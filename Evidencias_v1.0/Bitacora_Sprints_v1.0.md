# Bitácora de evidencias por sprint

**Proyecto:** Aplicación móvil con IA para recomendación gastronómica en Tunja  
**Versión del registro:** 1.0  
**Última actualización:** 23 de octubre de 2026
**Propósito:** conservar evidencia verificable de planificación, ejecución, pruebas, decisiones y resultados para el informe final en Word.

> **Regla de registro:** los datos no confirmados quedan como “Pendiente”. No completar horas, puntos, pruebas, despliegues ni resultados por estimación retrospectiva.

## 1. Resumen del roadmap

| Sprint | Alcance planificado | SP planificados (peso) | Estado de evidencia |
|---|---|---:|---|
| Sprint 0 | Validación, requisitos, arquitectura, diseño y preparación | No aplica | Cierre documental por reconciliar |
| Sprint 1 | Infraestructura, HU-027 y HU-001 | 34 | Cierre pendiente de verificaciones y datos del equipo |
| Sprint 2 | HU-002 y HU-004; HU-007 candidata según capacidad | 34 + candidata | Planificado; validar al cierre de S1 |
| Sprint 3 | HU-005 y HU-006 | 28 | Planificado |
| Sprint 4 | HU-012 y HU-013 | 28 | Planificado |
| Sprint 5 | HU-014 y baseline de recomendaciones HU-016 | 36 | Planificado |
| Sprint 6 | Presentación de recomendaciones HU-018 y cold start HU-025 | 26 | Planificado |
| Incremento 2 | Motor Python/pgvector, entrenamiento y evaluación, gestión de catálogo y evolución del recomendador | 364 SP según planificación | Pendiente de planificación y ejecución |

La secuencia S1–S6 es provisional y debe ajustarse con la capacidad y velocidad reales del equipo. La referencia es `Sprint_0_Plan_v1.0/Capacidad_y_Sprint_1_v1.0.md`.

## 2. Sprint 0 — Diseño y preparación

**Estado registrado:** por reconciliar entre artefactos.  
**Periodo indicado en los artefactos:** septiembre de 2026.

### Alcance y entregables documentados

- Validación de requisitos e historias, priorización del MVP y decisiones V-01 a V-20.
- Arquitectura de app móvil, FastAPI, Supabase/PostgreSQL y recomendador Python.
- Contrato de API, modelo de datos, diseño de recomendación, privacidad y plan de datos/IA.
- Backlog de 36 historias refinadas y priorización P0/P1/P2.

### Observación de control documental

El README de `Sprint_0_Plan_v1.0/` declara Sprint 0 completado. El `Acta_Validacion_Sprint_0_v1.0.md` indica que esa acta cierra S0-01 a S0-03 y que, en la fecha del acta, el Sprint 0 completo no se declaraba DONE. Por eso esta bitácora no asigna un estado final hasta que el equipo confirme qué actividades y DoD se completaron después.

### Evidencia vinculada

- `Sprint_0_Plan_v1.0/Acta_Validacion_Sprint_0_v1.0.md`
- `Sprint_0_Plan_v1.0/README.md`
- `Arquitectura_v1.0/`
- `API_v1.0/Contrato_API_v1.0.md`
- `Modelo_Datos_v1.0/`
- `Recomendador_v1.0/`
- `Seguridad_Privacidad_v1.0/`
- `Historias_Usuario_v1.1_Refinadas/`

### Datos de cierre

| Campo | Registro |
|---|---|
| Actividades S0-01 a S0-15 completadas | Pendiente de reconciliar |
| DoD final del Sprint 0 | Pendiente de reconciliar |
| Decisiones abiertas | Dataset sintético, revisión jurídica de privacidad y otros pendientes anotados en los documentos de origen |
| Responsable de confirmar el estado | Pendiente |

## 3. Sprint 1 — Infraestructura, registro y base multiplataforma

**Estado del plan:** en ejecución; cierre pendiente.  
**Duración planificada:** 2 semanas.  
**Capacidad planificada:** 54 horas efectivas; 90 horas brutas.  
**Alcance:** 34 SP provisionales: HU-027 (13 SP) y HU-001 (21 SP), además de tareas transversales.

**Nota de versión:** el estado descrito en la sección siguiente corresponde al corte del 18-oct-2026 y al PR #1 de ese momento. Al 23-oct-2026, los cambios de migración y prueba de privilegios, formulario de registro, endpoint de eventos y cliente móvil están publicados en la rama `feat/sprint-1-close` del repositorio de producto. No se debe interpretar la referencia histórica al PR #1 como confirmación de su estado remoto actual.

### Alcance planificado

