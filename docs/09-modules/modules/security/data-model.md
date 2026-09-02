<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Modelo de datos — security

**Estado:** En progreso
**Fecha:** 2026-09-01

</div>

</div>

> Última actualización: 2026-09-01

## Entidades propias

### user

Representa a los usuarios del sistema.

| Campo | Tipo | Nullable | Descripción |
|-------|------|----------|-------------|
| `id` | UUID | No | PK |
| `email` | VARCHAR(255) | No | Email único, login |
| `password_hash` | TEXT | No | Hash BCrypt/Argon2 |
| `first_name` | VARCHAR(100) | No | Nombre(s) |
| `last_name` | VARCHAR(100) | No | Apellido(s) |
| `phone` | VARCHAR(30) | Sí | Teléfono de contacto |
| `is_active` | BOOLEAN | No | Soft delete — por defecto TRUE |
| `email_verified_at` | TIMESTAMPTZ | Sí | Verificación de email |
| `last_login_at` | TIMESTAMPTZ | Sí | Último login exitoso |
| `failed_login_attempts` | SMALLINT | No | Contador de fallos consecutivos |
| `locked_until` | TIMESTAMPTZ | Sí | Bloqueo temporal por intentos fallidos |
| `created_at` | TIMESTAMPTZ | No | Fecha de creación (UTC) |
| `created_by` | UUID | No | Usuario creador (FK user.id o SYSTEM_ACTOR_ID) |
| `updated_at` | TIMESTAMPTZ | No | Última modificación (UTC) |
| `updated_by` | UUID | No | Último modificador (FK user.id o SYSTEM_ACTOR_ID) |
| `deleted_at` | TIMESTAMPTZ | Sí | Soft delete timestamp |
| `deleted_by` | UUID | Sí | Usuario que eliminó (FK user.id) |
| `version` | INTEGER | No | Optimistic locking (inicia en 1) |
| `status` | VARCHAR(50) | Sí | Estado de negocio (FK parameterization.status) |
| `status_category` | VARCHAR(30) | Sí | Categoría de estado (FK parameterization.status_category) |

**Reglas de negocio aplicables:** [RN-SEC-01] Email único, [RN-SEC-02] Bloqueo tras 5 intentos fallidos

### role

Define los roles del sistema.

| Campo | Tipo | Nullable | Descripción |
|-------|------|----------|-------------|
| `id` | UUID | No | PK |
| `code` | VARCHAR(50) | No | Código único (`admin`, `user`) |
| `name` | VARCHAR(100) | No | Nombre visible |
| `description` | TEXT | Sí | Descripción del rol |
| `is_active` | BOOLEAN | No | Por defecto TRUE |
| `created_at` | TIMESTAMPTZ | No | Fecha de creación (UTC) |
| `created_by` | UUID | Sí | Usuario creador (seed = SYSTEM_ACTOR_ID) |
| `updated_at` | TIMESTAMPTZ | No | Última modificación (UTC) |
| `updated_by` | UUID | Sí | Último modificador |

**Reglas de negocio aplicables:** [RN-SEC-03] Código único inmutable, solo 2 roles permitidos

### module

Agrupa funcionalidades del sistema.

| Campo | Tipo | Nullable | Descripción |
|-------|------|----------|-------------|
| `id` | UUID | No | PK |
| `code` | VARCHAR(50) | No | Código único (`security`, `device_management`, `telemetry`, `monitoring`, `analytics`, `parameterization`) |
| `name` | VARCHAR(100) | No | Nombre visible |
| `description` | TEXT | Sí | Descripción del módulo |
| `created_at` | TIMESTAMPTZ | No | Fecha de creación (UTC) |
| `created_by` | UUID | Sí | Usuario creador (seed = SYSTEM_ACTOR_ID) |
| `updated_at` | TIMESTAMPTZ | No | Última modificación (UTC) |
| `updated_by` | UUID | Sí | Último modificador |

