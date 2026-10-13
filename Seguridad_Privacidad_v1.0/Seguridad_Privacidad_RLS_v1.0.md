# Seguridad, Privacidad y RLS — v1.0 (S0-12)

**Proyecto:** Aplicación móvil con IA para recomendación gastronómica en Tunja
**Fecha:** septiembre de 2026
**Estado:** Diseño inicial — sujeto a pruebas de seguridad y revisión jurídica

## 1. Propósito

Definir cómo se protege el sistema y los datos personales: autenticación, autorización por roles, políticas RLS, tratamiento de datos (incluida la ubicación), manejo de secretos, protección de la API y respuesta ante incidentes. Cubre RNF-006 y RNF-007, y las decisiones V-01, V-02, V-03, V-14 y V-15.

## 2. Activos y modelo de amenazas (resumen)

| Activo | Amenaza principal | Mitigación |
|---|---|---|
| Credenciales y sesiones | Robo de token, fuerza bruta | Supabase Auth, tokens de corta vida, HTTPS, almacenamiento seguro en el dispositivo, rate limiting |
| Datos personales (perfil, preferencias, eventos) | Acceso indebido, fuga | RLS, minimización, anonimización, cifrado en tránsito/reposo |
| Ubicación | Inferencia de domicilio, persistencia indebida | No persistir coordenadas exactas, consentimiento explícito, uso solo en consulta |
| Catálogo (restaurantes/platos) | Alteración no autorizada | Roles gestor/admin con RLS, auditoría de cambios |
| Motor de recomendación | Manipulación de resultados, fuga por features | Backend como única vía, sin datos personales en embeddings, versionado |
| Secretos (service key, API keys) | Exposición en cliente o repositorio | Solo backend, variables de entorno, rotación, `.gitignore` |
| API | Abuso, inyección, scraping | Validación de entrada, rate limiting, paginación, límites de lote |

## 3. Autenticación

- **Supabase Auth** gestiona registro, inicio/cierre de sesión y emisión de JWT (ADR-002).
- El backend **valida el JWT** en cada petición; el cliente nunca usa la clave de servicio.
- Tokens de acceso de corta duración con refresh token; cierre de sesión invalida el refresh.
- Contraseñas: gestionadas por Supabase (hash seguro); no se almacenan en el dominio.
- **MFA opcional para administradores** (recomendado; evaluar en S0-14).
- Recuperación de contraseña por correo gestionado por Supabase Auth.
- Errores de autenticación genéricos (no revelar si el correo existe).

## 4. Autorización y roles

### 4.1 Roles

| Actor | Cómo se determina | Alcance |
|---|---|---|
| Usuario | Autenticado sin rol especial | Su perfil, preferencias, eventos, recomendaciones |
| Gestor | Presencia en `restaurante_gestores` | Solo sus restaurantes, platos, horarios, ubicación |
| Administrador | `perfiles.rol_global = 'administrador'` | Todo el catálogo, aprobación, suspensión, auditoría |
| Investigación | Acceso separado (vistas anonimizadas y exportaciones) | Datos agregados/anonimizados; nunca datos personales crudos |
| Servicio (backend) | Rol de servicio acotado | Insertar eventos/recomendaciones y jobs; nunca expuesto al cliente |

### 4.2 Matriz de permisos (resumen)

| Recurso | Usuario | Gestor | Admin | Investigación |
|---|:---:|:---:|:---:|:---:|
| Perfil/preferencias propias | CRUD | CRUD | CRUD | — |
| Restaurantes activos (lectura) | R | R | R | R (anonimizado) |
| Restaurantes propios | — | CRUD | CRUD | — |
| Aprobar/suspender | — | — | ✅ | — |
| Platos/horarios/ubicación | R | CRUD (suyos) | CRUD | — |
| Eventos propios | Insert | Insert | R | R (agregado) |
| Recomendaciones propias | R | R | R | R (agregado) |
| Auditoría | — | — | R | — |
| Métricas/exportaciones | — | — | R | R |

