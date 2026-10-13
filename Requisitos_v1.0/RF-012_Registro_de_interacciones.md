# RF-012 — Registro de interacciones
**Tipo:** Requisito funcional  
**Prioridad:** Alta  
**Actor principal:** Sistema

## Descripción
El sistema deberá registrar determinadas interacciones realizadas por los usuarios con restaurantes y platos.

## Eventos definidos (Sprint 0)
- `busqueda`
- `ver_restaurante`
- `ver_plato`
- `recomendacion_impresion`
- `clic_recomendacion`

Metadatos: usuario pseudonimizado, entidad, origen, sesión y timestamp; en recomendaciones: `recomendacion_id`, `request_id`, `modelo_version` y `posicion` (V-05).

## Criterios de aceptación
1. Las interacciones definidas se registran con una estructura consistente.
2. Las interacciones pueden asociarse al usuario cuando corresponda.
3. Se registra el elemento involucrado y la fecha/hora cuando sea necesario.

## Importancia
Estos datos podrán utilizarse para personalización y evaluación del recomendador.

## Estado
Validado en Sprint 0 (septiembre de 2026). Ver `../Sprint_0_Plan_v1.0/Acta_Validacion_Sprint_0_v1.0.md`.
