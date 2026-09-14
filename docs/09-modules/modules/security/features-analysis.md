<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Análisis y estandarización de Features — security

**Estado:** Alineado (DB + API)
**Fecha:** 2026-09-14

</div>

</div>

> Última actualización: 2026-09-14
> Fuente: somnguard-db seeds (011_insert_security_features.sql, 012_insert_security_role_feature.sql) + docs/04-requirements/ + HU-DB-003 (provision/claim)

---

## Resumen ejecutivo

| Métrica | Valor |
|---------|-------|
| Roles en BD | 2 (`admin`, `user`) |
| Módulos en BD | 6 |
| Features previas en BD | 19 (17 base + `device.provision` + `device.claim` HU-DB-003) |
| Features estandarizadas | 25 (23 propuesta + 2 provisioning preservadas) |
| **Renombradas (UPDATE, preserva FK)** | 4 |
| **Creadas nuevas** | 6 |
| **Preservadas HU-DB-003** | 2 (`device.provision`, `device.claim`) |

---

## Features previas en BD (19: 17 base + 2 provisioning HU-DB-003)

| Módulo | Feature | Descripción | Problema detectado |
|--------|---------|-------------|-------------------|
| security | `user.read` | Ver lista y detalles de usuarios | OK (admin); user debería ser solo own |
| security | `user.write` | Crear, actualizar, eliminar usuarios | Ambigüedad: ¿incluye soft-delete? → split en `user.write` + `user.own_write` + `user.delete` |
| security | `role.read` | Ver roles y permisos | OK |
| security | `role.write` | Gestionar roles y asignaciones | Mezcla CRUD roles + asignar a usuarios → split en `role.write` + `role.assign` |
| security | `audit.read` | Ver logs de auditoría y login | OK |
| device_management | `device.read` | Ver dispositivos y asignaciones | OK (admin); user solo own vía `device.assign` |
| device_management | `device.write` | Registrar, asignar, configurar dispositivos | Muy amplio: alta + asignación + config → `device.write` (admin CRUD) + `device.assign` |
| device_management | `device.config` | Gestionar configuración remota | No es acción CRUD estándar → renombrar a `device.config_write` + crear `device.config_read` |
| device_management | `device.provision` | Crear tokens de aprovisionamiento (solo admin) | OK — HU-DB-003, se preserva |
| device_management | `device.claim` | Reclamar dispositivo con claim_code (admin+user) | OK — HU-DB-003, se preserva y se otorga a `user` |
| telemetry | `event.read` | Consultar eventos y evidencias | OK |
| telemetry | `event.write` | Ingestar eventos (device) | Es ingesta device, no acción usuario → renombrar a `event.ingest` (solo API key, sin roles) |
| telemetry | `alert.read` | Ver histórico de alarmas | OK |
| monitoring | `notification.read` | Ver notificaciones propias | OK |
| monitoring | `notification.write` | Enviar notificaciones (sistema) | Es acción sistema/admin → renombrar a `notification.send` |
| analytics | `analytics.read` | Ver reportes y métricas | OK |
| analytics | `analytics.report` | Crear y exportar reportes | Renombrar a `analytics.generate` (acción clara) |
| parameterization | `catalog.read` | Ver catálogos de parametrización | OK |
| parameterization | `catalog.write` | Gestionar catálogos (admin) | OK |

> Nota: el módulo en BD es `telemetry` (no `telemetry_service`). Este doc usa `telemetry` en todo el SQL.

---

## Análisis por rol según funcionalidad real del sistema

### Rol `admin` — Acceso total al sistema
**Necesita:** CRUD completo en todos los módulos + gestión de usuarios/roles + configuración sistema

| Módulo | Acciones necesarias |
|--------|---------------------|
| security | Ver/crear/editar/eliminar usuarios, ver/crear/editar roles, asignar roles, ver auditoría |
| device_management | Ver/crear/editar/eliminar dispositivos, asignar/desasignar, configurar remoto |
| telemetry | Ver eventos/evidencias/alertas, **ingestar eventos** (para testing/admin) |
| monitoring | Ver notificaciones, **enviar notificaciones** (manual admin) |
| analytics | Ver métricas, generar/exportar reportes |
| parameterization | Ver/crear/editar/eliminar catálogos |

### Rol `user` (conductor) — Solo sus datos y dispositivos
**Necesita:** Ver/gestionar SU dispositivo, ver SUS eventos/alertas/notificaciones, ver SUS métricas, ver catálogos (referencia)

