<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Estrategia de Migraciones Liquibase SomnGuard

**Estado:** Borrador  
**Fecha:** 2026-08-22

</div>

</div>

> **Objetivo:** Que `somnguard-db` genere changelogs consistentes y que `somnguard-api`, `somnguard-portal`, `somnguard-app`, `somnguard-device` desarrollen contra contratos compartidos sin bloqueos de FK ni datos inconsistentes.

> **Fuente de verdad:** Estructura real de `somnguard-db` (changelogs pure `include` + `sqlFile`, sin cambios inline).

---

## 1. Visión General

El repositorio `somnguard-db` administra **1 solo PostgreSQL 16** con **6 esquemas** (módulos), cada uno con su propio changelog independiente. No hay cambios DDL inline en yaml; todo se resuelve mediante `sqlFile` que apuntan a archivos `.sql` externos.

El punto de entrada único es `changelog-master.yaml` que incluye los módulos en orden definido. Las migraciones son **idempotentes** por diseño (seed data con `ON CONFLICT DO NOTHING`) y tienen **rollback espejo** en `05_rollbacks/` que también usan `sqlFile`.

> **Arquitectura:** Cada módulo es dueño de su schema y sus objetos (tablas, FK, índices, datos). No hay un solo archivo gigante; la separación por módulo permite que los 5 repos (API, DB, APP, PORTAL, DEVICE) trabajen en paralelo sin bloqueos.

---

## 2. Estructura de Changelogs

### 2.1 `changelog-master.yaml` (punto de entrada único)

Ubicación: `somnguard-db/changelog/changelog-master.yaml`

```yaml
databaseChangeLog:
  - include:
      file: ../01_ddl/changelog.yaml
      relativeToChangelogFile: true
      contextFilter: 01_ddl
      labels: 00_extensions
  - include:
      file: ../02_dml/changelog.yaml
      relativeToChangelogFile: true
      contextFilter: 02_dml
      labels: dml-base
  - include:
      file: ../03_dcl/changelog.yaml
      relativeToChangelogFile: true
      contextFilter: 03_dcl
      labels: dcl
  - include:
      file: ../04_tcl/changelog.yaml
      relativeToChangelogFile: true
      contextFilter: 04_tcl
      labels: tcl
```

### 2.2 Estructura por módulo en `01_ddl/changelog.yaml`

Ubicación: `somnguard-db/01_ddl/changelog.yaml`

```yaml
databaseChangeLog:
  - include:
      file: 00_extensions/0000changelog.yaml
      relativeToChangelogFile: true
      contextFilter: 01_ddl
      labels: 00_extensions
  - include:
      file: 01_schemas/0000changelog.yaml
      relativeToChangelogFile: true
      contextFilter: 01_ddl
      labels: 01_schemas
  - include:
      file: 02_types/0000changelog.yaml
      relativeToChangelogFile: true
      contextFilter: 01_ddl
      labels: 02_types
  - include:
      file: 03_tables/0000changelog.yaml
      relativeToChangelogFile: true
      contextFilter: 01_ddl
      labels: 03_tables
  - include:
      file: 04_alter/0000changelog.yaml
      relativeToChangelogFile: true
      contextFilter: 01_ddl
      labels: 04_alter
  - include:
      file: 05_views/0000changelog.yaml
      relativeToChangelogFile: true
      contextFilter: 01_ddl
      labels: 05_views
  - include:
      file: 06_materialized_views/0000changelog.yaml
      relativeToChangelogFile: true
      contextFilter: 01_ddl
      labels: 06_materialized_views
  - include:
      file: 07_functions/0000changelog.yaml
      relativeToChangelogFile: true
      contextFilter: 01_ddl
      labels: 07_functions
  - include:
      file: 08_procedures/0000changelog.yaml
      relativeToChangelogFile: true
      contextFilter: 01_ddl
      labels: 08_procedures
  - include:
      file: 09_triggers/0000changelog.yaml
      relativeToChangelogFile: true
      contextFilter: 01_ddl
      labels: 09_triggers
  - include:
      file: 10_indexes/0000changelog.yaml
      relativeToChangelogFile: true
      contextFilter: 01_ddl
      labels: 10_indexes
```

### 2.3 Ejemplo real de changeset (formato real)

