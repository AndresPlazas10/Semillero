# RF-015 — Generación de recomendaciones
**Tipo:** Requisito funcional  
**Prioridad:** Alta  
**Actor principal:** Motor de recomendación

## Descripción
El sistema deberá generar recomendaciones gastronómicas personalizadas a partir de la información disponible del usuario.

## Criterios de aceptación
1. El motor recibe los datos disponibles necesarios.
2. El motor procesa dichos datos.
3. Se genera una lista de recomendaciones.
4. Las recomendaciones pueden asociarse a restaurantes y/o platos.
5. El resultado puede ser consumido por la aplicación móvil.

## Decisión (Sprint 0)
Unidad principal: restaurantes (platos como atributos/contenido). Candidato: baseline de popularidad + recuperación por contenido con embeddings y pgvector, con ranking en Python; híbrido condicionado a datos reales. Cold start: cercanía + preferencias explícitas, con popularidad como respaldo. Métricas y protocolo según RF-023/RIA-006 (V-07, V-08, V-19, V-20).

## Estado
Validado en Sprint 0 (septiembre de 2026). Ver `../Sprint_0_Plan_v1.0/Acta_Validacion_Sprint_0_v1.0.md`.
