# Propuestas de Validación — Sprint 0 (v1.0)

Documento de propuestas y decisiones para las actividades **S0-01 (validar requisitos)**, **S0-02 (validar HU)** y **S0-03 (priorizar MVP)** del Sprint 0. Decisiones aprobadas en septiembre de 2026.

> Este documento **no modifica** los artefactos oficiales. Las decisiones se registran aquí y, una vez aprobadas, se trasladan al Acta de Validación y a los documentos fuente (requisitos, HU, backlog, matriz, checklist y estimación).

**Fecha:** septiembre de 2026
**Estado:** ✅ Aprobado con ajustes (septiembre de 2026) — decisiones trasladadas al `Acta_Validacion_Sprint_0_v1.0.md`

## Cómo usar este documento

- Cada ficha **V-XX** contiene: situación actual, opciones, recomendación e impacto.
- El equipo marca una opción por ficha con los estados: ⬜ Pendiente · ✅ Aprobado · ✏️ Ajustar · ❌ Rechazado.
- Las decisiones aprobadas se aplican al final en un solo pase, y luego se revalida la consistencia (SP, IDs, estados, trazabilidad).
- Convención: *(recomendada)* indica la opción sugerida por el análisis, no una decisión tomada.

## Contexto ya cerrado (no se reabre)

| Tema | Decisión | Fuente |
|---|---|---|
| Móvil | React Native + Expo + TypeScript | Plan Sprint 0 v1.0 |
| Backend/API | FastAPI + Python como frontera de servicio | Plan Sprint 0 v1.0 |
| Datos/Auth | PostgreSQL + Supabase (Auth, RLS) | Plan Sprint 0 v1.0 |
| Vectorial | pgvector en PostgreSQL | Plan Sprint 0 v1.0 |
| ML | Python; scikit-learn y MLflow condicionales | Plan Sprint 0 v1.0 |
| Pruebas/Versionado/Release | Postman + pytest; Git + GitHub; Expo EAS | Plan Sprint 0 v1.0 |
| Algoritmo concreto | Sujeto a validación de datos y experimentos | Plan Sprint 0 v1.0 |

---

# A. Alcance y actores

## V-01 — Actores, roles y permisos

**Requisitos/HU:** RF-001, RF-002, RF-003, RF-018–RF-022, RNF-006; HU-001–HU-023.
**Situación actual:** las HU v1.1 identifican actores (usuario, usuario nuevo, administrador/gestor, investigador/desarrollador, sistema) pero no existe matriz de permisos. S0-02 exige confirmarlos.

**Opciones:**
- **A (recomendada):** 4 roles — **Usuario final**, **Gestor** (comerciante), **Administrador** e **Investigador** — con permisos por rol aplicados con RLS en Supabase. En el MVP un Administrador global y Gestores asociados a restaurantes.
- **B:** 2 roles en el MVP — Usuario y Administrador (el administrador carga todo el catálogo); Gestor/Comerciante queda para una fase posterior.
- **C:** roles granulares por restaurante (multi-tenant) con invitaciones y auditoría desde el MVP.

**Impacto si se aprueba:** RF-018/019/020, HU v1.1, matriz de trazabilidad, modelo de datos (roles/permisos), RLS (S0-12).
**Decisión final (Acta):** ✅ A con ajuste — 3 roles de producto (Usuario, Gestor, Administrador); el acceso de investigación se separa (datos anonimizados/agregados + entorno de análisis), no como rol de la aplicación.

## V-02 — Canal de gestión gastronómica y aprobación de establecimientos

**Requisitos/HU:** RF-018, RF-019, RF-020, RF-021, RF-022; HU-019–HU-023.
**Situación actual:** pendiente definir si la gestión se realiza desde la aplicación móvil, un panel administrativo independiente o ambos. RF-019 también deja pendientes permisos, aprobación de establecimientos y validación de la información.

