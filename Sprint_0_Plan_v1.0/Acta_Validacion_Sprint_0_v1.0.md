# Acta de Validación — Sprint 0 (v1.0)

**Proyecto:** Aplicación móvil con IA para recomendación gastronómica en Tunja
**Actividades:** S0-01 (validar requisitos), S0-02 (validar historias de usuario), S0-03 (priorizar MVP)
**Fecha:** septiembre de 2026
**Estado:** ✅ Aprobada
**Alcance:** esta acta cierra **S0-01, S0-02 y S0-03**. El Sprint 0 completo requiere las actividades **S0-04 a S0-15** y sus criterios DoD (ver `Sprint_0_Backlog_y_Checklist_v1.0.xlsx`).
**Fuentes:** `Propuestas_Validacion_Sprint_0_v1.0.md`, backlog HU v1.1, requisitos v1.0, plan de Sprint 0
**Participantes:** Equipo del semillero

## Propósito

Registrar las decisiones de validación del Sprint 0 sobre requisitos, historias de usuario, datos/IA, requisitos no funcionales y priorización del MVP, junto con sus impactos y pendientes de seguimiento.

## 1. Decisiones

| Ficha | Tema | Decisión |
|---|---|---|
| V-01 | Actores, roles y permisos | 3 roles de producto: Usuario, Gestor y Administrador. La investigación se separa como acceso a datos anonimizados/agregados y entorno de análisis; no es rol de la aplicación. Permisos con RLS en Supabase. |
| V-02 | Canal de gestión y aprobación | Gestión desde la app móvil con rol Gestor. Administrador con herramientas internas (Supabase Studio + pantalla mínima); panel web pospuesto. Estados del establecimiento: pendiente → activo → suspendido, con aprobación del Administrador. |
| V-03 | Perfil y ubicación | Perfil: nombre/apodo, rango de precio, preferencias alimentarias y restricciones (opcional); **sin zona habitual**. El motor usa la ubicación actual aproximada (GPS con permiso, radio con expansión) y las preferencias priorizan sobre los disponibles. Sin permiso → popularidad. No se persisten coordenadas exactas. |
| V-04 | Filtros del MVP | MVP: tipo de comida, rango de precio y radio de cercanía. "Abierto ahora" se aplica como regla de disponibilidad del motor, no como filtro. Horario, categoría, restaurante y plato pasan a P1/P2. |
| V-05 | Eventos de interacción | 5 eventos: `busqueda`, `ver_restaurante`, `ver_plato`, `recomendacion_impresion`, `clic_recomendacion`. Metadatos: usuario pseudonimizado, entidad, origen, sesión, timestamp; y para recomendaciones: `recomendacion_id`, `request_id`, `modelo_version`, `posicion`. |
| V-06 | Preferencias | Explícitas en onboarding + inferidas progresivamente desde interacciones; lo explícito domina mientras no haya historial. |
| V-07 | Algoritmo y baseline | Baseline de popularidad → embeddings y pgvector para recuperación de candidatos → ranking en Python; híbrido (contenido + colaborativo) se evalúa cuando existan datos reales. pgvector es infraestructura de recuperación, no el algoritmo. |
| V-08 | Cold start | Cercanía (si hay permiso de ubicación) + preferencias explícitas; sin ubicación o historial → popularidad. Aplica a usuario, restaurante y plato nuevos. |
| V-09 | Datos de entrenamiento | Datos sintéticos reproducibles para pipeline y pruebas técnicas; datos reales para validación experimental. Split temporal (pasado → entrenamiento, siguiente → validación, reciente → prueba; alternativa leave-last-out por usuario) y estrategia para usuarios con pocas interacciones. Documentar limitaciones. |
| V-10 | Métricas y umbrales | P@10, R@10, NDCG@10 y cobertura sobre restaurantes + CTR (clics/impresiones). Baseline de popularidad. Objetivo de mejora provisional; el criterio final se fija antes del experimento definitivo. |
| V-11 | Integración del motor | Motor Python como módulo interno a través de FastAPI (un solo despliegue); separación como servicio solo si la carga o el equipo lo justifican. |
| V-12 | Plataformas objetivo | React Native + Expo + TypeScript con Expo SDK 57: Android 7+ e iOS 16.4+ (ADR-001). App Store exige Xcode 26+ desde abril de 2026. |
| V-13 | Rendimiento | p95 < 2 s en catálogo/búsqueda y < 3 s en recomendaciones; medición en gama media y red 4G simulada. Consultas de cercanía con PostGIS e índice espacial. |
| V-14 | Disponibilidad | Objetivo del MVP: 99 % mensual con definición de "disponible" por componente (API/BD/recomendador). El SLA contractual de Supabase (99.9 %) solo aplica a Enterprise; Pro no incluye SLA de uptime y Free pausa proyectos tras 7 días de inactividad (evaluar Pro para el piloto). |
| V-15 | Privacidad | Minimización; consentimiento previo, expreso e informado (interacciones/preferencias y ubicación); RLS por rol; retención justificada por la finalidad (propuesta del proyecto: anonimizar interacciones a los 12 meses y eliminar al borrar la cuenta); ubicación aproximada solo en sesión; Ley 1581 de 2012. |
| V-16 | Calidad ISO 25010 | Evaluar adecuación funcional, eficiencia de desempeño, usabilidad y seguridad con indicadores ligados a las pruebas del backlog. |
| V-17 | Priorización MVP | P0 = 23, P1 = 10, P2 = 3. HU-024 y HU-026 pasan a P0 por ser núcleo del recomendador. P0 define alcance, no orden de sprint. |
| V-18 | HU-027 | Variante C: Sprint 0 ejecuta repo/infraestructura (T-027-01/02, alineadas con S0-13/14, sin doble conteo); Sprint 1 ejecuta funcionalidad y pruebas multiplataforma (T-027-03/04/05 y P-027-*). |
| V-19 | Unidad de recomendación | El restaurante es la unidad principal; los platos son atributos/contenido del restaurante. |
| V-20 | Representación del usuario | Preferencias explícitas + historial ponderado (centroide de embeddings de ítems) + restricciones de precio/ubicación. Cold start: solo lo explícito. Evolución a embeddings aprendidos con datos reales. |

