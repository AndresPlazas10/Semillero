# Registro de experimentos del recomendador

**Proyecto:** Aplicación móvil con IA para recomendación gastronómica en Tunja  
**Versión del registro:** 1.0  
**Estado inicial:** no hay corridas de entrenamiento/evaluación registradas en el código inspeccionado al 18 de octubre de 2026. El módulo ML se encuentra descrito como esqueleto; los resultados se añadirán cuando se ejecuten experimentos.

> Mantener una ficha por corrida y una ficha de comparación por conjunto experimental. No reemplazar resultados anteriores. Etiquetar claramente cada resultado como **sintético (experimento técnico)** o **real (validación experimental)**.

## 1. Protocolo de referencia del proyecto

- Unidad recomendada: restaurante.
- Baseline: popularidad, calculada únicamente con datos previos al corte de evaluación.
- Estrategias a comparar cuando estén disponibles: baseline de popularidad, contenido/pgvector e híbrido.
- Métricas offline: Precision@10, Recall@10, NDCG@10 y cobertura del catálogo de restaurantes activos.
- Métricas de producto con datos reales: CTR (clics/impresiones), CTR por posición/modelo y tiempo a primera interacción.
- Split temporal: entrenamiento con datos pasados, validación con el periodo siguiente y prueba out-of-time con el periodo más reciente. No usar split aleatorio para la evaluación principal.
- Evitar fuga de información: features y popularidad solo con datos anteriores al corte; reservar usuarios fríos para pruebas cold start; no usar eventos posteriores al momento evaluado.
- Protocolo de referencia: mínimo tres corridas por configuración cuando el generador o el muestreo permita variar semilla; reportar media y dispersión.
- Resultados sintéticos no se presentan como evidencia de comportamiento real.
- El umbral definitivo de mejora frente al baseline se fija antes del experimento final.

Referencia metodológica: `Recomendador_v1.0/Metricas_Evaluacion_v1.0.md` y `Recomendador_v1.0/Plan_Datos_IA_v1.0.md`.

## 2. Índice de experimentos

| ID | Fecha | Pregunta / objetivo | Dataset y tipo | Estrategia | Estado | Resultado principal | Enlace a informe/artefactos |
|---|---|---|---|---|---|---|---|
| — | — | No se han registrado corridas todavía | — | — | Pendiente | — | — |

## 3. Ficha para cada corrida

Copia esta ficha una vez por corrida. Para comparaciones, conserva un ID común de experimento y un ID de corrida diferente.

### Identificación

| Campo | Valor |
|---|---|
| ID de experimento | Pendiente |
| ID de corrida | Pendiente |
| Fecha y hora (UTC) | Pendiente |
| Responsable | Pendiente |
| Sprint / incremento relacionado | Pendiente |
| Estado | Planificado / En ejecución / Completado / Fallido |
| Commit o versión de código | Pendiente |

### Pregunta experimental

| Campo | Registro |
|---|---|
| Objetivo / pregunta | Pendiente |
| Hipótesis previa al experimento | Pendiente |
| Estrategia evaluada | Baseline / contenido / híbrido / otra (especificar) |
| Criterio de comparación predefinido | Pendiente |

### Dataset y protocolo

| Campo | Valor |
|---|---|
| Tipo de datos | Sintético / Real autorizado |
| Nombre, versión y hash del dataset | Pendiente |
| Semilla del generador o muestreo | Pendiente / No aplica |
| Número de usuarios, restaurantes, platos e interacciones | Pendiente |
| Periodos/cortes de entrenamiento, validación y prueba | Pendiente |
| Método de relevancia y ventana temporal | Pendiente |
| Segmentos reportados | Con historial / preferencias sin historial / cold start puro |
| Controles contra fuga de información | Pendiente |
| Consentimiento y tratamiento de datos reales verificados | Pendiente / No aplica |

### Modelo y configuración

| Campo | Valor |
|---|---|
| Algoritmo / estrategia | Pendiente |
| Modelo de embeddings, versión y dimensiones | Pendiente / No aplica |
| Recuperación vectorial y configuración pgvector | Pendiente / No aplica |
| Features y señales utilizadas | Pendiente |
| Pesos e hiperparámetros | Pendiente |
| Configuración de cold start | Pendiente |
| Dependencias y versiones | Pendiente |
| Hardware / entorno de ejecución | Pendiente |
| Ruta de artefactos, modelo y logs | Pendiente |
| Hashes de artefactos | Pendiente |

### Resultados observados de esta corrida

| Métrica | Resultado | Cantidad / denominador | Notas |
|---|---:|---:|---|
| Precision@10 | Pendiente | Pendiente | |
| Recall@10 | Pendiente | Pendiente | |
| NDCG@10 | Pendiente | Pendiente | |
| Cobertura de restaurantes | Pendiente | Pendiente | |
| CTR | Pendiente / No aplica | Impresiones y clics | Calcular solo con eventos reales válidos |
| CTR por posición/modelo | Pendiente / No aplica | Pendiente | |
| Tiempo p95 de recomendación | Pendiente | Número de solicitudes | |

**No completar métricas hasta ejecutar el cálculo y registrar el artefacto de resultados.** Si una métrica no puede calcularse, usar “No disponible” y explicar por qué.

### Conclusión de la corrida

- **Resultado principal:** Pendiente.
- **Comparación frente al baseline:** Pendiente.
- **Limitaciones, anomalías o errores:** Pendiente.
- **Decisión:** Pendiente; no avanzar / repetir / aceptar provisionalmente / descartar, con justificación.
- **Enlaces a notebook, logs, tablas, figuras o informe:** Pendiente.

## 4. Resumen comparativo de un experimento

Completar después de ejecutar las corridas comparables, usando el mismo dataset de prueba y protocolo.

| Estrategia | N corridas | P@10 media ± dispersión | R@10 media ± dispersión | NDCG@10 media ± dispersión | Cobertura media ± dispersión | CTR (si aplica) |
|---|---:|---:|---:|---:|---:|---:|
| Baseline popularidad | Pendiente | Pendiente | Pendiente | Pendiente | Pendiente | Pendiente |
| Contenido / pgvector | Pendiente | Pendiente | Pendiente | Pendiente | Pendiente | Pendiente |
| Híbrido | Pendiente | Pendiente | Pendiente | Pendiente | Pendiente | Pendiente |

**Conclusión comparativa:** Pendiente.  
**¿Se superó el baseline de forma consistente?:** Pendiente.  
**Decisión de siguiente etapa:** Pendiente.  
**Aprobación del criterio de mejora antes del experimento final:** Pendiente.