**Opciones:**
- **A (recomendada):** en el MVP la gestión se hace **desde la app móvil** con rol Gestor/Administrador; el establecimiento tiene estados **pendiente → activo → suspendido**, con aprobación del Administrador. El panel web se evalúa en P1/P2 según necesidades del piloto.
- **B:** panel administrativo web desde el MVP; la app móvil solo consulta.
- **C:** alta directa de comerciantes sin aprobación, con validación automática de campos obligatorios.

**Impacto si se aprueba:** RF-018/019, HU-019–HU-023, priorización MVP, modelo de datos (estados y auditoría), contrato API.
**Decisión final (Acta):** ✅ A con ajuste — el Gestor gestiona desde la app móvil; el Administrador usa herramientas internas (Supabase Studio + pantalla mínima) y el panel web queda pospuesto. Estados: pendiente → activo → suspendido con aprobación.

## V-03 — Campos del perfil y uso en el recomendador

**Requisitos/HU:** RF-003, RF-013, RIA-001, RIA-002; HU-003, HU-014.
**Situación actual:** pendiente definir los campos exactos del perfil y cuáles utiliza el recomendador.

**Opciones:**
- **A (recomendada):** campos — nombre/apodo, zona habitual, rango de precio preferido, preferencias alimentarias (categorías) y restricciones (opcional). El recomendador usa **zona, rango de precio y preferencias**; el resto es informativo.
- **B:** perfil mínimo (solo nombre o apodo); la personalización depende únicamente de las interacciones.
- **C:** perfil ampliado (datos demográficos, presupuesto exacto, frecuencia de consumo).

**Impacto si se aprueba:** RF-003, RF-013, HU-003/014, modelo de datos, RIA-002, pantallas de onboarding.
**Decisión final (Acta):** ✅ A con ajuste — perfil: nombre/apodo, rango de precio, preferencias alimentarias y restricciones; sin zona habitual. El motor usa la ubicación actual aproximada (GPS con permiso, radio con expansión) y las preferencias priorizan sobre los disponibles. Sin permiso → solo popularidad. No se persisten coordenadas exactas. Ver V-19 y V-20.

## V-04 — Filtros concretos del MVP

**Requisitos/HU:** RF-010; HU-010, HU-011.
**Situación actual:** hay 7 filtros candidatos sin priorizar: tipo de comida, precio, ubicación, restaurante, plato, horario y categoría.

**Opciones:**
- **A (recomendada):** MVP con **tipo de comida, rango de precio y zona**; horario, categoría y filtros por restaurante/plato pasan a P1/P2.
- **B:** MVP con 5 filtros (los 3 anteriores + horario + categoría).
- **C:** todos los filtros candidatos en el MVP.

**Impacto si se aprueba:** RF-010, HU-010/011, criterios de aceptación, parámetros del contrato API, estimación.
**Decisión final (Acta):** ✅ A + mejora — filtros MVP: tipo de comida, rango de precio y radio de cercanía; "abierto ahora" se aplica como regla de disponibilidad del motor, no como filtro. El resto pasa a P1/P2.

---

# B. Datos e IA

## V-05 — Eventos exactos de interacción

**Requisitos/HU:** RF-012, RF-014, RIA-002; HU-013, HU-015.
**Situación actual:** pendiente definir los eventos exactos que se registrarán; RIA-002 solo enumera interacción, historial y preferencias.

**Opciones:**
- **A (recomendada):** 4 eventos en el MVP — `busqueda`, `ver_restaurante`, `ver_plato`, `clic_recomendacion` — con campos comunes: usuario (pseudonimizado), tipo, entidad, origen (búsqueda/recomendación), sesión y marca de tiempo. "Guardar favorito" a P1.
- **B:** 7 eventos (agrega favorito, compartir y tiempo de permanencia).
- **C:** registrar todos los eventos de UI (analítica completa) desde el inicio.

