# Capacidad y Sprint 1 — v1.0 (S0-15)

**Proyecto:** Aplicación móvil con IA para recomendación gastronómica en Tunja
**Fecha:** septiembre de 2026
**Estado:** Aprobado — secuencia provisional sujeta a velocidad real

## 1. Capacidad inicial

| Parámetro | Valor |
|---|---|
| Personas | 3 |
| Dedicación | 15 h/semana por persona |
| Duración del sprint | 2 semanas |
| Capacidad bruta | 3 × 15 × 2 = **90 h/sprint** |
| Factor de foco | 0.6 (propuesto) |
| **Capacidad efectiva** | **54 h/sprint** |

**Velocidad inicial: a determinar.** Se estima con **Planning Poker** al inicio del Sprint 1 y se **recalibra con la velocidad real observada** al cierre del Sprint 1. Las cifras en SP de la secuencia son **pesos de alcance provisionales**, no una capacidad comprometida.

## 2. Alcance del MVP (aprobado)

**MVP = 186 SP (12 HU):** HU-027, HU-001, HU-002, HU-004, HU-005, HU-006, HU-012, HU-013, HU-014, HU-016, HU-018, HU-025.

- Catalogo del MVP alimentado por **datos iniciales (seed)**; la gestión de catálogo (HU-019 a HU-023) pasa a Incremento 2.
- La recomendación del MVP es un **baseline por reglas** (preferencias, interacciones, ubicación, precio, disponibilidad) implementado en FastAPI/SQL.
- HU-016 depende de HU-013 y HU-014; HU-018 depende de HU-016 (no de HU-017).

**Incremento 2 = 364 SP:** motor Python + pgvector, entrenamiento/evaluación, gestión de catálogo, búsqueda/filtros avanzados, híbrido y calidad.

## 3. Secuencia provisional

| Sprint | Contenido | SP (peso) |
|---|---|---:|
| **S1** | Verificación multiplataforma (HU-027) + registro (HU-001); base del repo/CI/entorno | 34 |
| **S2** | Inicio de sesión (HU-002) + explorar restaurantes con seed (HU-004) | 34 |
| **S3** | Detalle de restaurante (HU-005) + platos (HU-006) | 28 |
| **S4** | Detalle de plato (HU-012) + captura de interacciones (HU-013) | 28 |
| **S5** | Preferencias (HU-014) + baseline de recomendación (HU-016) | 36 |
| **S6** | Presentación de recomendaciones (HU-018) + cold start (HU-025) | 26 |
| **Total MVP** | | **186** |

Candidata a entrar si la velocidad del S1 lo permite: **HU-007 (precios, 11 SP)**.

## 4. Regla transversal: captura de datos desde S1

No se espera al modelo del Incremento 2 para instrumentar:

- En S1 se habilitan la tabla de eventos, `POST /v1/eventos` y un logger de cliente.
- Cada pantalla implementada instrumenta sus eventos al momento de construirse (`busqueda`, `ver_restaurante`, `ver_plato`, `recomendacion_impresion`, `clic_recomendacion`).
- Se mantiene un **dataset mínimo** de desarrollo (sintético reproducible) y el **baseline de popularidad** desde S5.
- Así, al llegar al Incremento 2 ya existen datos reales para entrenar y evaluar.

## 5. Nota académica: el baseline es la línea base, no el resultado final

- El baseline del MVP es la **referencia experimental**, no el aporte de IA del proyecto.
- El núcleo de investigación es el **Incremento 2**: modelo Python + pgvector + experimentos + comparación baseline vs modelo + evaluación (RIA-001 a RIA-006).
- El reporte final debe separar resultados con datos sintéticos (técnicos) y datos reales (validación).

## 6. Criterios de entrada al Sprint 1 (Definition of Ready)

- [x] Repositorio y entorno operativos (S0-13/S0-14).
- [x] Migraciones iniciales y RLS aplicadas en Supabase dev.
- [x] Contrato API v1 definido (S0-08).
- [x] Modelo de datos y arquitectura v1 aprobados (S0-04/S0-06).
- [x] HUs del MVP priorizadas y dependencias corregidas.
- [ ] Velocidad inicial estimada con Planning Poker.
- [ ] Sin bloqueadores críticos sin propietario o mitigación.

## 7. Riesgos

| Riesgo | Mitigación |
|---|---|
| Velocidad real menor a la esperada | Recalibrar tras S1 y recortar candidatas (p. ej. HU-007) |
| Capacidad afectada por carga académica | Factor de foco revisado por sprint |
| Dependencia del seed para el catálogo | Seed versionado y validado en CI |
| Captura de datos insuficiente | Regla transversal de la sección 4 |

## 8. Trazabilidad

- Estimación y dependencias: `../Estimacion_y_Dependencias_v1.0/`.
- Recomendador y métricas: `../Recomendador_v1.0/`.
- Acta de validación: `Acta_Validacion_Sprint_0_v1.0.md`.
