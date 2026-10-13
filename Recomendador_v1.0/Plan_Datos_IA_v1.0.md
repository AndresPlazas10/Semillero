# Plan de Datos de IA — v1.0 (S0-10)

**Proyecto:** Aplicación móvil con IA para recomendación gastronómica en Tunja
**Fecha:** septiembre de 2026
**Estado:** Diseño inicial — volumen inicial pendiente de fijar

## 1. Propósito

Definir qué datos alimentan al recomendador, cómo se generan y versionan los datos sintéticos, cómo se separan de los reales, y cómo se evita la fuga de información (V-09, RIA-004).

## 2. Separación estricta: sintético vs real

| Tipo de dato | Para qué sirve | Para qué NO sirve |
|---|---|---|
| **Sintético** | Desarrollar el pipeline, probar código, validar el flujo completo, pruebas controladas de algoritmos | No es evidencia de comportamiento real de usuarios de Tunja |
| **Real** | Validación experimental del proyecto (calidad de recomendaciones) | No se usa para desarrollo con datos personales sin anonimizar |

**Regla:** los resultados con datos sintéticos se reportan como "experimento técnico". Los resultados con datos reales se reportan como "validación experimental". Nunca se presentan como equivalentes.

## 3. Datos reales

### 3.1 Qué se captura
- Los 5 eventos (`busqueda`, `ver_restaurante`, `ver_plato`, `recomendacion_impresion`, `clic_recomendacion`) con sus metadatos (V-05).
- Preferencias explícitas e inferidas (V-06).
- Estado del catálogo (restaurantes activos, platos, horarios, precios).
- Atribución de recomendaciones (`recomendacion_id`, `request_id`, `modelo_version`, `posicion`).

### 3.2 Reglas
- Solo con consentimiento vigente (V-15).
- Usuario pseudonimizado; sin coordenadas exactas (solo `uso_ubicacion` y `radio_km`).
- Retención: anonimización a los 12 meses; borrado al eliminar cuenta.
- **Captura progresiva:** cuanto antes empiece el piloto, antes habrá datos reales para validar.

### 3.3 Volumen esperado
- Sin definir hasta el piloto. Referencia: para métricas estables por usuario se necesitan al menos ~10 interacciones por usuario activo en el período de prueba.

## 4. Dataset sintético (reproducible)

### 4.1 Generador
- Script en Python con **semilla versionada** (p. ej. `seed=2026`), dependencias fijadas y salida con hash.
- Versión del dataset en el nombre (`synthetic_v1_seed2026`).

### 4.2 Entidades a generar

| Entidad | Volumen inicial propuesto | Notas |
|---|---:|---|
| Categorías | 10-15 | Comida típica, italiana, rápida, asiática, saludable, postres… |
| Restaurantes | 80-150 | Distribución por zona de Tunja; 2-5% cerrados/suspendidos |
| Platos | 10-20 por restaurante | Precios dentro del rango del restaurante |
| Horarios | 1-2 intervalos/día | Con días cerrados realistas |
| Usuarios | 200-500 | Con y sin preferencias (para medir cold start) |
| Interacciones | ~20.000 | Long tail + sesgo de popularidad + estacionalidad horaria |

**Pendiente:** el equipo debe aprobar o ajustar estos volúmenes (V-09).

### 4.3 Comportamiento simulado
- Preferencias declaradas por el 60% de usuarios.
- Sesgo de popularidad (pocos restaurantes concentran muchas interacciones).
- Estacionalidad: más actividad al mediodía y en la noche; más viernes-sábado.
- Tasa de clic sintética (~15-25% sobre impresiones) coherente con el CTR esperado.
- Usuarios "fríos" reservados exclusivamente para test.

### 4.4 Salida
- Archivos Parquet/CSV versionados: `usuarios`, `restaurantes`, `platos`, `horarios`, `eventos`, `preferencias`.
- Metadatos: semilla, versión, fecha, hash, autor.

## 5. Split temporal (V-09)

```
Pasado ────────────────▶ Entrenamiento
        Siguiente ──────▶ Validación
              Reciente ─▶ Prueba (out-of-time)
```

- **Nunca split aleatorio** para evaluación: se filtra información del futuro.
- Alternativa por usuario: **leave-last-out** (última interacción para test, penúltima para validación).
- Reglas:
  - Usuarios con < 3 interacciones: solo test de cold start; no entran a entrenamiento.
  - Restaurantes nuevos: reservar un porcentaje para medir cold start de ítems.
  - El test out-of-time se usa una sola vez para el reporte final.

## 6. Prevención de fuga de información (leakage)

1. Las features de un evento solo usan información **anterior** a ese evento.
2. Las agregaciones de popularidad se calculan con la ventana previa al corte, nunca con el período de test.
3. Los embeddings de contenido pueden precalcularse (no usan interacciones), pero su uso en evaluación respeta el corte temporal.
4. Deduplicación de eventos (mismo usuario/tipo/entidad/segundo).
5. Prohibido usar `clic_recomendacion` para construir la representación del usuario en la misma sesión antes de medir la impresión correspondiente.

## 7. Pipeline de datos

```
Generador sintético ─┐
Captura real ────────┼─▶ Limpieza y validación ─▶ Features (ventanas temporales)
                     │                                      │
                     └─▶ Versionado de dataset ◀────────────┘
                                      │
                                      ▼
                         Entrenamiento / evaluación
                                      │
                                      ▼
                         Registro de resultados (S0-11)
```

- Herramientas: Python (pandas/polars), Parquet, **MLflow** si el volumen de experimentos lo justifica (condicional).
- Reproducibilidad: semilla, versiones de dependencias y hash del dataset en cada experimento.

## 8. Trazabilidad

| Tarea | Artefacto |
|---|---|
| T-026-01a/b (dataset de entrenamiento) | Este plan + generador |
| T-026-02 (dataset de evaluación) | Sección de split temporal |
| T-026-03 (preparación de datos) | Pipeline (sección 7) |
| T-026-05 (separar y proteger evaluación) | Secciones 5 y 6 |
| P-026-01/03 (separación, sin fuga) | Pruebas automatizadas del generador y del split |

## 9. Riesgos y pendientes

| Riesgo | Mitigación |
|---|---|
| Presentar resultados sintéticos como reales | Regla de reporte (sección 2) y etiquetado |
| Volumen insuficiente | Ampliar generador; priorizar captura real del piloto |
| Sesgo del generador | Documentar supuestos y contrastar con datos reales |
| Datos personales en exportaciones | Anonimización y vistas de investigación (S0-06/S0-12) |

**Pendientes:** fijar volumen inicial (V-09), definir herramienta de versionado (DVC o nombres), confirmar uso de MLflow.