**Impacto si se aprueba:** RF-012/014, HU-013/015, modelo de datos (tabla de interacciones), RNF-007 (privacidad), insumos del recomendador.
**Decisión final (Acta):** ✅ A con ajuste obligatorio — 5 eventos: busqueda, ver_restaurante, ver_plato, recomendacion_impresion y clic_recomendacion; metadatos: posición, recomendacion_id, request_id, modelo_version, sesión y timestamp.

## V-06 — Preferencias: explícitas, inferidas o combinadas

**Requisitos/HU:** RF-013, RIA-001; HU-014, HU-016.
**Situación actual:** pendiente crítico — definir qué preferencias se solicitan directamente y cuáles se infieren del comportamiento.

**Opciones:**
- **A (recomendada):** **combinación** — preferencias explícitas en onboarding (tipo de comida, rango de precio, restricciones) más preferencias inferidas progresivamente desde las interacciones. Mientras no haya historial, domina lo explícito.
- **B:** solo explícitas.
- **C:** solo inferidas.

**Impacto si se aprueba:** RF-013, HU-014, modelo de datos, cold start (V-08), onboarding móvil.
**Decisión final (Acta):** ✅ A — preferencias explícitas en onboarding + inferidas progresivamente; lo explícito domina mientras no haya historial.

## V-07 — Algoritmo candidato y baseline

**Requisitos/HU:** RIA-003, RF-015; HU-016, HU-017, HU-026.
**Situación actual:** alternativas contempladas (colaborativo, contenido, híbrido); pendiente de decisión y de justificación experimental.

**Opciones:**
- **A (recomendada):** **baseline de popularidad + filtrado basado en contenido con pgvector**; evaluar el híbrido (contenido + colaborativo) cuando existan interacciones reales. La selección final se justifica con experimentos (RIA-004/RIA-006).
- **B:** filtrado colaborativo desde el MVP.
- **C:** solo filtrado basado en contenido.

**Impacto si se aprueba:** RIA-003, RF-015/016, HU-016/017/026, arquitectura del recomendador, dependencias de datos en la estimación.
**Decisión final (Acta):** ✅ A + precisión — baseline de popularidad → embeddings y pgvector para recuperación de candidatos → ranking en Python; híbrido condicionado a datos reales. pgvector es infraestructura de recuperación, no el algoritmo.

## V-08 — Estrategia de cold start

**Requisitos/HU:** RIA-005, RF-015; HU-025.
**Situación actual:** pendiente de diseño.

**Opciones:**
- **A (recomendada):** preferencias declaradas (V-06) + **popularidad por zona/rango de precio** como respaldo; transición gradual a personalización conforme se acumula historial.
- **B:** solo popularidad global mientras no haya historial.
- **C:** recomendaciones de contenido diversificado (sin señal de usuario).

**Impacto si se aprueba:** RIA-005, HU-025, RF-015, criterios de aceptación, relación con datos sintéticos (V-09).
**Decisión final (Acta):** ✅ A corregida — cold start: cercanía (si hay permiso de ubicación) + preferencias explícitas; sin ubicación o historial → popularidad (sin zona declarada).

## V-09 — Datos de entrenamiento y evaluación

**Requisitos/HU:** RIA-004; HU-026, HU-016.
**Situación actual:** pendientes las fuentes, la estructura, el volumen, la separación entrenamiento/validación/prueba y la relación entre datos sintéticos y reales.

**Opciones:**
- **A (recomendada):** dataset **sintético reproducible** (generador con semilla versionada) + captura progresiva de datos reales; split 70/15/15 por usuario; separación estricta y prevención de fuga de datos; volumen inicial a definir por el equipo.
- **B:** solo datos sintéticos durante el MVP.
- **C:** posponer el entrenamiento hasta contar con datos reales.

