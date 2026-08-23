# ADR-004: Database Strategy — 1 PostgreSQL + Esquemas por Módulo + Liquibase

**Estado:** Aceptada
**Fecha:** 2026-08-22
**Autores:** Equipo SomnGuard
**Equipos involucrados:** Arquitectura, Desarrollo, DBA, Device

---

## Contexto

SomnGuard es un **monolito modular** (no microservicios) con 7 módulos de negocio en el backend:
- `security`, `parameterization`, `device_management`, `telemetry_service`, `monitoring`, `analytics`

Además, el dispositivo edge (Raspberry Pi) usa **SQLite local** para buffer offline.

Necesitamos una estrategia de base de datos que:
- Garantice aislamiento de datos por módulo (ownership claro)
- Permita migraciones versionadas y automatizadas (CI/CD)
- Sea operativamente simple (1 servidor, 1 backup, 1 monitoring)
- Escale verticalmente para MVP (1000 usuarios concurrentes, < 10k eventos/día)
- No añada complejidad de red (transacciones distribuidas, saga, etc.)

---

## Decisión

### 1. Un único servidor PostgreSQL 16
- **1 instancia** (local: contenedor Docker; prod: managed PG o VM)
- **1 base de datos** llamada `somnguard`
- **6 esquemas** (uno por módulo backend): `security`, `parameterization`, `device_management`, `telemetry_service`, `monitoring`, `analytics`
- **Esquema `public`**: solo extensiones y objetos compartidos (ninguna tabla de negocio)

### 2. Ownership estricto por esquema
| Esquema | Módulo Owner | Tablas Principales |
|---------|--------------|-------------------|
| `security` | Security | `user`, `role`, `feature`, `role_feature`, `user_role`, `password_reset_request`, `audit_login` |
| `parameterization` | Parameterization | `event_category`, `severity`, `media_type`, `sound_pattern`, `event_type`, `status_category`, `status`, `status_transition` |
| `device_management` | Device Management | `device`, `device_assignment`, `device_config`, `device_config_history` |
| `telemetry_service` | Telemetry Service | `event`, `evidence`, `alert_log` |
| `monitoring` | Monitoring | `notification`, `notification_delivery` |
| `analytics` | Analytics | **Sin tablas propias** — vistas materializadas y proyecciones sobre esquemas ajenos |

> **Regla:** Un módulo **solo escribe** en su esquema. Puede **leer** de otros esquemas **solo via puertos de entrada** (use cases) del módulo owner — nunca `JOIN` directo ni `SELECT` crudo cross-esquema en código de aplicación.

### 3. Migraciones con Liquibase por módulo
- Cada módulo tiene su `changelog/changelog-master.yaml` en `somnguard-db/<modulo>/`
- `changelog-master.yaml` incluye changesets ordenados (001_create_tables, 002_add_fks, 003_seed_data...)
- **Orden de ejecución global** (respetando FKs cross-esquema):
  1. `parameterization` (catálogos base, sin deps)
  2. `security` (users, roles; FK a parameterization.status)
  3. `device_management` (device; FK a security.user)
  4. `telemetry_service` (event, evidence, alert_log; FK a device, parameterization)
  5. `monitoring` (notification; FK a telemetry.event, security.user)
  6. `analytics` (vistas materializadas; depende de todos)

- **Runner único** en Docker Compose (`somnguard-docker-infra`): perfil `tooling` ejecuta Liquibase contra la BD única aplicando todos los changelogs en orden.

### 4. Convenciones de Modelado (Obligatorias — ver `modeling-conventions.md`)
- **Naming:** `snake_case` tablas y columnas (`event_type_id`, `created_at`)
- **PK:** UUID v7 (generado en app, no `gen_random_uuid()` en BD)
- **Auditoría en TODAS las tablas transaccionales:**
  - `created_at` TIMESTAMPTZ NOT NULL DEFAULT now()
  - `created_by` UUID NULL (user_id o device_id)
  - `updated_at` TIMESTAMPTZ NOT NULL DEFAULT now()
  - `updated_by` UUID NULL
  - `deleted_at` TIMESTAMPTZ NULL (soft delete)
  - `version` INTEGER NOT NULL DEFAULT 1 (optimistic locking)