## 5. Row Level Security (RLS)

- RLS **habilitado** en todas las tablas con datos de usuario o gestión.
- Funciones auxiliares (helper) para evitar subconsultas repetidas:

```sql
create or replace function es_admin() returns boolean
language sql stable security definer as $$
  select exists (
    select 1 from perfiles p
    where p.id = auth.uid() and p.rol_global = 'administrador'
  );
$$;

create or replace function es_gestor_de(restaurante uuid) returns boolean
language sql stable security definer as $$
  select exists (
    select 1 from restaurante_gestores g
    where g.restaurante_id = restaurante and g.usuario_id = auth.uid()
  );
$$;
```

### 5.1 Políticas de ejemplo

```sql
-- Perfiles: cada usuario ve y edita el suyo; el admin todo
alter table perfiles enable row level security;
create policy perfiles_propios on perfiles
  for all using (id = auth.uid()) with check (id = auth.uid());
create policy perfiles_admin on perfiles
  for all using (es_admin());

-- Restaurantes: lectura de activos; gestor/admin sobre los suyos
alter table restaurantes enable row level security;
create policy restaurantes_lectura on restaurantes
  for select using (estado = 'activo' or es_admin() or es_gestor_de(id));
create policy restaurantes_gestion on restaurantes
  for all using (es_gestor_de(id) or es_admin())
  with check (es_gestor_de(id) or es_admin());

-- Platos y horarios heredan del restaurante
alter table platos enable row level security;
create policy platos_lectura on platos
  for select using (exists (
    select 1 from restaurantes r
    where r.id = restaurante_id and (r.estado = 'activo' or es_gestor_de(r.id) or es_admin())
  ));
create policy platos_gestion on platos
  for all using (es_gestor_de(restaurante_id) or es_admin())
  with check (es_gestor_de(restaurante_id) or es_admin());

-- Eventos: inserción del propio usuario; lectura solo admin
alter table eventos enable row level security;
create policy eventos_insert on eventos
  for insert with check (usuario_id = auth.uid());
create policy eventos_admin on eventos
  for select using (es_admin());

-- Recomendaciones: lectura del dueño; escritura del backend (servicio)
alter table recomendaciones enable row level security;
create policy recomendaciones_propias on recomendaciones
  for select using (usuario_id = auth.uid() or es_admin());
```

- Las **escrituras de sistema** (eventos de recomendación, items, embeddings) usan el rol de servicio desde el backend con validaciones propias.
- Las **vistas de investigación** (`v_interacciones_anonimas`, `v_metricas_*`) se exponen solo al rol de investigación/admin.
- Cada política tiene pruebas de permisos por rol (P-002-*, P-032-*, P-033-*).

## 6. Privacidad y tratamiento de datos (Ley 1581 de 2012)

### 6.1 Principios aplicados
- **Finalidad:** personalización de recomendaciones y operación del servicio.
- **Libertad y consentimiento previo, expreso e informado.**
- **Minimización:** solo los datos necesarios; sin datos demográficos ni presupuesto exacto.
- **Seguridad:** RLS, cifrado y acceso mínimo.
- **Transparencia:** política de privacidad y aviso de tratamiento en el onboarding.

### 6.2 Datos tratados

| Dato | Finalidad | Base | Retención |
|---|---|---|---|
| Cuenta (correo/credencial) | Autenticación | Contrato | Mientras exista la cuenta |
| Perfil (nombre, rango de precio, restricciones) | Personalización | Consentimiento | Mientras exista la cuenta |
| Preferencias explícitas | Personalización | Consentimiento | Mientras exista la cuenta |
| Preferencias inferidas | Personalización | Consentimiento | Anonimizadas a los 12 meses |
| Interacciones/eventos | Recomendación y evaluación | Consentimiento | Anonimizadas a los 12 meses |
| Ubicación aproximada | Cercanía en la consulta | Consentimiento | **No se persiste** (solo `uso_ubicacion`/`radio_km`) |

