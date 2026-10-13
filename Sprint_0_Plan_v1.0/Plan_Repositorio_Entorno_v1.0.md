# Plan de Repositorio y Entorno — v1.0 (S0-13/S0-14)

**Proyecto:** Aplicación móvil con IA para recomendación gastronómica en Tunja
**Fecha:** septiembre de 2026
**Estado:** Ejecutado — esqueleto del monorepo creado en `../recomendacion-gastronomica-tunja/`

## 1. Propósito

Definir y ejecutar la estructura del repositorio, las convenciones de trabajo, la integración continua y los entornos de desarrollo, de modo que el Sprint 1 pueda iniciar con una base de código ejecutable (infraestructura-first).

## 2. Estructura del monorepo

```
recomendacion-gastronomica-tunja/
├── README.md · CONTRIBUTING.md · .gitignore · .editorconfig · .env.example
├── .github/workflows/
│   ├── api.yml          # ruff + pytest + docker build
│   ├── mobile.yml       # typecheck + lint + expo-doctor
│   └── supabase.yml     # validación de migraciones (supabase db lint)
├── apps/mobile/         # React Native + Expo SDK 57 + TypeScript
│   ├── app/             # pantallas (Expo Router)
│   ├── src/lib/         # supabase, env, logger de eventos
│   └── package.json · app.json · tsconfig.json
├── services/api/        # FastAPI + Python 3.12
│   ├── app/{main.py, core, api/v1}
│   ├── tests/
│   ├── pyproject.toml · Dockerfile · .env.example
├── ml/                  # recomendador y experimentos
│   ├── recommender/
│   └── experiments/
├── supabase/            # config.toml · migrations/0001_init.sql · seed.sql
├── docs/                # índice hacia la documentación de planificación
└── scripts/             # dev.sh · check.sh
```

**Decisión:** monorepo (un solo repositorio) para simplificar coordinación, CI y trazabilidad en un equipo de 3 personas.

## 3. Convenciones de trabajo

| Tema | Regla |
|---|---|
| Ramas | `main` protegido; ramas cortas `feat/…`, `fix/…`, `docs/…`, `chore/…` |
| Commits | Conventional Commits (`feat:`, `fix:`, `docs:`, `test:`, `chore:`) |
| PR | 1 revisión mínima; squash merge; CI verde obligatorio |
| Versionado | SemVer para la API; versión en la ruta `/v1` |
| Estilo | TypeScript `strict`; Ruff (lint+format) en Python; Prettier/ESLint en móvil |
| Datos | `data/` y `.env` fuera del repositorio; nunca secretos ni datos personales |

## 4. Herramientas y versiones

| Capa | Herramienta | Versión |
|---|---|---|
| Móvil | Node.js | 22 LTS (mínimo de Expo SDK 57) |
| Móvil | Expo SDK / React Native / TypeScript | 57 / 0.86 / 5.x |
| API | Python | 3.12 |
| API | uv (gestor de dependencias) + FastAPI + Uvicorn | últimas estables |
| API | Ruff + pytest | últimas estables |
| Datos | Supabase CLI (migraciones), extensiones `postgis` y `vector` | últimas estables |
| CI/CD | GitHub Actions | — |
| Build móvil | Expo EAS | — |

## 5. Integración continua (CI)

| Workflow | Dispara en | Pasos |
|---|---|---|
| `api.yml` | cambios en `services/api/**` | instalar uv → `ruff check` → `pytest` → `docker build` |
| `mobile.yml` | cambios en `apps/mobile/**` | `npm ci` → `tsc --noEmit` → `eslint` → `expo-doctor` |
| `supabase.yml` | cambios en `supabase/**` | `supabase start` → `supabase db lint` (validación de migraciones) |

Nota: `supabase.yml` es más lento (levanta contenedores); puede marcarse como no bloqueante durante el primer sprint.

## 6. Entornos y secretos

- **Supabase dev** y **Supabase prod**: proyectos separados; mismas migraciones aplicadas en ambos.
- Variables (`services/api/.env`): `ENV`, `SUPABASE_URL`, `SUPABASE_ANON_KEY`, `SUPABASE_SERVICE_ROLE_KEY`, `DATABASE_URL`.
- Variables móviles (públicas): `EXPO_PUBLIC_SUPABASE_URL`, `EXPO_PUBLIC_SUPABASE_ANON_KEY`, `EXPO_PUBLIC_API_URL`.
- La `service_role` **solo** en backend; nunca en el cliente ni en el repositorio.
- `.env.example` documenta todas las variables.

## 7. Despliegue del backend (decisión en S0-14)

| Opción | Costo | Cold start | Notas |
|---|---|---|---|
| **Render Starter** | ~USD 7/mes | No (Starter no duerme) | Más simple; recomendado para el piloto |
| Fly.io | Bajo/medio | Bajo | Más control regional; configuración algo más técnica |
| Cloud Run | Por uso | Sí (escala a cero) | Flexible, cold starts afectan p95 |
| Supabase Edge Functions | Incluido | Sí | No apto para Python/FastAPI completo |

**Criterios de decisión:** costo, cold start (RNF-003), región (latencia desde Colombia), facilidad operativa y plan de Supabase (V-14). El esqueleto incluye `Dockerfile` para cualquiera de las opciones.

## 8. Checklist de verificación del entorno

- [ ] `services/api` responde `GET /health` en local.
- [ ] `pytest` pasa en `services/api`.
- [ ] `tsc --noEmit` y lint pasan en `apps/mobile`.
- [ ] La app Expo arranca en Android y iOS (dev build o Expo Go).
- [ ] Migración `0001_init.sql` aplicada en Supabase dev con `postgis` y `vector`.
- [ ] `seed.sql` carga categorías, rangos de precio y catálogo de ejemplo.
- [ ] RLS activo y probado con los roles usuario/gestor/administrador.
- [ ] CI verde en el primer PR.
- [ ] `.env.example` completo y sin secretos reales en el repositorio.

## 9. Trazabilidad

- Decisiones: `Acta_Validacion_Sprint_0_v1.0.md` (V-01, V-11, V-12, V-14).
- Arquitectura: `../Arquitectura_v1.0/`.
- Modelo de datos: `../Modelo_Datos_v1.0/Modelo_ER_v1.0.md` (base de `0001_init.sql`).
- Contrato API: `../API_v1.0/Contrato_API_v1.0.md`.