- **Soft delete obligatorio:** Nunca `DELETE` físico; `UPDATE SET deleted_at = now()`
- **JSONB** para configuraciones flexibles (`device_config.config_json`, `event.metadata`)
- **FKs** con `ON DELETE RESTRICT` (integridad referencial estricta)
- **Índices** en columnas de filtro frecuente: `device_id`, `occurred_at`, `event_type_id`, `severity_id`

### 5. Device Edge — SQLite Local (Offline-First)
- **Archivo:** `data/db/somnguard_local.db` (no versionado)
- **Tablas:** `pending_events` (event_json, evidence_path, retries, created_at), `device_config_cache`, `catalogs_cache`
- **Sync:** Ver ADR-005 (Offline-first Device)

---

## Consecuencias

### Positivas
- **Operacionalmente simple:** 1 BD, 1 backup, 1 punto de monitoring, 1 conexión pool
- **Aislamiento lógico:** Esquemas = ownership claro; migraciones por módulo = deploy independiente
- **Transacciones ACID cross-módulo** cuando sea necesario (ej. asignar device + crear assignment en misma tx)
- **Liquibase nativo:** Versionado, rollback, diff, seed data — estándar de la industria
- **Esquema `analytics` read-only:** Vistas materializadas optimizadas sin afectar ingesta

### Negativas / Trade-offs
- **Acoplamiento de despliegue:** Todos los módulos comparten BD → migración coordinada (mitigado: orden fijo + CI valida)
- **Ruido de vecino:** Query pesada en analytics puede afectar telemetría (mitigado: vistas materializadas + índices + pool separado opcional)
- **Escalabilidad vertical:** Límite en write throughput de PG único (mitigado: MVP < 10k eventos/día; futuro: read replicas + particionado)

### Riesgos
- **Migración fallida en un módulo bloquea a todos** → Mitigación: CI ejecuta `liquibase update` en BD limpia en cada PR; rollback probado
- **FK cross-esquema mal gestionado** → Mitigación: Revisión de arquitectura en PR; tests de integración con BD real (Testcontainers)
- **Drift schema vs código** → Mitigación: `liquibase diff` en CI; convenciones estrictas en `modeling-conventions.md`

---

## Alternativas consideradas

| Alternativa | Por qué se descartó |
|-------------|---------------------|
| 1 BD por módulo (7 BDs) | Complejidad operativa: 7 backups, 7 pools, 7 monitorings, transacciones distribuidas para operaciones cross-módulo (ej. crear device + assignment). Overkill para monolito. |
| 1 BD, 1 esquema `public` para todo | Sin ownership claro; migraciones conflictivas; imposible saber qué módulo "posee" qué tabla. |
| Microservicios con BD propia c/u | Decidido en ADR-001/002: monolito modular. Complejidad operativa injustificada para tamaño actual. |
| MongoDB / NoSQL | Datos relacionales fuertes (users, devices, events, FKs); consistencia ACID requerida; equipo experto en PG. |
| Flyway en lugar de Liquibase | Liquibase ya en uso; soporte XML/YAML/SQL; diff automático; rollback mejor; equipo familiarizado. |

---

## Referencias

- [cross-cutting.md](../cross-cutting.md#2-auditoría-y-trazabilidad-de-datos)
- [modeling-conventions.md](../../../06-data-architecture/modeling-conventions.md)
- [migration-strategy.md](../../../06-data-architecture/migration-strategy.md)
- [software-design-report.md](../../software-design-report.md) §3.2
- [module-catalog.md](../../../09-modules/module-catalog.md)
- Liquibase docs: https://docs.liquibase.com