**Impacto si se aprueba:** RIA-004, HU-026 (T-026-01a/b, T-026-04a/b), pipeline de datos, métricas (V-10).
**Decisión final (Acta):** ✅ A con ajuste fuerte — datos sintéticos para pipeline y pruebas técnicas; datos reales para validación experimental. Split temporal (pasado → entrenamiento, siguiente → validación, reciente → prueba; alternativa leave-last-out) y estrategia para usuarios con pocas interacciones. Documentar limitaciones.

## V-10 — Métricas, baseline y umbrales de evaluación

**Requisitos/HU:** RIA-006, RF-023; HU-024.
**Situación actual:** pendientes críticos — métricas, baseline, protocolo experimental, datos de evaluación y criterios de comparación.

**Opciones:**
- **A (recomendada):** métricas offline **Precision@10, Recall@10, NDCG@10 y cobertura**; baseline = popularidad; métrica de producto = CTR sobre recomendaciones; comparación de modelos con el mismo split; **umbral inicial: mejora ≥ 10 % sobre el baseline** para promover un modelo (revisable).
- **B:** métricas offline sin umbrales (solo reporte).
- **C:** solo métricas de producto (CTR).

**Impacto si se aprueba:** RIA-006, RF-023, HU-024, criterios de aceptación de HU-016/024, protocolo experimental.
**Decisión final (Acta):** ✅ A con ajuste — métricas P@10, R@10, NDCG@10 y cobertura sobre restaurantes + CTR (clics/impresiones); baseline de popularidad; objetivo de mejora provisional hasta fijar el criterio final antes del experimento definitivo.

## V-11 — Integración del motor de recomendación

**Requisitos/HU:** RF-016, RIA-002; HU-017, HU-035.
**Situación actual:** pendiente definir si el motor Python se expone como API independiente y cómo se despliega.

**Opciones:**
- **A (recomendada):** motor Python como **módulo interno expuesto a través de FastAPI** en el MVP (un solo despliegue); separarlo como servicio cuando la carga o el equipo lo justifiquen. Coherente con la decisión de FastAPI como frontera de servicio.
- **B:** microservicio independiente desde el MVP.
- **C:** funciones serverless.

**Impacto si se aprueba:** RF-016, HU-017/035, contrato API (S0-08), arquitectura, ADR-004.
**Decisión final (Acta):** ✅ A — motor Python como módulo interno a través de FastAPI; separación como servicio solo si la carga lo justifica.

---

# C. No funcionales

## V-12 — Plataformas objetivo y versiones mínimas

**Requisitos/HU:** RNF-001, HU-027.
**Situación actual:** pendiente validar la tecnología definitiva y las versiones; RNF-001 solo exige diseño multiplataforma.

**Opciones:**
- **A (recomendada):** **Android e iOS** con Expo/EAS; mínimo **Android 10 (API 29)** e **iOS 15**; verificación de funciones críticas en ambas plataformas.
- **B:** Android primero; iOS en una fase posterior.
- **C:** definir versiones después del primer build.

**Impacto si se aprueba:** RNF-001, HU-027, pruebas P-027-01/02, criterios de entrada al Sprint 1.
**Decisión final (Acta):** ✅ A con ajuste — Expo SDK 57: Android 7+ e iOS 16.4+, documentado en ADR-001 (App Store exige Xcode 26+ desde abril de 2026).

## V-13 — Umbrales de rendimiento

**Requisitos/HU:** RNF-003, HU-029.
**Situación actual:** pendiente crítico — no hay objetivos cuantitativos de tiempo de respuesta ni condiciones de medición.

**Opciones:**
- **A (recomendada):** **p95 < 2 s** en consultas de catálogo/búsqueda y **p95 < 3 s** en generación de recomendaciones; medición en dispositivo de gama media y red 4G simulada; condiciones documentadas.
- **B:** umbrales más estrictos (p95 < 1 s / < 2 s).
- **C:** definir umbrales tras el primer prototipo.

