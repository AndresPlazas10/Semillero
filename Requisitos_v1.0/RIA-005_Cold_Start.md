# RIA-005 — Manejo del Cold Start
**Tipo:** Requisito de inteligencia artificial  
**Prioridad:** Alta

## Descripción
El sistema deberá contemplar el comportamiento ante usuarios, restaurantes o platos nuevos sin historial suficiente.

## Escenarios
1. Usuario nuevo.
2. Restaurante nuevo.
3. Plato nuevo.

## Criterio de aceptación
Debe existir una estrategia definida y evaluable para cada escenario aplicable.

## Decisión (Sprint 0)
Usuario nuevo: cercanía (si hay permiso de ubicación) + preferencias explícitas; sin ubicación o historial → popularidad. Restaurantes y platos nuevos: atributos de contenido y popularidad (V-08).

## Estado
Validado en Sprint 0 (septiembre de 2026). Ver `../Sprint_0_Plan_v1.0/Acta_Validacion_Sprint_0_v1.0.md`.
