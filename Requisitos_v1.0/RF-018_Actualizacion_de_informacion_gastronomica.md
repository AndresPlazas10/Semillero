# RF-018 — Actualización de información gastronómica
**Tipo:** Requisito funcional  
**Prioridad:** Alta  
**Actor principal:** Comerciante / Administrador

## Descripción
El sistema deberá permitir mantener actualizada la información de restaurantes, menús, precios y horarios.

## Criterios de aceptación
1. Un actor autorizado puede modificar información.
2. Los cambios se almacenan correctamente.
3. Los datos actualizados se reflejan en las consultas posteriores.
4. Se controlan permisos de modificación.

## Decisión (Sprint 0)
La gestión se realiza desde la aplicación móvil con rol Gestor; el Administrador usa herramientas internas (Supabase Studio + pantalla mínima) y el panel web queda pospuesto. Los establecimientos tienen estados pendiente → activo → suspendido con aprobación del Administrador (V-02).

## Estado
Validado en Sprint 0 (septiembre de 2026). Ver `../Sprint_0_Plan_v1.0/Acta_Validacion_Sprint_0_v1.0.md`.
