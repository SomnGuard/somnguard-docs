<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Preocupaciones Transversales (Cross-Cutting Concerns)

**Estado:** Borrador
**Fecha:** 2026-08-22

</div>

</div>

> **Reglas obligatorias para TODOS los módulos/repos** (API, DB, DEVICE, PORTAL, APP).
> Basado en: [SRS](../04-requeriments/01-srs/) (RNF-*), [functional.md](./functional.md) (RF-*),
> [architecture-document.md](../05-architecture/architecture-document.md),
> [pattern-guide.md](../05-architecture/pattern-guide.md) (Hexagonal + DDD),
> Guía ADR-004 (estados parametrizados + auditoría).

---

## 1. Autenticación y Autorización

### 1.1 Usuarios (Portal, App, Admin)
| Aspecto | Especificación |
|---------|----------------|
| **Protocolo** | JWT RS256 (asymmetric) — ADR-001 |
| **Access Token** | 15 min, `Authorization: Bearer <jwt>` |
| **Refresh Token** | 7 días, rotación en cada uso, hash en BD, blocklist en logout |
| **Claims obligatorios** | `sub` (user_id UUID), `roles` (array), `features` (array), `exp`, `iat`, `jti` |
| **JWKS Endpoint** | `GET /.well-known/jwks.json` para verificación distribuida |
| **Rate Limit Auth** | 5 req/min en `/auth/*` por IP |

### 1.2 Dispositivos (Edge → API)
| Aspecto | Especificación |
|---------|----------------|
| **Credencial** | API Key (opaque, 32 bytes base64url) + `device_id` (UUID) |
| **Header** | `X-Device-ID: <uuid>` + `X-API-Key: <key>` |
| **Validación** | HMAC-SHA256 de `api_key` comparado con `device.api_key_hash` |
| **Rate Limit Device** | 100 req/min por API Key (token bucket) |
| **Scope** | Solo endpoints `/telemetry/*` y `/devices/{id}/config` |

### 1.3 RBAC (Role-Based Access Control)
| Aspecto | Especificación |
|---------|----------------|
| **Modelo** | `role` ↔ `feature` (N:M via `role_feature`) → `user_role` (N:M) |
| **Roles base** | `admin` (todas las features), `user` (features propias) |
| **Enforcement** | Middleware en cada endpoint: `@RequireFeature("feature.code")` |
| **Denegación** | 403 Forbidden con `{code: "FORBIDDEN", message: "Feature X required"}` |

---

## 2. Auditoría y Trazabilidad de Datos

### 2.1 Campos Estándar (por tipo de tabla)

#### Transaccionales con UPDATE concurrente (user, device, device_config, notification, device_assignment, event_type)
| Campo | Tipo | Descripción |
|-------|------|-------------|
| `id` | UUID | PK, generado por app (UUID v7) |
| `created_at` | TIMESTAMPTZ | UTC, `now()` en insert |
| `created_by` | UUID | User ID o Device ID (nullable para system) |
| `updated_at` | TIMESTAMPTZ | UTC, `now()` en update |
| `updated_by` | UUID | User ID o Device ID |
| `deleted_at` | TIMESTAMPTZ | **Soft delete** — NULL = activo |
| `deleted_by` | UUID | User ID o Device ID que eliminó |
| `version` | INTEGER | Optimistic locking (empezar en 1) |
| `is_active` | BOOLEAN | **Campo de soft delete** — por defecto TRUE. FALSE = inactivo. **Es el campo base en todas las tablas.** |
| `status` | VARCHAR(50) | **Solo en `parameterization.event_type`** (FK `parameterization.status`). En las demás tablas NULL = no aplica. |
| `status_category` | VARCHAR(30) | **Solo en `parameterization.event_type`** (FK `parameterization.status_category`). En las demás tablas NULL = no aplica. |

#### Transaccionales solo INSERT / append-only (event, evidence, alert_log, audit_login, password_reset_request, device_config_history)
| Campo | Tipo | Descripción |
|-------|------|-------------|
| `id` | UUID | PK, generado por app (UUID v7) |
| `created_at` | TIMESTAMPTZ | UTC, `now()` en insert |
| `created_by` | UUID | User ID o Device ID (nullable para system) |
| `status` | VARCHAR(50) | Estado de negocio — **solo event** |
| `status_category` | VARCHAR(30) | Categoría — **solo event** |

