# Sprint 1 — Materiales de Planning Poker (v1.0)

**Objetivo:** confirmar las estimaciones de HU-027 y HU-001 y calibrar la velocidad inicial.
**Duración sugerida:** 60 minutos.
**Participantes:** 3 integrantes del equipo + facilitador (puede ser uno del equipo).

## 1. Guía de facilitación (timebox)

| Paso | Tiempo | Descripción |
|---|---:|---|
| 1. Contexto | 5 min | Recordar objetivo del sprint y capacidad (54 h efectivas). |
| 2. Leer tarea | 2 min/tarea | El facilitador lee la tarjeta; dudas se aclaran sin estimar. |
| 3. Votar | 1 min | Cada persona elige una carta (1, 2, 3, 5, 8, 13). |
| 4. Discutir | 2 min | Solo si hay diferencias; la persona con el valor extremo explica. |
| 5. Consensuar | 1 min | Se acuerda un valor (o se repite la votación). |
| 6. Registrar | — | Anotar en la tabla de resultados y en el backlog. |

Escala: **1, 2, 3, 5, 8, 13** (Fibonacci). Una tarea ≥ 8 SP se divide.

## 2. Tarjetas de estimación

### HU-027 — Verificación multiplataforma (referencia actual: 13 SP)

| ID | Tarea | Referencia | Voto |
|---|---|---:|---|
| T-027-03 | Verificar navegación y componentes en ambas plataformas | 1 | |
| T-027-04 | Ejecutar build por plataforma (EAS) | 1 | |
| T-027-05 | Registrar incompatibilidades y corregirlas | 1 | |
| P-027-01 | Prueba: build plataforma 1 | 3 | |
| P-027-02 | Prueba: build plataforma 2 | 3 | |
| P-027-03 | Prueba: funciones críticas multiplataforma | 3 | |
| P-027-04 | Prueba: navegación consistente | 1 | |

### HU-001 — Registro de usuario (referencia actual: 21 SP)

| ID | Tarea | Referencia | Voto |
|---|---|---:|---|
| T-001-01 | Definir campos y reglas de validación del registro | 1 | |
| T-001-02 | Configurar entidad/tabla de usuario y restricciones | 3 | |
| T-001-03 | Implementar formulario de registro en móvil | 3 | |
| T-001-04 | Implementar servicio de creación de usuario y manejo de errores | 3 | |
| T-001-05 | Integrar validaciones cliente/servidor | 3 | |
| T-001-06 | Documentar contrato y comportamiento del registro | 3 | |
| P-001-01 | Prueba: registro válido | 1 | |
| P-001-02 | Prueba: campos obligatorios inválidos | 1 | |
| P-001-03 | Prueba: identificador duplicado | 1 | |
| P-001-04 | Prueba: persistencia de usuario | 1 | |
| P-001-05 | Prueba: error controlado de servicio | 1 | |

### Infraestructura (transversal, no suma SP de HU)

| ID | Tarea | Referencia | Voto |
|---|---|---:|---|
| I-01 | Repo + CI + protección de `main` | — | |
| I-02 | Supabase dev + migraciones + seed | — | |
| I-03 | Pruebas RLS por rol | — | |
| I-04 | Despliegue Render Starter | — | |
| I-05 | Endpoint y logger de eventos | — | |

## 3. Plantilla de resultados

| Tarea | Persona A | Persona B | Persona C | Acordado | Notas |
|---|---:|---:|---:|---:|---|
| T-027-03 | | | | | |
| T-027-04 | | | | | |
| T-027-05 | | | | | |
| P-027-01 | | | | | |
| P-027-02 | | | | | |
| P-027-03 | | | | | |
| P-027-04 | | | | | |
| T-001-01 | | | | | |
| T-001-02 | | | | | |
| T-001-03 | | | | | |
| T-001-04 | | | | | |
| T-001-05 | | | | | |
| T-001-06 | | | | | |
| P-001-01 | | | | | |
| P-001-02 | | | | | |
| P-001-03 | | | | | |
| P-001-04 | | | | | |
| P-001-05 | | | | | |

## 4. Cálculo de velocidad

```
Velocidad del sprint = suma de SP de tareas terminadas (cumplen DoD)
Factor de foco real = horas efectivas usadas / horas brutas planificadas
Capacidad del siguiente sprint = (personas × horas/semana × semanas) × factor de foco real
```

- La capacidad inicial era 54 h efectivas; la velocidad en SP se registra al cierre.
- Regla: si la velocidad real queda por debajo de 34 SP, se recorta alcance del Sprint 2 (nunca la calidad de lo entregado).
- Si una tarea resultó mal estimada, se anota para mejorar el siguiente Planning.

## 5. Criterios de estimación acordados

1. Se estima complejidad relativa, no horas exactas (las horas del CSV son orientativas).
2. Una tarea que incluya UI + API + pruebas puede dividirse.
3. Las pruebas se estiman aparte de la implementación.
4. Cualquier tarea ≥ 8 SP se divide antes de entrar al sprint.