| Módulo | Acciones necesarias |
|--------|---------------------|
| security | Ver SU perfil, actualizar SU perfil, eliminar SU cuenta (soft-delete), recuperar password |
| device_management | Ver SU dispositivo, **asociar/desasociar** SU dispositivo, ver config de SU dispositivo |
| telemetry | Ver SUS eventos, ver SUS alertas, **NO ingerir eventos** (eso es device via API key) |
| monitoring | Ver SUS notificaciones, **NO enviar notificaciones** |
| analytics | Ver SUS métricas, generar SUS reportes |
| parameterization | Ver catálogos (solo lectura, referencia) |

---

## Propuesta de features estandarizadas (25: 23 base + 2 provisioning HU-DB-003; implementadas en seeds)

### Convención: `{recurso}.{accion}` en minúscula | Acciones: `read`, `write`, `delete`, `assign`, `ingest`, `generate`

| # | Feature | Módulo | Descripción | Acción | Admin | User |
|---|---------|--------|-------------|--------|-------|------|
| 1 | `user.read` | security | Ver usuarios (lista y detalle) | READ | ✅ | ❌ (solo own via user.own_read) |
| 2 | `user.write` | security | Crear y actualizar usuarios | WRITE | ✅ | ❌ |
| 3 | `user.delete` | security | Eliminar usuario (soft-delete) | DELETE | ✅ | ✅ (own) |
| 4 | `user.own_read` | security | Ver propio perfil | READ | ✅ | ✅ |
| 5 | `user.own_write` | security | Actualizar propio perfil | WRITE | ✅ | ✅ |
| 6 | `role.read` | security | Ver roles y features | READ | ✅ | ❌ |
| 7 | `role.write` | security | CRUD roles (crear/editar/eliminar) | WRITE | ✅ | ❌ |
| 8 | `role.assign` | security | Asignar/quitar roles a usuarios | ASSIGN | ✅ | ❌ |
| 9 | `audit.read` | security | Ver logs de auditoría y login | READ | ✅ | ❌ |
| 10 | `device.read` | device_management | Ver dispositivos (lista y detalle) | READ | ✅ | ❌ (solo own) |
| 11 | `device.write` | device_management | Crear/editar/eliminar dispositivos (admin) | WRITE | ✅ | ❌ |
| 12 | `device.assign` | device_management | Asociar/desasociar dispositivo a usuario | ASSIGN | ✅ | ✅ (own) |
| 13 | `device.config_read` | device_management | Ver configuración de dispositivo | READ | ✅ | ✅ (own) |
| 14 | `device.config_write` | device_management | Gestionar configuración remota | WRITE | ✅ | ❌ |
| 15 | `event.read` | telemetry | Consultar eventos y evidencias | READ | ✅ | ✅ (own) |
| 16 | `event.ingest` | telemetry | Ingestar eventos (desde device via API key) | INGEST | ❌ | ❌ (device only) |
| 17 | `alert.read` | telemetry | Ver histórico de alarmas | READ | ✅ | ✅ (own) |
| 18 | `notification.read` | monitoring | Ver notificaciones propias | READ | ✅ | ✅ (own) |
| 19 | `notification.send` | monitoring | Enviar notificaciones (sistema/admin manual) | WRITE | ✅ | ❌ |
| 20 | `analytics.read` | analytics | Ver reportes y métricas | READ | ✅ | ✅ (own) |
| 21 | `analytics.generate` | analytics | Generar y exportar reportes | GENERATE | ✅ | ✅ (own) |
| 22 | `catalog.read` | parameterization | Ver catálogos de parametrización | READ | ✅ | ✅ |
| 23 | `catalog.write` | parameterization | Gestionar catálogos (CRUD) | WRITE | ✅ | ❌ |
| 24 | `device.provision` | device_management | Crear tokens de aprovisionamiento (solo admin, HU-DB-003) | WRITE | ✅ | ❌ |
| 25 | `device.claim` | device_management | Reclamar dispositivo con claim_code (HU-DB-003) | ASSIGN | ✅ | ✅ (own) |

---

## Matriz final: Admin vs User