Ubicación: `somnguard-db/01_ddl/03_tables/0000changelog.yaml` (fragmento real)

```yaml
databaseChangeLog:
  - changeSet:
      id: 001-create-security-user
      author: CristianJPalma
      labels: "BD-01,ddl,tables,security"
      comment: Creates the security.user table
      changes:
        - sqlFile:
            path: 001_create_security_user.sql
            relativeToChangelogFile: true
            stripComments: false
  - changeSet:
      id: 002-create-security-role
      author: CristianJPalma
      labels: "BD-01,ddl,tables,security"
      comment: Creates the security.role table
      changes:
        - sqlFile:
            path: 002_create_security_role.sql
            relativeToChangelogFile: true
            stripComments: false
```

### 2.4 Rollback espejo (formato real)

Siempre usa `sqlFile` con ruta relativa al rollback:

```yaml
  - changeSet:
      id: 001-create-security-user
      author: CristianJPalma
      labels: "BD-01,ddl,tables,security"
      comment: Creates the security.user table
      changes:
        - sqlFile:
            path: 001_create_security_user.sql
            relativeToChangelogFile: true
            stripComments: false
      rollback:
        - sqlFile:
            path: ../../05_rollbacks/01_ddl/03_tables/001_drop_security_user.sql
            relativeToChangelogFile: true
            stripComments: false
```

### 2.5 Estructura `02_dml/changelog.yaml` (datos semilla)

Ubicación: `somnguard-db/02_dml/changelog.yaml`

```yaml
databaseChangeLog:
  - include:
      file: 00_inserts/0000changelog.yaml
      relativeToChangelogFile: true
      contextFilter: 02_dml
      labels: 00_inserts
  - include:
      file: 01_updates/0000changelog.yaml
      relativeToChangelogFile: true
      contextFilter: 02_dml
      labels: 01_updates
  - include:
      file: 02_deletes/0000changelog.yaml
      relativeToChangelogFile: true
      contextFilter: 02_dml
      labels: 02_deletes
  - include:
      file: 03_upserts/0000changelog.yaml
      relativeToChangelogFile: true
      contextFilter: 02_dml
      labels: 03_upserts
  - include:
      file: 04_patches/0000changelog.yaml
      relativeToChangelogFile: true
      contextFilter: 02_dml
      labels: 04_patches
```

### 2.6 Rollback en `02_dml` (usando UUIDs auto-generados)

```yaml
  - changeSet:
      id: 60888613-1e9b-40bc-b368-86d62f9cb2c1
      author: JuanCarlos
      labels: "BD-01-dev,dml,base,inserts"
      comment: Seeds the initial event_category catalog records.
      changes:
        - sqlFile:
            path: 001_insert_event_category.sql
            relativeToChangelogFile: true
            stripComments: false
      rollback:
        - sqlFile:
            path: ../../05_rollbacks/02_dml/00_inserts/001_delete_event_category.sql
            relativeToChangelogFile: true
            stripComments: false
```

---

## 3. Orden de Ejecución y Dependencias Críticas

La secuencia **obligatoria** para `docker compose --profile tooling run liquibase update` es:

1. **`01_ddl`** (siempre primero): Extensiones, schemas, tipos, tablas sin FK, alter (FKs), views, materialized views, functions, procedures, triggers, índices
2. **`02_dml`** (después de DDL): Datos semilla idempotentes (catálogos, roles, estados iniciales)
3. **`03_dcl`** (después de datos): Roles y GRANT/REVOKE (least-privilege)
4. **`04_tcl`** (opcional): Bloques transaccionales y recuperaciones manuales

### 3.1 Orden interno por módulo (siempre el mismo)

Dentro de cada módulo (security, parameterization, device-management, telemetry-service, monitoring), el orden es fijo:

```
00_extensions → 01_schemas → 02_types → 03_tables → 04_alter → 05_views → 06_materialized_views → 07_functions → 08_procedures → 09_triggers → 10_indexes
```

### 3.2 Dependencias entre módulos (CRÍTICO)