## 2. Principio arquitectónico aprobado

```
UBICACIÓN APROXIMADA + PREFERENCIAS + HISTORIAL + INTERACCIONES
        ↓
GENERACIÓN DE CANDIDATOS (pgvector + reglas/SQL)
        ↓
RANKING EN PYTHON
        ↓
RECOMENDACIONES (restaurantes)
        ↓
FASTAPI / APP MÓVIL
```

## 3. Impactos aplicados a los artefactos

| Artefacto | Cambio |
|---|---|
| `Requisitos_v1.0/` (39 docs) | Pendientes resueltos con las decisiones; umbrales en RNF-003/005/007/010; estado → Validado en Sprint 0 |
| `Historias_Usuario_v1.1_Refinadas/` | Notas de decisión en HU afectadas; backlog CSV/XLSX con estado "Validada en Sprint 0" y MVP actualizado (HU-024/026 → P0) |
| `Priorizacion_MVP_v1.0.md` | Aprobada; P0 = 23, P1 = 10, P2 = 3; decisiones pendientes cerradas |
| `Matriz_Trazabilidad_v1.0/` | Columna Estado → "Validado (Sprint 0)" |
| `Estimacion_y_Dependencias_v1.0/` | HU-027 en variante C (T-027-01/02 Sprint 0; resto Sprint 1) |
| `Sprint_0_Backlog_y_Checklist_v1.0.xlsx` | S0-01, S0-02 y S0-03 completados |

## 4. Pendientes de seguimiento

| # | Pendiente | Responsable | Bloquea |
|---|---|---|---|
| 1 | Fijar el volumen inicial del dataset sintético (V-09) | Equipo | Entrenamiento (Sprint 3) |
| 2 | Registrar la revisión jurídica de privacidad (V-15) | Equipo / asesoría | Lanzamiento |
| 3 | Medir la capacidad real del equipo y re-secuenciar sprints con P0 = 23 HU (V-17, V-18) | Equipo | Sprint Planning |
| 4 | Validar el plan de Supabase para el piloto: pausa de proyectos Free, SLA (V-14) | Equipo | Despliegue |
| 5 | Documentar el contrato de API y el modelo de datos (S0-06, S0-08) | Equipo | Sprint 1 |

## 5. Criterios de cierre de la validación (S0-01/02/03)

- ✅ Los 39 requisitos tienen decisión registrada (sin secciones "Pendiente" abiertas).
- ✅ Actores y permisos definidos (3 roles + acceso de investigación).
- ✅ Eventos, filtros, algoritmo, cold start, métricas y unidad de recomendación documentados.
- ✅ RNF con umbrales medibles (rendimiento, disponibilidad, privacidad, calidad).
- ✅ Priorización aprobada y HU-027 resuelta.
- ✅ Estados sincronizados en backlog, matriz, checklist y estimación.

**Estado del Sprint 0:** **NO declarado DONE.** Al cierre de esta acta hay **2/10 criterios DoD completados** y **12/15 actividades pendientes** (S0-04 a S0-15). El **Sprint 1 no se aprueba** hasta medir la capacidad real del equipo (S0-15).

## Trazabilidad

- Propuestas y decisiones: `Propuestas_Validacion_Sprint_0_v1.0.md`
- Requisitos: `../Requisitos_v1.0/`
- Backlog refinado: `../Historias_Usuario_v1.1_Refinadas/`
- Priorización: `../Historias_Usuario_v1.1_Refinadas/Priorizacion_MVP_v1.0.md`
- Matriz: `../Matriz_Trazabilidad_v1.0/`
- Estimación: `../Estimacion_y_Dependencias_v1.0/`
