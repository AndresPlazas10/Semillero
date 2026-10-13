# RIA-003 — Algoritmo de recomendación
**Tipo:** Requisito de inteligencia artificial  
**Prioridad:** Alta

## Descripción
Deberá seleccionarse e implementarse un algoritmo de recomendación apropiado para el problema.

## Alternativas contempladas
- Filtrado colaborativo.
- Filtrado basado en contenido.
- Sistema híbrido.

## Criterio de aceptación
La selección deberá justificarse mediante criterios técnicos y/o experimentales.

## Decisión (Sprint 0)
Candidatos: baseline de popularidad + filtrado basado en contenido con embeddings/pgvector para recuperación de candidatos y ranking en Python; el híbrido (contenido + colaborativo) se evalúa cuando existan datos reales. pgvector es infraestructura de recuperación, no el algoritmo (V-07).

## Estado
Validado en Sprint 0 (septiembre de 2026). Ver `../Sprint_0_Plan_v1.0/Acta_Validacion_Sprint_0_v1.0.md`.
