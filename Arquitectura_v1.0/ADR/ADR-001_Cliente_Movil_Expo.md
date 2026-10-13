# ADR-001 — Cliente móvil con React Native + Expo + TypeScript

**Estado:** Aceptada
**Fecha:** septiembre de 2026
**Decisores:** Equipo del semillero
**Relacionados:** RNF-001, HU-027, V-12, V-18

## Contexto

El proyecto requiere una aplicación móvil multiplataforma (Android e iOS) para una startup universitaria, con un equipo pequeño y sin experiencia previa confirmada en desarrollo nativo doble. RNF-001 exige diseño multiplataforma y HU-027 cubre la verificación en ambas plataformas.

## Decisión

Usar **React Native + Expo + TypeScript** como cliente móvil, con **Expo SDK 57**, soportando **Android 7+** e **iOS 16.4+**, y builds/publicaciones mediante **Expo EAS**.

- Un solo código para Android e iOS.
- TypeScript para tipado y mantenibilidad.
- EAS para build, firma y actualizaciones OTA.
- La navegación y las librerías de UI se definirán durante la implementación (Sprint 1).

## Alternativas consideradas

| Alternativa | Motivo de descarte |
|---|---|
| Desarrollo nativo separado (Kotlin + Swift) | Duplica esfuerzo y equipo; inviable para el alcance del semillero |
| Flutter (Dart) | Viable, pero el anteproyecto y el plan ya fijaron React Native; cambiar agrega riesgo y curva de aprendizaje |
| PWA / web móvil | Menor acceso a permisos (ubicación en segundo plano, notificaciones) y peor experiencia; no cumple el objetivo de app móvil |

## Consecuencias

- ✅ Un solo desarrollo para dos plataformas; ecosistema Expo simplifica builds y despliegue.
- ✅ Actualizaciones OTA para cambios JS sin pasar por tienda.
- ⚠️ Dependencia del ciclo de Expo/React Native y de EAS.
- ⚠️ Mínimos de OS atados al SDK: con SDK 57, iOS 16.4+ y Android 7+.
- ⚠️ Desde abril de 2026, App Store exige compilar con Xcode 26+; el SDK 57 ya lo cumple.
- Verificación obligatoria en ambas plataformas (T-027-03/04/05 y P-027-*).

## Referencias

- `../../Sprint_0_Plan_v1.0/Acta_Validacion_Sprint_0_v1.0.md` (V-12, V-18)
- Plan de Sprint 0: stack tecnológico
- Documentación Expo: mínimos por SDK y requisitos de tienda