#### Catálogos inmutables (role, module, feature, event_category, severity, media_type, sound_pattern, status_category, status, status_transition)
| Campo | Tipo | Descripción |
|-------|------|-------------|
| `id` / `code` | UUID / VARCHAR | PK |
| `created_at` | TIMESTAMPTZ | UTC, `now()` en insert |
| `created_by` | UUID | User ID creador (seed = SYSTEM_ACTOR_ID) |
| `updated_at` | TIMESTAMPTZ | UTC, `now()` en update |
| `updated_by` | UUID | User ID modificador |

> **Nota:** `SYSTEM_ACTOR_ID = 00000000-0000-0000-0000-000000000000` para seeds y acciones automáticas.

### 2.2 Reglas de Auditoría
| Regla | Descripción |
|-------|-------------|
| **Append-only** | Tablas de auditoría (`audit_login`, `alert_log`, `device_config_history`) solo INSERT |
| **Soft delete obligatorio** | DELETE lógico vía `deleted_at`; nunca `DELETE` físico en app |
| **Cascada suave** | Al borrar parent → `deleted_at` en children (no FK cascade delete) |
| **Histórico de cambios** | Tablas `_history` (trigger PG) para entidades críticas: `user`, `device`, `event_type`, `device_config` |
| **Estados parametrizados** | Solo 5 tablas core usan `status` + `status_category` (ADR-009): user, device, event, device_config, notification |

### 2.3 Tablas de Auditoría Base
| Tabla | Propósito | Retención |
|-------|-----------|-----------|
| `audit_login` | Login attempts (éxito/fallo, IP, UA) | 2 años |
| `alert_log` | Alertas emitidas (AS-XX, event_id, severity) | 5 años |
| `device_config_history` | Cambios de config remota | 3 años |

---

## 3. Observabilidad (Logs, Métricas, Trazas)

### 3.1 Logs Estructurados (JSON Lines)
```json
{
  "timestamp": "2026-08-22T14:30:00.123Z",
  "level": "INFO",
  "service": "somnguard-api",
  "module": "telemetry_service",
  "trace_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "span_id": "1234567890abcdef",
  "user_id": "optional-uuid",
  "device_id": "optional-uuid",
  "event": "telemetry.events.ingested",
  "message": "Batch of 23 events ingested for device abc-123",
  "details": {"count": 23, "duration_ms": 45}
}
```
**Campos obligatorios:** `timestamp` (ISO8601 UTC), `level`, `service`, `module`, `trace_id`, `span_id`, `event`, `message`.
**Campos contextuales:** `user_id`, `device_id`, `details` (objeto libre).

### 3.2 Métricas RED (Rate, Errors, Duration) por Endpoint
| Métrica | Tipo | Labels |
|---------|------|--------|
| `http_requests_total` | Counter | `method`, `path`, `status`, `module` |
| `http_request_duration_seconds` | Histogram | `method`, `path`, `module` |
| `http_request_size_bytes` | Histogram | `method`, `path` |
| `http_response_size_bytes` | Histogram | `method`, `path` |

**Exposición:** `/actuator/prometheus` (Spring Boot Micrometer) en API.

### 3.3 Trazas Distribuidas (OpenTelemetry)
| Aspecto | Especificación |
|---------|----------------|
| **Propagación** | `traceparent` (W3C) + `X-Request-ID` (UUID v4) en headers HTTP |
| **Sampling** | 10% base; 100% para errores (status >= 500) y endpoints críticos (`/telemetry/events`, `/auth/*`) |
| **Exportador** | OTLP gRPC → OTel Collector → Tempo (traces) |
| **Instrumentación** | Auto-instrumentation Java agent + spans manuales en use cases clave |

### 3.4 Dashboards Grafana (Mínimos por Módulo)
| Dashboard | Paneles Clave |
|-----------|---------------|
| **API Global** | RED global, JVM (heap, GC, threads), DB pool, Redis pool |
| **Auth** | Login success/failed rate, token refresh rate, 401/403 rate |
| **Telemetry** | Events ingested/sec, batch size, MinIO upload latency, duplicate rate |
| **Device Sync** | Devices online/offline, sync success/failed, buffer age, config pull rate |
| **Monitoring** | Notifications sent/delivered/failed, retry queue depth |
| **Analytics** | Query latency p50/p95, report generation time, AI summary tokens |

---

## 4. Idempotencia y Consistencia en Sincronización

