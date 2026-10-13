# Sprint 1 — Plan (v1.0)

**Proyecto:** Aplicación móvil con IA para recomendación gastronómica en Tunja
**Duración:** 2 semanas
**Capacidad:** 54 h efectivas (3 personas × 15 h/semana × factor de foco 0.6)
**Alcance comprometido:** 34 SP provisionales
**Estado:** En ejecución

## 1. Objetivo

Dejar operativa la base técnica (repo, CI, Supabase dev, API desplegada, app corriendo en Android/iOS, captura de eventos) y entregar el **registro de usuario** de extremo a extremo.

## 2. Alcance

| Bloque | Contenido | SP |
|---|---|---:|
| HU-027 | Verificación multiplataforma (T-027-03/04/05, P-027-01…04) | 13 |
| HU-001 | Registro de usuario (T-001-01…06, P-001-01…05) | 21 |
| Transversal | Repo + CI, Supabase dev + RLS, Render, endpoint y logger de eventos | — |
| **Total** | | **34** |

## 3. Backlog del sprint

### 3.1 Infraestructura (transversal)

| ID | Tarea | Criterio de cierre |
|---|---|---|
| I-01 | Publicar monorepo en GitHub privado + `main` protegido | CI verde en el primer PR |
| I-02 | Supabase dev con migraciones + seed | `0001_init.sql` y `seed.sql` aplicados |
| I-03 | Probar RLS con roles usuario/gestor/administrador | 3 pruebas de acceso por rol |
| I-04 | Desplegar API en Render Starter | `GET /health` público responde |
| I-05 | Endpoint `POST /v1/eventos` + logger móvil | Evento de prueba registrado con origin y sesión |

### 3.2 HU-027 — Verificación multiplataforma

| ID | Tarea | SP |
|---|---|---:|
| T-027-03 | Verificar navegación y componentes | 1 |
| T-027-04 | Ejecutar build por plataforma | 1 |
| T-027-05 | Registrar incompatibilidades y corregirlas | 1 |
| P-027-01 | Build plataforma 1 | 3 |
| P-027-02 | Build plataforma 2 | 3 |
| P-027-03 | Funciones críticas multiplataforma | 3 |
| P-027-04 | Navegación consistente | 1 |

### 3.3 HU-001 — Registro de usuario

| ID | Tarea | SP |
|---|---|---:|
| T-001-01 | Definir campos y reglas de validación del registro | 1 |
| T-001-02 | Configurar entidad/tabla de usuario y restricciones | 3 |
| T-001-03 | Implementar formulario de registro en móvil | 3 |
| T-001-04 | Implementar servicio de creación de usuario y manejo de errores | 3 |
| T-001-05 | Integrar validaciones cliente/servidor | 3 |
| T-001-06 | Documentar contrato y comportamiento del registro | 3 |
| P-001-01 | Registro válido | 1 |
| P-001-02 | Campos obligatorios inválidos | 1 |
| P-001-03 | Identificador duplicado | 1 |
| P-001-04 | Persistencia de usuario | 1 |
| P-001-05 | Error controlado de servicio | 1 |

## 4. Reparto (equipo full-stack)

Todos toman tareas de cualquier capa, asignadas por dependencia. Regla: **una tarea en curso por persona** y revisiones cruzadas de PR.

| Orden | Tarea | Persona |
|---|---|---|
| 1 | I-01 Repo + CI | A |
| 2 | I-02/I-03 Supabase + RLS | B |
| 3 | I-04 Render | C |
| 4 | T-001-01/02 + I-05 eventos | A + B |
| 5 | T-001-03/04/05 móvil + API | B + C |
| 6 | T-027-03/04/05 verificación | C + A |
| 7 | Pruebas P-001/P-027 + T-001-06 | todos |

Las personas se asignan en el Planning (A, B, C = integrantes del equipo).

## 5. Definition of Done del sprint

- [ ] CI verde en `main` (api, mobile, supabase).
- [ ] Migraciones y seed aplicados en Supabase dev; RLS probado con 3 roles.
- [ ] API desplegada en Render Starter; `GET /health` responde.
- [ ] App instalada y funcionando en Android e iOS.
- [ ] Registro de usuario funcional con pruebas P-001-01…05 en verde.
- [ ] Endpoint de eventos y logger probados con un evento de prueba.
- [ ] `Velocidad_Sprint_1_v1.0.md` completado con la velocidad real.
- [ ] Sin defectos críticos abiertos.

## 6. Ceremonias (ligeras)

| Ceremonia | Cuándo | Duración |
|---|---|---|
| Planning + Planning Poker | Día 0 | 60 min |
| Daily asíncrono | Cada día (canal del equipo) | 10 min |
| Review + demo | Último día | 45 min |
| Retro | Último día | 30 min |

## 7. Riesgos

| Riesgo | Mitigación |
|---|---|
| Velocidad real < 34 SP | Priorizar I-01…I-04 + HU-001; HU-027 no se recorta (P0) |
| Credenciales (Supabase/Render) tardías | Gestionarlas el día 0; la infra local no bloquea HU-001 |
| Contexto full-stack | Tareas por dependencia + revisiones cruzadas |
| Repos privados Free sin branch protection | Documentar la regla y activar al pasar a Pro |
| iOS sin dispositivo físico | Usar simulador de Xcode/EAS dev build |

## 8. Trazabilidad

- Capacidad y secuencia: `../Sprint_0_Plan_v1.0/Capacidad_y_Sprint_1_v1.0.md`.
- Estimación de tareas: `../Estimacion_y_Dependencias_v1.0/Estimacion_Tareas_v1.0.csv`.
- Contrato API: `../API_v1.0/Contrato_API_v1.0.md`.