### 6.3 Consentimiento
- Se registra por tipo (`interacciones`, `preferencias`, `ubicacion`), versión y fecha (`consentimientos`).
- Rechazar el consentimiento no bloquea la consulta básica del catálogo.
- La ubicación se solicita en el momento de uso (permiso del sistema operativo) y es revocable.
- El retiro del consentimiento detiene la captura y programa la anonimización.

### 6.4 Derechos del titular
- Acceso, rectificación, cancelación y revocatoria: implementados en `GET/PATCH/DELETE /v1/usuarios/me`.
- La eliminación de cuenta borra perfil, preferencias, consentimientos y recomendaciones personales; los eventos se anonimizan.
- Canal de atención para solicitudes (correo del proyecto).

### 6.5 Retención
- 12 meses para eventos y preferencias inferidas **es una decisión del proyecto**, justificada por la finalidad; no se presenta como mandato legal.
- Proceso programado de anonimización (S0-14) y registro de su ejecución.
- **Revisión jurídica pendiente** antes del lanzamiento.

## 7. Secretos y configuración

- Variables de entorno en backend y EAS: `SUPABASE_URL`, `SUPABASE_ANON_KEY`, `SUPABASE_SERVICE_ROLE_KEY` (solo backend), `DATABASE_URL`, claves del modelo de embeddings.
- **Nunca** se incluyen secretos en el repositorio ni en la app móvil; `.env` en `.gitignore` + `.env.example` documentado.
- Rotación de claves ante sospecha de exposición; claves distintas por entorno (dev/prod).
- La `service_role` solo se usa en el servidor y en jobs controlados.

## 8. Protección de la API

- Validación de esquemas (Pydantic) y límites de tamaño/lote (máx. 50 eventos por petición).
- Rate limiting por usuario/IP (429 + `Retry-After`); límites a definir en S0-14.
- Paginación obligatoria en listados (máx. 100).
- Errores sin datos internos (no exponer SQL ni stack traces).
- Logs sin PII: se registran IDs y códigos, no coordenadas ni datos personales.
- Idempotencia de eventos para evitar duplicados.

## 9. Seguridad de la app móvil

- Tokens en almacenamiento seguro (Expo SecureStore/Keychain/Keystore), nunca en texto plano.
- Permiso de ubicación solicitado en contexto y solo mientras se usa la app.
- Sin logs de datos sensibles en producción.
- Actualizaciones OTA firmadas por EAS.
- HTTPS obligatorio; certificate pinning evaluado como mejora futura.

## 10. Monitoreo, auditoría y respuesta

- `auditoria_cambios` registra aprobaciones, suspensiones y ediciones de catálogo con actor y fecha.
- Alertas básicas: errores 5xx, picos de 401/403, jobs fallidos, embeddings desactualizados.
- Plan de incidentes mínimo: detectar → contener (rotar claves, revocar sesiones) → evaluar alcance → notificar si aplica → documentar.
- Backups gestionados por Supabase; restaurar en entorno de prueba al menos una vez antes del piloto.

## 11. Trazabilidad

| Requisito/decisión | Dónde se cumple |
|---|---|
| RNF-006 (seguridad) | Secciones 3, 4, 5, 7, 8 |
| RNF-007 (privacidad) | Sección 6 |
| V-01/V-02 (roles y gestión) | Secciones 4 y 5 |
| V-03/V-15 (ubicación y consentimiento) | Sección 6.2–6.4 |
| V-14 (disponibilidad/plan) | Sección 10 y pendientes |
| HU-032/HU-033 | Matriz de permisos y tratamiento de datos |

## 12. Pendientes

1. Registrar la revisión jurídica de la Ley 1581 (Acta, pendiente #2).
2. Redactar la política de privacidad y el aviso de tratamiento para el onboarding.
3. Definir límites de tasa concretos y monitoreo (S0-14).
4. Implementar y probar las políticas RLS con tests por rol.
5. Evaluar MFA para administradores.
6. Probar la restauración de backups antes del piloto.
