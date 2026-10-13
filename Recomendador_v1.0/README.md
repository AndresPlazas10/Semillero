# Recomendador_v1.0

Entregables de las actividades **S0-09 (diseñar recomendador)**, **S0-10 (definir datos de IA)** y **S0-11 (definir métricas)** del Sprint 0.

## Archivos
- `Diseno_Recomendador_v1.0.md`: pipeline, recuperación, ranking, cold start y evolución al híbrido.
- `Plan_Datos_IA_v1.0.md`: datos sintéticos y reales, split temporal, anti-leakage y reproducibilidad.
- `Metricas_Evaluacion_v1.0.md`: métricas offline y de producto, protocolo experimental y umbrales provisionales.

## Trazabilidad
- Decisiones: `../Sprint_0_Plan_v1.0/Acta_Validacion_Sprint_0_v1.0.md` (V-07, V-08, V-09, V-10, V-19, V-20).
- Arquitectura: `../Arquitectura_v1.0/ADR/ADR-004_Python_pgvector_Recomendacion.md`.
- Modelo de datos y vectores: `../Modelo_Datos_v1.0/`.
- Contrato API: `../API_v1.0/Contrato_API_v1.0.md`.

## Estado
Diseño inicial v1.0 (septiembre de 2026). El modelo de embeddings y los pesos de ranking se validan con experimentos (HU-024/HU-026).
