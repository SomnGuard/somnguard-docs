<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Diccionario de Datos Unificado

**Estado:** Borrador
**Fecha:** 2026-08-22

</div>

</div>

> **Fuentes:** [02-modules-entities.md](../06-data-architecture/02-modules-entities.md) (MER + atributos),
> [entities-and-rules.md](../02-domain/entities-and-rules.md) (RN-*),
> [ADR-004](../05-architecture/decisions/records/ADR-004-database-strategy.md) (BD strategy + convenciones),
> [ADR-009](../05-architecture/decisions/records/ADR-009-status-parametrized-audit.md) (estados + auditoría),
> [ADR-001](../05-architecture/decisions/records/ADR-001-backend-java-spring-boot.md) (auth),
> [SRS](../04-requeriments/01-srs/) (RF-*, RNF-*).
>
> **Convenciones base (ADR-004):**
> - **Naming:** `snake_case` tablas/columnas
> - **PK:** UUID v7 (generado en app)
> - **Auditoría obligatoria:** `created_at`, `created_by`, `updated_at`, `updated_by`, `deleted_at`, `deleted_by`, `version` (optimistic lock)
> - **Soft delete:** `deleted_at` NOT NULL = eliminado; nunca `DELETE` físico
> - **Estados:** `status_category` + `status` (catálogo `parameterization.status_category` / `status`) — ADR-009
> - **JSONB** para configuraciones flexibles
> - **FKs:** `ON DELETE RESTRICT`
> - **Índices** en columnas de filtro frecuente

---

## Resumen de Entidades por Módulo/Esquema

| Esquema (Módulo) | Entidades (Tablas) | Tipo |
|------------------|-------------------|------|
| `security` | `user`, `role`, `module`, `feature`, `role_feature`, `user_role`, `password_reset_request`, `audit_login` | Transaccional |
| `parameterization` | `event_category`, `severity`, `media_type`, `sound_pattern`, `event_type`, `status_category`, `status`, `status_transition` | Catálogo / Config |
| `device_management` | `device`, `device_assignment`, `device_config`, `device_config_history` | Transaccional |
| `telemetry_service` | `event`, `evidence`, `alert_log` | Transaccional (alta escritura) |
| `monitoring` | `notification`, `notification_delivery` | Transaccional |
| `analytics` | *Sin tablas propias* — vistas materializadas (`v_event_timeline`, `v_metrics_daily`) | Analítico |

**Total: 20 tablas transaccionales + 3 catálogos de estado + vistas analíticas**

---

## Convenciones de Columna (Aplican a TODAS las tablas)

| Campo | Tipo | Null | Default | Descripción |
|-------|------|------|---------|-------------|
| `id` | UUID | NO | gen_random_uuid() v7* | PK — *generado en app, no en BD |
| `created_at` | TIMESTAMPTZ | NO | `now()` | Auditoría: creación (UTC) |
| `created_by` | UUID | SÍ | — | User ID o Device ID que creó |
| `updated_at` | TIMESTAMPTZ | NO | `now()` | Auditoría: última modificación (UTC) |
| `updated_by` | UUID | SÍ | — | User ID o Device ID que modificó |
| `deleted_at` | TIMESTAMPTZ | SÍ | NULL | **Soft delete** — NULL = activo |
| `deleted_by` | UUID | SÍ | — | User ID o Device ID que eliminó |
| `version` | INTEGER | NO | 1 | Optimistic locking (incrementa en cada UPDATE) |

> **Nota:** Tablas de solo auditoría/append-only (`audit_login`, `alert_log`, `notification_delivery`, `device_config_history`, `status_transition`) **no** tienen `deleted_at` / `deleted_by` / `version` (solo INSERT).

---

## Esquema: `security` (Módulo Security)

### `security.user` — Usuarios del sistema (Portal, App, Admin)

| Columna | Tipo | Null | Default | FK | Índice | Descripción | RN |
|---------|------|------|---------|----|--------|-------------|-----|
| `id` | UUID | NO | — | PK | PK | Identificador único | |
| `email` | VARCHAR(255) | NO | — | — | **UNIQUE** | Correo único (login) | RN-01 |
| `password_hash` | TEXT | NO | — | — | — | Hash bcrypt (costo 12) | RN-01 |
| `first_name` | VARCHAR(100) | NO | — | — | — | Nombre | |
| `last_name` | VARCHAR(100) | NO | — | — | — | Apellido | |
| `phone` | VARCHAR(30) | SÍ | — | — | **UNIQUE** | Teléfono único (opcional) | RN-01 |
| `status` | VARCHAR(50) | NO | `PENDING_VERIFICATION` | `parameterization.status(code)` | — | Estado: PENDING_VERIFICATION, ACTIVE, SUSPENDED, SOFT_DELETED | ADR-009 |
| `status_category` | VARCHAR(30) | NO | `PENDING` | `parameterization.status_category(code)` | — | Categoría derivada | ADR-009 |
| `email_verified_at` | TIMESTAMPTZ | SÍ | — | — | — | Timestamp verificación correo | |
| `last_login_at` | TIMESTAMPTZ | SÍ | — | — | — | Último login exitoso | |
| `failed_login_attempts` | SMALLINT | NO | 0 | — | — | Contador fallos consecutivos | |
| `locked_until` | TIMESTAMPTZ | SÍ | — | — | — | Bloqueo temporal tras 5 fallos | |
| *auditoría estándar* | — | — | — | — | — | `created_at`, `created_by`, `updated_at`, `updated_by`, `deleted_at`, `deleted_by`, `version` | |

**Índices compuestos:** `idx_user_status_active (status) WHERE deleted_at IS NULL`

