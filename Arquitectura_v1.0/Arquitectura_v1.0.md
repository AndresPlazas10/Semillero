# Arquitectura del Sistema — v1.0 (S0-04)

**Proyecto:** Aplicación móvil con IA para recomendación gastronómica en Tunja
**Fecha:** septiembre de 2026
**Estado:** Documentada — sujeta a revisión al cerrar S0-06 a S0-12

## 1. Propósito

Definir los componentes del sistema, sus responsabilidades, los flujos principales y los límites entre capas, de modo que el Sprint 1 pueda iniciar con una base técnica clara y sin decisiones abiertas de alto nivel.

## 2. Principios de diseño

1. **FastAPI como frontera de servicio:** toda la lógica de negocio y de recomendación pasa por la API. El cliente móvil no ejecuta lógica contra la base de datos.
2. **Seguridad por rol con RLS:** PostgreSQL/Supabase aplica Row Level Security; el backend usa el JWT del usuario y nunca expone claves de servicio al cliente.
3. **pgvector es recuperación, no el algoritmo:** los embeddings y la búsqueda por similitud generan candidatos; el ranking y la evaluación viven en Python.
4. **Restaurante como unidad principal de recomendación:** los platos son atributos/contenido del restaurante (V-19).
5. **Ubicación aproximada y no persistida:** la ubicación se usa durante la consulta (radio con expansión) y no se almacenan coordenadas exactas (V-03, V-15).
6. **Datos sintéticos y reales separados:** sintético para pipeline y pruebas técnicas; real para validación experimental (V-09).
7. **Simplicidad primero:** un solo despliegue de backend en el MVP; la separación de servicios se evalúa cuando la carga lo justifique (V-11).

## 3. Diagrama de componentes

```
┌─────────────────────────────┐
│  App móvil                  │
│  React Native + Expo + TS   │
│  (Expo SDK 57, EAS)         │
└──────────┬──────────────────┘
           │ HTTPS (JWT)
           ▼
┌─────────────────────────────┐        ┌──────────────────────────┐
│  FastAPI (Python)           │───────▶│  Motor de recomendación  │
│  - Autenticación/autoriz.   │        │  (módulo Python interno) │
│  - Catálogo y gestión       │        │  - Recuperación          │
│  - Interacciones y eventos  │        │  - Ranking                │
│  - Recomendaciones          │        │  - Evaluación             │
│  - Contrato API v1          │        └──────────────────────────┘
└──────────┬──────────────────┘
           │ SQL (rol de servicio acotado)
           ▼
┌───────────────────────────────────────────────┐
│  Supabase / PostgreSQL                        │
│  - Supabase Auth (JWT)                        │
│  - Datos: usuarios, restaurantes, platos,     │
│    horarios, ubicación, preferencias,         │
│    interacciones, recomendaciones             │
│  - RLS por rol (usuario, gestor, admin)       │
│  - PostGIS: cercanía                          │
│  - pgvector: embeddings y similitud           │
└───────────────────────────────────────────────┘
```

## 4. Componentes y responsabilidades

| Componente | Responsabilidad | No hace |
|---|---|---|
| **App móvil (React Native + Expo + TS)** | Interfaz, permisos de ubicación, captura de eventos, presentación de recomendaciones, sesión con Supabase Auth | Lógica de negocio, acceso directo a tablas de negocio, cálculo de ranking |
| **FastAPI (Python)** | Frontera única: valida JWT, aplica reglas de negocio, orquesta el recomendador, expone contrato API v1, registra eventos | No entrena modelos ni calcula embeddings en la ruta de request (delega al módulo de recomendación) |
| **Motor de recomendación (Python)** | Genera candidatos (pgvector + reglas), aplica ranking, resuelve cold start, produce artefactos de evaluación | No expone endpoints propios en el MVP |
| **Supabase Auth** | Registro, inicio/cierre de sesión, emisión y validación de JWT | No contiene lógica de negocio |
| **PostgreSQL (Supabase)** | Persistencia, integridad, RLS, PostGIS (cercanía) y pgvector (embeddings) | No contiene lógica de aplicación |
| **Expo EAS** | Build y publicación Android/iOS, actualizaciones OTA | — |

## 5. Flujos principales

### 5.1 Autenticación
1. El usuario se registra/inicia sesión con Supabase Auth desde la app.
2. La app obtiene un JWT y lo envía en cada llamada a FastAPI.
3. FastAPI valida el JWT y determina el rol (usuario, gestor, administrador).

### 5.2 Catálogo y búsqueda
1. La app solicita restaurantes al backend.
2. FastAPI lee con RLS y aplica filtros MVP: tipo de comida, rango de precio y radio de cercanía (si hay permiso de ubicación).
3. Regla de disponibilidad: "abierto ahora" excluye o penaliza restaurantes cerrados.
4. Sin permiso de ubicación: orden por popularidad/relevancia.

### 5.3 Registro de interacciones
1. La app emite eventos (`busqueda`, `ver_restaurante`, `ver_plato`, `recomendacion_impresion`, `clic_recomendacion`).
2. FastAPI valida y persiste el evento con usuario pseudonimizado, entidad, origen, sesión y timestamp.
3. En recomendaciones se registran `recomendacion_id`, `request_id`, `modelo_version` y `posicion` (base del CTR).

