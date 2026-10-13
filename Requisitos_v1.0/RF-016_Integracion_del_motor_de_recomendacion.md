# RF-016 — Integración del motor de recomendación
**Tipo:** Requisito funcional  
**Prioridad:** Alta  
**Actor principal:** Aplicación móvil / Motor de recomendación

## Descripción
El sistema móvil deberá comunicarse con el componente encargado de generar las recomendaciones.

## Criterios de aceptación
1. Existe un mecanismo de comunicación definido.
2. La aplicación puede solicitar recomendaciones.
3. El motor devuelve una respuesta estructurada.
4. Se contemplan errores de comunicación.
5. La interfaz de integración queda documentada.

## Decisión (Sprint 0)
El motor Python se expone como módulo interno a través de FastAPI (un solo despliegue); la separación como servicio independiente se evalúa si la carga o el equipo lo justifican (V-11).

## Estado
Validado en Sprint 0 (septiembre de 2026). Ver `../Sprint_0_Plan_v1.0/Acta_Validacion_Sprint_0_v1.0.md`.