**Reglas:** RN-01 (unicidad email/phone, hash pwd), RN-02 (roles/permisos via `user_role` + `role_feature`)

---

### `security.role` — Roles del sistema

| Columna | Tipo | Null | Default | FK | Índice | Descripción |
|---------|------|------|---------|----|--------|-------------|
| `id` | UUID | NO | — | PK | PK | Identificador |
| `code` | VARCHAR(50) | NO | — | — | **UNIQUE** | Código técnico: `admin`, `user` |
| `name` | VARCHAR(100) | NO | — | — | — | Nombre legible |
| `description` | TEXT | SÍ | — | — | — | Descripción |
| *auditoría estándar* | — | — | — | — | — | (sin `deleted_at` — catálogo inmutable) |

**Seed:** `admin` (todas features), `user` (features propias)

---

### `security.module` — Módulos funcionales (agrupadores de features)

| Columna | Tipo | Null | Default | FK | Índice | Descripción |
|---------|------|------|---------|----|--------|-------------|
| `id` | UUID | NO | — | PK | PK | Identificador |
| `code` | VARCHAR(50) | NO | — | — | **UNIQUE** | Código: `security`, `device_management`, `telemetry`, `monitoring`, `analytics`, `parameterization` |
| `name` | VARCHAR(100) | NO | — | — | — | Nombre legible |
| `description` | TEXT | SÍ | — | — | — | Descripción |
| *auditoría estándar* | — | — | — | — | — | (sin `deleted_at`) |

---

### `security.feature` — Funcionalidades protegidas (permisos atómicos)

| Columna | Tipo | Null | Default | FK | Índice | Descripción |
|---------|------|------|---------|----|--------|-------------|
| `id` | UUID | NO | — | PK | PK | Identificador |
| `module_id` | UUID | NO | — | `security.module(id)` | IDX | Módulo al que pertenece |
| `code` | VARCHAR(50) | NO | — | — | **UNIQUE (module_id, code)** | Código: `device.read`, `event.write`, `analytics.report`, etc. |
| `name` | VARCHAR(100) | NO | — | — | — | Nombre legible |
| `description` | TEXT | SÍ | — | — | — | Descripción |
| *auditoría estándar* | — | — | — | — | — | (sin `deleted_at`) |

---

### `security.role_feature` — Asignación Role ↔ Feature (N:M)

| Columna | Tipo | Null | Default | FK | Índice | Descripción |
|---------|------|------|---------|----|--------|-------------|
| `id` | UUID | NO | — | PK | PK | Identificador |
| `role_id` | UUID | NO | — | `security.role(id)` | **UNIQUE (role_id, feature_id)** | Role |
| `feature_id` | UUID | NO | — | `security.feature(id)` | IDX | Feature |
| *auditoría estándar* | — | — | — | — | — | (solo `created_at`, `created_by`) |

---

### `security.user_role` — Asignación User ↔ Role (N:M) con vigencia

| Columna | Tipo | Null | Default | FK | Índice | Descripción |
|---------|------|------|---------|----|--------|-------------|
| `id` | UUID | NO | — | PK | PK | Identificador |
| `user_id` | UUID | NO | — | `security.user(id)` | **UNIQUE (user_id, role_id) WHERE deleted_at IS NULL** | Usuario |
| `role_id` | UUID | NO | — | `security.role(id)` | IDX | Role |
| `assigned_at` | TIMESTAMPTZ | NO | `now()` | — | — | Cuándo se asignó |
| `expires_at` | TIMESTAMPTZ | SÍ | — | — | — | Expiración opcional (NULL = indefinido) |
| *auditoría estándar* | — | — | — | — | — | (soft delete = revocación) |

---

### `security.password_reset_request` — Solicitudes de recuperación de contraseña

| Columna | Tipo | Null | Default | FK | Índice | Descripción |
|---------|------|------|---------|----|--------|-------------|
| `id` | UUID | NO | — | PK | PK | Identificador |
| `user_id` | UUID | NO | — | `security.user(id)` | IDX | Usuario solicitante |
| `token_hash` | TEXT | NO | — | — | — | Hash del token (SHA-256) |
| `expires_at` | TIMESTAMPTZ | NO | `now() + 1 hour` | — | — | Expiración (1h) |
| `is_used` | BOOLEAN | NO | FALSE | — | — | Si ya se consumió |
| `used_at` | TIMESTAMPTZ | SÍ | — | — | — | Cuándo se usó |
| *auditoría estándar* | — | — | — | — | — | (solo `created_at`, `created_by`) |

---

### `security.audit_login` — Auditoría de intentos de autenticación (Append-only)

| Columna | Tipo | Null | Default | FK | Índice | Descripción |
|---------|------|------|---------|----|--------|-------------|
| `id` | UUID | NO | — | PK | PK | Identificador |
| `user_id` | UUID | SÍ | — | `security.user(id)` | IDX | Usuario (NULL si email no existe) |
| `email_attempted` | VARCHAR(255) | NO | — | — | IDX | Email usado en intento |
| `outcome` | VARCHAR(50) | NO | — | — | — | `SUCCESS`, `INVALID_CREDENTIALS`, `ACCOUNT_LOCKED`, `ACCOUNT_SUSPENDED`, `EMAIL_NOT_VERIFIED` |
| `ip_address` | VARCHAR(45) | NO | — | — | — | IPv4 o IPv6 |
| `user_agent` | TEXT | SÍ | — | — | — | User-Agent del cliente |
| `attempted_at` | TIMESTAMPTZ | NO | `now()` | — | **IDX (attempted_at DESC)** | Timestamp intento |
| *solo auditoría* | — | — | — | — | — | `created_at` = `attempted_at`, `created_by` = `user_id` |

