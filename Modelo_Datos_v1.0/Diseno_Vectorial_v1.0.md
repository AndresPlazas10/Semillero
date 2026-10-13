# Diseño Vectorial — v1.0 (S0-07)

**Proyecto:** Aplicación móvil con IA para recomendación gastronómica en Tunja
**Fecha:** septiembre de 2026
**Estado:** Diseño inicial — sujeto a experimentación (S0-09/S0-11)

## 1. Propósito

Definir qué entidades tendrán embeddings, con qué modelo y dimensión, cómo se versionan y actualizan, cómo se usa la búsqueda por similitud y qué reglas de privacidad aplican (V-07, V-19, V-20).

## 2. Principio rector

**pgvector es infraestructura de recuperación, no el algoritmo** (V-07). El flujo es:

```
Embeddings de restaurantes (platos como features)
        ↓
Búsqueda por similitud (pgvector) + filtros SQL (estado, precio, radio)
        ↓
Candidatos
        ↓
Ranking en Python (preferencias, cercanía, popularidad)
        ↓
Recomendaciones
```

## 3. Entidades con embeddings

| Entidad | Prioridad | Uso |
|---|---|---|
| **Restaurante** (principal) | MVP | Recuperación de candidatos por similitud con el perfil del usuario |
| **Plato** | P1 | Búsqueda de platos y enriquecimiento del embedding del restaurante |
| **Usuario** | No se persiste | Representación calculada en el momento (V-20); no se almacena como embedding |

### 3.1 Embedding de restaurante (principal)

**Contenido fuente (texto concatenado):**
- Nombre y descripción del restaurante.
- Categoría (tipo de comida).
- Rango de precio (etiqueta).
- Nombres y descripciones de sus platos disponibles (máx. N para acotar costo).

**Tabla:** `restaurante_embeddings` (PK `restaurante_id`, ver S0-06), con `modelo`, `version`, `dims`, `contenido_hash` y `actualizado_en`.

- Un embedding vigente por restaurante; el histórico se reconstruye si se requiere mediante re-generación con versiones.
- `contenido_hash` evita re-generar cuando nada cambió.

### 3.2 Embedding de plato (P1)

- Mismo modelo; permite búsqueda semántica de platos y detalle.
- No bloquea el MVP; se evalúa tras la primera iteración de calidad.

## 4. Modelo, dimensión y versionado

| Parámetro | Propuesta | Notas |
|---|---|---|
| Modelo | Multilingüe de embeddings (candidatos: `multilingual-e5-small/base`, `bge-m3`) ejecutado en Python; alternativa API (`text-embedding-3-small`) | La elección se justifica con experimentos (RIA-003) |
| Dimensión | Según modelo (384/768/1024) | Se almacena en `dims`; validar que coincida con el índice |
| Distancia | Coseno (`vector_cosine_ops`) | Textos normalizados |
| Versión | `modelo` + `version` (p. ej. `e5-small` + `v1`) | Nunca mezclar versiones en un mismo índice |
| Índice | HNSW (mejor recall/latencia en MVP); alternativa IVFFlat | Reconstruir al cambiar de modelo/versión |

**Regla de oro:** si cambia el modelo o la versión, se re-generan todos los embeddings y se reconstruye el índice antes de usarlo en producción.

## 5. Actualización

1. **Disparadores:** alta/edición de restaurante o de sus platos; cambio de estado a `activo`.
2. **Estrategia:** marcar `contenido_hash` desactualizado y ejecutar un job por lotes (no en la ruta de request).
3. **Momentos:** job al cerrar el día y bajo demanda tras cambios de catálogo relevantes.
4. **Idempotencia:** se re-embebe solo si `contenido_hash` cambió.
5. **Monitoreo:** registrar cuántos embeddings están pendientes o desactualizados; alertar si supera un umbral.

## 6. Búsqueda y uso en el recomendador

### 6.1 Sin historial (cold start, V-08)
1. Construir un vector "semilla" con las **preferencias explícitas** (categorías) → similar a promedio de embeddings de categorías o de restaurantes representativos.
2. Recuperar candidatos por similitud + filtros (estado `activo`, rango de precio, radio).
3. Si no hay preferencias ni ubicación → **popularidad**.

### 6.2 Con historial (V-20)
1. Construir la **representación del usuario** en memoria: promedio ponderado (centroide) de los embeddings de restaurantes/platos con los que interactuó, más peso a los eventos recientes y a `clic_recomendacion` sobre `recomendacion_impresion`.
2. Recuperar candidatos por similitud coseno (k variando según resultados).
3. Aplicar reglas: `activo`, "abierto ahora", rango de precio, radio (PostGIS) con expansión si hay pocos resultados.
4. Rankings en Python: `score = w1·similitud + w2·cercanía + w3·preferencia + w4·popularidad` (pesos a calibrar).

### 6.3 Re-ranking y evaluación
- Medir **cobertura de recuperación** (¿está el restaurante relevante entre los candidatos?).
- Métricas finales: P@10, R@10, NDCG@10 y cobertura (V-10).

## 7. Privacidad (V-15)

- **No se persiste el embedding del usuario**; el centroide se calcula por petición y se descarta.
- Los embeddings de restaurantes son contenido público del catálogo.
- Los eventos usados para la representación respetan consentimiento y anonimización.
- No se incluye ubicación exacta en el texto ni en el embedding.

## 8. Costos y operación

- Modelo local (open source) evita costo por token; contrapartida: tiempo de cómputo y tamaño de despliegue.
- Si se usa API de embeddings: cachear por `contenido_hash` y limitar re-generaciones.
- El índice HNSW consume memoria; dimensionar el plan de Supabase (V-14).
- Batch nocturno para re-embeddings masivos.

## 9. Pendientes

1. Elegir el modelo definitivo mediante experimento comparativo (S0-09/S0-11).
2. Definir N máximo de platos usados como features.
3. Validar el índice HNSW con el volumen esperado.
4. Fijar el volumen inicial del dataset sintético (V-09).
5. Definir el job de actualización en S0-14.