**Impacto si se aprueba:** RNF-003, HU-029, pruebas P-029-*, diseño de consultas/índices.
**Decisión final (Acta):** ✅ A provisional — p95 < 2 s en catálogo/búsqueda y < 3 s en recomendaciones, en gama media y 4G simulada; consultas de cercanía con PostGIS e índice espacial.

## V-14 — Disponibilidad y manejo de fallos

**Requisitos/HU:** RNF-005, HU-031.
**Situación actual:** pendiente objetivo cuantitativo de disponibilidad y estrategia de manejo de fallos.

**Opciones:**
- **A (recomendada):** objetivo **99 % mensual** en el MVP (servicios gestionados de Supabase); manejo de fallos con reintentos en el cliente, respuestas degradadas (catálogo en caché) y monitoreo básico.
- **B:** 99.9 % desde el MVP (implica plan/instancia superior y mayor costo).
- **C:** sin objetivo numérico en el MVP.

**Impacto si se aprueba:** RNF-005, HU-031, arquitectura, operación y presupuesto.
**Decisión final (Acta):** ✅ A con definición — disponibilidad 99 % mensual como objetivo del MVP, con definición de "disponible" (API/BD/recomendador); el SLA contractual 99.9 % de Supabase solo aplica a Enterprise; evaluar Pro para el piloto (Free se pausa tras 7 días de inactividad).

## V-15 — Privacidad: datos, finalidad, consentimiento y retención

**Requisitos/HU:** RNF-007, RF-012/013/014; HU-033.
**Situación actual:** pendiente el análisis jurídico y de privacidad; los criterios exigen identificar datos, finalidad, consentimiento, acceso y conservación.

**Opciones:**
- **A (recomendada):** **minimización** (solo datos necesarios); consentimiento **explícito en onboarding** para interacciones/preferencias (rechazable); acceso por rol con RLS; retención definida (interacciones anonimizadas a los 12 meses o al eliminar la cuenta); opción de eliminar cuenta/datos; revisión jurídica documentada bajo la **Ley 1581 de 2012 (Colombia)**.
- **B:** aviso de uso con consentimiento implícito.
- **C:** postergar la política hasta producción.

**Impacto si se aprueba:** RNF-007, HU-033, RF-012/013/014, modelo de datos (anonimización), onboarding.
**Decisión final (Acta):** ✅ A con ajuste — retención justificada por la finalidad (12 meses es decisión del proyecto, no mandato legal); consentimiento previo, expreso e informado; anexo específico de ubicación; Ley 1581 de 2012.

## V-16 — Características de calidad ISO/IEC 25010

**Requisitos/HU:** RNF-010, HU-036.
**Situación actual:** pendiente seleccionar las características/subcaracterísticas que se evaluarán y definir indicadores medibles.

**Opciones:**
- **A (recomendada):** evaluar 4 características — **adecuación funcional, eficiencia de desempeño, usabilidad y seguridad** — con indicadores ligados a las pruebas del backlog (P-XXX-*).
- **B:** evaluar todas las características de la norma.
- **C:** usar solo el checklist de calidad del Sprint 0.

**Impacto si se aprueba:** RNF-010, HU-036, matriz de trazabilidad, criterios de aceptación.
**Decisión final (Acta):** ✅ A — se evalúan adecuación funcional, eficiencia de desempeño, usabilidad y seguridad, con indicadores ligados a las pruebas del backlog.

---

# D. Priorización y plan

## V-17 — Priorización MVP P0/P1/P2

**Requisitos/HU:** todas; S0-03.
**Situación actual:** existe la propuesta en `../Historias_Usuario_v1.1_Refinadas/Priorizacion_MVP_v1.0.md` (P0 = 21, P1 = 12, P2 = 3), pendiente de aprobación.

**Opciones:**
- **A (recomendada):** aprobar tal cual.
- **B:** aprobar con ajustes (p. ej. mover HU-024/HU-026 a P0 si se prioriza el rigor del recomendador, o HU-025 a P1 si se acepta un cold start básico).
- **C:** reclasificar por sprint (P0 = Sprints 1–3, P1 = Sprint 4, P2 = Sprint 5).

