<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Análisis y estandarización de Features — security

**Estado:** En progreso
**Fecha:** 2026-09-01

</div>

</div>

> Última actualización: 2026-09-01
> Fuente: somnguard-db seeds (011_insert_security_features.sql, 012_insert_security_role_feature.sql) + docs/04-requeriments/

---

## Resumen ejecutivo

| Métrica | Valor |
|---------|-------|
| Roles en BD | 2 (`admin`, `user`) |
| Módulos en BD | 6 |
| Features actuales en BD | 17 |
| Features propuestas | 23 |
| **A eliminar** | 2 |
| **A renombrar** | 5 |
| **A crear nuevas** | 7 |

---

## Features actuales en BD (17)

| Módulo | Feature | Descripción | Problema detectado |
|--------|---------|-------------|-------------------|
| security | `user.read` | Ver lista y detalles de usuarios | OK |
| security | `user.write` | Crear, actualizar, eliminar usuarios | Ambigüedad: ¿incluye soft-delete? |
| security | `role.read` | Ver roles y permisos | OK |
| security | `role.write` | Gestionar roles y asignaciones | Mezcla CRUD roles + asignar a usuarios |
| security | `audit.read` | Ver logs de auditoría y login | OK |
| device_management | `device.read` | Ver dispositivos y asignaciones | OK |
| device_management | `device.write` | Registrar, asignar, configurar dispositivos | Muy amplio: alta + asignación + config |
| device_management | `device.config` | Gestionar configuración remota | No es acción CRUD estándar |
| telemetry_service | `event.read` | Consultar eventos y evidencias | OK |
| telemetry_service | `event.write` | Ingestar eventos (device) | Es ingesta device, no acción usuario |
| telemetry_service | `alert.read` | Ver histórico de alarmas | OK |
| monitoring | `notification.read` | Ver notificaciones propias | OK |
| monitoring | `notification.write` | Enviar notificaciones (sistema) | Es acción sistema, no usuario |
| analytics | `analytics.read` | Ver reportes y métricas | OK |
| analytics | `analytics.report` | Crear y exportar reportes | OK |
| parameterization | `catalog.read` | Ver catálogos de parametrización | OK |
| parameterization | `catalog.write` | Gestionar catálogos (admin) | OK |

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

## Propuesta de features estandarizadas (20 features)

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

---

## Cambios requeridos en BD (seeds)

### ELIMINAR (2 features - no aplican a usuarios)
```sql
-- monitoring.notification.write → Es acción del sistema, no feature de usuario
-- telemetry.event.write → Es ingesta de device, no acción de usuario
```

### RENOMBRAR (5 features - aclarar semántica)
```sql
-- device.write        → device.write (solo admin CRUD) + device.assign (nuevo)
-- device.config       → device.config_write (admin) + device.config_read (nuevo)
-- user.write          → user.write (admin) + user.own_write (user) + user.delete (nuevo)
-- role.write          → role.write (CRUD roles) + role.assign (nuevo)
-- analytics.report    → analytics.generate (acción clara)
```

### CREAR NUEVAS (7 features)
```sql
-- user.delete           -- soft-delete usuario (admin + own)
-- user.own_read         -- ver propio perfil
-- user.own_write        -- editar propio perfil
-- role.assign           -- asignar roles a usuarios (separado de role.write)
-- device.assign         -- asociar/desasociar device (user own + admin)
-- device.config_read    -- ver config device (user own + admin)
-- event.ingest          -- ingesta desde device (solo API key, no roles)
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
-- Admin: TODAS las 23 features
INSERT INTO security.role_feature (id, role_id, feature_id, created_at, created_by)
SELECT gen_random_uuid(), r.id, f.id, NOW(), '00000000-0000-0000-0000-000000000000'
FROM security.role r
CROSS JOIN security.feature f
WHERE r.code = 'admin'
ON CONFLICT (role_id, feature_id) DO NOTHING;

-- User: Solo features propias (13 features)
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
WHERE r.code = 'user' AND f.code IN ('device.assign', 'device.config_read')
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
| `device.config` | DIVIDIR | `device.config_write` (admin) + `device.config_read` (nueva) |
| `event.read` | MANTENER | `event.read` |
| `event.write` | ELIMINAR/RENOMBRAR | `event.ingest` (solo device, no roles) |
| `alert.read` | MANTENER | `alert.read` |
| `notification.read` | MANTENER | `notification.read` |
| `notification.write` | ELIMINAR | (acción sistema, no feature usuario) |
| `analytics.read` | MANTENER | `analytics.read` |
| `analytics.report` | RENOMBRAR | `analytics.generate` |
| `catalog.read` | MANTENER | `catalog.read` |
| `catalog.write` | MANTENER | `catalog.write` |

---

## Próximos pasos

1. **Aprobar esta propuesta** con equipo
2. **Actualizar seeds** en `somnguard-db/02_dml/00_inserts/`:
   - `011_insert_security_features.sql` → 23 features
   - `012_insert_security_role_feature.sql` → Matriz admin/user arriba
3. **Ejecutar migración** (Liquibase) en entorno de desarrollo
4. **Actualizar código** backend para usar nuevos códigos de feature
5. **Documentar** en `docs/09-modules/modules/security/data-model.md` y `decisions.md`