> **Retención:** 2 años (política de limpieza job mensual)

---

## Esquema: `parameterization` (Módulo Parameterization)

### `parameterization.event_category` — Categorías de eventos (alto nivel)

| Columna | Tipo | Null | Default | FK | Índice | Descripción |
|---------|------|------|---------|----|--------|-------------|
| `id` | UUID | NO | — | PK | PK | Identificador |
| `code` | VARCHAR(30) | NO | — | — | **UNIQUE** | Código: `SOMNOLENCE`, `DISTRACTION`, `SEATBELT`, `SYSTEM` |
| `name` | VARCHAR(100) | NO | — | — | — | Nombre legible |
| `description` | TEXT | SÍ | — | — | — | Descripción |
| `sort_order` | INTEGER | NO | 0 | — | — | Orden visual |
| *auditoría estándar* | — | — | — | — | — | (sin `deleted_at`) |

---

### `parameterization.severity` — Niveles de severidad

| Columna | Tipo | Null | Default | FK | Índice | Descripción |
|---------|------|------|---------|----|--------|-------------|
| `id` | UUID | NO | — | PK | PK | Identificador |
| `code` | VARCHAR(20) | NO | — | — | **UNIQUE** | Código: `info`, `warning`, `high`, `critical` |
| `name` | VARCHAR(50) | NO | — | — | — | Nombre legible |
| `priority` | SMALLINT | NO | 0 | — | — | Prioridad numérica (1=info..4=critical) |
| *auditoría estándar* | — | — | — | — | — | (sin `deleted_at`) |

---

### `parameterization.media_type` — Tipos de evidencia multimedia

| Columna | Tipo | Null | Default | FK | Índice | Descripción |
|---------|------|------|---------|----|--------|-------------|
| `id` | UUID | NO | — | PK | PK | Identificador |
| `code` | VARCHAR(20) | NO | — | — | **UNIQUE** | Código: `image_jpeg`, `video_mp4` |
| `name` | VARCHAR(50) | NO | — | — | — | Nombre legible |
| `mime_type` | VARCHAR(50) | NO | — | — | — | MIME type: `image/jpeg`, `video/mp4` |
| `max_size_mb` | INTEGER | NO | 10 | — | — | Tamaño máximo permitido |
| *auditoría estándar* | — | — | — | — | — | (sin `deleted_at`) |

---

### `parameterization.sound_pattern` — Patrones de sonido (AS-XX)

| Columna | Tipo | Null | Default | FK | Índice | Descripción |
|---------|------|------|---------|----|--------|-------------|
| `id` | UUID | NO | — | PK | PK | Identificador |
| `code` | VARCHAR(30) | NO | — | — | **UNIQUE** | Código: `AS-01`..`AS-09` |
| `description` | TEXT | NO | — | — | — | Descripción legible |
| `frequency_hz` | INTEGER | NO | — | — | — | Frecuencia en Hz |
| `duration_ms` | INTEGER | NO | — | — | — | Duración por pitido (ms) |
| `repetitions` | SMALLINT | NO | 1 | — | — | Repeticiones (0 = continuo/hasta respuesta) |
| `pattern_type` | VARCHAR(20) | NO | `beep` | — | — | `beep`, `continuous`, `intermittent`, `escalating` |
| `interval_ms` | INTEGER | SÍ | — | — | — | Intervalo entre repeticiones (ms) |
| *auditoría estándar* | — | — | — | — | — | (sin `deleted_at`) |

**Seed (Apéndice 1 SRS):**
| code | description | frequency_hz | duration_ms | repetitions | pattern_type | interval_ms |
|------|-------------|--------------|-------------|-------------|--------------|-------------|
| AS-01 | Somnolencia leve | 800 | 500 | 1 | beep | — |
| AS-02 | Somnolencia moderada | 950 | 400 | 2 | beep | 200 |
| AS-03 | Somnolencia severa | 1100 | 300 | 3 | beep | 200 |
| AS-04 | Estado crítico | 1200 | 2000 | 0 | continuous | — |
| AS-05 | Distracción teléfono | 900 | 700 | 2 | beep | 300 |
| AS-06 | Mirada fuera vía | 950 | 500 | 2 | beep | 200 |
| AS-07 | Cinturón no detectado | 700 | 1000 | 0 | intermittent | 1000 |
| AS-08 | Confirmación inicio | 600 | 300 | 1 | beep | — |
| AS-09 | Error sistema | 1000→700 | 500 | 2 | beep | 200 |

---

### `parameterization.event_type` — Tipos de eventos detectados (configurables)

| Columna | Tipo | Null | Default | FK | Índice | Descripción |
|---------|------|------|---------|----|--------|-------------|
| `id` | UUID | NO | — | PK | PK | Identificador |
| `code` | VARCHAR(30) | NO | — | — | **UNIQUE** | Código: `EV-SOM-01`..`EV-SYS-06` |
| `name` | VARCHAR(100) | NO | — | — | — | Nombre legible |
| `event_category_id` | UUID | NO | — | `parameterization.event_category(id)` | IDX | Categoría |
| `default_severity_id` | UUID | NO | — | `parameterization.severity(id)` | — | Severidad por defecto |
| `default_sound_pattern_id` | UUID | NO | — | `parameterization.sound_pattern(id)` | — | Sonido por defecto |
| `threshold_config` | JSONB | SÍ | `{}` | — | — | Umbrales configurables (ej: `{"blink_rate_max": 25, "eye_closed_min_sec": 2}`) |
| `is_active` | BOOLEAN | NO | TRUE | — | — | Si está vigente |
| *auditoría estándar* | — | — | — | — | — | (sin `deleted_at`) |