- I-01: repositorio privado, `main` protegido y CI.
- I-02: proyecto Supabase dev, migraciones y seed.
- I-03: pruebas RLS con usuario, gestor y administrador.
- I-04: despliegue de API en Render y respuesta de `/health`.
- I-05: `POST /v1/eventos` y logger móvil, con prueba de evento.
- HU-027: comprobación multiplataforma.
- HU-001: registro de usuario de extremo a extremo.

### Trabajo y verificaciones registrados al 18-oct-2026

| Elemento | Resultado observado | Estado de evidencia |
|---|---|---|
| I-01 repositorio y CI | Repositorio privado en GitHub; `main` protegida con checks requeridos y una aprobación. Los checks de API, Mobile y Supabase pasan en el PR #1 para el commit `9ab7337`. | Checks del PR aprobados; merge/revisión pendiente |
| I-02 migraciones y seed | `supabase db reset` local aplicó `0001_init.sql`, `0002_profile_privileges.sql` y `seed.sql` sin error. El workflow de Supabase también pasó en el PR #1. Falta confirmar el proyecto Supabase dev remoto. | Local y CI completados; remoto pendiente |
| I-03 RLS por rol | `supabase/tests/rls_roles.test.sql` ejecutó 15 pruebas locales aprobadas y el mismo workflow pasó en el PR #1: perfil propio, rol no modificable, aislamiento de restaurantes para usuario/gestor y acceso administrativo. | Local y CI completados; remoto pendiente |
| I-04 Render | Servicio publicado en https://recomendacion-gastronomica-api.onrender.com. El usuario confirmó que `GET /health` devuelve `{"status":"ok"}` el 18-oct-2026. | Verificado por respuesta del health check |
| I-05 eventos | En la otra copia local se probó `POST /v1/eventos`: sin token devuelve 401; con usuario sin consentimiento devuelve 403; con un consentimiento sintético local devuelve 200 y persiste un evento. El usuario y los datos de prueba se eliminaron después. El endpoint/helper no están incluidos en el PR #1. | Prueba de integración local parcial; pendiente portar al clon activo y repetir con consentimiento de prueba |
| HU-001 registro | Contra Supabase Auth local: registro válido devuelve 200, correo inválido 400, duplicado 422 con mensaje de cuenta ya registrada; el trigger creó el perfil. La cuenta temporal se eliminó. No se probó la interfaz móvil ni todos los casos P-001-01…05. | Servicio Auth verificado localmente; aceptación de la app pendiente |
| HU-027 multiplataforma | TypeScript, ESLint y Expo Doctor (21/21) pasan en local. No se han documentado builds ni pruebas funcionales en Android/iOS. | Parcial; builds y pruebas de plataforma pendientes |
| Defectos críticos | Se detectó el riesgo de que el usuario actualice su rol global. `0002_profile_privileges.sql` limita las columnas editables; las 15 pruebas RLS pasan en local y CI. | Corrección verificada localmente y en CI |

### Verificaciones locales

- API: `pytest` → **2 passed**; Ruff lint y formato → **passed**.
- Móvil: TypeScript y ESLint → **passed**; Expo Doctor → **21/21 checks passed**.
- Supabase local: `supabase db reset` aplicó migraciones y seed; `supabase test db` → **15 passed**.
- Registro/Auth local: válido **200**, email inválido **400**, duplicado **422**; perfil confirmado por trigger y usuario temporal eliminado.
- Eventos en copia de implementación local: sin token **401**, sin consentimiento **403**, con consentimiento sintético de prueba **200** y evento persistido; se limpiaron los datos de prueba.
- Render: `GET https://recomendacion-gastronomica-api.onrender.com/health` → **200**, `{"status":"ok"}`.
- `supabase db lint --level warning` terminó con código **0**; los diagnósticos emitidos mencionaron funciones PostGIS incluidas por la extensión.
- `git diff --check` → **passed**.
- Estos resultados corresponden al árbol de trabajo local; no implican despliegue ni prueba real con Supabase.

### Referencias de CI del commit base

- API: https://github.com/AndresPlazas10/recomendacion-gastronomica-tunja/actions/runs/35884985613
- Mobile: https://github.com/AndresPlazas10/recomendacion-gastronomica-tunja/actions/runs/35884985609
- Supabase: https://github.com/AndresPlazas10/recomendacion-gastronomica-tunja/actions/runs/35884985453

### Pull request de migración y RLS

- PR #1: https://github.com/AndresPlazas10/recomendacion-gastronomica-tunja/pull/1
- Commit: `9ab7337` (`fix(supabase): protect profile roles and test RLS`).
- API: https://github.com/AndresPlazas10/recomendacion-gastronomica-tunja/actions/runs/39188126873
- Mobile: https://github.com/AndresPlazas10/recomendacion-gastronomica-tunja/actions/runs/39188126891
- Supabase: https://github.com/AndresPlazas10/recomendacion-gastronomica-tunja/actions/runs/39188126990
- PR #1 se fusionó el 19-oct-2026 con el merge commit `135820a`; los tres checks pasaron.

