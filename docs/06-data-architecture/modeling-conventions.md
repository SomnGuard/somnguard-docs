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

### Tablas transaccionales (todas)

| Columna | Tipo | Nullable | Descripción |
|---------|------|----------|-------------|
| `created_at` | TIMESTAMPTZ | No | Momento de creación (UTC) |
| `updated_at` | TIMESTAMPTZ | No | Última modificación (UTC) |
| `deleted_at` | TIMESTAMPTZ | Sí | Soft delete; `NULL` = no eliminado |
| `created_by` | UUID | No | Usuario que creó (FK `user.id` o `SYSTEM_ACTOR_ID`) |
| `updated_by` | UUID | No | Último usuario que modificó (FK `user.id` o `SYSTEM_ACTOR_ID`) |
| `deleted_by` | UUID | Sí | Usuario que eliminó (FK `user.id` o `SYSTEM_ACTOR_ID`) |
| `is_active` | BOOLEAN | No | Habilitado/deshabilitado sin eliminar (DEFAULT true) |

### Actor de sistema

Las acciones automáticas (workers, jobs, seeds) usan el UUID reservado:

```
SYSTEM_ACTOR_ID = 00000000-0000-0000-0000-000000000000
```

### Tablas catálogo (parameterization)

Solo requieren `created_at`, `updated_at`, `is_active`. No llevan soft delete: un valor de catálogo se desactiva con `is_active = false`, no se elimina.

### Tablas append-only

`audit_login` y similares conservan únicamente su timestamp de inserción. No tienen `updated_*`, `deleted_*` ni `is_active`: son inmutables por definición.

### Regla de consulta

Toda query de lectura sobre tablas transaccionales filtra por defecto `WHERE deleted_at IS NULL`.

## 2. Estados de negocio vs enums técnicos

| Concepto | Qué representa | Cómo se modela |
|----------|----------------|----------------|
| Ciclo de vida del registro | ¿La fila existe y está habilitada? | `is_active` + `deleted_at` (soft delete) |
| Estado de negocio | Posición en una máquina de estados (ej. estados del dispositivo en `device`, del evento en `event`) | FK a catálogo parametrizable (vía `parameterization`) o `VARCHAR + CHECK` según decisión |
| Enum técnico cerrado | Conjunto fijo e inmutable (ej. `media_type`, `severity`) | Catálogo `parameterization` o `VARCHAR + CHECK` |

> Regla de oro: un registro puede estar `is_active = true` (habilitado) y a la vez tener un estado de negocio intermedio. Son ejes ortogonales.
> Nota: los estados de negocio de SomnGuard se modelan con los catálogos de `parameterization` (`event_category`, `severity`, `media_type`, `sound_pattern`, `event_type`); la decisión de introducir `status` genérico parametrizable queda abierta (ver [open-questions.md](../15-project-control/open-questions.md)).

## 3. Otras convenciones (vigentes)

- **Sin `ENUM` nativo de Postgres** (dificulta migraciones).
- **Acciones referenciales**: cada FK declara `ON UPDATE`/`ON DELETE`. Por defecto: catálogo/padre → `RESTRICT`; hijo de agregado (composición) → `CASCADE`; FK opcional → `SET NULL`.
- **Nomenclatura de constraints**: `pk_<tabla>`, `uq_<tabla>_<cols>`, `fk_<tabla>_<ref>`, `ck_<tabla>_<regla>`, `ix_<tabla>_<cols>`.
- **PK**: UUID v4 en todas las tablas.
- **Timestamps**: siempre `TIMESTAMPTZ` (UTC); la conversión a hora local es de la capa de presentación.

## 4. Estructura DDL y orden de aplicación (Liquibase)

Los changelogs se organizan en carpetas numeradas que definen el orden de ejecución:

```
01_ddl/
  00_extensions/   -- CREATE EXTENSION (pgcrypto / gen_random_uuid)
  01_schemas/      -- CREATE SCHEMA del módulo
  02_types/        -- DOMAIN / tipos (si aplica)
  03_tables/       -- CREATE TABLE (SIN llaves foráneas)
  04_alter/        -- ALTER TABLE ... ADD CONSTRAINT (llaves foráneas)
  05_views/        -- vistas
  06_functions/    -- funciones
  07_procedures/   -- procedimientos
  08_triggers/     -- triggers
  10_indexes/      -- índices (incluye un índice por cada FK)
02_dml/            -- datos semilla (seeds), con control de duplicados
03_dcl/            -- roles y GRANT/REVOKE (least-privilege)
04_tcl/            -- tags de versión / release
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