### 4.1 Clave de Idempotencia
| Contexto | Clave | Almacenamiento |
|----------|-------|----------------|
| Device → API (eventos) | `event_id` (UUID v7, generado en device) | Índice único `telemetry.event(event_id)` |
| Device → API (evidencia) | `evidence_id` (UUID) + `event_id` | Índice único `evidence(evidence_id)` |
| API → Push (notificaciones) | `notification_id` (UUID) + `device_id` + `event_id` | Tabla `notification` (cols: `sent_at`, `delivered_at`, `read_at`, `retry_count`, `error_message`) |

### 4.2 Patrón de Reintentos (Device Side)
```python
# Backoff exponencial con jitter
base_delay = 60  # 1 min
max_delay = 3600  # 1 hora
max_retries = 10
delay = min(base_delay * (2 ** attempt) + random(0, 30), max_delay)
```

### 4.3 Limpieza de Buffer (Device)
- **ACK recibido (201):** Borrar `event_id` confirmados de `pending_events`
- **Error 409 (duplicado):** Borrar duplicado local (ya persistido en server)
- **Error 4xx/5xx:** Incrementar `retries`, reprogramar con backoff
- **Retención máxima:** 7 días para fallidos persistentes; luego AS-09 + log

---

## 5. Estados Parametrizados (ADR-004 Guía Adaptado)

### 5.1 Modelo Unificado de Estados
Toda entidad con ciclo de vida usa **dos campos**:
| Campo | Tipo | Descripción |
|-------|------|-------------|
| `status_category` | VARCHAR(30) | Grupo semántico: `ACTIVE`, `INACTIVE`, `PENDING`, `ERROR`, `ARCHIVED` |
| `status` | VARCHAR(50) | Valor específico dentro de la categoría |

### 5.2 Catálogo Base (Parameterization → `status_category`, `status`)
| Entidad | status_category | status (ejemplos) |
|---------|-----------------|-------------------|
| `device` | `REGISTERED`, `ASSIGNED`, `ACTIVE`, `OFFLINE`, `SUSPENDED`, `RETIRED` | `REGISTERED`, `ASSIGNED`, `ACTIVE`, `OFFLINE`, `SUSPENDED`, `RETIRED` |
| `event` | `DETECTED`, `REGISTERED`, `SYNCHRONIZED`, `ANALYZED`, `ARCHIVED` | `DETECTED`, `REGISTERED`, `SYNCHRONIZED`, `ANALYZED`, `ARCHIVED` |
| `user` | `PENDING`, `ACTIVE`, `SUSPENDED`, `DELETED` | `PENDING_VERIFICATION`, `ACTIVE`, `SUSPENDED`, `SOFT_DELETED` |
| `device_config` | `DRAFT`, `PUBLISHED`, `DEPRECATED` | `DRAFT`, `PUBLISHED`, `DEPRECATED` |

### 5.3 Reglas de Transición
- Definidas en `status_transition` (from_category, from_status, to_category, to_status, allowed_roles)
- Validación en domain service antes de persistir cambio de estado
- Evento de dominio `EntityStatusChanged` publicado en transición

---

## 6. Zona Horaria y Temporalidad

| Regla | Especificación |
|-------|----------------|
| **Almacenamiento BD** | **UTC exclusivamente** (`TIMESTAMPTZ` → normaliza a UTC) |
| **Display usuario** | Zona horaria del usuario (default `America/Bogota`) |
| **Device timestamps** | `occurred_at` en UTC (device sincroniza NTP al arrancar) |
| **Agregaciones analytics** | Bucketing por día/semana/mes en UTC; conversión a local solo en presentación |
| **Scheduling jobs** | Cron en UTC (ej. refresh vistas 0 */5 * * * *) |

---

## 7. Manejo de Errores (RFC 7807 Problem Details)

### 7.1 Formato Unificado
```json
{
  "type": "https://somnguard.com/errors/VALIDATION_ERROR",
  "title": "Validation failed",
  "status": 422,
  "detail": "Field 'email' must be a valid email address",
  "instance": "/api/v1/auth/register",
  "trace_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "errors": [
    {"field": "email", "code": "INVALID_FORMAT", "message": "Invalid email format"}
  ]
}
```

