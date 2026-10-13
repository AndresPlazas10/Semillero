# ADR-003 — FastAPI + Python como frontera de servicio

**Estado:** Aceptada
**Fecha:** septiembre de 2026
**Decisores:** Equipo del semillero
**Relacionados:** RF-016, RNF-009, V-11, HU-017, HU-035

## Contexto

El cliente móvil necesita consumir catálogo, gestionar datos, registrar interacciones y obtener recomendaciones. El anteproyecto ya usaba Python para el motor de recomendaciones. Existe el riesgo de acoplar el móvil directamente a Supabase y de repartir lógica de negocio entre capas (riesgo R4 del plan de Sprint 0).

## Decisión

Usar **FastAPI (Python)** como **frontera única de servicio**:

- Toda la lógica de negocio y de recomendación pasa por la API.
- El cliente móvil solo usa Supabase Auth para la sesión y JWT; **no** ejecuta lógica contra tablas de negocio.
- El **motor de recomendación es un módulo Python interno** invocado por FastAPI en el MVP (un solo despliegue).
- La separación del motor como microservicio se evaluará si la carga o el equipo lo justifican.
- El contrato de API se especifica en S0-08.

## Alternativas consideradas

| Alternativa | Motivo de descarte |
|---|---|
| Móvil accediendo directo a Supabase | Dispersa la lógica, dificulta validaciones y auditoría; acopla el cliente al esquema |
| Microservicios desde el MVP | Sobrearquitectura para un equipo pequeño y una etapa temprana |
| Serverless functions para cada operación | Fragmenta la lógica y complica el motor Python y el manejo de estado |
| Backend en Node/TypeScript | Viable, pero obligaría a reimplementar el ecosistema de datos/ML en otro lenguaje |

## Consecuencias

- ✅ Un único punto para validar JWT, permisos y reglas de negocio.
- ✅ Facilita pruebas de integración (Postman + pytest) y contrato estable para el móvil.
- ✅ El motor Python evoluciona sin exponer endpoints propios.
- ⚠️ FastAPI es un punto único de fallo en el MVP; requiere monitoreo y reintentos en el cliente.
- ⚠️ Hay que definir el despliegue (S0-14) y el manejo de secretos.
- ⚠️ La latencia del backend suma a los umbrales de RNF-003.

## Referencias

- `../../Sprint_0_Plan_v1.0/Acta_Validacion_Sprint_0_v1.0.md` (V-11)
- `../Arquitectura_v1.0.md`
- Plan de Sprint 0: frontera de servicio y riesgo R4