### Actualización del checkout local — 23-oct-2026

Los siguientes resultados se ejecutaron durante la revisión del checkout local. Son evidencia nueva y no sustituyen los checks de CI ni las verificaciones pendientes en Supabase remoto.

| Área | Resultado observado | Estado |
|---|---|---|
| Registro móvil (HU-001) | El formulario y la llamada a Supabase Auth están implementados en el checkout. TypeScript y ESLint pasan. No se ejecutaron los cinco casos P-001-01…05 de extremo a extremo ni una prueba en dispositivo. | Implementación local; aceptación pendiente |
| Eventos (I-05) | Existe `POST /v1/eventos`, con validación de sesión, consentimiento de interacciones y persistencia; el cliente móvil envía lotes autenticados. API: Ruff lint y formato pasan; pytest: 5 pruebas pasan. | Implementado y verificado unitariamente; integración con Supabase pendiente |
| Privilegios de perfiles | Está en el checkout `0002_profile_privileges.sql` y una prueba pgTAP con 5 aserciones sobre permisos y `rol_global`. | Migración y prueba preparadas; ejecución local pendiente |
| Calidad móvil | `npx expo-doctor`: 21/21; `npm run typecheck` y `npm run lint`: pasan. Se actualizaron dependencias Expo para resolver el fallo de compatibilidad detectado inicialmente. No existe script `npm test`. | Verificado localmente; pruebas unitarias móviles pendientes |
| Supabase | `supabase status` no pudo conectar porque Docker no está disponible, así que no se ejecutó localmente. En el PR #2, CI pasó `supabase start`, `db reset`, `test db` y `db lint`. | CI aprobado; validación del proyecto dev remoto pendiente |
| Android/iOS | No se ejecutaron builds ni pruebas funcionales por plataforma en esta revisión. | Pendiente; HU-027 no cumple aún toda su evidencia de aceptación |
| Auditoría npm | Tras `npm audit fix` sin `--force`, `npm audit --omit=dev` aún reporta 7 vulnerabilidades (4 altas, 3 moderadas); las correcciones sugeridas implican cambios mayores de Expo. | Riesgo abierto para triage; no se aplicó una actualización mayor |
| Credenciales | El `.env.example` actual contiene placeholders, no tiene cambios locales y el usuario confirma que los valores anteriores nunca se publicaron. | Sin credenciales en el cambio actual |

