# RF-003 — Gestión básica del perfil
**Tipo:** Requisito funcional  
**Prioridad:** Alta  
**Actor principal:** Usuario final

## Descripción
El sistema deberá permitir almacenar información básica del usuario necesaria para personalizar su experiencia.

## Criterios de aceptación
1. El sistema mantiene un perfil asociado a la cuenta.
2. Los campos definitivos del perfil se encuentran definidos antes de implementación.
3. La información del perfil puede consultarse y actualizarse según los permisos establecidos.

## Dependencias
- RF-001.
- RF-002.
- Diseño de datos.

## Decisión (Sprint 0)
Campos del perfil: nombre/apodo, rango de precio preferido, preferencias alimentarias (categorías) y restricciones (opcional). No se incluye zona habitual. El recomendador usa rango de precio y preferencias; la ubicación actual (permiso del dispositivo, radio con expansión) se usa en la consulta y no se persiste exacta (V-03).

## Estado
Validado en Sprint 0 (septiembre de 2026). Ver `../Sprint_0_Plan_v1.0/Acta_Validacion_Sprint_0_v1.0.md`.