| Feature | Admin | User | Notas |
|---------|-------|------|-------|
| `user.read` | ✅ | ❌ | Admin ve todos |
| `user.write` | ✅ | ❌ | Admin crea/edit users |
| `user.delete` | ✅ | ✅ own | Soft-delete |
| `user.own_read` | ✅ | ✅ | Perfil propio |
| `user.own_write` | ✅ | ✅ | Editar perfil propio |
| `role.read` | ✅ | ❌ | |
| `role.write` | ✅ | ❌ | CRUD roles |
| `role.assign` | ✅ | ❌ | Asignar roles a users |
| `audit.read` | ✅ | ❌ | |
| `device.read` | ✅ | ❌ | Admin ve todos |
| `device.write` | ✅ | ❌ | Admin CRUD devices |
| `device.assign` | ✅ | ✅ own | User asocia SU device |
| `device.config_read` | ✅ | ✅ own | Ver config |
| `device.config_write` | ✅ | ❌ | Admin configura remoto |
| `event.read` | ✅ | ✅ own | User ve SUS eventos |
| `event.ingest` | ❌ | ❌ | Solo device (API key) |
| `alert.read` | ✅ | ✅ own | User ve SUS alertas |
| `notification.read` | ✅ | ✅ own | User ve SUS notifs |
| `notification.send` | ✅ | ❌ | Admin/sistema |
| `analytics.read` | ✅ | ✅ own | User ve SUS métricas |
| `analytics.generate` | ✅ | ✅ own | User genera SU reporte |
| `catalog.read` | ✅ | ✅ | Referencia global |
| `catalog.write` | ✅ | ❌ | Solo admin |
| `device.provision` | ✅ | ❌ | Solo admin (HU-DB-003) |
| `device.claim` | ✅ | ✅ own | User reclama SU device |

---

## Cambios aplicados en BD (seeds, rama `fix/security-feature-dev`)

### RENOMBRADOS con UPDATE (4 features - preservan `role_feature`) — aplicados en `011`
```sql
-- device.config       → device.config_write (UPDATE por module device_management)
-- event.write         → event.ingest (UPDATE por module telemetry; solo API key, sin roles)
-- notification.write  → notification.send (UPDATE por module monitoring)
-- analytics.report    → analytics.generate (UPDATE por module analytics)
```

### CREADAS NUEVAS (6 features)
```sql
-- user.delete           -- soft-delete usuario (admin + own)
-- user.own_read         -- ver propio perfil
-- user.own_write        -- editar propio perfil
-- role.assign           -- asignar roles a usuarios (separado de role.write)
-- device.assign         -- asociar/desasociar device (user own + admin)
-- device.config_read    -- ver config device (user own + admin)
```

### PRESERVADAS HU-DB-003 (2 features, ya existían)
```sql
-- device.provision      -- crear tokens de aprovisionamiento (solo admin)
-- device.claim          -- reclamar con claim_code (admin + user own)
```

---

## SQL para seeds actualizados (011_insert_security_features.sql)