**Impacto si se aprueba:** backlog v1.1 (CSV/XLSX), `Priorizacion_MVP_v1.0.md`, plan de sprints, estimación.
**Decisión final (Acta):** ✅ A con ajuste — HU-024 y HU-026 pasan a P0 (núcleo del recomendador): P0 = 23, P1 = 10, P2 = 3. P0 define alcance, no orden de sprint.

## V-18 — HU-027: ¿Sprint 0 o Sprint 1?

**Requisitos/HU:** RNF-001, HU-027; HU-001, HU-002.
**Situación actual:** la estimación asigna HU-027 (multiplataforma) y sus 9 elementos (17 SP) al Sprint 0, pero el Sprint 0 está definido como preparación (requisitos, arquitectura, ADR, entorno), no implementación. Todas las HU del Sprint 1 dependen de HU-027.

**Opciones:**
- **A (recomendada):** mover HU-027 al **Sprint 1** como primera actividad y ajustar las dependencias de HU-001/HU-002 y la estimación. El Sprint 0 queda como planificación/entorno puro.
- **B:** mantener HU-027 en Sprint 0 y reconocerlo como preparación técnica (configuración del proyecto, no funcionalidad).
- **C:** dividir — configuración del proyecto (T-027-01/02) en Sprint 0 y verificación multiplataforma (T-027-03/04/05, P-027-*) en Sprint 1.

**Impacto si se aprueba:** estimación (CSV/XLSX/MD), dependencias de HU, plan de sprints, priorización P0.
**Decisión final (Acta):** ✅ A variante C — Sprint 0: repo/infraestructura (T-027-01/02, alineadas con S0-13/14, sin doble conteo); Sprint 1: funcionalidad y pruebas multiplataforma (T-027-03/04/05 y P-027-*).

## V-19 — Unidad de recomendación (nueva)

**Requisitos/HU:** RF-015, RF-017, RIA-001; HU-016, HU-018.
**Situación:** el requisito admitía recomendar restaurantes y/o platos sin fijar la unidad principal.

**Decisión final (Acta):** ✅ A — el **restaurante** es la unidad principal de recomendación; los platos son atributos/contenido y se muestran en el detalle del restaurante. Implica: embeddings de restaurante (platos como features), métricas sobre restaurantes, eventos y criterios de HU-016/HU-018.

## V-20 — Representación del usuario para recomendación (nueva)

**Requisitos/HU:** RIA-001, RIA-002; HU-014, HU-016, HU-025.
**Situación:** faltaba definir cómo se representa al usuario para el ranking basado en contenido.

**Decisión final (Acta):** ✅ A — representación simple: preferencias explícitas + historial ponderado de interacciones (centroide de embeddings de ítems) + restricciones de precio/ubicación. En cold start se usa solo lo explícito; la evolución a embeddings aprendidos se evalúa con datos reales. Se documenta en RIA-002 y en el diseño del recomendador (S0-09).

---

# Anexos (propuestas)

## Anexo A — Matriz rol → operación

| Operación | Usuario | Gestor | Administrador | Investigación (acceso separado) |
|---|:---:|:---:|:---:|:---:|
| Registrarse / iniciar sesión | ✔ | ✔ | ✔ | — |
| Consultar catálogo, buscar y filtrar | ✔ | ✔ | ✔ | — |
| Registrar interacciones y preferencias | ✔ | ✔ | ✔ | — |
| Ver recomendaciones | ✔ | ✔ | ✔ | — |
| Gestionar perfil | ✔ | ✔ | ✔ | — |
| CRUD de restaurantes/platos/horarios/ubicación | — | ✔ (su restaurante) | ✔ | — |
| Aprobar/suspender establecimientos | — | — | ✔ | — |
| Evaluar modelo y consultar métricas | — | — | ✔ | ✔ (datos anonimizados/agregados) |
| Ejecutar el motor y registrar resultados | — | — | — | — |