### 7.2 Códigos de Error Estándar
| HTTP Status | Type (URI) | Código | Cuándo |
|-------------|------------|--------|--------|
| 400 | `BAD_REQUEST` | `MALFORMED_JSON` | JSON inválido |
| 401 | `UNAUTHORIZED` | `INVALID_TOKEN` | JWT expirado/firmado mal |
| 401 | `UNAUTHORIZED` | `INVALID_API_KEY` | Device key inválida |
| 403 | `FORBIDDEN` | `FEATURE_REQUIRED` | Falta feature en role |
| 404 | `NOT_FOUND` | `RESOURCE_NOT_FOUND` | Entidad no existe |
| 409 | `CONFLICT` | `DUPLICATE_KEY` | `event_id` duplicado, email único |
| 409 | `CONFLICT` | `STATE_TRANSITION_INVALID` | Transición estado no permitida |
| 422 | `VALIDATION_ERROR` | `FIELD_VALIDATION` | @Valid falló |
| 429 | `RATE_LIMITED` | `TOO_MANY_REQUESTS` | Rate limit excedido |
| 500 | `INTERNAL_ERROR` | `UNEXPECTED` | Bug no controlado |
| 503 | `SERVICE_UNAVAILABLE` | `DOWNSTREAM_FAILED` | MinIO/Redis/BD caído |

### 7.3 Logging de Errores
- **SIEMPRE** loguear con `level: ERROR` + `trace_id` + `details` (stack trace sanitizado)
- **NUNCA** loguear: passwords, tokens, API keys, PII sensible
- **Correlación:** `trace_id` propagado en toda la cadena (device→API→DB→MinIO)

---

## 8. Paginación, Filtrado y Ordenación (API REST)

### 8.1 Paginación
| Parámetro | Default | Máximo | Descripción |
|-----------|---------|--------|-------------|
| `page` | 1 | — | 1-based |
| `page_size` | 20 | 100 | Items por página |

**Respuesta:**
```json
{
  "data": [...],
  "meta": {
    "total": 1234,
    "page": 1,
    "page_size": 20,
    "total_pages": 62
  }
}
```

### 8.2 Filtrado
```
GET /telemetry/events?filter[device_id]=uuid&filter[severity]=critical&filter[occurred_at][$gte]=2026-01-01
```
**Operadores:** `$eq`, `$ne`, `$gt`, `$gte`, `$lt`, `$lte`, `$in`, `$nin`, `$like`, `$ilike`

### 8.3 Ordenación
```
GET /telemetry/events?sort=-occurred_at,event_type_id
```
**Prefijo `-` = DESC; sin prefijo = ASC. Múltiples separados por coma.**

---

## 9. Versionado de API

| Regla | Especificación |
|-------|----------------|
| **Path versioning** | `/api/v1/` obligatorio en todas las rutas |
| **Deprecación** | Header `Sunset: Sat, 01 Jan 2027 00:00:00 GMT` + `Deprecation: true` |
| **Compatibilidad** | No breaking changes en v1 (añadir campos opcionales OK, quitar/renombrar NO) |
| **Nueva versión** | `/api/v2/` con migración documentada; v1 mantiene 12 meses tras v2 GA |

---

## 10. Configuración por Ambiente (12-Factor)

| Variable | Descripción | Ejemplo Local |
|----------|-------------|---------------|
| `SPRING_PROFILES_ACTIVE` | Perfil Spring | `docker` |
| `JWT_PRIVATE_KEY_PATH` | Ruta clave privada RS256 | `/keys/private.pem` |
| `JWT_PUBLIC_KEY_PATH` | Ruta clave pública | `/keys/public.pem` |
| `MINIO_ENDPOINT` | Endpoint MinIO | `http://minio:9000` |
| `MINIO_BUCKET` | Bucket evidencias | `somnguard-evidence` |
| `REDIS_HOST` / `REDIS_PORT` | Redis cache | `redis:6379` |
| `OTEL_EXPORTER_OTLP_ENDPOINT` | OTel Collector | `http://otel-collector:4317` |
| `LOG_LEVEL` | Nivel log root | `INFO` |
| `RATE_LIMIT_USER_RPM` | Rate limit user | `100` |
| `RATE_LIMIT_DEVICE_RPM` | Rate limit device | `1000` |

**NUNCA** commitear `.env` real; usar `.env.example` + secretos en CI/CD (GitHub Secrets, Vault).

---

## 11. Seguridad Adicional

