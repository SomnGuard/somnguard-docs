<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Convenciones de modelado

**Estado:** En progreso
**Fecha:** 2026-08-19

</div>

</div>

Convenciones transversales del modelo de datos de SomnGuard. Aplica a las 20 entidades transaccionales y a los catálogos de `parameterization`. Es la **fuente única** de convenciones; los modelos por módulo deben referenciar este estándar.

## 1. Estándar de auditoría (columnas obligatorias)

### Tablas transaccionales con UPDATE concurrente (user, device, device_config, notification, device_assignment, event, event_type)
| Columna | Tipo | Nullable | Descripción |
|---------|------|----------|-------------|
| `created_at` | TIMESTAMPTZ | No | Momento de creación (UTC) |
| `updated_at` | TIMESTAMPTZ | No | Última modificación (UTC) |
| `deleted_at` | TIMESTAMPTZ | Sí | Soft delete; `NULL` = no eliminado |
| `created_by` | UUID | Sí | Usuario/Device que creó (FK `user.id`/`device.id` o `SYSTEM_ACTOR_ID`; NULL = auto-registro/sistema) |
| `updated_by` | UUID | Sí | Último usuario/dispositivo que modificó (NULL si no hubo UPDATE) |
| `deleted_by` | UUID | Sí | Usuario que eliminó (FK `user.id` o `SYSTEM_ACTOR_ID`) |
| `version` | INTEGER | No | Optimistic locking (empezar en 1) |
| `is_active` | BOOLEAN | No | **Campo de soft delete** — por defecto TRUE. FALSE = inactivo. |
| `status` | VARCHAR(50) | Sí (default NULL; NOT NULL DEFAULT solo en `event_type`) | **Estado de negocio** — en `user, device, device_config, notification, event, event_type` (FK a `parameterization.status`). `device_assignment` NO tiene estado. |
| `status_category` | VARCHAR(30) | Sí (default NULL; NOT NULL DEFAULT solo en `event_type`) | **Categoría de estado** — en las mismas 6 tablas (FK a `parameterization.status_category`). |

### Tablas transaccionales solo INSERT / append-only (evidence, alert_log, audit_login, password_reset_request, device_config_history)
| Columna | Tipo | Nullable | Descripción |
|---------|------|----------|-------------|
| `created_at` | TIMESTAMPTZ | No | Momento de creación (UTC) |
| `created_by` | UUID | Sí | Usuario/Device que creó (FK `user.id` o `device.id`) |
| `is_active` | BOOLEAN | No | **Soft delete** — por defecto TRUE. FALSE = inactivo. (Ausente en `device_config_history`.) |

### Tablas catálogo (parameterization + security.role/module/feature)
| Columna | Tipo | Nullable | Descripción |
|---------|------|----------|-------------|
| `created_at` | TIMESTAMPTZ | No | Momento de creación (UTC) |
| `created_by` | UUID | Sí | Usuario que creó (seed = SYSTEM_ACTOR_ID) |
| `updated_at` | TIMESTAMPTZ | No | Última modificación (UTC) |
| `updated_by` | UUID | Sí | Usuario que modificó |

> `role_feature` y `user_role` son transaccionales (llevan `version`/`deleted_at`/`is_active`), no catálogos.

> **Nota:** Catálogos con `is_active` (`role`, `event_type`, `event_category`, `severity`, `media_type`, `sound_pattern`) se desactivan con `is_active = false`; `module`/`feature`/`status_*` son inmutables (sin `is_active`, sin UPDATE).

### Actor de sistema

Las acciones automáticas (workers, jobs, seeds) usan el UUID reservado:

```
SYSTEM_ACTOR_ID = 00000000-0000-0000-0000-000000000000
```

### Regla de consulta

Toda query de lectura sobre tablas transaccionales filtra por defecto `WHERE deleted_at IS NULL`.

## 2. Estados de negocio vs enums técnicos