**Reglas de negocio aplicables:** [RN-SEC-04] Código único inmutable, 6 módulos fijos

### feature

Representa una funcionalidad protegida del sistema.

| Campo | Tipo | Nullable | Descripción |
|-------|------|----------|-------------|
| `id` | UUID | No | PK |
| `module_id` | UUID | No | FK a module.id |
| `code` | VARCHAR(50) | No | Código único con formato `recurso.accion` (`user.read`, `device.write`, etc.) |
| `name` | VARCHAR(100) | No | Nombre visible |
| `description` | TEXT | Sí | Descripción de la feature |
| `created_at` | TIMESTAMPTZ | No | Fecha de creación (UTC) |
| `created_by` | UUID | Sí | Usuario creador (seed = SYSTEM_ACTOR_ID) |
| `updated_at` | TIMESTAMPTZ | No | Última modificación (UTC) |
| `updated_by` | UUID | Sí | Último modificador |

**Reglas de negocio aplicables:** [RN-SEC-05] Código único por módulo, [RN-SEC-06] Formato: `recurso.accion` en minúscula

### role_feature

Relaciona roles con funcionalidades (sin scope).

| Campo | Tipo | Nullable | Descripción |
|-------|------|----------|-------------|
| `id` | UUID | No | PK |
| `role_id` | UUID | No | FK a role.id |
| `feature_id` | UUID | No | FK a feature.id |
| `created_at` | TIMESTAMPTZ | No | Fecha de creación (UTC) |
| `created_by` | UUID | Sí | Usuario creador (seed = SYSTEM_ACTOR_ID) |
| `updated_at` | TIMESTAMPTZ | No | Última modificación (UTC) |
| `updated_by` | UUID | Sí | Último modificador |
| `deleted_at` | TIMESTAMPTZ | Sí | Soft delete timestamp |
| `deleted_by` | UUID | Sí | Usuario que eliminó |
| `version` | INTEGER | No | Optimistic locking (inicia en 1) |
| `is_active` | BOOLEAN | No | Por defecto TRUE |

**Reglas de negocio aplicables:** [RN-SEC-07] Unicidad (role_id, feature_id), **no hay scope_type**

### user_role

Asigna roles a los usuarios con vigencia temporal.

| Campo | Tipo | Nullable | Descripción |
|-------|------|----------|-------------|
| `id` | UUID | No | PK |
| `user_id` | UUID | No | FK a user.id |
| `role_id` | UUID | No | FK a role.id |
| `assigned_at` | TIMESTAMPTZ | No | Momento de asignación |
| `expires_at` | TIMESTAMPTZ | Sí | Expiración opcional (NULL = indefinido) |
| `created_at` | TIMESTAMPTZ | No | Fecha de creación (UTC) |
| `created_by` | UUID | No | Usuario que asignó |
| `updated_at` | TIMESTAMPTZ | No | Última modificación (UTC) |
| `updated_by` | UUID | No | Último modificador |
| `deleted_at` | TIMESTAMPTZ | Sí | Soft delete |
| `deleted_by` | UUID | Sí | Usuario que revocó |
| `version` | INTEGER | No | Optimistic locking |
| `is_active` | BOOLEAN | No | Por defecto TRUE |

**Reglas de negocio aplicables:** [RN-SEC-08] Usuario puede tener múltiples roles, [RN-SEC-09] Roles activos = deleted_at IS NULL AND (expires_at IS NULL OR expires_at > NOW())

### password_reset_request

Solicitudes de recuperación de contraseña.

| Campo | Tipo | Nullable | Descripción |
|-------|------|----------|-------------|
| `id` | UUID | No | PK |
| `user_id` | UUID | No | FK a user.id |
| `token_hash` | TEXT | No | Hash del token (no almacenar en claro) |
| `expires_at` | TIMESTAMPTZ | No | Expiración del token (ej. 1 hora) |
| `is_used` | BOOLEAN | No | Por defecto FALSE |
| `used_at` | TIMESTAMPTZ | Sí | Momento de uso |
| `created_at` | TIMESTAMPTZ | No | Fecha de creación (UTC) |
| `created_by` | UUID | Sí | Usuario solicitante |

