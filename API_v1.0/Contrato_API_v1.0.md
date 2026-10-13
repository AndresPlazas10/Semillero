# Contrato de API — v1.0 (S0-08)

**Proyecto:** Aplicación móvil con IA para recomendación gastronómica en Tunja
**Fecha:** septiembre de 2026
**Base URL:** `https://<host>/v1`
**Estado:** Diseño inicial — el OpenAPI definitivo se genera desde FastAPI

## 1. Convenciones

| Tema | Regla |
|---|---|
| Formato | JSON UTF-8; fechas ISO 8601 en UTC |
| Versionado | En la ruta (`/v1`) |
| Autenticación | `Authorization: Bearer <JWT de Supabase Auth>` |
| Roles | `usuario`, `gestor`, `administrador` (el gestor se deriva de `restaurante_gestores`) |
| Paginación | `?page=1&page_size=20` (máx. 100); respuesta con `meta` |
| Orden | `?orden=cercania|popularidad|precio|nombre` |
| Idempotencia de eventos | `Idempotency-Key` opcional; el backend deduplica por `sesion_id + tipo + entidad_id + timestamp` |
| Límite de tasa | 429 con `Retry-After` (definir en S0-12) |
| CORS | No requerido para cliente móvil; limitado en desarrollo |

## 2. Formato de error

```json
{
  "error": {
    "code": "validation_error",
    "message": "El campo 'precio' debe ser mayor que cero",
    "details": [{"campo": "precio", "motivo": "debe ser > 0"}]
  }
}
```

| HTTP | `code` | Uso |
|---|---|---|
| 400 | `bad_request` | Petición malformada |
| 401 | `unauthorized` | JWT ausente o inválido |
| 403 | `forbidden` | Rol sin permiso sobre el recurso |
| 404 | `not_found` | Recurso inexistente |
| 409 | `conflict` | Duplicado o transición de estado inválida |
| 422 | `validation_error` | Campos inválidos |
| 429 | `rate_limited` | Límite excedido |
| 500 | `internal_error` | Error no controlado |

## 3. Autenticación y autorización

- El registro y el inicio de sesión se hacen con **Supabase Auth** en la app; el backend valida el JWT.
- El backend aplica **RLS** y además valida permisos de negocio (p. ej., el gestor solo toca su restaurante).
- `GET /v1/usuarios/me` crea el `perfil` la primera vez si no existe (sincronización post-registro).

## 4. Endpoints

### 4.1 Usuario y perfil

| Método y ruta | Rol | Descripción | Requisitos |
|---|---|---|---|
| `GET /v1/usuarios/me` | autenticado | Perfil, preferencias y estado de consentimientos | RF-003 |
| `PATCH /v1/usuarios/me` | autenticado | Actualiza nombre, rango de precio y restricciones (sin zona) | RF-003, V-03 |
| `PUT /v1/usuarios/me/preferencias` | autenticado | Reemplaza preferencias explícitas (categorías) | RF-013, V-06 |
| `POST /v1/usuarios/me/consentimientos` | autenticado | Registra/actualiza consentimiento por tipo y versión | RNF-007, V-15 |
| `DELETE /v1/usuarios/me` | autenticado | Elimina cuenta y datos asociados | RNF-007, V-15 |

Ejemplo `PUT /v1/usuarios/me/preferencias`:

```json
{
  "categorias": ["uuid-italiana", "uuid-comida-rapida"],
  "rango_precio_id": "uuid-rango-2",
  "restricciones": ["sin gluten"]
}
```

### 4.2 Catálogo

| Método y ruta | Rol | Descripción | Requisitos |
|---|---|---|---|
| `GET /v1/categorias` | autenticado | Lista de tipos de comida | RF-010, V-04 |
| `GET /v1/rangos-precio` | autenticado | Lista de rangos de precio | RF-007 |
| `GET /v1/restaurantes` | autenticado | Catálogo con filtros y cercanía | RF-004, RF-010, V-03/V-04 |
| `GET /v1/restaurantes/{id}` | autenticado | Detalle (incluye horarios y ubicación) | RF-005 |
| `GET /v1/restaurantes/{id}/platos` | autenticado | Platos del restaurante | RF-006 |
| `GET /v1/platos/{id}` | autenticado | Detalle del plato | RF-011 |

