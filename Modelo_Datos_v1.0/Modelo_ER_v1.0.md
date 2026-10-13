# Modelo Entidad-Relación — v1.0 (S0-06)

**Proyecto:** Aplicación móvil con IA para recomendación gastronómica en Tunja
**Fecha:** septiembre de 2026
**Base de datos:** PostgreSQL (Supabase) con extensiones `postgis` y `vector`
**Estado:** Diseño inicial — las migraciones se implementan en S0-13/S0-14

## 1. Convenciones

- Claves primarias `uuid` (`gen_random_uuid()`), salvo tablas de alto volumen (`eventos`) que usan `bigint identity`.
- Fechas en `timestamptz` (UTC). Nombres de tabla y columna en `snake_case` y singular/plural consistente.
- Enumeraciones con `CREATE TYPE` (evitan strings libres).
- `auth.users` es gestionada por Supabase Auth; el dominio la referencia con `perfiles.id`.
- No se persisten coordenadas exactas de usuarios (V-03/V-15): solo coordenadas de restaurantes.

## 2. Diagrama entidad-relación

```
auth.users 1──1 perfiles 1──N preferencias_usuario
                    │ 1──N consentimientos
                    │ 1──N restaurante_gestores N──1 restaurantes
                    │                                    │ 1──N platos
                    │                                    │ 1──N horarios
                    │                                    │ 1──1 restaurante_embeddings
                    │                                    │ 1──N recomendacion_items
                    └──1──N eventos               recomendaciones 1──N recomendacion_items
categorias 1──N restaurantes / platos
rangos_precio 1──N restaurantes
auditoria_cambios (transversal)
```

- `categorias`: tipo de comida (italiana, rápida, típica…).
- `rangos_precio`: etiquetas con mínimo y máximo.
- `restaurantes`: entidad principal del catálogo y de la recomendación (V-19).
- `platos`: atributos/contenido del restaurante.
- `eventos`: las 5 interacciones definidas (V-05).
- `recomendaciones` + `recomendacion_items`: trazabilidad de qué se mostró y qué modelo lo generó.

## 3. Diccionario de entidades

### 3.1 Identidad y perfil

| Tabla | Columnas principales | Notas |
|---|---|---|
| `perfiles` | `id` (=`auth.users.id`), `nombre`, `rango_precio_id`, `restricciones text[]`, `rol_global` (`usuario`/`administrador`), `creado_en`, `actualizado_en` | Sin `zona` (V-03). El rol `gestor` se deriva de `restaurante_gestores` |
| `preferencias_usuario` | `id`, `usuario_id`, `categoria_id`, `origen` (`explicita`/`inferida`), `peso numeric(3,2)`, `creado_en`, `actualizado_en` | Única por (`usuario_id`, `categoria_id`); explícitas dominan al inicio (V-06) |
| `consentimientos` | `id`, `usuario_id`, `tipo` (`interacciones`,`preferencias`,`ubicacion`), `version`, `otorgado bool`, `fecha` | Autorización previa, expresa e informada (V-15) |

### 3.2 Catálogo

| Tabla | Columnas principales | Notas |
|---|---|---|
| `categorias` | `id`, `nombre`, `slug` | Catálogo de tipos de comida |
| `rangos_precio` | `id`, `etiqueta`, `min`, `max` | Filtro y perfil |
| `restaurantes` | `id`, `nombre`, `descripcion`, `categoria_id`, `rango_precio_id`, `direccion`, `ubicacion geography(Point,4326)`, `estado` (`pendiente`/`activo`/`suspendido`), `creado_por`, `aprobado_por`, `aprobado_en`, `creado_en`, `actualizado_en` | Aprobación por administrador (V-02); PostGIS para cercanía |
| `platos` | `id`, `restaurante_id`, `nombre`, `descripcion`, `precio numeric(10,2)`, `categoria_id`, `disponible bool`, `imagen_url`, `creado_en`, `actualizado_en` | Precio e imagen opcionales |
| `horarios` | `id`, `restaurante_id`, `dia_semana smallint`, `abre time`, `cierra time`, `cerrado bool` | Varias filas por día permitidas; alimenta "abierto ahora" |
| `restaurante_gestores` | `restaurante_id`, `usuario_id`, `creado_en` | PK compuesta; asocia gestor ↔ restaurante (V-01/V-02) |

### 3.3 Analítica y recomendación