| Módulo | Depende de | Justificación |
|--------|------------|---------------|
| `security` | Ninguna | Tabla base: users, roles, features - el foundation |
| `parameterization` | Independiente | Catálogos independientes (event_type, severity, media_type, sound_pattern, status_category, status); no tienen FKs a security |
| `device-management` | `security` | Devices → user asignation; FKs a users/roles para config y asignaciones |
| `telemetry-service` | `device-management` + `parameterization` | Events → device FK, event_type/severity FK a catálogos |
| `monitoring` | `telemetry-service` | Notifications → alert_log FK, user FK a security.user |
| `analytics` | Todas (solo lectura) | Vistas materializadas sobre tables de todos los módulos anteriores |

**Regla de oro:** El orden de módulos es: `security` → `device-management` → `parameterization` → `telemetry-service` → `monitoring` → `analytics`. `parameterization` es independiente y puede aplicarse antes o después que `security` sin conflictos de FK.

### 3.3 Aplicación por módulo (para desarrollo paralelo)

```bash
# Aplica solo security (para que API desarrolle auth)
docker compose --profile tooling run --rm liquibase update --contexts=security

# Aplica todos en orden
docker compose --profile tooling run --rm liquibase update

# Rollback específico módulo
docker compose --profile tooling run --rm liquibase rollback-count 1 --contexts=telemetry-service
```

---

## 4. Convenciones Obligatorias en Every Changeset

Basadas en la estructura real de `somnguard-db`, **todas** las changesets deben cumplir:

| Convención | Especificación (real) |
|------------|----------------------|
| **Formato** | Siempre `changeSet { id, author, labels, comment }` + `changes: - sqlFile { path, relativeToChangelogFile, stripComments }` |
| **Naming** | `snake_case` en nombres de archivos `.sql` (no en yaml ids) |
| **PK** | UUID generado en app (archivo `.sql`: `gen_random_uuid()`), **nunca** definición de tipo en yaml |
| **Auditoría base** | Cada tabla SQL debe tener: `created_at` TIMESTAMPTZ NOT NULL, `created_by` UUID NULL |
| **Auditoría extendida** | Tablas con UPDATE real: `updated_at`, `updated_by`, `version` INTEGER NOT NULL DEFAULT 1 |
| **Soft delete** | Tablas transaccionales: `deleted_at` TIMESTAMPTZ NULL, `deleted_by` UUID NULL; NULL = activo; nunca `DELETE` físico |
| **Estados (ADR-009)** | 6 tablas core: `status_category` + `status` con FK a `parameterization` (`parameterization.event_type`, `security.user`, `device_management.device`, `telemetry_service.event`, `device_management.device_config`, `monitoring.notification`) |
| **JSONB** | Para configuraciones flexibles, almacenado en columnas `configuration JSONB` |
| **FKs** | `ON DELETE RESTRICT` por defecto; `ON DELETE CASCADE` solo en composición (evidence→event, notification→alert_log) |
| **Índices** | Definidos en archivos `.sql` separados, no en yaml |
| **Contexto** | `contextFilter` y `labels` obligatorios en cada `include` |
| **Rollback** | Siempre `sqlFile` con ruta relativa `../../05_rollbacks/...` que espeje el upgrade |

### 4.1 Estructura de archivo `.sql` (ejemplo real)

Archivo: `somnguard-db/01_ddl/03_tables/001_create_security_user.sql` (contenido real esperado):

```sql
CREATE TABLE security.user (
    id UUID PRIMARY KEY,  -- sin DEFAULT, generado en app (UUID v7)
    email VARCHAR(255) NOT NULL UNIQUE,
    password_hash TEXT NOT NULL,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    phone VARCHAR(30) UNIQUE,
    status VARCHAR(50) NOT NULL DEFAULT 'PENDING_VERIFICATION',
    status_category VARCHAR(30) NOT NULL DEFAULT 'PENDING',
    email_verified_at TIMESTAMPTZ,
    last_login_at TIMESTAMPTZ,
    failed_login_attempts SMALLINT NOT NULL DEFAULT 0,
    locked_until TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by UUID NULL,
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_by UUID NULL,
    deleted_at TIMESTAMPTZ,
    deleted_by UUID NULL,
    version INTEGER NOT NULL DEFAULT 1
);

-- Índices y constraints después del CREATE TABLE
CREATE UNIQUE INDEX idx_user_email ON security.user (email);
CREATE UNIQUE INDEX idx_user_phone ON security.user (phone WHERE deleted_at IS NULL);
CREATE INDEX idx_user_status_active ON security.user (status) WHERE deleted_at IS NULL;
ALTER TABLE security.user ADD CONSTRAINT fk_user_created_by FOREIGN KEY (created_by) REFERENCES security.user(id);
ALTER TABLE security.user ADD CONSTRAINT fk_user_updated_by FOREIGN KEY (updated_by) REFERENCES security.user(id);
ALTER TABLE security.user ADD CONSTRAINT fk_user_deleted_by FOREIGN KEY (deleted_by) REFERENCES security.user(id);
```