Parámetros de `GET /v1/restaurantes`:

```
q, categoria_id, rango_precio_id,
lat, lng, radio_km (default 5, expansión automática),
abierto_ahora (bool), orden, page, page_size
```

Respuesta (resumen):

```json
{
  "data": [
    {
      "id": "uuid",
      "nombre": "La Trattoria",
      "categoria": {"id": "uuid", "nombre": "Italiana"},
      "rango_precio": {"id": "uuid", "etiqueta": "$$"},
      "distancia_km": 1.2,
      "abierto_ahora": true,
      "estado": "activo"
    }
  ],
  "meta": {"page": 1, "page_size": 20, "total": 37}
}
```

### 4.3 Recomendaciones

| Método y ruta | Rol | Descripción | Requisitos |
|---|---|---|---|
| `POST /v1/recomendaciones` | autenticado | Genera recomendaciones (unidad: restaurantes) | RF-015/016/017, V-19 |
| `GET /v1/recomendaciones/{id}` | autenticado (dueño) | Recupera una recomendación emitida | RF-017 |

Petición:

```json
{
  "lat": 5.5353,
  "lng": -73.3678,
  "radio_km": 5,
  "limite": 10,
  "contexto": "inicio"
}
```

- `lat`/`lng` son opcionales; si faltan o no hay consentimiento de ubicación, se usa popularidad (V-08).
- La respuesta incluye `recomendacion_id` y `request_id` para atribución de eventos.

Respuesta:

```json
{
  "recomendacion_id": "uuid",
  "request_id": "uuid",
  "modelo_version": "contenido-v1",
  "estrategia": "cold_start",
  "items": [
    {
      "posicion": 1,
      "restaurante": {"id": "uuid", "nombre": "La Trattoria", "distancia_km": 1.2},
      "score": 0.87,
      "razones": ["preferencia: italiana", "cerca de ti"]
    }
  ]
}
```

### 4.4 Eventos

| Método y ruta | Rol | Descripción | Requisitos |
|---|---|---|---|
| `POST /v1/eventos` | autenticado | Registra hasta 50 eventos en lote | RF-012, V-05 |

```json
{
  "eventos": [
    {
      "tipo": "recomendacion_impresion",
      "entidad_tipo": "restaurante",
      "entidad_id": "uuid",
      "origen": "recomendacion",
      "sesion_id": "uuid",
      "recomendacion_id": "uuid",
      "request_id": "uuid",
      "modelo_version": "contenido-v1",
      "posicion": 1,
      "creado_en": "2026-09-10T14:03:00Z"
    },
    {
      "tipo": "clic_recomendacion",
      "entidad_tipo": "restaurante",
      "entidad_id": "uuid",
      "origen": "recomendacion",
      "sesion_id": "uuid",
      "recomendacion_id": "uuid",
      "posicion": 1,
      "creado_en": "2026-09-10T14:03:07Z"
    }
  ]
}
```

Reglas:
- Sin consentimiento de interacciones → `403` y no se registra nada.
- `recomendacion_impresion` es obligatorio para calcular CTR (V-05/V-10).
- Los eventos son inmutables (solo inserción).

### 4.5 Gestión (gestor y administrador)

| Método y ruta | Rol | Descripción | Requisitos |
|---|---|---|---|
| `POST /v1/restaurantes` | gestor/admin | Crea restaurante (estado `pendiente`) | RF-018/019 |
| `PATCH /v1/restaurantes/{id}` | gestor (suyo)/admin | Actualiza información | RF-018/019 |
| `PATCH /v1/restaurantes/{id}/ubicacion` | gestor (suyo)/admin | Actualiza coordenadas y dirección | RF-022 |
| `PUT /v1/restaurantes/{id}/horarios` | gestor (suyo)/admin | Reemplaza horarios | RF-021 |
| `POST /v1/restaurantes/{id}/platos` | gestor (suyo)/admin | Crea plato | RF-020 |
| `PATCH /v1/platos/{id}` | gestor (suyo)/admin | Actualiza plato | RF-020 |
| `DELETE /v1/platos/{id}` | gestor (suyo)/admin | Desactiva plato (`disponible=false`) | RF-020 |

