# Sprint 0 — Aplicación móvil de recomendación gastronómica en Tunja

## Decisión técnica central
Python y pgvector no son alternativas:
- Python: ML, preparación de datos, ranking y evaluación.
- PostgreSQL/pgvector: almacenamiento de vectores y búsqueda por similitud.
- FastAPI: frontera de servicio.
- Supabase: PostgreSQL, Auth y servicios gestionados.
- React Native + Expo + TypeScript: cliente móvil.

El algoritmo concreto de recomendación queda sujeto a la validación de datos y experimentos.

## Referencias
- Propuestas de validación para S0-01, S0-02 y S0-03 (20 fichas de decisión y anexos): `Propuestas_Validacion_Sprint_0_v1.0.md`.
- Acta con las decisiones aprobadas: `Acta_Validacion_Sprint_0_v1.0.md`.
- Arquitectura y ADRs (S0-04/05): `../Arquitectura_v1.0/`.
- Modelo de datos y diseño vectorial (S0-06/07): `../Modelo_Datos_v1.0/`.
- Contrato API v1 (S0-08): `../API_v1.0/`.
- Recomendador, datos IA y métricas (S0-09/10/11): `../Recomendador_v1.0/`.
- Seguridad, privacidad y RLS (S0-12): `../Seguridad_Privacidad_v1.0/`.
- Repositorio y entorno (S0-13/14): `Plan_Repositorio_Entorno_v1.0.md` y `../recomendacion-gastronomica-tunja/`.
- Capacidad y Sprint 1 (S0-15): `Capacidad_y_Sprint_1_v1.0.md`.
- Priorización P0/P1/P2 aprobada (P0 = 23): `../Historias_Usuario_v1.1_Refinadas/Priorizacion_MVP_v1.0.md`.
- HU-027 resuelta (V-18, variante C): configuración del proyecto en Sprint 0 (T-027-01/02, alineadas con S0-13/14) y verificación multiplataforma en Sprint 1 (T-027-03/04/05 y P-027-*).

## Estado actual (septiembre de 2026)

**✅ Sprint 0 DONE.** 15/15 actividades y 10/10 criterios DoD completados.

| Estado | Actividades |
|---|---|
| ✅ Completadas | S0-01 a S0-15 (validación, arquitectura, datos, API, recomendador, seguridad, repositorio, entorno y capacidad) |

- Base de código ejecutable: `../recomendacion-gastronomica-tunja/` (monorepo con CI, migraciones, FastAPI y shell Expo).
- Capacidad inicial: **54 h efectivas por sprint**; velocidad a calibrar después del Sprint 1.
- **Sprint 1 autorizado:** infraestructura + HU-027 + HU-001, con la secuencia documentada en `Capacidad_y_Sprint_1_v1.0.md`.