**Seed (Apéndice 2 SRS):**
| code | name | event_category | default_severity | default_sound | threshold_config (ejemplo) |
|------|------|----------------|------------------|---------------|---------------------------|
| EV-SOM-01 | Parpadeo anómalo | SOMNOLENCE | info | AS-01 | `{"blink_rate_max": 25, "blink_rate_min": 5, "window_sec": 15}` |
| EV-SOM-02 | Cierre prolongado ojos | SOMNOLENCE | warning | AS-02 | `{"eye_closed_min_sec": 2}` |
| EV-SOM-03 | Bostezo detectado | SOMNOLENCE | warning | AS-02 | `{"yawn_count_min": 2, "window_min": 5}` |
| EV-SOM-04 | Cabeceo/inclinación | SOMNOLENCE | high | AS-03 | `{"head_tilt_deg_min": 20, "duration_sec_min": 3}` |
| EV-SOM-05 | Microsueño detectado | SOMNOLENCE | critical | AS-04 | `{"eye_closed_min_sec": 3, "head_tilt_deg_min": 20, "simultaneous": true}` |
| EV-DIS-01 | Uso teléfono móvil | DISTRACTION | info | AS-05 | `{"detection_confidence_min": 0.7, "duration_sec_min": 2}` |
| EV-DIS-02 | Uso prolongado teléfono | DISTRACTION | high | AS-05 | `{"duration_sec_min": 5}` |
| EV-DIS-03 | Mirada fuera vía | DISTRACTION | info | AS-06 | `{"duration_sec_min": 3}` |
| EV-DIS-04 | Mirada prolongada fuera | DISTRACTION | high | AS-06 | `{"duration_sec_min": 5}` |
| EV-DIS-05 | Movimiento anómalo | DISTRACTION | info | AS-05 | `{"duration_sec_min": 3}` |
| EV-CIN-01 | Cinturón no detectado | SEATBELT | info | AS-07 | `{"no_detection_sec_min": 10}` |
| EV-CIN-02 | Cinturón mal colocado | SEATBELT | info | AS-07 | `{"incorrect_position_sec_min": 10}` |
| EV-SYS-01 | Inicialización exitosa | SYSTEM | info | AS-08 | `{}` |
| EV-SYS-02 | Error cámara/obstrucción | SYSTEM | warning | AS-09 | `{"invalid_image_sec_min": 10}` |
| EV-SYS-03 | Rostro no detectado | SYSTEM | info | AS-09 | `{"no_face_sec_min": 30}` |
| EV-SYS-04 | Conectividad perdida | SYSTEM | info | — | `{}` |
| EV-SYS-05 | Conectividad restaurada | SYSTEM | info | — | `{}` |
| EV-SYS-06 | Almacenamiento casi lleno | SYSTEM | warning | AS-09 | `{"usage_pct_min": 90}` |

---

### `parameterization.status_category` — Categorías de estado (ADR-009)

| Columna | Tipo | Null | Default | FK | Índice | Descripción |
|---------|------|------|---------|----|--------|-------------|
| `code` | VARCHAR(30) | NO | — | PK | PK | `ACTIVE`, `INACTIVE`, `PENDING`, `ERROR`, `ARCHIVED` |
| `name` | VARCHAR(100) | NO | — | — | — | Nombre legible |
| `description` | TEXT | SÍ | — | — | — | Descripción |
| `sort_order` | INTEGER | NO | 0 | — | — | Orden |
| `is_final` | BOOLEAN | NO | FALSE | — | — | Si es estado terminal (no hay salidas) |
| *auditoría estándar* | — | — | — | — | — | (sin `deleted_at`) |

---

### `parameterization.status` — Estados específicos por entidad (ADR-009)

| Columna | Tipo | Null | Default | FK | Índice | Descripción |
|---------|------|------|---------|----|--------|-------------|
| `code` | VARCHAR(50) | NO | — | PK | PK | Código estado: `ACTIVE`, `OFFLINE`, `SYNCHRONIZED`, etc. |
| `status_category` | VARCHAR(30) | NO | — | `parameterization.status_category(code)` | IDX | Categoría padre |
| `name` | VARCHAR(100) | NO | — | — | — | Nombre legible |
| `description` | TEXT | SÍ | — | — | — | Descripción |
| `entity_type` | VARCHAR(50) | NO | — | — | **IDX (entity_type, code)** | `device`, `event`, `user`, `device_config`, `notification` |
| `sort_order` | INTEGER | NO | 0 | — | — | Orden dentro de categoría |
| `is_initial` | BOOLEAN | NO | FALSE | — | — | Estado inicial al crear entidad |
| `is_terminal` | BOOLEAN | NO | FALSE | — | — | Estado terminal (sin salidas) |
| *auditoría estándar* | — | — | — | — | — | (sin `deleted_at`) |

**Seed clave (ver ADR-009 para completo):**
| entity_type | code | status_category | is_initial | is_terminal |
|-------------|------|-----------------|------------|-------------|
| device | REGISTERED | PENDING | TRUE | FALSE |
| device | ASSIGNED | PENDING | FALSE | FALSE |
| device | ACTIVE | ACTIVE | FALSE | FALSE |
| device | OFFLINE | INACTIVE | FALSE | FALSE |
| device | SUSPENDED | INACTIVE | FALSE | FALSE |
| device | RETIRED | ARCHIVED | FALSE | TRUE |
| event | DETECTED | PENDING | TRUE | FALSE |
| event | REGISTERED | PENDING | FALSE | FALSE |
| event | SYNCHRONIZED | ACTIVE | FALSE | FALSE |
| event | ANALYZED | ACTIVE | FALSE | FALSE |
| event | ARCHIVED | ARCHIVED | FALSE | TRUE |
| user | PENDING_VERIFICATION | PENDING | TRUE | FALSE |
| user | ACTIVE | ACTIVE | FALSE | FALSE |
| user | SUSPENDED | INACTIVE | FALSE | FALSE |
| user | SOFT_DELETED | ARCHIVED | FALSE | TRUE |

