# Diseño del Recomendador — v1.0 (S0-09)

**Proyecto:** Aplicación móvil con IA para recomendación gastronómica en Tunja
**Fecha:** septiembre de 2026
**Estado:** Diseño inicial — pesos y modelo se validan experimentalmente

## 1. Propósito

Definir el funcionamiento del motor de recomendación: señales de entrada, generación de candidatos, reglas, ranking, cold start, disponibilidad y evolución hacia el híbrido (V-07, V-08, V-19, V-20).

## 2. Principios

1. **pgvector no es el algoritmo:** recupera candidatos por similitud; Python ordena y evalúa.
2. **Unidad principal: restaurante** (V-19); los platos son atributos/contenido.
3. **Señales explícitas primero** cuando no hay historial (V-06).
4. **Ubicación aproximada, no persistida** (V-03).
5. **Disponibilidad real:** un restaurante cerrado no se recomienda.
6. **Explicabilidad:** cada recomendación expone "razones" (preferencia, cercanía, popularidad).

## 3. Pipeline

```
ENTRADAS
  Usuario: preferencias explícitas, historial ponderado, restricciones
  Contexto: ubicación aproximada (opcional), hora/día, sesión
  Catálogo: restaurantes activos, platos, horarios, precios, popularidad
        ↓
RECUPERACIÓN DE CANDIDATOS
  A. Similitud: embedding del usuario vs restaurante_embeddings (pgvector)
  B. Reglas: cercanía (PostGIS, radio con expansión), preferencias explícitas,
     popularidad por zona
  C. Unión + deduplicación (máx. ~100 candidatos)
        ↓
FILTROS DUROS
  estado = activo · abierto_ahora · rango de precio · radio · datos vigentes
        ↓
RANKING (Python)
  score = w1·similitud + w2·cercanía + w3·preferencia + w4·popularidad
  (+ ajustes: penalización por cierre próximo, diversidad de categorías)
        ↓
POST-PROCESO
  Diversidad (máx. 2-3 por categoría) · razones por ítem · recorte a N (10)
        ↓
SALIDA
  Lista ordenada + recomendacion_id + request_id + modelo_version
```

## 4. Señales y representación del usuario (V-20)

### 4.1 Preferencias explícitas
- Categorías declaradas en onboarding; peso alto y estable.
- Restricciones (p. ej. "sin gluten") actúan como filtro duro.

### 4.2 Historial e interacciones
Representación en memoria (centroide ponderado de embeddings de ítems):

| Evento | Peso base | Nota |
|---|---:|---|
| `clic_recomendacion` | 1.0 | Señal más fuerte |
| `ver_plato` | 0.6 | Interés específico |
| `ver_restaurante` | 0.5 | Interés general |
| `busqueda` | 0.3 | Intención; usar término/filtros |
| `recomendacion_impresion` | 0.0 | No suma afinidad; sirve para CTR |

- **Decaimiento temporal:** peso × `exp(-λ·días)` (λ inicial ≈ 0.02, vida media ~35 días).
- Solo se usan eventos con consentimiento vigente (V-15).
- El vector no se persiste; se calcula por petición y se descarta.

### 4.3 Rango de precio y restricciones
- Rango de precio: filtro suave (penaliza fuera del rango) + señal de ranking.
- Restricciones: filtro duro.

## 5. Recuperación de candidatos

| Fuente | Cómo | Uso |
|---|---|---|
| Vectorial | kNN coseno sobre `restaurante_embeddings` | Usuarios con preferencias/historial |
| Preferencias | Restaurantes de las categorías declaradas | Cold start con preferencias |
| Cercanía | `ST_DWithin` con radio inicial 5 km | Usuarios con ubicación |
| Popularidad | Ranking por interacciones recientes (p. ej. 30 días) | Fallback universal |

- Si el radio devuelve pocos resultados (< límite), se **expande** progresivamente (5 → 10 → 20 km) (V-03).
- Sin ubicación: se omite la fuente de cercanía y se pondera más popularidad/preferencia.

## 6. Ranking

### 6.1 Normalización
Cada señal se normaliza a [0, 1]: similitud coseno, cercanía (`1 - distancia/radio`), coincidencia de preferencia, popularidad (percentil).

### 6.2 Pesos iniciales propuestos (a calibrar)

| Perfil | w1 similitud | w2 cercanía | w3 preferencia | w4 popularidad |
|---|---:|---:|---:|---:|
| Con historial + ubicación | 0.40 | 0.30 | 0.20 | 0.10 |
| Con preferencias, sin historial | 0.30 | 0.35 | 0.25 | 0.10 |
| Sin historial ni preferencias | 0.00 | 0.40 | 0.10 | 0.50 |

La calibración se hace con el protocolo de S0-11, no a ojo.