Notas: la investigación no es un rol operativo de la aplicación; accede a datos anonimizados/agregados y a un entorno de análisis. El motor de recomendación es un componente del sistema, no un rol de usuario.

## Anexo B — Taxonomía de eventos MVP

| Evento | Descripción | Entidad | Origen |
|---|---|---|---|
| `busqueda` | Consulta con término y filtros | Término/filtros | UI |
| `ver_restaurante` | Apertura del detalle de restaurante | Restaurante | UI |
| `ver_plato` | Apertura del detalle de plato | Plato | UI |
| `recomendacion_impresion` | Recomendación mostrada en pantalla | Recomendación (posición) | UI |
| `clic_recomendacion` | Clic sobre un ítem recomendado | Recomendación (posición) | UI |

Campos comunes: `usuario_id` (pseudonimizado), `tipo`, `entidad_id`, `origen`, `sesion_id`, `timestamp`. Para recomendaciones: `recomendacion_id`, `request_id`, `modelo_version` y `posicion` (permiten calcular CTR = clics / impresiones).

## Anexo C — Filtros MVP vs P1

| Filtro | MVP | P1/P2 |
|---|:---:|:---:|
| Tipo de comida | ✔ | |
| Rango de precio | ✔ | |
| Radio de cercanía (ubicación actual, con permiso) | ✔ | |
| Horario | | ✔ |
| Categoría | | ✔ |
| Restaurante específico | | ✔ |
| Plato específico | | ✔ |
| "Abierto ahora" | regla de disponibilidad del motor (siempre activa) | |

## Anexo D — Métricas y umbrales

| Ámbito | Métrica | Objetivo |
|---|---|---|
| Recomendador (offline) | Precision@10, Recall@10, NDCG@10, cobertura (unidad: restaurantes) | Comparar contra baseline de popularidad; objetivo de mejora provisional, criterio final se fija antes del experimento definitivo |
| Recomendador (producto) | CTR = clics / impresiones | Requiere el evento `recomendacion_impresion`; comparar por iteración |
| Rendimiento | p95 consultas / p95 recomendaciones | < 2 s / < 3 s en gama media y 4G simulada |
| Disponibilidad | Disponibilidad mensual (definida por componente) | 99 % objetivo del MVP (SLA contractual solo Enterprise) |

## Anexo E — Plataformas objetivo

| Plataforma | Versión mínima | Build |
|---|---|---|
| Android | 7+ | Expo SDK 57 / EAS |
| iOS | 16.4+ | Expo SDK 57 / EAS |

Nota: los mínimos derivan del Expo SDK elegido (ADR-001). Con SDK 57: Android 7+ e iOS 16.4+. Desde abril de 2026, App Store exige compilar con Xcode 26+.

## Anexo F — Privacidad

| Aspecto | Propuesta |
|---|---|
| Datos recopilados | Cuenta (identificador y credenciales gestionadas por Supabase Auth), perfil declarado (sin zona), preferencias, interacciones e historial. |
| Ubicación | Permiso explícito del dispositivo; uso en la consulta para cercanía (radio con expansión); no se persisten coordenadas exactas; a lo sumo zona aproximada con consentimiento. |
| Finalidad | Personalización de recomendaciones y operación del servicio. |
| Consentimiento | Previo, expreso e informado para interacciones/preferencias y ubicación; rechazable sin bloquear la consulta básica. |
| Acceso | RLS por rol; la investigación accede solo a datos anonimizados/agregados. |
| Retención | Definida según finalidad y obligaciones aplicables. Propuesta del proyecto: anonimizar interacciones a los 12 meses y eliminar al borrar la cuenta (decisión del proyecto, no mandato legal). |
| Marco | Ley 1581 de 2012 (Colombia); registrar revisión jurídica. |

---

# Checklist de aprobación