---

### `parameterization.status_transition` — Transiciones permitidas (ADR-009)

| Columna | Tipo | Null | Default | FK | Índice | Descripción |
|---------|------|------|---------|----|--------|-------------|
| `from_status` | VARCHAR(50) | NO | — | `parameterization.status(code)` | PK | Estado origen |
| `to_status` | VARCHAR(50) | NO | — | `parameterization.status(code)` | PK | Estado destino |
| `allowed_roles` | VARCHAR(100)[] | SÍ | NULL | — | — | Roles permitidos (NULL = system/any) |
| `description` | TEXT | SÍ | — | — | — | Descripción transición |
| `created_at` | TIMESTAMPTZ | NO | `now()` | — | — | Timestamp creación |
| *append-only* | — | — | — | — | — | Solo INSERT |

**Ejemplos seed (ver ADR-009):**
| from_status | to_status | allowed_roles | description |
|-------------|-----------|---------------|-------------|
| REGISTERED | ASSIGNED | `{user}` | Usuario asocia device |
| ASSIGNED | ACTIVE | `{system}` | Primer heartbeat |
| ACTIVE | OFFLINE | `{system}` | Sin heartbeat > 5 min |
| OFFLINE | ACTIVE | `{system}` | Heartbeat recibido |
| ACTIVE | SUSPENDED | `{admin}` | Admin suspende |
| SUSPENDED | ACTIVE | `{admin}` | Admin reactiva |
| DETECTED | REGISTERED | `{system}` | Persistido en buffer local |
| REGISTERED | SYNCHRONIZED | `{system}` | ACK recibido de API |

---

## Esquema: `device_management` (Módulo Device Management)

### `device_management.device` — Dispositivos físicos

| Columna | Tipo | Null | Default | FK | Índice | Descripción | RN |
|---------|------|------|---------|----|--------|-------------|-----|
| `id` | UUID | NO | — | PK | PK | Identificador | |
| `serial_number` | VARCHAR(100) | NO | — | — | **UNIQUE** | Número de serie único | RN-03 |
| `api_key_hash` | TEXT | NO | — | — | — | HMAC-SHA256 de API Key | RN-03 |
| `firmware_version` | VARCHAR(50) | NO | — | — | — | Versión firmware instalada | |
| `status` | VARCHAR(50) | NO | `REGISTERED` | `parameterization.status(code)` | IDX | Estado dispositivo | ADR-009 |
| `status_category` | VARCHAR(30) | NO | `PENDING` | `parameterization.status_category(code)` | — | Categoría derivada | ADR-009 |
| `last_heartbeat_at` | TIMESTAMPTZ | SÍ | — | — | IDX | Último heartbeat recibido | |
| `last_sync_at` | TIMESTAMPTZ | SÍ | — | — | — | Última sincronización exitosa | |
| `last_config_pull_at` | TIMESTAMPTZ | SÍ | — | — | — | Última descarga config | |
| `last_seen_ip` | VARCHAR(45) | SÍ | — | — | — | IP origen último request | |
| *auditoría estándar* | — | — | — | — | — | `created_at`, `created_by`, `updated_at`, `updated_by`, `deleted_at`, `deleted_by`, `version` | |

**Índices:** `idx_device_status_active (status) WHERE deleted_at IS NULL`, `idx_device_heartbeat (last_heartbeat_at)`

**Reglas:** RN-03 (device pertenece a cuenta asignada, API key válida para eventos), RN-10 (config se descarga y aplica)

---

### `device_management.device_assignment` — Historial de asignación Device ↔ User

| Columna | Tipo | Null | Default | FK | Índice | Descripción |
|---------|------|------|---------|----|--------|-------------|
| `id` | UUID | NO | — | PK | PK | Identificador |
| `device_id` | UUID | NO | — | `device_management.device(id)` | **UNIQUE (device_id) WHERE unassigned_at IS NULL** | Device |
| `user_id` | UUID | NO | — | `security.user(id)` | IDX | Usuario |
| `assigned_at` | TIMESTAMPTZ | NO | `now()` | — | — | Cuándo se asignó |
| `unassigned_at` | TIMESTAMPTZ | SÍ | — | — | — | Cuándo se desasignó (NULL = actual) |
| `assigned_by` | UUID | NO | — | `security.user(id)` | — | Quién asignó (admin o user) |
| *auditoría estándar* | — | — | — | — | — | (soft delete = desasignación) |

> **Regla:** Un device solo tiene UNA asignación activa (`unassigned_at IS NULL`)

---

### `device_management.device_config` — Configuración remota del dispositivo (JSONB)