**Reglas de negocio aplicables:** [RN-SEC-10] Token de un solo uso, [RN-SEC-11] Expiración máxima 24h

### audit_login

Registro de intentos de autenticación.

| Campo | Tipo | Nullable | Descripción |
|-------|------|----------|-------------|
| `id` | UUID | No | PK |
| `user_id` | UUID | Sí | FK a user.id (NULL si email no existe) |
| `email_attempted` | VARCHAR(255) | No | Email usado en el intento |
| `outcome` | VARCHAR(50) | No | SUCCESS, FAILED_CREDENTIALS, FAILED_LOCKED, FAILED_INACTIVE |
| `ip_address` | VARCHAR(45) | Sí | IPv4 o IPv6 |
| `user_agent` | TEXT | Sí | User-Agent del cliente |
| `attempted_at` | TIMESTAMPTZ | No | Momento del intento (UTC) |
| `created_at` | TIMESTAMPTZ | No | Fecha de creación (UTC) |
| `created_by` | UUID | Sí | SYSTEM_ACTOR_ID |

**Reglas de negocio aplicables:** [RN-SEC-12] Append-only, no UPDATE ni DELETE

## Referencias a otras tablas (misma BD)

| Campo | Tabla referenciada | Acción referencial |
|-------|--------------------|-------------------|
| `user_role.user_id` | `security.user` | ON DELETE CASCADE |
| `user_role.role_id` | `security.role` | ON DELETE RESTRICT |
| `role_feature.role_id` | `security.role` | ON DELETE CASCADE |
| `role_feature.feature_id` | `security.feature` | ON DELETE RESTRICT |
| `feature.module_id` | `security.module` | ON DELETE RESTRICT |
| `password_reset_request.user_id` | `security.user` | ON DELETE CASCADE |
| `audit_login.user_id` | `security.user` | ON DELETE SET NULL |

## Índices relevantes

| Tabla | Campos indexados | Tipo | Motivo |
|-------|-----------------|------|--------|
| `user` | `email` | UNIQUE | Login por email |
| `user` | `is_active, deleted_at` | BTREE | Filtro soft delete |
| `role` | `code` | UNIQUE | Lookup por código |
| `module` | `code` | UNIQUE | Lookup por código |
| `feature` | `module_id, code` | UNIQUE | Feature único por módulo |
| `role_feature` | `role_id, feature_id` | UNIQUE | Evitar duplicados |
| `user_role` | `user_id, role_id, deleted_at` | UNIQUE | Rol activo único por usuario |
| `user_role` | `user_id, expires_at` | BTREE | Consultar roles vigentes |
| `password_reset_request` | `user_id, is_used, expires_at` | BTREE | Buscar token válido |
| `audit_login` | `user_id, attempted_at` | BTREE | Historial por usuario |
| `audit_login` | `email_attempted, attempted_at` | BTREE | Detección fuerza bruta |

## Notas de integridad

- **Catálogos inmutables**: `role`, `module`, `feature`, `role_feature` no llevan soft delete; se desactivan con `is_active = false` solo si nunca se usaron (ver modeling-conventions.md)
- **SYSTEM_ACTOR_ID**: `00000000-0000-0000-0000-000000000000` para seeds y procesos automáticos
- **Roles actuales**: Solo `admin` y `user` (según seeds en somnguard-db)
- **Features actuales**: 17 features en minúscula formato `recurso.accion` (ver seeds 011_insert_security_features.sql)
- **Inicialización**: Seeds en `somnguard-db/02_dml/00_inserts/` crean módulos, features, roles y role_feature base