### 4.2 Estructura archivo rollback SQL

Archivo: `somnguard-db/05_rollbacks/01_ddl/03_tables/001_drop_security_user.sql` (formato real):

```sql
DROP INDEX IF EXISTS idx_user_status_active;
DROP INDEX IF EXISTS idx_user_phone;
DROP INDEX IF EXISTS idx_user_email;
DROP TABLE IF EXISTS security.user CASCADE;
```

---

## 5. Datos Semilla (Seed Data) - Idempotente por Diseño

### 5.1 Ubicación y formato

Todos los seeds están en `02_dml/`, cada uno es un archivo `.sql` independiente con `ON CONFLICT DO NOTHING`.

### 5.2 Catálogos base (archivos reales esperados)

| Archivo SQL | Contenido | Referencia |
|-------------|-----------|------------|
| `001_insert_event_category.sql` | AS-01..AS-09, SOMNOLENCE, DISTRACTION, SEATBELT, SYSTEM | `data-dictionary.md` |
| `002_insert_severity.sql` | info, warning, high, critical | `data-dictionary.md` |
| `003_insert_media_type.sql` | image_jpeg, video_mp4 | `data-dictionary.md` |
| `004_insert_sound_pattern.sql` | AS-01..AS-09 con freq/dur/repeticiones | `data-dictionary.md` |
| `005_insert_event_type.sql` | EV-SOM-01..EV-SYS-06 con thresholds | `data-dictionary.md` |

### 5.3 Seed idempotente (patrón real)

```sql
INSERT INTO parameterization.event_category (code, name, description, sort_order)
VALUES ('SOMNOLENCE', 'Somnolencia', 'Eventos de somnolencia', 0)
ON CONFLICT (code) DO UPDATE SET
  name = EXCLUDED.name,
  description = EXCLUDED.description,
  sort_order = EXCLUDED.sort_order;
```

### 5.4 Roles y estados iniciales

Archivos en `02_dml/00_inserts/`:
- `006_insert_roles.sql` - roles admin, user
- `007_insert_user_role.sql` - asignaciones base
- `010_insert_status_init.sql` - estados device: REGISTERED, ASSIGNED, ACTIVE

### 5.5 Estrategia de aplicación

```bash
# Aplica todos los módulos en orden (solo DDL + DML, sin DCL aún)
docker compose --profile tooling run --rm liquibase update

# Solo DDL (primeros 2 perfiles)
docker compose --profile tooling run --rm liquibase update --contexts=01_ddl

# Solo datos semilla (después de DDL)
docker compose --profile tooling run --rm liquibase update --contexts=02_dml
```

---

## 6. Estrategia de Rollback

### 6.1 Rollback por changeset (formato real)

Cada changeset en `01_ddl` y `02_dml` debe tener `rollback` que use `sqlFile` relativo:

```yaml
  - changeSet:
      id: 001-create-security-user
      author: CristianJPalma
      labels: "BD-01,ddl,tables,security"
      comment: Creates the security.user table
      changes:
        - sqlFile:
            path: 001_create_security_user.sql
            relativeToChangelogFile: true
            stripComments: false
      rollback:
        - sqlFile:
            path: ../../05_rollbacks/01_ddl/03_tables/001_drop_security_user.sql
            relativeToChangelogFile: true
            stripComments: false
```

### 6.2 Rollback por módulo (orden inverso)

```bash
# Rollback módulo telemetry-service (último en aplicarse, primero en rollback)
docker compose --profile tooling run --rm liquibase rollback 018 --contexts=telemetry-service

# Rollback completo a etiqueta anterior
docker compose --profile tooling run --rm liquibase rollback count 5
```

