# ADR-004 — Python + pgvector para la recomendación híbrida

**Estado:** Aceptada
**Fecha:** septiembre de 2026
**Decisores:** Equipo del semillero
**Relacionados:** RF-015, RF-023, RIA-001…RIA-006, RNF-003, RIA-003, V-07, V-08, V-09, V-10, V-19, V-20

## Contexto

Se necesita un sistema de recomendación para un arranque en frío (pocos usuarios e interacciones) que use preferencias, historial y ubicación. Existe el riesgo de **confundir embeddings con el recomendador completo** (riesgo R3 del plan de Sprint 0) y de elegir un algoritmo antes de validar datos.

## Decisión

Arquitectura de recomendación en dos etapas, con responsabilidades separadas:

```
Recuperación de candidatos        Ranking y evaluación
(PostgreSQL + reglas + pgvector)  (Python)
        ↓                               ↓
embeddings de restaurantes        preferencias + cercanía + precio
(platos como features)            + popularidad + señales de historial
        ↓                               ↓
candidatos similares        →     lista ordenada de restaurantes
```

Puntos concretos:

1. **pgvector es infraestructura de recuperación**, no el algoritmo (V-07).
2. **Unidad principal: restaurante**; los platos son atributos/contenido (V-19).
3. **Representación del usuario** (V-20): preferencias explícitas + historial ponderado (centroide de embeddings de ítems) + restricciones de precio/ubicación.
4. **Cold start** (V-08): cercanía (si hay permiso) + preferencias explícitas; sin ubicación o historial → popularidad.
5. **Datos** (V-09): sintético reproducible para pipeline y pruebas técnicas; real para validación; split temporal (pasado → entrenamiento, siguiente → validación, reciente → prueba; alternativa leave-last-out).
6. **Evaluación** (V-10): P@10, R@10, NDCG@10 y cobertura (sobre restaurantes) + CTR (clics/impresiones); baseline de popularidad; criterio de mejora provisional que se fija antes del experimento definitivo.
7. **Disponibilidad**: los restaurantes cerrados se excluyen o penalizan ("abierto ahora").
8. **Híbrido**: el componente colaborativo se incorpora cuando existan interacciones reales suficientes.

## Alternativas consideradas

| Alternativa | Motivo de descarte |
|---|---|
| Filtrado colaborativo desde el MVP | Sin datos suficientes; cold start inviable |
| Solo filtrado por contenido (sin vectores) | Menor capacidad de similitud semántica; pgvector ya está disponible |
| Tratar pgvector como "el algoritmo" | Confusión conceptual; no resuelve ranking, filtros ni evaluación |
| Modelo profundo desde el inicio | Sobredimensionado para los datos disponibles; difícil de justificar académicamente |

## Consecuencias

- ✅ Encaja con el arranque en frío y con la justificación experimental por etapas.
- ✅ Cada componente es medible: recuperación (cobertura) y ranking (P@10, R@10, NDCG@10).
- ⚠️ Requiere definir embeddings, dimensión y estrategia de actualización (S0-07).
- ⚠️ Requiere dataset y protocolo experimental documentados (S0-10/S0-11).
- ⚠️ El volumen inicial del dataset sintético sigue pendiente de fijar.

## Referencias

- `../../Sprint_0_Plan_v1.0/Acta_Validacion_Sprint_0_v1.0.md` (V-07, V-08, V-09, V-10, V-19, V-20)
- `../Arquitectura_v1.0.md`
- Requisitos `RIA-001` a `RIA-006` en `../../Requisitos_v1.0/`
