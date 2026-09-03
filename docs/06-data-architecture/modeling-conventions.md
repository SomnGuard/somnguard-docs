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

### Tablas transaccionales con UPDATE concurrente (user, device, device_config, notification, device_assignment, event_type)
| Columna | Tipo | Nullable | Descripción |
|---------|------|----------|-------------|
| `created_at` | TIMESTAMPTZ | No | Momento de creación (UTC) |
| `updated_at` | TIMESTAMPTZ | No | Última modificación (UTC) |
| `deleted_at` | TIMESTAMPTZ | Sí | Soft delete; `NULL` = no eliminado |
| `created_by` | UUID | No | Usuario que creó (FK `user.id` o `SYSTEM_ACTOR_ID`) |
| `updated_by` | UUID | No | Último usuario que modificó (FK `user.id` o `SYSTEM_ACTOR_ID`) |
| `deleted_by` | UUID | Sí | Usuario que eliminó (FK `user.id` o `SYSTEM_ACTOR_ID`) |
| `version` | INTEGER | No | Optimistic locking (empezar en 1) |
| `is_active` | BOOLEAN | No | **Campo de soft delete** — por defecto TRUE. FALSE = inactivo. |
| `status` | VARCHAR(50) | SÍ | NULL | **Estado de negocio** — solo en `parameterization.event_type` (tiene FK a `parameterization.status`). En las demás tablas (user, device, notification) NULL = no aplica. Esta columna no es obligatoria en la base de datos. |
| `status_category` | VARCHAR(30) | SÍ | NULL | **Categoría de estado** — solo en `parameterization.event_type` (tiene FK a `parameterization.status_category`). En las demás tablas NULL = no aplica. No es obligatoria en la base de datos. |

### Tablas transaccionales solo INSERT / append-only (event, evidence, alert_log, audit_login, password_reset_request, device_config_history)
| Columna | Tipo | Nullable | Descripción |
|---------|------|----------|-------------|
| `created_at` | TIMESTAMPTZ | No | Momento de creación (UTC) |
| `created_by` | UUID | Sí | Usuario/Device que creó (FK `user.id` o `device.id`) |
| `is_active` | BOOLEAN | No | **Soft delete** — por defecto TRUE. FALSE = inactivo. — controla eliminación lógica. |
| `status` | VARCHAR(50) | SÍ | NULL | **Estado de negocio** — solo en `parameterization.event_type` (tiene FK a `parameterization.status`). En las demás tablas NULL = no aplica. No es columna base. |
| `status_category` | VARCHAR(30) | SÍ | NULL | **Categoría de estado** — solo en `parameterization.event_type` (tiene FK a `parameterization.status_category`). En las demás tablas NULL = no aplica. No es columna base. |

### Tablas catálogo (parameterization + security.role/module/feature/role_feature)
| Columna | Tipo | Nullable | Descripción |
|---------|------|----------|-------------|
| `created_at` | TIMESTAMPTZ | No | Momento de creación (UTC) |
| `created_by` | UUID | Sí | Usuario que creó (seed = SYSTEM_ACTOR_ID) |
| `updated_at` | TIMESTAMPTZ | No | Última modificación (UTC) |
| `updated_by` | UUID | Sí | Usuario que modificó |

> **Nota:** Catálogos no llevan soft delete (`deleted_at`/`deleted_by`/`version`); se desactivan con `is_active = false` en `event_type` o no se modifican (role, module, feature, status_category, status, status_transition).

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
| Ciclo de vida del registro | ¿La fila existe y está habilitada? | `deleted_at` IS NULL (activo) / `is_active = FALSE` (inactivo) / timestamp (eliminado) — `is_active` es el campo de soft delete en todas las tablas |
| Estado de negocio | Posición en una máquina de estados (ej. estados del dispositivo en `device`, del evento en `event`) | FK a catálogo parametrizable: `status` + `status_category` (ADR-009) — **solo en 5 tablas core** |
| Enum técnico cerrado | Conjunto fijo e inmutable (ej. `media_type`, `severity`, `event_category`) | Catálogo `parameterization` (inmutables, solo `created_at`/`updated_at`/`created_by`/`updated_by`) |

> **Regla de oro:** el soft delete (`is_active = FALSE` / `deleted_at IS NULL`) y el estado de negocio (`status`/`status_category`) son ejes ortogonales.
> **Estado actual en la BD:**
> - `is_active` BOOLEAN es el campo de soft delete **en todas las tablas** (por defecto TRUE).
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