### 6.3 Casos edge - Rollback con datos

| Escenario | Acción |
|-----------|--------|
| Datos semilla ya insertados | `INSERT ... ON CONFLICT DO NOTHING` ignora re-insert (idempotente) |
| FKs referenciadas | Rollback falla con error; se deben borrar datos dependientes manualmente primero |
| Vistas materializadas | Rollback `DROP MATERIALIZED VIEW` + `DROP VIEW` en archivo `.sql` |
| Triggers | Rollback `DROP TRIGGER` preserva datos; solo quita la lógica automática |

---

## 7. Integración CI/CD

### 7.1 GitHub Actions - Validación de PR (formato real)

```yaml
name: Validate Liquibase

on:
  pull_request:
    paths:
      - 'somnguard-db/**'

jobs:
  liquibase-validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up JDK 21
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '21'
          
      - name: Build Liquibase Docker image
        run: docker build -t liquibase-somnguard ./somnguard-db
      
      - name: Validate changelogs
        run: |
          docker run --rm \
            -v $(PWD)/somnguard-db:/liquibase \
            -e LIQUIBASE_URL=jdbc:postgresql://localhost:5432/somnguard \
            -e LIQUIBASE_USERNAME=somnguard_user \
            -e LIQUIBASE_PASSWORD=test \
            liquibase-somnguard validate
      
      - name: Check for uncommitted changes
        run: |
          docker run --rm \
            -v $(PWD)/somnguard-db:/liquibase \
            liquibase-somnguard diffChangeLog > /tmp/changelog.diff
          if [ -s /tmp/changelog.diff ]; then
            echo "⚠️  Hay cambios pendientes en el changelog"
            cat /tmp/changelog.diff
            exit 1
          fi
```

### 7.2 Checklist de validación por PR (formato real)

- [ ] `liquibase validate` pasa sin errores
- [ ] Todos los `include` tienen `contextFilter` y `labels` consistentes
- [ ] Order de módulos correcto: `01_ddl` antes que `02_dml` antes que `03_dcl`
- [ ] Seed data idempotente (`ON CONFLICT DO NOTHING` probado en entorno develop)
- [ ] Rollback probado en `develop` (no en `main` ni `producción`)
- [ ] Todos los `sqlFile` paths son relativos correctos y existen
- [ ] Convenciones de naming en archivos `.sql` (snake_case, prefijos por módulo)
- [ ] Documentación actualizada: `data-dictionary.md`, `modeling-conventions.md`

---

## 8. Checklist de Migración a Nuevo Entorno

Para levantar un entorno nuevo (local/dev/qa/prod):

```bash
# 1. Levantar infraestructura (BD, MinIO, Redis, Traefik, API, Portal)
docker compose --profile api --profile observability up -d

# 2. Aplicar migraciones ordenadas (DDL → DML → DCL)
docker compose --profile tooling run --rm liquibase update

# 3. Verificar estado de schemas y tables
docker compose exec postgres psql -U somnguard_user -d somnguard -c "
  SELECT schema_name, count(table_name) as tables 
  FROM information_schema.tables 
  WHERE table_schema NOT IN ('pg_catalog', 'information_schema') 
  GROUP BY schema_name ORDER BY schema_name;
"

# 4. Verificar rollback funcional
docker compose --profile tooling run --rm liquibase rollback-count 1
```

---

## 9. Referencias Cruzadas

| Documento | Sección |
|-----------|---------|
| `data-dictionary.md` | Convenciones de columna, naming, PK, auditoría |
| `modeling-conventions.md` | Estructura DDL, orden Liquibase, FKs, índices, convención `.sql` |
| `cross-cutting.md` | Estándares de auditoría, estados, TZ, errores |
| `ADR-003` | Decisión de 1 BD PostgreSQL + esquemas por módulo |
| `ADR-004` | Estados parametrizados (`status_category` + `status`) |
| `ADR-006` | MinIO/S3 para evidencia (retención ILM) |
| `software-analysis.md` | F-01..F-10 funcionalidades que requieren BD |
| `01_ddl/changelog.yaml` | Estructura real de includes por módulo |
| `02_dml/changelog.yaml` | Estructura real de includes de datos semilla |
| `changelog-master.yaml` | Punto de entrada único |

---
