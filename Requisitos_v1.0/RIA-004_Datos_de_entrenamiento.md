# RIA-004 — Datos de entrenamiento y evaluación
**Tipo:** Requisito de inteligencia artificial  
**Prioridad:** Alta

## Descripción
El sistema deberá contar con datos suficientes para entrenar, probar o evaluar el algoritmo seleccionado.

## Contexto inicial
El anteproyecto contempla datos sintéticos para pruebas offline del motor.

## Decisión (Sprint 0)
Datos sintéticos reproducibles para pipeline y pruebas técnicas; datos reales para validación experimental. Split temporal (pasado → entrenamiento, siguiente → validación, reciente → prueba; alternativa leave-last-out por usuario) y estrategia para usuarios con pocas interacciones. El volumen inicial queda pendiente de fijar por el equipo (V-09).

## Estado
Validado en Sprint 0 (septiembre de 2026). Ver `../Sprint_0_Plan_v1.0/Acta_Validacion_Sprint_0_v1.0.md`.