| Concepto | Qué representa | Cómo se modela |
|----------|----------------|----------------|
| Ciclo de vida del registro | ¿La fila existe y está habilitada? | `deleted_at` IS NULL (activo) / `is_active = FALSE` (inactivo) / timestamp (eliminado) — `is_active` base en transaccionales (sin `is_active`: `module`, `feature`, `status_*`, `device_config_history`) |
| Estado de negocio | Posición en una máquina de estados (ej. estados del dispositivo en `device`, del evento en `event`) | FK a catálogo parametrizable: `status` + `status_category` (ADR-009) — 5 core (`user, device, event, device_config, notification`) + `event_type` (workflow de catálogo) |
| Enum técnico cerrado | Conjunto fijo e inmutable (ej. `media_type`, `severity`, `event_category`) | Catálogo `parameterization` (inmutables, solo `created_at`/`updated_at`/`created_by`/`updated_by`) |

> **Regla de oro:** el soft delete (`is_active = FALSE` / `deleted_at IS NULL`) y el estado de negocio (`status`/`status_category`) son ejes ortogonales.
> **Estado actual en la BD:**
> - `is_active` BOOLEAN es el campo de soft delete por defecto TRUE (ausente en `module`, `feature`, `status_category`, `status`, `status_transition` y `device_config_history`).
> - `status` + `status_category` VARCHARs existen en: `parameterization.event_type` (catálogo), `security.user`, `device_management.device`, `telemetry_service.event`, `device_management.device_config`, `monitoring.notification` (6 tablas total, ADR-009 aplicado).

## 3. Otras convenciones (vigentes)

- **Sin `ENUM` nativo de Postgres** (dificulta migraciones).
- **Acciones referenciales**: cada FK declara `ON UPDATE`/`ON DELETE`. Por defecto: catálogo/padre → `RESTRICT`; hijo de agregado (composición) → `CASCADE`; FK opcional → `SET NULL`.
- **Nomenclatura de constraints**: `pk_<tabla>` o el nombre automático de PostgreSQL (`<tabla>_pkey`), `uq_<tabla>_<cols>`, `fk_<tabla>_<ref>`, `ck_<tabla>_<regla>`, `ix_<tabla>_<cols>`.
- **PK**: UUID v7 (generado en app, no en BD) en todas las tablas.
- **Timestamps**: siempre `TIMESTAMPTZ` (UTC); la conversión a hora local es de la capa de presentación.

## 4. Estructura DDL y orden de aplicación (Liquibase)

Los changelogs se organizan en carpetas numeradas que definen el orden de ejecución:

```
01_ddl/
  00_extensions/   -- (no extensiones requeridas; gen_random_uuid() es nativa en PostgreSQL 16)
  01_schemas/      -- CREATE SCHEMA del módulo
  02_types/        -- DOMAIN / tipos (si aplica)
  03_tables/       -- CREATE TABLE (SIN llaves foráneas)
  04_alter/        -- ALTER TABLE ... ADD CONSTRAINT (llaves foráneas)
  05_views/        -- vistas
  06_materialized_views/  -- vistas materializadas
  07_functions/    -- funciones
  08_procedures/   -- procedimientos
  09_triggers/     -- triggers
  10_indexes/      -- índices (incluye un índice por cada FK)
02_dml/            -- datos semilla (seeds), con control de duplicados
03_dcl/            -- roles y GRANT/REVOKE (least-privilege)
04_tcl/            -- bloques transaccionales y recuperaciones manuales
05_rollbacks/      -- rollbacks espejo de cada changeset
```

### Regla: las llaves foráneas van en `04_alter`, no en `03_tables`

- `03_tables`: `CREATE TABLE` con PK, columnas, `NOT NULL`, `UNIQUE` y `CHECK` locales. **Sin `REFERENCES`.**
- `04_alter`: un changeset por grupo de FKs — `ALTER TABLE <hija> ADD CONSTRAINT fk_<tabla>_<ref> FOREIGN KEY (...) REFERENCES <padre> (...) ON UPDATE ... ON DELETE ...`.
- `10_indexes`: crear el índice de cada columna FK (Postgres no lo crea automáticamente).
- `05_rollbacks`: espejo exacto — `DROP CONSTRAINT` para `04_alter`, `DROP TABLE` para `03_tables`.

Motivo: separar la creación de estructura del cableado referencial hace el despliegue determinista e independiente del orden entre tablas.

## Ver también

- [Modelo de datos vigente](./02-modules-entities.md)
- [Entidades y reglas de negocio](../02-domain/entities-and-rules.md)
- [Catálogos](../01-project-context/software-technical-proposal.md)