| Ficha | Tema | Estado | Responsable | Fecha |
|---|---|---|---|---|
| V-01 | Actores, roles y permisos | ✅ Aprobada (con ajuste) | Equipo del semillero | 2026-09 |
| V-02 | Canal de gestión y aprobación | ✅ Aprobada (con ajuste) | Equipo del semillero | 2026-09 |
| V-03 | Campos del perfil y ubicación | ✅ Aprobada (con ajuste) | Equipo del semillero | 2026-09 |
| V-04 | Filtros del MVP | ✅ Aprobada (+ "abierto ahora") | Equipo del semillero | 2026-09 |
| V-05 | Eventos de interacción | ✅ Aprobada (ajuste obligatorio: impresiones) | Equipo del semillero | 2026-09 |
| V-06 | Preferencias explícitas/inferidas | ✅ Aprobada | Equipo del semillero | 2026-09 |
| V-07 | Algoritmo y baseline | ✅ Aprobada (+ precisión pgvector) | Equipo del semillero | 2026-09 |
| V-08 | Cold start | ✅ Aprobada (corregida: sin zona) | Equipo del semillero | 2026-09 |
| V-09 | Datos de entrenamiento | ✅ Aprobada (split temporal) | Equipo del semillero | 2026-09 |
| V-10 | Métricas y umbrales | ✅ Aprobada (objetivo provisional) | Equipo del semillero | 2026-09 |
| V-11 | Integración del motor | ✅ Aprobada | Equipo del semillero | 2026-09 |
| V-12 | Plataformas objetivo | ✅ Aprobada (Expo SDK 57) | Equipo del semillero | 2026-09 |
| V-13 | Rendimiento | ✅ Aprobada (provisional) | Equipo del semillero | 2026-09 |
| V-14 | Disponibilidad | ✅ Aprobada (con definición) | Equipo del semillero | 2026-09 |
| V-15 | Privacidad | ✅ Aprobada (con ajuste) | Equipo del semillero | 2026-09 |
| V-16 | Calidad ISO 25010 | ✅ Aprobada | Equipo del semillero | 2026-09 |
| V-17 | Priorización P0/P1/P2 | ✅ Aprobada (HU-024/026 a P0) | Equipo del semillero | 2026-09 |
| V-18 | HU-027: Sprint 0 vs 1 | ✅ Aprobada (variante C) | Equipo del semillero | 2026-09 |
| V-19 | Unidad de recomendación | ✅ Aprobada (restaurante primario) | Equipo del semillero | 2026-09 |
| V-20 | Representación del usuario | ✅ Aprobada | Equipo del semillero | 2026-09 |

## Después de la aprobación

1. ✅ Decisiones registradas en `Acta_Validacion_Sprint_0_v1.0.md`.
2. ✅ Cambios aplicados en:
   - `../Requisitos_v1.0/*.md` (pendientes resueltos, umbrales agregados, estado validado).
   - `../Historias_Usuario_v1.1_Refinadas/` (notas de decisión y backlog CSV/XLSX).
   - `../Historias_Usuario_v1.1_Refinadas/Priorizacion_MVP_v1.0.md` (aprobada, P0 = 23).
   - `../Matriz_Trazabilidad_v1.0/` (columna Estado → Validado).
   - `Sprint_0_Backlog_y_Checklist_v1.0.xlsx` (S0-01/02/03 completados).
   - `../Estimacion_y_Dependencias_v1.0/` (HU-027 en variante C).
3. ✅ Consistencia revalidada: SP, IDs, estados y trazabilidad.

## Pendientes de seguimiento (no bloquean S0-01/02/03)

1. Fijar el volumen inicial del dataset sintético (V-09).
2. Registrar la revisión jurídica de privacidad (V-15).
3. Definir la capacidad real del equipo y re-secuenciar sprints con P0 = 23 HU (V-17, V-18).
4. Validar el plan de Supabase para el piloto (pausa de proyectos Free, SLA) (V-14).