| Columna | Tipo | Null | Default | FK | Índice | Descripción |
|---------|------|------|---------|----|--------|-------------|
| `id` | UUID | NO | — | PK | PK | Identificador |
| `device_id` | UUID | NO | — | `device_management.device(id)` | **UNIQUE** | Device (1 config por device) |
| `configuration` | JSONB | NO | `'{}'` | — | — | Config completa (umbrales, sound_pattern, volumen, sync_interval) |
| `status` | VARCHAR(50) | NO | `DRAFT` | `parameterization.status(code)` | IDX | Estado config: DRAFT, PUBLISHED, DEPRECATED | ADR-009 |
| `status_category` | VARCHAR(30) | NO | `PENDING` | `parameterization.status_category(code)` | — | Categoría | ADR-009 |
| `version` | INTEGER | NO | 1 | — | — | Versión config (incrementa en cada publish) |
| `published_at` | TIMESTAMPTZ | SÍ | — | — | — | Cuándo se publicó |
| *auditoría estándar* | — | — | — | — | — | `created_at`, `created_by`, `updated_at`, `updated_by`, `deleted_at`, `deleted_by`, `version` |

**Ejemplo `configuration` JSONB:**
```json
{
  "thresholds": {
    "blink_rate_max": 25,
    "eye_closed_min_sec": 2,
    "head_tilt_deg_min": 20
  },
  "sound_patterns": {
    "EV-SOM-01": "AS-01",
    "EV-SOM-02": "AS-02",
    "EV-SYS-02": "AS-09"
  },
  "volume_pct": 80,
  "sync_interval_seconds": 30,
  "retention_days": 7
}
```

---

### `device_management.device_config_history` — Historial de cambios de config (Append-only)

| Columna | Tipo | Null | Default | FK | Índice | Descripción |
|---------|------|------|---------|----|--------|-------------|
| `id` | UUID | NO | — | PK | PK | Identificador |
| `device_config_id` | UUID | NO | — | `device_management.device_config(id)` | IDX | Config padre |
| `configuration` | JSONB | NO | — | — | — | Snapshot completo en ese momento |
| `changed_by` | UUID | NO | — | `security.user(id)` | — | Admin que cambió |
| `change_reason` | VARCHAR(200) | SÍ | — | — | — | Motivo del cambio |
| `created_at` | TIMESTAMPTZ | NO | `now()` | — | **IDX (created_at DESC)** | Timestamp |
| *append-only* | — | — | — | — | — | Solo INSERT |

---

## Esquema: `telemetry_service` (Módulo Telemetry Service)

### `telemetry_service.event` — Eventos detectados (alta escritura)

| Columna | Tipo | Null | Default | FK | Índice | Descripción | RN |
|---------|------|------|---------|----|--------|-------------|-----|
| `id` | UUID | NO | — | PK | PK | **event_id** — generado en device (UUID v7) | RN-08 |
| `device_id` | UUID | NO | — | `device_management.device(id)` | **IDX (device_id, occurred_at DESC)** | Device origen | RN-03 |
| `event_type_id` | UUID | NO | — | `parameterization.event_type(id)` | IDX | Tipo de evento | RN-04 |
| `occurred_at` | TIMESTAMPTZ | NO | — | — | **IDX (occurred_at DESC)** | Timestamp evento (UTC, device) | |
| `severity_id` | UUID | NO | — | `parameterization.severity(id)` | IDX | Severidad (puede diferir de default) | |
| `sound_pattern_id` | UUID | SÍ | — | `parameterization.sound_pattern(id)` | — | Sonido reproducido | |
| `is_offline_sync` | BOOLEAN | NO | FALSE | — | — | TRUE si generado offline | RN-08 |
| `metadata` | JSONB | SÍ | `'{}'` | — | — | Datos extra (confianza, coordenadas, etc.) | |
| `status` | VARCHAR(50) | NO | `DETECTED` | `parameterization.status(code)` | IDX | Estado: DETECTED, REGISTERED, SYNCHRONIZED, ANALYZED, ARCHIVED | ADR-009 |
| `status_category` | VARCHAR(30) | NO | `PENDING` | `parameterization.status_category(code)` | — | Categoría | ADR-009 |
| *auditoría estándar* | — | — | — | — | — | `created_at`, `created_by` (device_id), `updated_at`, `updated_by`, `deleted_at`, `deleted_by`, `version` | |

**Índices compuestos críticos:**
- `idx_event_device_time (device_id, occurred_at DESC)` — timeline por device
- `idx_event_type_severity (event_type_id, severity_id)` — filtros dashboard
- `idx_event_status_sync (status) WHERE status IN ('REGISTERED','SYNCHRONIZED')` — sync status

**Reglas:** RN-04 (clasificación via catálogos), RN-05 (alert_log asociado), RN-06 (evidencia + retención), RN-08 (idempotencia via `id` UUID v7)

---

### `telemetry_service.evidence` — Evidencia multimedia (frames, video)

| Columna | Tipo | Null | Default | FK | Índice | Descripción |
|---------|------|------|---------|----|--------|-------------|
| `id` | UUID | NO | — | PK | PK | Identificador evidencia |
| `event_id` | UUID | NO | — | `telemetry_service.event(id)` | **UNIQUE** | Evento asociado (1 evidencia por evento en MVP) |
| `media_type_id` | UUID | NO | — | `parameterization.media_type(id)` | — | Tipo: image_jpeg, video_mp4 |
| `minio_key` | VARCHAR(500) | NO | — | — | — | Key en MinIO: `{device_id}/{YYYY}/{MM}/{DD}/{event_id}.jpg` | ADR-006 |
| `size_bytes` | BIGINT | NO | — | — | — | Tamaño archivo |
| `checksum_sha256` | VARCHAR(64) | NO | — | — | — | Integridad |
| `created_at` | TIMESTAMPTZ | NO | `now()` | — | — | Timestamp subida |
| `created_by` | UUID | NO | — | `device_management.device(id)` | — | Device que subió |
| *auditoría estándar* | — | — | — | — | — | (sin `deleted_at` — append-only por retención) |