```sql
INSERT INTO security.feature (id, module_id, code, name, description, created_at, created_by, updated_at, updated_by)
VALUES
    -- Security (8 features)
    (gen_random_uuid(), (SELECT id FROM security.module WHERE code = 'security'), 'user.read', 'Leer usuarios', 'Ver lista y detalles de todos los usuarios', NOW(), '00000000-0000-0000-0000-000000000000', NOW(), '00000000-0000-0000-0000-000000000000'),
    (gen_random_uuid(), (SELECT id FROM security.module WHERE code = 'security'), 'user.write', 'Escribir usuarios', 'Crear y actualizar usuarios', NOW(), '00000000-0000-0000-0000-000000000000', NOW(), '00000000-0000-0000-0000-000000000000'),
    (gen_random_uuid(), (SELECT id FROM security.module WHERE code = 'security'), 'user.delete', 'Eliminar usuario', 'Soft-delete de usuario (ventana 30 días)', NOW(), '00000000-0000-0000-0000-000000000000', NOW(), '00000000-0000-0000-0000-000000000000'),
    (gen_random_uuid(), (SELECT id FROM security.module WHERE code = 'security'), 'user.own_read', 'Ver propio perfil', 'Ver datos de su propio usuario', NOW(), '00000000-0000-0000-0000-000000000000', NOW(), '00000000-0000-0000-0000-000000000000'),
    (gen_random_uuid(), (SELECT id FROM security.module WHERE code = 'security'), 'user.own_write', 'Editar propio perfil', 'Actualizar datos de su propio usuario', NOW(), '00000000-0000-0000-0000-000000000000', NOW(), '00000000-0000-0000-0000-000000000000'),
    (gen_random_uuid(), (SELECT id FROM security.module WHERE code = 'security'), 'role.read', 'Leer roles', 'Ver roles y sus features asignados', NOW(), '00000000-0000-0000-0000-000000000000', NOW(), '00000000-0000-0000-0000-000000000000'),
    (gen_random_uuid(), (SELECT id FROM security.module WHERE code = 'security'), 'role.write', 'Escribir roles', 'CRUD de roles (crear, editar, eliminar)', NOW(), '00000000-0000-0000-0000-000000000000', NOW(), '00000000-0000-0000-0000-000000000000'),
    (gen_random_uuid(), (SELECT id FROM security.module WHERE code = 'security'), 'role.assign', 'Asignar roles', 'Asignar y quitar roles a usuarios', NOW(), '00000000-0000-0000-0000-000000000000', NOW(), '00000000-0000-0000-0000-000000000000'),
    (gen_random_uuid(), (SELECT id FROM security.module WHERE code = 'security'), 'audit.read', 'Leer auditoría', 'Ver logs de auditoría y login', NOW(), '00000000-0000-0000-0000-000000000000', NOW(), '00000000-0000-0000-0000-000000000000'),

    -- Device Management (5 features)
    (gen_random_uuid(), (SELECT id FROM security.module WHERE code = 'device_management'), 'device.read', 'Leer dispositivos', 'Ver lista y detalles de dispositivos', NOW(), '00000000-0000-0000-0000-000000000000', NOW(), '00000000-0000-0000-0000-000000000000'),
    (gen_random_uuid(), (SELECT id FROM security.module WHERE code = 'device_management'), 'device.write', 'Escribir dispositivos', 'Crear, actualizar y eliminar dispositivos (admin)', NOW(), '00000000-0000-0000-0000-000000000000', NOW(), '00000000-0000-0000-0000-000000000000'),
    (gen_random_uuid(), (SELECT id FROM security.module WHERE code = 'device_management'), 'device.assign', 'Asignar dispositivo', 'Asociar y desasociar dispositivo a usuario', NOW(), '00000000-0000-0000-0000-000000000000', NOW(), '00000000-0000-0000-0000-000000000000'),
    (gen_random_uuid(), (SELECT id FROM security.module WHERE code = 'device_management'), 'device.config_read', 'Leer config dispositivo', 'Ver configuración remota de dispositivo', NOW(), '00000000-0000-0000-0000-000000000000', NOW(), '00000000-0000-0000-0000-000000000000'),
    (gen_random_uuid(), (SELECT id FROM security.module WHERE code = 'device_management'), 'device.config_write', 'Escribir config dispositivo', 'Gestionar configuración remota de dispositivo', NOW(), '00000000-0000-0000-0000-000000000000', NOW(), '00000000-0000-0000-0000-000000000000'),
    (gen_random_uuid(), (SELECT id FROM security.module WHERE code = 'device_management'), 'device.provision', 'Aprovisionar dispositivos', 'Crear tokens de aprovisionamiento (solo admin)', NOW(), '00000000-0000-0000-0000-000000000000', NOW(), '00000000-0000-0000-0000-000000000000'),
    (gen_random_uuid(), (SELECT id FROM security.module WHERE code = 'device_management'), 'device.claim', 'Reclamar dispositivos', 'Reclamar dispositivo con claim_code (admin+user)', NOW(), '00000000-0000-0000-0000-000000000000', NOW(), '00000000-0000-0000-0000-000000000000'),

    -- Telemetry (3 features)
    (gen_random_uuid(), (SELECT id FROM security.module WHERE code = 'telemetry'), 'event.read', 'Leer eventos', 'Consultar eventos y evidencias', NOW(), '00000000-0000-0000-0000-000000000000', NOW(), '00000000-0000-0000-0000-000000000000'),
    (gen_random_uuid(), (SELECT id FROM security.module WHERE code = 'telemetry'), 'event.ingest', 'Ingestar eventos', 'Ingesta de eventos desde dispositivo (API key)', NOW(), '00000000-0000-0000-0000-000000000000', NOW(), '00000000-0000-0000-0000-000000000000'),
    (gen_random_uuid(), (SELECT id FROM security.module WHERE code = 'telemetry'), 'alert.read', 'Leer alarmas', 'Ver histórico de alarmas', NOW(), '00000000-0000-0000-0000-000000000000', NOW(), '00000000-0000-0000-0000-000000000000'),

    -- Monitoring (2 features)
    (gen_random_uuid(), (SELECT id FROM security.module WHERE code = 'monitoring'), 'notification.read', 'Leer notificaciones', 'Ver notificaciones propias', NOW(), '00000000-0000-0000-0000-000000000000', NOW(), '00000000-0000-0000-0000-000000000000'),
    (gen_random_uuid(), (SELECT id FROM security.module WHERE code = 'monitoring'), 'notification.send', 'Enviar notificaciones', 'Enviar notificaciones (sistema/admin manual)', NOW(), '00000000-0000-0000-0000-000000000000', NOW(), '00000000-0000-0000-0000-000000000000'),

    -- Analytics (2 features)
    (gen_random_uuid(), (SELECT id FROM security.module WHERE code = 'analytics'), 'analytics.read', 'Leer analíticas', 'Ver reportes y métricas', NOW(), '00000000-0000-0000-0000-000000000000', NOW(), '00000000-0000-0000-0000-000000000000'),
    (gen_random_uuid(), (SELECT id FROM security.module WHERE code = 'analytics'), 'analytics.generate', 'Generar reportes', 'Crear y exportar reportes (PDF/HTML)', NOW(), '00000000-0000-0000-0000-000000000000', NOW(), '00000000-0000-0000-0000-000000000000'),

    -- Parameterization (2 features)
    (gen_random_uuid(), (SELECT id FROM security.module WHERE code = 'parameterization'), 'catalog.read', 'Leer catálogos', 'Ver catálogos de parametrización', NOW(), '00000000-0000-0000-0000-000000000000', NOW(), '00000000-0000-0000-0000-000000000000'),
    (gen_random_uuid(), (SELECT id FROM security.module WHERE code = 'parameterization'), 'catalog.write', 'Escribir catálogos', 'Gestionar catálogos (CRUD admin)', NOW(), '00000000-0000-0000-0000-000000000000', NOW(), '00000000-0000-0000-0000-000000000000')
ON CONFLICT (module_id, code) DO UPDATE SET
    name = EXCLUDED.name,
    description = EXCLUDED.description,
    updated_at = NOW(),
    updated_by = '00000000-0000-0000-0000-000000000000';
```

