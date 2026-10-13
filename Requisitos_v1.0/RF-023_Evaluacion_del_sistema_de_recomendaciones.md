# RF-023 — Evaluación del sistema de recomendaciones
**Tipo:** Requisito funcional / investigativo  
**Prioridad:** Alta  
**Actor principal:** Equipo de investigación

## Descripción
El sistema deberá permitir obtener los datos necesarios para evaluar el desempeño del motor de recomendaciones mediante métricas definidas durante la etapa de diseño y experimentación.

## Criterios de aceptación
1. Se dispone de datos para pruebas.
2. Se define un conjunto de evaluación.
3. Se calculan las métricas seleccionadas.
4. Los resultados quedan registrados para comparación.
5. Es posible comparar el desempeño de alternativas cuando corresponda.

## Decisión (Sprint 0)
Métricas offline Precision@10, Recall@10, NDCG@10 y cobertura (sobre restaurantes); baseline de popularidad; CTR de producto (clics/impresiones); split temporal con conjunto de evaluación separado; el criterio final de mejora se fija antes del experimento definitivo (V-09, V-10).

## Estado
Validado en Sprint 0 (septiembre de 2026). Ver `../Sprint_0_Plan_v1.0/Acta_Validacion_Sprint_0_v1.0.md`.