| Control | Implementación |
|---------|----------------|
| **CORS** | Solo orígenes permitidos (portal domain, app bundle ID); `credentials: true` |
| **CSP** | `default-src 'self'; script-src 'self'; img-src 'self' data: https://minio;` |
| **HSTS** | `Strict-Transport-Security: max-age=31536000; includeSubDomains` (prod) |
| **Headers** | `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, `Referrer-Policy: strict-origin-when-cross-origin` |
| **Secrets** | Vault/Sealed Secrets en K8s; local `.env` gitignored |
| **Dependencias** | `dependency-check` (OWASP) en CI; actualización mensual |

---

## 12. Convenciones de Código (Hexagonal + DDD)

| Capa | Responsabilidad | Depende de |
|------|-----------------|------------|
| `domain/model` | Entidades, value objects, eventos de dominio, reglas (RN-*) | **Nada** (pure Java/Kotlin/Python) |
| `domain/service` | Servicios de dominio (lógica transversal entre entidades) | `domain/model` |
| `application/port/in` | Use case interfaces (Input Ports) | `domain/*` |
| `application/port/out` | Repository/External interfaces (Output Ports) | `domain/*` |
| `application/usecase` | Implementación use cases (orquesta domain + ports) | `application/port/*`, `domain/*` |
| `adapter/in/web` | Controllers REST, DTOs, mappers | `application/port/in` |
| `adapter/in/amqp` | Consumidores de eventos (si aplica) | `application/port/in` |
| `adapter/out/persistence` | JPA/SQLite repos, mappers, Liquibase | `application/port/out` |
| `adapter/out/storage` | MinIO/S3 client | `application/port/out` |
| `platform` | Config, security, observability, error handling, shared utils | **Transversal** (usado por adapters) |

**Regla de Oro:** `domain` **nunca** depende de Spring, JPA, HTTP, frameworks. Las dependencias apuntan **hacia adentro**.

---

## 13. Referencias a ADRs (Decision Records)

| ADR | Título | Estado | Archivo |
|-----|--------|--------|---------|
| ADR-001 | Backend en Java Spring Boot | Aceptada | `decisions/records/ADR-001-backend-java-spring-boot.md` |
| ADR-002 | Arquitectura Hexagonal por módulo (Monolito Modular) | Aceptada | `decisions/records/ADR-002-hexagonal-architecture.md` |
| ADR-003 | Analytics Module | Aceptada | `decisions/records/ADR-003-analytics-module.md` |
| ADR-004 | Database Strategy — 1 PostgreSQL + Esquemas por Módulo + Liquibase | Aceptada | `decisions/records/ADR-004-database-strategy.md` |
| ADR-005 | Offline-First en Device Edge (SQLite Local + Sync Idempotente) | Aceptada | `decisions/records/ADR-005-offline-first-device.md` |
| ADR-006 | MinIO Evidence Storage | Aceptada | `decisions/records/ADR-006-minio-evidence-storage.md` |
| ADR-007 | Observabilidad OpenTelemetry + LGTM | Aceptada | `decisions/records/ADR-007-observability-otel-lgtm.md` |
| ADR-008 | Traefik Edge Gateway | Aceptada | `decisions/records/ADR-008-traefik-edge-gateway.md` |
| ADR-009 | Estados Parametrizados + Auditoría Append-Only (status_category + status) | Aceptada | `decisions/records/ADR-009-status-parametrized-audit.md` |

> **Nota:** ADRs 004-009 ya existen y están aceptados. Ver carpeta `decisions/records/`.

---

## 14. Checklist de Cumplimiento por Módulo/Repo

| Repo | Auth | Audit | Obs | Idempotencia | Estados | TZ | Errores | Paginación | Versionado | Config | Seguridad | Hexagonal |
|------|------|-------|-----|--------------|---------|-----|---------|------------|------------|--------|-----------|-----------|
| `somnguard-api` | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| `somnguard-db` | N/A | ✅ | N/A | N/A | ✅ | ✅ | N/A | N/A | N/A | ✅ | ✅ | N/A |
| `somnguard-device` | ✅ (API Key) | ✅ (local) | ✅ (logs JSON) | ✅ | ✅ | ✅ | ✅ | N/A | N/A | ✅ | ✅ | ✅ (Python) |
| `somnguard-portal` | ✅ (JWT) | N/A | ✅ (RUM) | N/A | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | N/A |
| `somnguard-app` | ✅ (JWT) | N/A | ✅ (RUM) | N/A | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | N/A |

**Leyenda:** ✅ = Implementar obligatorio | N/A = No aplica en ese repo | **Estados** = ADR-009 (5 tablas core)

---

## Próximos Pasos

1. **Crear 8 ADRs** en `docs/05-architecture/decisions/records/` (usar `_template-adr.md`).
2. **Actualizar `modeling-conventions.md`** con campos de auditoría obligatorios.
3. **Implementar middleware/base classes** en `somnguard-api/platform` para: auth, error handling, observabilidad, paginación.
4. **Configurar OTel Collector + LGTM** en `somnguard-docker-infra` (ver propuesta separada).
5. **Validar con equipo** en reunión de arquitectura (30 min).