| Tabla | Columnas principales | Notas |
|---|---|---|
| `eventos` | `id bigint`, `usuario_id` (nullable), `tipo` (enum 5 eventos), `entidad_tipo` (`restaurante`/`plato`/`consulta`), `entidad_id`, `origen`, `sesion_id`, `termino`, `filtros jsonb`, `recomendacion_id`, `request_id`, `modelo_version`, `posicion`, `creado_en` | Particionable por mes; base del CTR y de las señales del recomendador |
| `recomendaciones` | `id`, `usuario_id`, `request_id`, `modelo_version`, `estrategia` (`cold_start`/`popularidad`/`contenido`/`hibrido`), `uso_ubicacion bool`, `radio_km`, `total_items`, `creado_en` | No almacena coordenadas exactas del usuario |
| `recomendacion_items` | `recomendacion_id`, `restaurante_id`, `posicion`, `score numeric`, `razones jsonb` | Permite join con `clic_recomendacion` y `recomendacion_impresion` |
| `auditoria_cambios` | `id`, `tabla`, `registro_id`, `accion`, `actor_id`, `datos jsonb`, `creado_en` | Cambios de estado y de catálogo (V-02) |
| `restaurante_embeddings` | `restaurante_id`, `modelo`, `version`, `dims`, `embedding vector(N)`, `contenido_hash`, `actualizado_en` | Diseño en `Diseno_Vectorial_v1.0.md` (S0-07) |

## 4. DDL de referencia

```sql
create extension if not exists postgis;
create extension if not exists vector;

create type rol_global as enum ('usuario', 'administrador');
create type estado_restaurante as enum ('pendiente', 'activo', 'suspendido');
create type origen_preferencia as enum ('explicita', 'inferida');
create type tipo_consentimiento as enum ('interacciones', 'preferencias', 'ubicacion');
create type tipo_evento as enum (
  'busqueda', 'ver_restaurante', 'ver_plato', 'recomendacion_impresion', 'clic_recomendacion'
);
create type entidad_evento as enum ('restaurante', 'plato', 'consulta');
create type origen_evento as enum ('busqueda', 'recomendacion', 'catalogo');
create type estrategia_recomendacion as enum ('cold_start', 'popularidad', 'contenido', 'hibrido');

create table perfiles (
  id uuid primary key references auth.users(id) on delete cascade,
  nombre text,
  rango_precio_id uuid references rangos_precio(id),
  restricciones text[] default '{}',
  rol_global rol_global not null default 'usuario',
  creado_en timestamptz not null default now(),
  actualizado_en timestamptz not null default now()
);

create table preferencias_usuario (
  id uuid primary key default gen_random_uuid(),
  usuario_id uuid not null references perfiles(id) on delete cascade,
  categoria_id uuid not null references categorias(id),
  origen origen_preferencia not null default 'explicita',
  peso numeric(3,2) not null default 1.00,
  creado_en timestamptz not null default now(),
  actualizado_en timestamptz not null default now(),
  unique (usuario_id, categoria_id)
);

create table consentimientos (
  id uuid primary key default gen_random_uuid(),
  usuario_id uuid not null references perfiles(id) on delete cascade,
  tipo tipo_consentimiento not null,
  version text not null,
  otorgado boolean not null,
  fecha timestamptz not null default now(),
  unique (usuario_id, tipo, version)
);

create table restaurantes (
  id uuid primary key default gen_random_uuid(),
  nombre text not null,
  descripcion text,
  categoria_id uuid references categorias(id),
  rango_precio_id uuid references rangos_precio(id),
  direccion text,
  ubicacion geography(Point, 4326),
  estado estado_restaurante not null default 'pendiente',
  creado_por uuid references perfiles(id),
  aprobado_por uuid references perfiles(id),
  aprobado_en timestamptz,
  creado_en timestamptz not null default now(),
  actualizado_en timestamptz not null default now()
);

create table platos (
  id uuid primary key default gen_random_uuid(),
  restaurante_id uuid not null references restaurantes(id) on delete cascade,
  nombre text not null,
  descripcion text,
  precio numeric(10,2),
  categoria_id uuid references categorias(id),
  disponible boolean not null default true,
  imagen_url text,
  creado_en timestamptz not null default now(),
  actualizado_en timestamptz not null default now()
);

create table horarios (
  id uuid primary key default gen_random_uuid(),
  restaurante_id uuid not null references restaurantes(id) on delete cascade,
  dia_semana smallint not null check (dia_semana between 0 and 6),
  abre time,
  cierra time,
  cerrado boolean not null default false
);

create table restaurante_gestores (
  restaurante_id uuid not null references restaurantes(id) on delete cascade,
  usuario_id uuid not null references perfiles(id) on delete cascade,
  creado_en timestamptz not null default now(),
  primary key (restaurante_id, usuario_id)
);

create table eventos (
  id bigint generated always as identity primary key,
  usuario_id uuid references perfiles(id) on delete set null,
  tipo tipo_evento not null,
  entidad_tipo entidad_evento not null,
  entidad_id uuid,
  origen origen_evento,
  sesion_id uuid,
  termino text,
  filtros jsonb,
  recomendacion_id uuid,
  request_id uuid,
  modelo_version text,
  posicion smallint,
  creado_en timestamptz not null default now()
);

create table recomendaciones (
  id uuid primary key default gen_random_uuid(),
  usuario_id uuid not null references perfiles(id) on delete cascade,
  request_id uuid not null,
  modelo_version text,
  estrategia estrategia_recomendacion not null,
  uso_ubicacion boolean not null default false,
  radio_km numeric(5,2),
  total_items smallint not null default 0,
  creado_en timestamptz not null default now()
);

create table recomendacion_items (
  recomendacion_id uuid not null references recomendaciones(id) on delete cascade,
  restaurante_id uuid not null references restaurantes(id),
  posicion smallint not null,
  score numeric(6,4),
  razones jsonb,
  primary key (recomendacion_id, restaurante_id)
);

create table auditoria_cambios (
  id bigint generated always as identity primary key,
  tabla text not null,
  registro_id uuid not null,
  accion text not null,
  actor_id uuid references perfiles(id),
  datos jsonb,
  creado_en timestamptz not null default now()
);

create table restaurante_embeddings (
  restaurante_id uuid primary key references restaurantes(id) on delete cascade,
  modelo text not null,
  version text not null,
  dims smallint not null,
  embedding vector not null,
  contenido_hash text not null,
  actualizado_en timestamptz not null default now()
);
```