> **Retención:** 90 días (eventos normales) / 5 años (severity=critical) — ADR-006 ILM

---

### `telemetry_service.alert_log` — Registro histórico de alarmas (Append-only)

| Columna | Tipo | Null | Default | FK | Índice | Descripción | RN |
|---------|------|------|---------|----|--------|-------------|-----|
| `id` | UUID | NO | — | PK | PK | Identificador | |
| `event_id` | UUID | NO | — | `telemetry_service.event(id)` | IDX | Evento origen | RN-05 |
| `sound_pattern_id` | UUID | NO | — | `parameterization.sound_pattern(id)` | — | Patrón reproducido (AS-XX) | RN-05 |
| `severity_id` | UUID | NO | — | `parameterization.severity(id)` | IDX | Severidad registrada | |
| `triggered_at` | TIMESTAMPTZ | NO | — | — | **IDX (triggered_at DESC)** | Timestamp alarma | |
| `device_id` | UUID | NO | — | `device_management.device(id)` | IDX | Device (denormalizado para queries) | |
| *append-only* | — | — | — | — | — | Solo `created_at` = `triggered_at`, `created_by` = `device_id` | |

> **Retención:** 5 años — RN-05, ADR-006

---

## Esquema: `monitoring` (Módulo Monitoring)

### `monitoring.notification` — Notificaciones a usuarios (push, email, in-app)

| Columna | Tipo | Null | Default | FK | Índice | Descripción | RN |
|---------|------|------|---------|----|--------|-------------|-----|
| `id` | UUID | NO | — | PK | PK | Identificador | |
| `user_id` | UUID | NO | — | `security.user(id)` | **IDX (user_id, sent_at DESC)** | Usuario destinatario | RN-07 |
| `alert_log_id` | UUID | NO | — | `telemetry_service.alert_log(id)` | IDX | Alarma origen | RN-07 |
| `title` | VARCHAR(200) | NO | — | — | — | Título notificación | |
| `message` | TEXT | NO | — | — | — | Cuerpo mensaje | |
| `channel` | VARCHAR(30) | NO | — | — | — | `push`, `email`, `in_app` | |
| `status` | VARCHAR(50) | NO | `PENDING` | `parameterization.status(code)` | IDX | Estado: PENDING, SENT, DELIVERED, READ, FAILED | ADR-009 |
| `status_category` | VARCHAR(30) | NO | `PENDING` | `parameterization.status_category(code)` | — | Categoría | ADR-009 |
| `sent_at` | TIMESTAMPTZ | SÍ | — | — | — | Cuándo se envió | |
| `delivered_at` | TIMESTAMPTZ | SÍ | — | — | — | Cuándo se entregó (push) | |
| `read_at` | TIMESTAMPTZ | SÍ | — | — | — | Cuándo se leyó (in_app) | |
| `retry_count` | SMALLINT | NO | 0 | — | — | Reintentos | |
| `error_message` | TEXT | SÍ | — | — | — | Error si FAILED | |
| *auditoría estándar* | — | — | — | — | — | `created_at`, `created_by`, `updated_at`, `updated_by`, `deleted_at`, `deleted_by`, `version` | |

**Reglas:** RN-07 (notificaciones críticas auto, propietario device), plantillas por event_type+severity

---

### `monitoring.notification_delivery` — Tracking de entrega (Append-only)

| Columna | Tipo | Null | Default | FK | Índice | Descripción |
|---------|------|------|---------|----|--------|-------------|
| `id` | UUID | NO | — | PK | PK | Identificador |
| `notification_id` | UUID | NO | — | `monitoring.notification(id)` | IDX | Notificación |
| `channel` | VARCHAR(30) | NO | — | — | — | `push`, `email`, `in_app` |
| `provider` | VARCHAR(50) | SÍ | — | — | — | `fcm`, `apns`, `sendgrid`, `internal` |
| `provider_message_id` | VARCHAR(200) | SÍ | — | — | — | ID externo (FCM message_id, etc.) |
| `status` | VARCHAR(50) | NO | — | — | — | `SENT`, `DELIVERED`, `READ`, `FAILED`, `BOUNCED` |
| `occurred_at` | TIMESTAMPTZ | NO | `now()` | — | **IDX (occurred_at DESC)** | Timestamp evento |
| `error_code` | VARCHAR(50) | SÍ | — | — | — | Código error proveedor |
| `error_message` | TEXT | SÍ | — | — | — | Detalle error |
| *append-only* | — | — | — | — | — | Solo `created_at` = `occurred_at` |

> **Retención:** 1 año — tracking delivery

---

## Esquema: `analytics` (Módulo Analytics)

> **Sin tablas transaccionales propias.** Solo **vistas materializadas** y **proyecciones** optimizadas para consultas.

### `analytics.v_event_timeline` — Vista: Timeline de eventos (denormalizada)

```sql
CREATE MATERIALIZED VIEW analytics.v_event_timeline AS
SELECT
  e.id AS event_id,
  e.device_id,
  d.serial_number,
  e.event_type_id,
  et.code AS event_type_code,
  et.name AS event_type_name,
  ec.code AS event_category_code,
  e.severity_id,
  s.code AS severity_code,
  s.priority AS severity_priority,
  e.occurred_at,
  e.is_offline_sync,
  e.status,
  e.status_category,
  ev.minio_key AS evidence_key,
  ev.media_type_id,
  mt.code AS media_type_code
FROM telemetry_service.event e
JOIN device_management.device d ON d.id = e.device_id
JOIN parameterization.event_type et ON et.id = e.event_type_id
JOIN parameterization.event_category ec ON ec.id = et.event_category_id
JOIN parameterization.severity s ON s.id = e.severity_id
LEFT JOIN telemetry_service.evidence ev ON ev.event_id = e.id
LEFT JOIN parameterization.media_type mt ON mt.id = ev.media_type_id
WHERE e.deleted_at IS NULL AND d.deleted_at IS NULL;
```
**Refresh:** Trigger on `event` insert/update + pg_cron cada 5 min
**Índices:** `CREATE INDEX ON analytics.v_event_timeline (device_id, occurred_at DESC)`