Los commits `17e10b5` (`feat(sprint-1): registro, eventos y privilegios RLS`) y `19b6cd7` (integración de `main` y resolución de conflictos) están publicados en `feat/sprint-1-close`. El [PR #2](https://github.com/AndresPlazas10/recomendacion-gastronomica-tunja/pull/2) está abierto y sin conflictos; los checks API, Mobile y Supabase pasan. GitHub aún requiere revisión antes de permitir el merge. No se hizo push directo a `main`.

### Pendientes de cierre

1. Obtener la revisión requerida y fusionar el PR #2; sus checks API, Mobile y Supabase ya pasaron.
2. Abrir el PR de `docs/sprint-1-evidence` después de actualizar el gitlink de la app al commit integrado definitivo.
3. Verificar los permisos requeridos en Supabase dev remoto y conservar la evidencia de CI de migraciones, seed y RLS.
4. Ejecutar P-001-01…05 de extremo a extremo, incluyendo persistencia del perfil y manejo de errores; registrar la evidencia.
5. Probar `POST /v1/eventos` con consentimiento de prueba explícito contra Supabase y verificar persistencia.
6. Ejecutar y documentar builds y funciones críticas Android/iOS para HU-027.
7. Triagear las vulnerabilidades npm restantes sin forzar un cambio mayor de Expo.
8. Registrar horas efectivas, historias que cumplieron DoD, SP completados y retrospectiva con el equipo; no estimar esos valores retrospectivamente.

### Cierre cuantitativo — completar con el equipo

| Indicador | Planificado | Real |
|---|---:|---:|
| Personas | 3 | Pendiente |
| Horas efectivas | 54 h | Pendiente |
| Horas brutas | 90 h | Pendiente |
| SP comprometidos | 34 | 34 |
| SP HU-027 aceptados con DoD | 13 | Pendiente |
| SP HU-001 aceptados con DoD | 21 | Pendiente |
| Velocidad | — | Pendiente de DoD y SP reales |
| Factor de foco | 0.6 propuesto | Pendiente: horas efectivas / 90 |
| Capacidad sugerida S2 | — | Pendiente: horas reales y disponibilidad del siguiente sprint |

**Retrospectiva:** Pendiente de la reunión de cierre.  
**Decisión de alcance para S2:** HU-002 + HU-004; considerar HU-007 según capacidad observada.

## 4. Sprint 2 — Inicio de sesión y catálogo con seed

**Estado:** planificado; confirmar tras el cierre de S1.  
**Alcance planificado:** HU-002 (inicio de sesión) y HU-004 (explorar restaurantes con seed), 34 SP provisionales. HU-007 (precios, 11 SP) es candidata según la velocidad.

| Registro | Resultado |
|---|---|
| Fechas reales / participantes | Pendiente |
| Historias comprometidas y SP | Pendiente de Planning |
| Trabajo completado y enlaces a PR/commits | Pendiente |
| Pruebas y resultados | Pendiente |
| Incidentes, decisiones y cambios de alcance | Pendiente |
| Horas efectivas | Pendiente |
| SP completados con DoD | Pendiente |
| Velocidad y factor de foco | Pendiente |
| Retrospectiva y acciones para S3 | Pendiente |

## 5. Sprint 3 — Detalle de restaurante y consulta de platos

**Estado:** planificado.  
**Alcance planificado:** HU-005 y HU-006, 28 SP provisionales.

| Registro | Resultado |
|---|---|
| Fechas reales / participantes | Pendiente |
| Historias comprometidas y SP | Pendiente de Planning |
| Trabajo completado y enlaces a PR/commits | Pendiente |
| Pruebas y resultados | Pendiente |
| Incidentes, decisiones y cambios de alcance | Pendiente |
| Horas efectivas | Pendiente |
| SP completados con DoD | Pendiente |
| Velocidad y factor de foco | Pendiente |
| Retrospectiva y acciones para S4 | Pendiente |

## 6. Sprint 4 — Detalle de plato e interacciones

**Estado:** planificado.  
**Alcance planificado:** HU-012 y HU-013, 28 SP provisionales.

| Registro | Resultado |
|---|---|
| Fechas reales / participantes | Pendiente |
| Historias comprometidas y SP | Pendiente de Planning |
| Trabajo completado y enlaces a PR/commits | Pendiente |
| Pruebas y resultados | Pendiente |
| Incidentes, decisiones y cambios de alcance | Pendiente |
| Horas efectivas | Pendiente |
| SP completados con DoD | Pendiente |
| Velocidad y factor de foco | Pendiente |
| Retrospectiva y acciones para S5 | Pendiente |

## 7. Sprint 5 — Preferencias y baseline de recomendación

**Estado:** planificado.  
**Alcance planificado:** HU-014 y HU-016, 36 SP provisionales. El baseline inicial se implementa con reglas y datos disponibles; todavía no equivale al entrenamiento del modelo del Incremento 2.

| Registro | Resultado |
|---|---|
| Fechas reales / participantes | Pendiente |
| Historias comprometidas y SP | Pendiente de Planning |
| Estrategia baseline, reglas y versión | Pendiente |
| Trabajo completado y enlaces a PR/commits | Pendiente |
| Pruebas y resultados | Pendiente |
| Incidentes, decisiones y cambios de alcance | Pendiente |
| Horas efectivas | Pendiente |
| SP completados con DoD | Pendiente |
| Velocidad y factor de foco | Pendiente |
| Retrospectiva y acciones para S6 | Pendiente |

## 8. Sprint 6 — Presentación y cold start

**Estado:** planificado.  
**Alcance planificado:** HU-018 y HU-025, 26 SP provisionales.

| Registro | Resultado |
|---|---|
| Fechas reales / participantes | Pendiente |
| Historias comprometidas y SP | Pendiente de Planning |
| Trabajo completado y enlaces a PR/commits | Pendiente |
| Pruebas y resultados | Pendiente |
| Incidentes, decisiones y cambios de alcance | Pendiente |
| Horas efectivas | Pendiente |
| SP completados con DoD | Pendiente |
| Velocidad y factor de foco | Pendiente |
| Retrospectiva y conclusión del MVP | Pendiente |

## 9. Incremento 2 — Motor, entrenamiento y evaluación

**Estado:** pendiente de planificación.  
**Alcance de referencia:** motor Python + pgvector, preparación de datos, entrenamiento/evaluación, comparación con baseline, gestión del catálogo y evolución hacia estrategias híbridas. Los resultados y las corridas se registran en `Registro_Experimentos_Recomendador_v1.0.md`.

| Registro | Resultado |
|---|---|
| Fechas, objetivo y participantes | Pendiente |
| Historias comprometidas | Pendiente de Planning |
| Dataset y protocolo aprobado | Pendiente |
| Experimentos vinculados | Pendiente |
| Cambios, conclusiones y decisión de producto | Pendiente |