### 6.3 Ajustes
- **Cierre próximo:** penalizar si cierra en < 60 min.
- **Diversidad:** máximo 2-3 restaurantes por categoría en el top 10.
- **Novedad:** penalización leve a restaurantes ya vistos recientemente.

## 7. Cold start (V-08)

| Escenario | Estrategia |
|---|---|
| Usuario nuevo con preferencias | Candidatos por categorías + cercanía (si hay permiso) + popularidad |
| Usuario nuevo sin preferencias | Popularidad (+ cercanía si hay ubicación) |
| Restaurante nuevo | Atributos de contenido (embedding) + exploración limitada; sin historial no compite por popularidad |
| Plato nuevo | No afecta el ranking principal (es atributo del restaurante) |

- La estrategia usada se registra en `recomendaciones.estrategia` (`cold_start`, `popularidad`, `contenido`, `hibrido`) para poder medirla.

## 8. Disponibilidad ("abierto ahora")

- Se calcula con `horarios` y zona horaria `America/Bogota`.
- Regla por defecto: excluir cerrados; si tras filtrar quedan < 5 resultados, incluirlos al final con penalización y aviso "cerrado".
- Forma parte de las pruebas P-016-04 ("Entidades no disponibles").

## 9. Evolución por etapas (RIA-003)

| Etapa | Estrategia | Cuándo |
|---|---|---|
| 1 | Baseline de popularidad | Siempre disponible; punto de comparación |
| 2 | Contenido + pgvector (recuperación) + ranking por reglas | MVP |
| 3 | Híbrido (contenido + colaborativo) | Cuando existan interacciones reales suficientes |
| 4 | Modelo aprendido (learning to rank) | Solo si los datos y la evaluación lo justifican |

El paso de una etapa a otra requiere mejora medible y reproducible (S0-11).

**Nota académica:** el baseline del MVP es la **línea base experimental**, no el resultado final de la investigación. El núcleo de IA del proyecto es el Incremento 2 (motor Python + pgvector + experimentos + comparación baseline vs modelo + evaluación).

## 10. Contrato con FastAPI

- Entrada: `POST /v1/recomendaciones` (ver `../API_v1.0/Contrato_API_v1.0.md`).
- Salida: `recomendacion_id`, `request_id`, `modelo_version`, `estrategia` e `items` con `score` y `razones`.
- El backend persiste la recomendación y sus items para atribución de impresiones/clics.

## 11. Riesgos y mitigaciones

| Riesgo | Mitigación |
|---|---|
| Confundir embeddings con el recomendador | Roles explícitos en ADR-004 y este diseño |
| Pesos mal calibrados | Protocolo de S0-11 + baseline de popularidad |
| Cold start pobre | Estrategias por escenario y fallback a popularidad |
| Sesgo hacia populares | Diversidad + exploración limitada |
| Datos insuficientes | Dataset sintético para pipeline; real para validación (S0-10) |

## 12. Técnicas y afirmación metodológica

### 12.1 Técnicas usadas por etapa

| Etapa | Técnica | Naturaleza |
|---|---|---|
| MVP (fallback) | Popularidad, reglas y filtros (precio, cercanía, "abierto ahora") | Heurística, sin ML |
| MVP (personalización) | Filtrado basado en contenido con **embeddings preentrenados** (transformers) y similitud coseno en pgvector | ML aplicado (sin entrenamiento propio) |
| Incremento 2 | **Filtrado colaborativo** (kNN ítem-usuario, factorización ALS/SVD) | ML entrenado con datos del proyecto |
| Incremento 2 | **Learning-to-rank** (regresión logística / GBDT) sobre impresiones y clics | ML entrenado |
| Incremento 2 | **Híbrido** (contenido + colaborativo + reglas) | Combinación de modelos |
| Evaluación | Split temporal, P@10/R@10/NDCG@10/cobertura y CTR | Metodología experimental |

### 12.2 Afirmación defendible

- **Hoy (MVP):** "Sistema de recomendación basado en contenido que utiliza representaciones vectoriales generadas por **modelos de machine learning preentrenados**, con ranking por reglas, evaluado experimentalmente contra un baseline de popularidad."
- **Al cerrar el Incremento 2:** "Sistema de recomendación **híbrido** que **entrena** un modelo (colaborativo o de ranking) con los datos de interacción y lo compara contra el baseline con métricas offline y de producto."

### 12.3 Precisión obligatoria

- No decir que "pgvector es el algoritmo": es infraestructura de recuperación.
- No presentar resultados con datos sintéticos como evidencia de comportamiento real.
- El baseline es la **línea base experimental**, no el resultado final de la investigación.

## 13. Pendientes

1. Elegir el modelo de embeddings definitivo (experimento).
2. Calibrar pesos y decaimiento con el protocolo de S0-11.
3. Definir el número máximo de platos por restaurante usados en el embedding.
4. Validar la regla de diversidad con usuarios reales.
5. Definir el criterio para promover el híbrido.
