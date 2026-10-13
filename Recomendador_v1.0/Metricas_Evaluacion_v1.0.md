# Métricas y Evaluación — v1.0 (S0-11)

**Proyecto:** Aplicación móvil con IA para recomendación gastronómica en Tunja
**Fecha:** septiembre de 2026
**Estado:** Diseño inicial — umbral definitivo se fija antes del experimento final

## 1. Propósito

Definir cómo se mide el desempeño del recomendador: métricas offline, métricas de producto, baseline, protocolo experimental y criterios de comparación (V-10, RIA-006, RF-023).

## 2. Métricas offline (unidad: restaurantes)

| Métrica | Definición | Interpretación |
|---|---|---|
| **Precision@10** | Proporción de restaurantes relevantes entre los 10 recomendados | Precisión de la lista |
| **Recall@10** | Proporción de restaurantes relevantes recuperados en el top 10 | Cobertura por usuario |
| **NDCG@10** | Ganancia acumulada descontada por posición, normalizada | Calidad del orden |
| **Cobertura** | % de restaurantes activos que aparecen al menos una vez en las listas | Diversidad del catálogo |

- **Relevancia:** interacción positiva (clic_recomendacion, ver_restaurante, ver_plato) dentro de la ventana de evaluación.
- Se reportan por segmento: con historial, con preferencias sin historial y cold start puro.

## 3. Métricas de producto

| Métrica | Fórmula | Fuente |
|---|---|---|
| **CTR** | `clic_recomendacion` / `recomendacion_impresion` | Eventos (V-05) |
| **CTR por posición** | CTR agrupado por `posicion` | Eventos |
| **CTR por modelo** | CTR agrupado por `modelo_version` | Eventos |
| **Tiempo a primera interacción** | Tiempo desde la recomendación hasta el primer clic | Eventos |
| **Uso de filtros/categorías** | Eventos `busqueda` con filtros | Eventos |

- El CTR requiere obligatoriamente el evento `recomendacion_impresion`; sin él la métrica no existe.
- Las métricas de producto se miden en el piloto, con datos reales.

## 4. Baseline

- **Baseline de popularidad:** restaurantes más interactuados en la ventana previa (p. ej. 30 días), sin personalización.
- Toda estrategia (contenido, híbrido) debe compararse contra este baseline con el **mismo split, mismos usuarios y misma ventana**.
- El baseline también es el fallback de cold start sin preferencias ni ubicación.
- **El baseline es la línea base experimental del proyecto, no su resultado final:** la comparación contra el modelo del Incremento 2 (Python + pgvector) es el aporte de investigación.

## 5. Protocolo experimental

1. **Datos:** dataset sintético (pruebas técnicas) o real (validación experimental), con split temporal (S0-10).
2. **Modelos a comparar:** baseline de popularidad, contenido + pgvector, híbrido (cuando aplique).
3. **Configuración fija:** mismo conjunto de test out-of-time, misma semilla por corrida, mismos pesos por variante documentados.
4. **Repeticiones:** mínimo 3 corridas por configuración (variando semilla del generador o de muestreo) y reportar media y dispersión.
5. **Registro:** cada corrida queda en el registro de experimentos con dataset, versión del modelo de embeddings, pesos, métricas y fecha.
6. **Reporte:** tabla comparativa + conclusiones; separar explícitamente resultados sintéticos y reales.

## 6. Criterio de comparación

- **Objetivo provisional:** mejorar el baseline en NDCG@10 y Precision@10 sin degradar la cobertura (margen preliminar: mejora visible y consistente en las repeticiones).
- **Criterio final:** se fija **antes del experimento definitivo**, considerando volumen, dispersión y dificultad del problema. No se adopta un porcentaje fijo (p. ej. 10 %) como ley.
- Un modelo que mejora poco pero de forma consistente puede ser preferible a uno con mejora alta e inestable.
- Si ningún modelo supera al baseline de forma consistente, se mantiene el baseline y se documenta el hallazgo.

## 7. Umbrales provisionales

| Métrica | Umbral provisional | Estado |
|---|---|---|
| Cobertura de recuperación | ≥ 80 % de los relevantes entre los candidatos | Por validar con datos |
| NDCG@10 | Superar el baseline de popularidad | A confirmar en el experimento final |
| CTR | Comparar por `modelo_version`; sin umbral hasta el piloto | Pendiente de datos reales |
| Rendimiento | p95 < 3 s para generar recomendaciones (RNF-003) | Aprobado en V-13 |

## 8. Registro de experimentos

- Herramienta: **MLflow** (condicional al volumen) o registro CSV/Markdown versionado.
- Cada experimento registra: `experimento_id`, fecha, dataset (versión y hash), modelo de embeddings, estrategia, pesos, métricas, responsable.
- Los resultados alimentan HU-024 (evaluación) y HU-026 (entrenamiento/evaluación del modelo).

## 9. Trazabilidad

| Requisito | Dónde se cumple |
|---|---|
| RIA-006 / RF-023 | Este documento (métricas, baseline, protocolo) |
| V-10 | Secciones 2, 3 y 6 |
| HU-024 | Protocolo de evaluación y reporte |
| HU-026 | Dataset y reproducibilidad (S0-10) |
| P-024-01…05 | Pruebas de evaluación y registro |
| P-026-01…04 | Separación, reproducibilidad y ausencia de fuga |

## 10. Pendientes

1. Fijar el criterio final de mejora antes del experimento definitivo.
2. Definir la ventana de relevancia para las etiquetas offline.
3. Confirmar el uso de MLflow.
4. Definir el tablero de métricas de producto del piloto.
5. Acordar el volumen mínimo de datos reales para la validación experimental.