## 5. Reglas de negocio

1. **Estados del restaurante:** `pendiente` (alta por gestor) → `activo` (aprobado por administrador) → `suspendido` (decisión del administrador). Solo `activo` se recomienda; `pendiente` y `suspendido` no aparecen en catálogo público. Todo cambio queda en `auditoria_cambios`.
2. **Permisos:** el gestor solo modifica restaurantes donde figura en `restaurante_gestores`; el administrador modifica todo y aprueba/suspende.
3. **Perfil:** sin zona; `rango_precio_id` y `restricciones` opcionales; preferencias explícitas e inferidas conviven con `peso` (V-06).
4. **Consentimiento:** los eventos de interacción y la ubicación requieren consentimiento vigente; sin consentimiento no se registran eventos (V-15).
5. **Ubicación:** `ubicacion` solo existe en restaurantes; las recomendaciones registran `uso_ubicacion` y `radio_km`, nunca coordenadas (V-03).
6. **Precio/rendimiento:** `precio` se guarda por plato; `rango_precio_id` a nivel restaurante para filtros.
7. **Retención:** proceso programado para anonimizar eventos a los 12 meses (eliminar `usuario_id`, conservar agregados) y borrar todo al eliminar la cuenta (V-15).

## 6. Row Level Security (RLS)

| Tabla | Política |
|---|---|
| `perfiles` | `select`/`update` propio; administrador todo |
| `preferencias_usuario`, `consentimientos` | `select`/`insert`/`update`/`delete` propio; administrador todo |
| `restaurantes`, `platos`, `horarios` | `select` autenticado solo `estado='activo'`; gestor `all` sobre los suyos; administrador todo (incluye `pendiente`/`suspendido`) |
| `restaurante_gestores` | `select` propio; administrador gestiona |
| `eventos` | `insert` autenticado propio; `select` administrador; investigación vía vistas anonimizadas |
| `recomendaciones`, `recomendacion_items` | `select` propio; escritura solo backend (rol de servicio) |
| `auditoria_cambios` | `select` administrador |
| `restaurante_embeddings` | `select` administrador/backend; escritura backend |

Notas:
- El backend usa JWT del usuario y un rol de servicio acotado solo para operaciones de sistema (insertar eventos/recomendaciones).
- Las políticas se prueban con tests específicos por rol (pruebas P-032-*, P-002-*).

## 7. Índices y rendimiento (RNF-003)

- `gist (ubicacion)` en `restaurantes` para cercanía.
- `btree (usuario_id, creado_en desc)` en `eventos`; `btree (recomendacion_id)`.
- `btree (restaurante_id)` en `platos` y `horarios`; `btree (estado)` en `restaurantes`.
- `gin (filtros jsonb)` y `gin (restricciones)` si se consultan.
- Índice vectorial HNSW (cosine) sobre `restaurante_embeddings.embedding` (ver S0-07).
- Particionado mensual de `eventos` cuando el volumen lo justifique.

## 8. Vistas de investigación y anonimización

- `v_interacciones_anonimas`: eventos sin `usuario_id` (hash irreversible por usuario), con tipo, entidad, origen y fecha.
- `v_metricas_eventos_diarios`: agregados por día/tipo restaurante.
- `v_metricas_recomendaciones`: impresiones, clics y CTR por `modelo_version` y posición.
- Acceso: solo administrador y acceso de investigación; nunca datos personales crudos (V-01/V-15).

## 9. Migraciones y evolución

- Migraciones versionadas con Supabase CLI en `supabase/migrations`.
- Cambios de esquema en rama + revisión + migración aplicada en dev y luego producción (ADR-002).
- Semilla de catálogo (categorías y rangos) como migración de datos.

## 10. Pendientes

1. Validar el catálogo inicial de categorías y rangos de precio (equipo).
2. Definir el volumen inicial del dataset sintético (V-09).
3. Confirmar la plataforma/plan de Supabase para el piloto (V-14).
4. Programar el job de anonimización/retención (S0-12/S0-14).
