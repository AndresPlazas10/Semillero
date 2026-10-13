# RNF-003 — Rendimiento
**Tipo:** Requisito no funcional  
**Prioridad:** Alta

## Descripción
Las operaciones principales de consulta y navegación deberán ejecutarse en tiempos adecuados para proporcionar una experiencia fluida.

## Decisión (Sprint 0)
p95 < 2 s en consultas de catálogo/búsqueda y p95 < 3 s en generación de recomendaciones; medición en dispositivo de gama media y red 4G simulada. Las consultas de cercanía requieren PostGIS e índice espacial (V-13).

## Estado
Validado en Sprint 0 (septiembre de 2026). Ver `../Sprint_0_Plan_v1.0/Acta_Validacion_Sprint_0_v1.0.md`.