### 5.4 Generación de recomendaciones
```
Ubicación aproximada + preferencias + historial + interacciones
        ↓
Generación de candidatos: reglas/SQL + pgvector (contenido)
        ↓
Ranking en Python (preferencias, cercanía, precio, popularidad)
        ↓
Filtros: disponibilidad ("abierto ahora"), datos vigentes
        ↓
Lista de restaurantes recomendados → FastAPI → app
```

### 5.5 Gestión gastronómica
1. El **Gestor** administra su restaurante, platos, horarios y ubicación desde la app (rol con RLS).
2. El **Administrador** aprueba establecimientos (pendiente → activo), puede suspender y usa herramientas internas.
3. La **investigación** accede solo a datos anonimizados/agregados y al entorno de análisis.

### 5.6 Evaluación del recomendador
1. Pipeline offline con dataset sintético reproducible y, después, datos reales.
2. Split temporal: pasado → entrenamiento, siguiente → validación, reciente → prueba.
3. Métricas P@10, R@10, NDCG@10 y cobertura contra baseline de popularidad; CTR en producto.

## 6. Límites y reglas

- El cliente **no** se conecta a PostgreSQL para lógica de negocio; solo usa Supabase Auth para sesión.
- Las claves de servicio y secretos viven únicamente en el backend.
- RLS activo en todas las tablas con datos de usuario o de gestión.
- La ubicación no se persiste en coordenadas exactas.
- Los eventos de recomendación permiten reconstruir qué modelo/posición produjo cada clic.
- Versionado del modelo y del dataset para reproducibilidad.

## 7. Entornos

| Entorno | Datos | Backend | Móvil |
|---|---|---|---|
| Desarrollo | Proyecto Supabase de desarrollo | FastAPI local | Expo Go / dev build |
| Producción | Proyecto Supabase de producción | Contenedor (objetivo por definir en S0-14) | Build EAS (Android/iOS) |

Pendiente S0-14: plataforma de despliegue del backend (candidatos: Render, Railway, Fly.io, Cloud Run), CI/CD y monitoreo básico.

## 8. Atributos de calidad (RNF)

| Requisito | Cómo lo cubre la arquitectura |
|---|---|
| RNF-001 Multiplataforma | React Native + Expo + TypeScript con SDK 57 (ADR-001) |
| RNF-003 Rendimiento | Índices, PostGIS para cercanía, p95 < 2 s consultas / < 3 s recomendaciones |
| RNF-004 Escalabilidad | Capas separadas; separación del motor en servicio futuro si la carga lo exige |
| RNF-005 Disponibilidad | 99 % objetivo del MVP; servicios gestionados; reintentos y respuestas degradadas |
| RNF-006 Seguridad | JWT, RLS por rol, secretos en servidor (ADR-002) |
| RNF-007 Privacidad | Minimización, consentimiento, retención justificada, sin ubicación exacta persistida |
| RNF-008 Mantenibilidad | Monorepo con capas claras, ADRs, pruebas automatizadas |
| RNF-009 Integrabilidad | FastAPI como frontera; contrato API v1 (S0-08) |
| RNF-010 Calidad | ISO 25010 acotada a 4 características con indicadores de pruebas |

## 9. Decisiones relacionadas

- `ADR/ADR-001_Cliente_Movil_Expo.md`
- `ADR/ADR-002_Datos_Supabase_RLS.md`
- `ADR/ADR-003_FastAPI_Frontera.md`
- `ADR/ADR-004_Python_pgvector_Recomendacion.md`
- Acta de validación: `../Sprint_0_Plan_v1.0/Acta_Validacion_Sprint_0_v1.0.md` (V-01…V-20)

## 10. Riesgos y pendientes

| Riesgo/Pendiente | Mitigación / siguiente paso |
|---|---|
| Elección de plataforma de despliegue del backend | S0-14 |
| Costos/plan de Supabase (pausa de Free, SLA) | Validar en S0-14 y Acta (V-14) |
| Modelo de datos y contrato API aún no definidos | S0-06 y S0-08 |
| Diseño detallado del recomendador y datos de IA | S0-09 y S0-10 |
| Observabilidad y rate limiting | Definir en S0-12/S0-14 |
| Volumen del dataset sintético | Fijar por el equipo (V-09) |

## 11. Glosario

| Término | Significado |
|---|---|
| Recomendador / motor de recomendación | Componente de software en Python que genera candidatos y los ordena; no es un usuario del sistema |
| Recuperación | Etapa que obtiene candidatos (reglas + SQL + pgvector) |
| Ranking | Etapa que ordena los candidatos según señales (preferencias, cercanía, precio, popularidad) |
| Embedding | Representación vectorial de un restaurante o plato para búsqueda por similitud |
| Gestor | Comerciante que administra su restaurante desde la app |
| Investigación | Acceso separado a datos anonimizados/agregados; no es rol operativo |
| ML aplicado | Uso de modelos de machine learning ya entrenados (p. ej. embeddings) sin entrenar con datos del proyecto |
| ML entrenado | Ajuste de un modelo con datos del proyecto (colaborativo o learning-to-rank) |
| Filtrado colaborativo | Recomendación basada en patrones de interacción de usuarios similares |
| Learning-to-rank | Modelo que aprende a ordenar resultados usando clics/impresiones |
| Baseline | Estrategia de referencia (popularidad) contra la que se mide cualquier modelo |