---

## SQL para role_feature actualizado (012_insert_security_role_feature.sql)

```sql
-- Admin: TODAS las 25 features (CROSS JOIN, incluye provision/claim)
INSERT INTO security.role_feature (id, role_id, feature_id, created_at, created_by)
SELECT gen_random_uuid(), r.id, f.id, NOW(), '00000000-0000-0000-0000-000000000000'
FROM security.role r
CROSS JOIN security.feature f
WHERE r.code = 'admin'
ON CONFLICT (role_id, feature_id) DO NOTHING;

-- Limpieza idempotente user: solo conserva matriz own (12)
-- (elimina user.read/device.read globales de seeds previos)
-- User: Solo features propias (12 features: 11 base + device.claim HU-DB-003)
INSERT INTO security.role_feature (id, role_id, feature_id, created_at, created_by)
SELECT gen_random_uuid(), r.id, f.id, NOW(), '00000000-0000-0000-0000-000000000000'
FROM security.role r
JOIN security.feature f ON f.module_id = (SELECT id FROM security.module WHERE code = 'security')
WHERE r.code = 'user' AND f.code IN ('user.own_read', 'user.own_write', 'user.delete')
ON CONFLICT (role_id, feature_id) DO NOTHING;

INSERT INTO security.role_feature (id, role_id, feature_id, created_at, created_by)
SELECT gen_random_uuid(), r.id, f.id, NOW(), '00000000-0000-0000-0000-000000000000'
FROM security.role r
JOIN security.feature f ON f.module_id = (SELECT id FROM security.module WHERE code = 'device_management')
WHERE r.code = 'user' AND f.code IN ('device.assign', 'device.config_read', 'device.claim')
ON CONFLICT (role_id, feature_id) DO NOTHING;

INSERT INTO security.role_feature (id, role_id, feature_id, created_at, created_by)
SELECT gen_random_uuid(), r.id, f.id, NOW(), '00000000-0000-0000-0000-000000000000'
FROM security.role r
JOIN security.feature f ON f.module_id = (SELECT id FROM security.module WHERE code = 'telemetry')
WHERE r.code = 'user' AND f.code IN ('event.read', 'alert.read')
ON CONFLICT (role_id, feature_id) DO NOTHING;

INSERT INTO security.role_feature (id, role_id, feature_id, created_at, created_by)
SELECT gen_random_uuid(), r.id, f.id, NOW(), '00000000-0000-0000-0000-000000000000'
FROM security.role r
JOIN security.feature f ON f.module_id = (SELECT id FROM security.module WHERE code = 'monitoring')
WHERE r.code = 'user' AND f.code IN ('notification.read')
ON CONFLICT (role_id, feature_id) DO NOTHING;

INSERT INTO security.role_feature (id, role_id, feature_id, created_at, created_by)
SELECT gen_random_uuid(), r.id, f.id, NOW(), '00000000-0000-0000-0000-000000000000'
FROM security.role r
JOIN security.feature f ON f.module_id = (SELECT id FROM security.module WHERE code = 'analytics')
WHERE r.code = 'user' AND f.code IN ('analytics.read', 'analytics.generate')
ON CONFLICT (role_id, feature_id) DO NOTHING;

INSERT INTO security.role_feature (id, role_id, feature_id, created_at, created_by)
SELECT gen_random_uuid(), r.id, f.id, NOW(), '00000000-0000-0000-0000-000000000000'
FROM security.role r
JOIN security.feature f ON f.module_id = (SELECT id FROM security.module WHERE code = 'parameterization')
WHERE r.code = 'user' AND f.code IN ('catalog.read')
ON CONFLICT (role_id, feature_id) DO NOTHING;
```

