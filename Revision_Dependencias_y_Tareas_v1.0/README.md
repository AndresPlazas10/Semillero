# Revisión de dependencias críticas y tareas grandes v1.0

## Resultado
Se revisaron las tareas técnicas derivadas de las 36 HU. Se marcaron como candidatas a división las tareas con alcance transversal o que pueden concentrar demasiada complejidad.

En la revisión inicial se identificaron T-026-01 y T-026-04 con una estimación de referencia de 8 SP. Tras el re-scoping y la estimación consolidada de `../Estimacion_y_Dependencias_v1.0/Estimacion_Tareas_v1.0.csv`, ambas quedaron en 3 SP, por lo que se subdividieron en subtareas verificables **sin alterar el total de 19 SP de HU-026**:

| Tarea original | Subdivisión aplicada | SP actual |
|---|---|---|
| T-026-01 Definir dataset de entrenamiento | T-026-01a (2 SP) + T-026-01b (1 SP) | 3 |
| T-026-04 Implementar entrenamiento | T-026-04a (2 SP) + T-026-04b (1 SP) | 3 |

El detalle y el estado de cierre están en `Tareas_a_dividir_v1.0.csv` y en `Revision_Dependencias_y_Tareas_v1.0.xlsx`.

## Regla
Una tarea grande no debe entrar al sprint como una unidad opaca. Debe dividirse en piezas verificables, preferiblemente de 3–5 SP, conservando la HU como unidad de valor.

## Dependencias críticas
1. Plataforma/entorno → autenticación → catálogo → interacciones → IA.
2. Captura de datos → entrenamiento/evaluación → recomendador → API → móvil.
3. Modelo de restaurante → gestión de datos → consulta.
4. Flujos MVP integrados → pruebas de calidad → aceptación.

## Próximo paso recomendado
Con esta revisión, el siguiente artefacto debe ser el **Product Backlog priorizado**, incorporando: HU, tarea, dependencia, SP, prioridad, riesgo, criterio de entrada al sprint y estado.
