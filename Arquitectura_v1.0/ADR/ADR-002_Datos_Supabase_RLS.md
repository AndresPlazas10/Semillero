# ADR-002 — Datos y autenticación con PostgreSQL/Supabase, RLS, PostGIS y pgvector

**Estado:** Aceptada
**Fecha:** septiembre de 2026
**Decisores:** Equipo del semillero
**Relacionados:** RF-004, RF-009, RNF-004, RNF-006, RNF-007, V-01, V-02, V-03, V-14, V-15

## Contexto

El sistema necesita persistir usuarios, restaurantes, platos, horarios, ubicación, preferencias, interacciones y recomendaciones; aplicar permisos por rol; resolver consultas de cercanía; y almacenar embeddings para búsqueda por similitud. El equipo es pequeño y necesita servicios gestionados.

## Decisión

Usar **PostgreSQL gestionado en Supabase** como única base de datos, con:

- **Supabase Auth** para registro, sesión y emisión de JWT.
- **Row Level Security (RLS)** por rol: usuario, gestor y administrador (V-01).
- **PostGIS** para consultas de cercanía (radio con expansión; V-03, V-13).
- **pgvector** para embeddings y búsqueda por similitud (ADR-004).
- Estados del establecimiento (pendiente → activo → suspendido) y auditoría de cambios (V-02).
- Minimización de datos y **sin persistencia de coordenadas exactas** del usuario (V-03, V-15).
- Proyectos separados de Supabase para desarrollo y producción.

## Alternativas consideradas

| Alternativa | Motivo de descarte |
|---|---|
| Firebase/Firestore | Modelo NoSQL dificulta integridad relacional del catálogo; menor encaje con SQL y RLS |
| MySQL gestionado | Sin PostGIS/pgvector integrados; obligaría a servicios adicionales |
| PostgreSQL autoalojado | Más operación y riesgo para un equipo pequeño; sin SLA práctico |
| Sin RLS (permisos solo en backend) | Mayor superficie de error; RLS da defensa en profundidad |

## Consecuencias

- ✅ Servicios gestionados (Auth, backups, panel) y menos operación.
- ✅ RLS + JWT permiten defensa en profundidad.
- ✅ PostGIS evita cálculos de distancia en la aplicación.
- ⚠️ El plan Free **pausa proyectos tras 7 días de inactividad** y **no tiene SLA de uptime**; el SLA 99.9 % solo aplica a Enterprise. Evaluar **Pro** para el piloto (V-14).
- ⚠️ Las migraciones deben versionarse (Supabase CLI) y las políticas RLS probarse.
- ⚠️ La anonimización/retensión requiere procesos programados (V-15).

## Referencias

- `../../Sprint_0_Plan_v1.0/Acta_Validacion_Sprint_0_v1.0.md` (V-01, V-02, V-03, V-14, V-15)
- `../Arquitectura_v1.0.md`
- `ADR-004_Python_pgvector_Recomendacion.md`