---

## Mapeo: Features actuales → Propuestas

| Feature actual | Acción | Feature nueva(s) |
|----------------|--------|------------------|
| `user.read` | MANTENER | `user.read` (admin) |
| `user.write` | DIVIDIR | `user.write` (admin) + `user.own_write` (user) + `user.delete` (nueva) |
| `role.read` | MANTENER | `role.read` |
| `role.write` | DIVIDIR | `role.write` (CRUD) + `role.assign` (nueva) |
| `audit.read` | MANTENER | `audit.read` |
| `device.read` | MANTENER | `device.read` (admin) |
| `device.write` | DIVIDIR | `device.write` (admin CRUD) + `device.assign` (nueva) |
| `device.config` | RENOMBRAR (UPDATE) | `device.config_write` (admin) + `device.config_read` (nueva) |
| `device.provision` | MANTENER (HU-DB-003) | `device.provision` (solo admin) |
| `device.claim` | MANTENER (HU-DB-003) | `device.claim` (admin + user own) |
| `event.read` | MANTENER | `event.read` |
| `event.write` | RENOMBRAR (UPDATE) | `event.ingest` (solo device, sin roles) |
| `alert.read` | MANTENER | `alert.read` |
| `notification.read` | MANTENER | `notification.read` |
| `notification.write` | RENOMBRAR (UPDATE) | `notification.send` (admin/sistema) |
| `analytics.read` | MANTENER | `analytics.read` |
| `analytics.report` | RENOMBRAR (UPDATE) | `analytics.generate` |
| `catalog.read` | MANTENER | `catalog.read` |
| `catalog.write` | MANTENER | `catalog.write` |

---

## Próximos pasos

1. ~~Aprobar esta propuesta con equipo~~ ✅ Aprobada y aplicada
2. ~~Actualizar seeds en `somnguard-db/02_dml/00_inserts/`~~ ✅ Aplicado en rama `fix/security-feature-dev`:
   - `011_insert_security_features.sql` → 25 features (4 UPDATE renombres + INSERT ... ON CONFLICT)
   - `012_insert_security_role_feature.sql` → admin 25 + user 12 + limpieza idempotente
   - `05_rollbacks/.../011_delete_security_features.sql` → 25 códigos
   - Changesets `011`/`012` tienen `runOnChange: true`, no requieren nuevo changeset
3. ~~Ejecutar migración (Liquibase) en entorno de desarrollo~~ → pendiente de `liquibase update` + `verify-rollback`
4. ~~Actualizar código backend para usar nuevos códigos de feature~~ ✅ Aplicado en API (rama actual, sin commit):
   - `DeviceController` list/get incluyen `device.assign`
   - `DeviceConfigController`/`DeviceConfigService` usan `device.config_read/write`
   - `AdminRbacController` asignaciones exigen `role.assign` (OR con `role.write` por compatibilidad)
   - `AccountController` exige `user.own_write`/`user.delete` (OR con `user.write`)
   - `DeviceAuthSupport.isAdmin()` reconoce `device.config_write` (+ legacy `device.config` temporal)
5. ~~Documentar en `docs/09-modules/modules/security/data-model.md` y `decisions.md`~~ → pendiente