### 4.6 Administración

| Método y ruta | Rol | Descripción | Requisitos |
|---|---|---|---|
| `GET /v1/admin/restaurantes?estado=pendiente` | admin | Revisa altas pendientes | RF-019, V-02 |
| `POST /v1/admin/restaurantes/{id}/aprobar` | admin | Activa el establecimiento | RF-019, V-02 |
| `POST /v1/admin/restaurantes/{id}/suspender` | admin | Suspende el establecimiento | RF-019, V-02 |
| `GET /v1/admin/auditoria` | admin | Consulta cambios auditados | V-02 |
| `GET /v1/admin/usuarios` | admin | Lista usuarios (sin datos sensibles) | RNF-006 |

### 4.7 Investigación

| Método y ruta | Rol | Descripción | Requisitos |
|---|---|---|---|
| `GET /v1/investigacion/metricas` | admin/investigación | Agregados (impresiones, clics, CTR, métricas offline) | RF-023, V-10 |
| `POST /v1/investigacion/exportaciones` | admin/investigación | Genera export anonimizado (asíncrono) | RIA-004, V-15 |
| `GET /v1/investigacion/exportaciones/{id}` | admin/investigación | Estado y enlace de descarga | RIA-004 |

- Solo datos anonimizados/agregados; nunca datos personales crudos (V-01/V-15).

### 4.8 Salud

| Método y ruta | Rol | Descripción |
|---|---|---|
| `GET /health` | público | Liveness (proceso vivo) |
| `GET /v1/health` | autenticado | Readiness (BD y dependencias) |

## 5. DTOs principales

| DTO | Campos |
|---|---|
| `UsuarioPerfil` | `id`, `nombre`, `rango_precio_id`, `restricciones[]`, `preferencias[]`, `consentimientos{}` |
| `RestauranteResumen` | `id`, `nombre`, `categoria`, `rango_precio`, `distancia_km`, `abierto_ahora`, `estado` |
| `RestauranteDetalle` | Resumen + `descripcion`, `direccion`, `ubicacion{lat,lng}`, `horarios[]`, `platos[]` |
| `Plato` | `id`, `nombre`, `descripcion`, `precio`, `categoria`, `disponible`, `imagen_url` |
| `Recomendacion` | `recomendacion_id`, `request_id`, `modelo_version`, `estrategia`, `items[]` |
| `RecomendacionItem` | `posicion`, `restaurante`, `score`, `razones[]` |
| `EventoEntrada` | `tipo`, `entidad_tipo`, `entidad_id`, `origen`, `sesion_id`, `recomendacion_id`, `request_id`, `modelo_version`, `posicion`, `termino`, `filtros`, `creado_en` |
| `Error` | `error.code`, `error.message`, `error.details[]` |

## 6. Reglas de negocio en la API

1. Solo restaurantes `activo` aparecen en catálogo y recomendaciones para usuarios.
2. `abierto_ahora` se calcula con horarios y zona horaria de Tunja (America/Bogota).
3. La cercanía usa `lat`/`lng` de la petición; no se persisten.
4. El gestor no puede aprobar su propio restaurante.
5. Las recomendaciones solo devuelven restaurantes existentes, activos y disponibles (HU-016).
6. Los eventos se aceptan en lote y son inmutables.
7. La eliminación de cuenta borra perfil y datos personales, y anonimiza eventos.

## 7. Pruebas y herramientas

- **OpenAPI** generado por FastAPI como fuente de verdad del contrato.
- **Postman**: colección de pruebas manuales/contractuales (P-017-*, P-035-*).
- **pytest**: pruebas automatizadas de endpoints, permisos y RLS.
- Casos mínimos: auth válida/ inválida, permisos por rol, filtros, eventos sin consentimiento, recomendación cold start, paginación y errores 4xx/5xx.

## 8. Pendientes

1. Definir la plataforma de despliegue y el host (S0-14).
2. Definir límites de tasa concretos (S0-12).
3. Definir el formato exacto de exportación de investigación (CSV/Parquet).
4. Versionar la colección Postman en el repositorio (S0-13).
