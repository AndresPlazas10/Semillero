# Priorización MVP (P0/P1/P2) — v1.0

> **Aprobada en el Sprint 0 (S0-03, septiembre de 2026).** Ver `../Sprint_0_Plan_v1.0/Acta_Validacion_Sprint_0_v1.0.md`. No modifica la prioridad Alta/Media registrada en el backlog.

## Criterios

- **P0 — Imprescindible:** sin esta funcionalidad no se puede demostrar el flujo principal del MVP (catálogo → interacción → recomendación → visualización), su base técnica o su núcleo de investigación (recomendador evaluable).
- **P1 — Importante:** aporta cobertura, calidad o rigor al MVP; puede incorporarse después del flujo principal sin invalidarlo.
- **P2 — Diferible:** no bloquea la validación del MVP; se planifica para un incremento posterior.

## Resultado

| Nivel | HU | Total |
|---|---|---:|
| P0 | HU-001, HU-002, HU-004, HU-005, HU-006, HU-007, HU-010, HU-012, HU-013, HU-014, HU-016, HU-017, HU-018, HU-019, HU-020, HU-021, HU-024, HU-025, HU-026, HU-027, HU-032, HU-035, HU-036 | 23 |
| P1 | HU-003, HU-008, HU-009, HU-011, HU-015, HU-022, HU-023, HU-028, HU-029, HU-033 | 10 |
| P2 | HU-030, HU-031, HU-034 | 3 |

## Justificación

### P0

| HU | Justificación |
|---|---|
| HU-001 | Puerta de entrada; habilita preferencias e historial por usuario. |
| HU-002 | Acceso seguro a las funcionalidades personalizadas. |
| HU-004 | Catálogo base de restaurantes. |
| HU-005 | Información necesaria para elegir un restaurante. |
| HU-006 | Oferta gastronómica consultable. |
| HU-007 | Comparación básica de precios. |
| HU-010 | Búsqueda como vía principal de acceso al catálogo. |
| HU-012 | Detalle de plato requerido para decidir y para registrar interacción. |
| HU-013 | Señal principal del recomendador. |
| HU-014 | Señal declarada del recomendador. |
| HU-016 | Núcleo del producto: generar recomendaciones. |
| HU-017 | Conexión móvil–recomendador sin acoplar el cliente a la base de datos. |
| HU-018 | Entrega del valor del MVP al usuario. |
| HU-019 | Mantener la información gastronómica vigente. |
| HU-020 | Administración del catálogo de restaurantes. |
| HU-021 | Administración del catálogo de platos. |
| HU-024 | **Núcleo de investigación:** evaluación del recomendador con métricas y protocolo. |
| HU-025 | Todo usuario nuevo carece de historial; el cold start es obligatorio. |
| HU-026 | **Núcleo de investigación:** entrenamiento y evaluación reproducible del modelo. |
| HU-027 | Base multiplataforma del cliente móvil. |
| HU-032 | Protección de cuentas y operaciones autenticadas. |
| HU-035 | Contrato de integración móvil/backend/recomendador. |
| HU-036 | Verificación de calidad y aceptación del MVP. |

### P1

| HU | Justificación |
|---|---|
| HU-003 | El perfil enriquece la personalización, pero el MVP puede operar con preferencias básicas. |
| HU-008 | Horarios deseables, no bloqueantes para recomendar. |
| HU-009 | Ubicación deseable; la vista de detalle cubre el mínimo del MVP. |
| HU-011 | Los filtros extra mejoran la búsqueda; la búsqueda básica ya cubre el MVP. |
| HU-015 | El historial mejora la experiencia y las señales, pero no bloquea el flujo. |
| HU-022 | Gestión de horarios complementaria. |
| HU-023 | Gestión de ubicación complementaria. |
| HU-028 | La usabilidad se mide sobre flujos integrados; puede iterarse después del P0. |
| HU-029 | El rendimiento se optimiza con el flujo principal ya operativo. |
| HU-033 | La privacidad requiere decisiones de tratamiento que pueden cerrarse en paralelo; los controles mínimos van en P0 (HU-032). |

### P2

| HU | Justificación |
|---|---|
| HU-030 | La escalabilidad se planifica en el diseño, pero no se valida en el primer incremento. |
| HU-031 | La disponibilidad objetivo se establece antes de operar en producción. |
| HU-034 | La mantenibilidad se evalúa de forma continua y no bloquea el MVP. |

## Decisiones aprobadas y seguimiento

1. ✅ Priorización aprobada (P0 = 23, P1 = 10, P2 = 3). P0 define alcance, no orden de sprint.
2. ✅ HU-027 resuelta en variante C: Sprint 0 (configuración, T-027-01/02) y Sprint 1 (verificación multiplataforma, T-027-03/04/05 y P-027-*). Ver V-18 del Acta.
3. ✅ HU-024 y HU-026 promovidas a P0 por ser el núcleo de investigación del proyecto.
4. Pendiente: medir la capacidad real del equipo y re-secuenciar sprints con 23 HU P0.
5. Pendiente: fijar el volumen inicial del dataset sintético (V-09).