---

### `analytics.v_metrics_daily` — Vista: Métricas diarias pre-agregadas

```sql
CREATE MATERIALIZED VIEW analytics.v_metrics_daily AS
SELECT
  DATE_TRUNC('day', e.occurred_at AT TIME ZONE 'America/Bogota') AS metric_date,
  e.device_id,
  d.user_id,
  e.event_type_id,
  et.event_category_id,
  e.severity_id,
  COUNT(*) AS event_count,
  COUNT(*) FILTER (WHERE e.severity_id = (SELECT id FROM parameterization.severity WHERE code='critical')) AS critical_count,
  COUNT(*) FILTER (WHERE e.severity_id = (SELECT id FROM parameterization.severity WHERE code='high')) AS high_count,
  MIN(e.occurred_at) AS first_event_at,
  MAX(e.occurred_at) AS last_event_at
FROM telemetry_service.event e
JOIN device_management.device d ON d.id = e.device_id
JOIN parameterization.event_type et ON et.id = e.event_type_id
WHERE e.deleted_at IS NULL AND d.deleted_at IS NULL
GROUP BY 1,2,3,4,5,6;
```
**Refresh:** pg_cron cada hora (agregación incremental)

---

## Tablas de Auditoría de Estado (ADR-009 - Triggers automáticos)

Para cada entidad con `status` + `status_category`, se crea tabla `_status_audit`:

| Tabla Auditoría | Entidad Origen | Esquema |
|-----------------|----------------|---------|
| `device_status_audit` | `device_management.device` | `device_management` |
| `event_status_audit` | `telemetry_service.event` | `telemetry_service` |
| `user_status_audit` | `security.user` | `security` |
| `device_config_status_audit` | `device_management.device_config` | `device_management` |
| `notification_status_audit` | `monitoring.notification` | `monitoring` |

**Estructura común:**
```sql
CREATE TABLE {schema}.{entity}_status_audit (
    id              BIGSERIAL PRIMARY KEY,
    {entity}_id     UUID NOT NULL REFERENCES {schema}.{entity}(id),
    from_status     VARCHAR(50),
    to_status       VARCHAR(50) NOT NULL,
    from_category   VARCHAR(30),
    to_category     VARCHAR(30) NOT NULL,
    changed_by      UUID,  -- user_id o device_id (NULL = system)
    changed_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    context_json    JSONB  -- metadata extra
);
CREATE INDEX ON {schema}.{entity}_status_audit ({entity}_id, changed_at DESC);
```

---

## Resumen de Convenciones por Tipo de Tabla

| Tipo de Tabla | Ejemplos | Auditoria | Soft Delete | Version | Estados |
|---------------|----------|-----------|-------------|---------|---------|
| **Catálogo inmutable** | `role`, `module`, `feature`, `event_category`, `severity`, `media_type`, `sound_pattern`, `status_category`, `status` | `created_at`, `created_by` | NO | NO | NO |
| **Configuración versionada** | `device_config`, `event_type` | Completa | SÍ | SÍ | `status_category` + `status` |
| **Transaccional principal** | `user`, `device`, `device_assignment`, `event`, `notification` | Completa | SÍ | SÍ | `status_category` + `status` |
| **Append-only (eventos)** | `audit_login`, `alert_log`, `password_reset_request`, `evidence`, `notification_delivery`, `device_config_history` | `created_at`/`created_by` | NO | NO | NO |
| **Histórico de estado** | `*_status_audit` | Append-only | NO | NO | N/A |

---

## Referencias Cruzadas Rápidas

| Necesitas | Documento |
|-----------|-----------|
| Reglas de negocio (RN-*) | [entities-and-rules.md](../02-domain/entities-and-rules.md) |
| Estrategia BD (esquemas, Liquibase) | [ADR-004](../05-architecture/decisions/records/ADR-004-database-strategy.md) |
| Estados parametrizados + auditoría | [ADR-009](../05-architecture/decisions/records/ADR-009-status-parametrized-audit.md) |
| Auth (JWT, API Keys) | [ADR-001](../05-architecture/decisions/records/ADR-001-backend-java-spring-boot.md) |
| Offline-first device | [ADR-005](../05-architecture/decisions/records/ADR-005-offline-first-device.md) |
| MinIO evidence storage | [ADR-006](../05-architecture/decisions/records/ADR-006-minio-evidence-storage.md) |
| Convenciones modelado Liquibase | [modeling-conventions.md](../06-data-architecture/modeling-conventions.md) |
| Estrategia migraciones | [migration-strategy.md](../06-data-architecture/migration-strategy.md) |
| Catálogo módulos | [module-catalog.md](../09-modules/module-catalog.md) |

---

## Próximos Pasos

1. **Validar** diccionario con DBA + Backend Lead (revisión 30 min)
2. **Generar changelogs Liquibase** iniciales por módulo (basados en este diccionario)
3. **Crear `modeling-conventions.md`** con reglas formales (naming, FKs, índices, JSONB, triggers)
4. **Crear `migration-strategy.md`** (orden ejecución, rollback, seed data, CI integration)
5. **Actualizar `module-template/data-model.md`** para cada módulo use este diccionario como fuente