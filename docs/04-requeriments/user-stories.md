<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Historias de Usuario — HU-<REPO>-NNN

**Estado:** Borrador
**Fecha:** 2026-08-23

</div>

</div>

> **Fuente:** [functional.md](./functional.md) (RF-*), [product-backlog.md](../03-product-definition/product-backlog.md) (épicas MoSCoW),
> [_template-hu.md](./_template-hu.md) (plantilla base extendida).
>
> **Convención:** `HU-<REPO>-NNN` donde REPO ∈ {API, DB, DEVICE, PORTAL, APP}.
> **Tracker:** GitHub Projects (vincular cada HU al crear issue).

---

## Resumen por Épica y Repo

| Épica | Repo | HUs | Prioridad |
|-------|------|-----|-----------|
| Setup y Foundation | API, DB | 3 | Must |
| Seguridad y cuentas | API, PORTAL, APP | 7 | Must |
| Gestión de dispositivos | API, DB | 3 | Must |
| Parametrización | API, DB | 2 | Must |
| Telemetría y sincronización | API, DB, DEVICE | 6 | Must |
| Monitoreo y notificaciones | API, APP | 2 | Must |
| Analítica y reportes | API, PORTAL, APP | 4 | Should |
| Device Edge (visión, alertas, offline) | DEVICE | 5 | Must |

**Total MVP: 32 HUs** (Must: 25, Should: 7) | **Post-MVP: 4 HUs** (Could: 38 SP)

---

## HU-API-SETUP: Setup proyecto API (Spring Boot, CI, OpenAPI, Health, Observabilidad)

> **Repo:** API | **Sprint:** 0 | **SP:** 8 | **MoSCoW:** Must
> **Épica:** Setup y Foundation | **Feature:** FEA-API-SETUP

### Historia
**Como** equipo de desarrollo
**quiero** proyecto Spring Boot 4.1.1 configurado con Maven, CI/CD, OpenAPI, Health checks, manejo de errores y observabilidad
**para** tener base sólida antes de implementar funcionalidades.

### Criterios de Aceptación

| ID | Criterio | Testeable |
|----|----------|-----------|
| AC-001 | `pom.xml` con Java 21, Spring Boot 4.1.1, starters: web, security, data-jpa, validation, postgresql, actuator | Sí |
| AC-002 | Estructura paquetes hexagonal: `com.somnguard.{security,parameterization,device_management,telemetry_service,monitoring,analytics,platform}` | Sí |
| AC-003 | `platform/` transversal: error handling global, logging estructurado (JSON), observabilidad (Micrometer/Prometheus), OpenAPI/Swagger | Sí |
| AC-004 | GitHub Actions CI: build, test, checkstyle, dependency-check, Docker build | Sí |
| AC-005 | Health checks: `/actuator/health` (liveness/readiness), `/actuator/info` | Sí |
| AC-006 | Configuración por perfiles: `application.yml` + `application-{dev,qa,prod}.yml`, variables de entorno | Sí |

### Notas Técnicas
- Arquitectura hexagonal por módulo (ADR-002); `platform` fuera de módulos
- Spring Security configurado para JWT RS256 (claves en `keys/`) y API Key para devices
- Liquibase integrado (migraciones desde `somnguard-db`)

### Dependencias

| HU / Artefacto | Tipo | Descripción |
|----------------|------|-------------|
| ADR-001, ADR-002 | Decisión | Stack y arquitectura |
| modeling-conventions.md | Estándar | Convenciones BD |

---

## HU-DB-001a: Esquemas BD y changelog maestro (Liquibase)

> **Repo:** DB | **Sprint:** 0 (pre-sprint) | **SP:** 8 | **MoSCoW:** Must
> **Épica:** Setup y Foundation | **Feature:** FEA-DB-SCHEMA-CORE

### Historia
**Como** desarrollador
**quiero** 6 esquemas BD con changelog maestro y changesets por tabla
**para** que cada módulo sea dueño de sus datos y deploy sea automático.

### Criterios de Aceptación

| ID | Criterio | Testeable |
|----|----------|-----------|
| AC-001 | 6 esquemas: `security`, `parameterization`, `device_management`, `telemetry_service`, `monitoring`, `analytics` | Sí |
| AC-002 | `changelog/changelog-master.yaml` incluye todos los módulos; orden de ejecución definido | Sí |
| AC-003 | Changesets por tabla (20 tablas): snake_case, PK UUID, auditoría, soft delete, JSONB, FKs, checks | Sí |
| AC-004 | `docker compose --profile tooling run liquibase update` aplica todo en orden | Sí |

### Dependencias

| HU / Artefacto | Tipo | Descripción |
|----------------|------|-------------|
| modeling-conventions.md | Estándar | [06-data-architecture/modeling-conventions.md](../06-data-architecture/modeling-conventions.md) |
| migration-strategy.md | Estrategia | [06-data-architecture/migration-strategy.md](../06-data-architecture/migration-strategy.md) |

---

## HU-DB-001b: Seed data, rollback y validación

> **Repo:** DB | **Sprint:** 0 (pre-sprint) | **SP:** 5 | **MoSCoW:** Must
> **Épica:** Setup y Foundation | **Feature:** FEA-DB-SEED-ROLLBACK

### Historia
**Como** desarrollador
**quiero** seed de catálogos y roles, rollback probado, y validación de migraciones
**para** tener datos base y reversibilidad garantizada.

### Criterios de Aceptación

| ID | Criterio | Testeable |
|----|----------|-----------|
| AC-001 | Seed data: catálogos `parameterization` (event_category, severity, media_type, sound_pattern, event_type desde Apéndices SRS) | Sí |
| AC-002 | Seed data: `security.role` (admin, user), `security.feature` (por módulo), `security.module` (6 módulos), `security.role_feature` | Sí |
| AC-003 | Rollback probado para últimos 3 changesets por módulo | Sí |
| AC-004 | Validación: `liquibase validate` y `liquibase status` en CI | Sí |

### Dependencias

| HU / Artefacto | Tipo | Descripción |
|----------------|------|-------------|
| HU-DB-001a | Bloqueante | Esquemas base creados |
| RF-SEC-*, RF-PAR-* | Requisitos | Datos semilla |

---

## HU-PORTAL-001: Autenticación en Portal Web (login, register, reset, session)

> **Repo:** PORTAL | **Sprint:** 1 | **SP:** 8 | **MoSCoW:** Must
> **Épica:** Seguridad y cuentas | **Feature:** FEA-PORTAL-AUTH

### Historia
**Como** usuario en portal web
**quiero** registrarme, iniciar/cerrar sesión y recuperar contraseña
**para** acceder a mi cuenta y dispositivos.

### Criterios de Aceptación

| ID | Criterio | Testeable |
|----|----------|-----------|
| AC-001 | Pantalla `/login`: formulario email/password, "¿Olvidaste contraseña?", validación cliente, error handling | Sí |
| AC-002 | Pantalla `/register`: datos cuenta, aceptación términos, llamada POST `/api/v1/auth/register`, redirige a login | Sí |
| AC-003 | Pantalla `/reset-password`: solicitud token (POST `/auth/forgot-password`) + confirmación (POST `/auth/reset-password`) | Sí |
| AC-004 | Guardado JWT (access + refresh) en httpOnly cookies / secure storage; refresh automático silencioso | Sí |
| AC-005 | Logout: llama POST `/api/v1/auth/logout`, limpia storage, redirige a login | Sí |
| AC-006 | Guards de ruta: redirige a `/login` si no autenticado; redirige a `/dashboard` si ya autenticado | Sí |

### Dependencias

| HU / Artefacto | Tipo | Descripción |
|----------------|------|-------------|
| HU-API-001, HU-API-002 | Bloqueante | Endpoints auth backend |
| RF-SEC-01..05,08,09 | Requisito | Base funcional |

---

## HU-APP-001: Autenticación en App Móvil (login, register, reset, session)

> **Repo:** APP | **Sprint:** 1 | **SP:** 8 | **MoSCoW:** Must
> **Épica:** Seguridad y cuentas | **Feature:** FEA-APP-AUTH

### Historia
**Como** usuario en app móvil
**quiero** registrarme, iniciar/cerrar sesión y recuperar contraseña
**para** acceder a mi cuenta y recibir notificaciones.

### Criterios de Aceptación

| ID | Criterio | Testeable |
|----|----------|-----------|
| AC-001 | Pantalla Login: email/password, biometría opcional, "¿Olvidaste contraseña?", validación, error handling | Sí |
| AC-002 | Pantalla Register: datos cuenta, términos, POST `/api/v1/auth/register`, auto-login tras registro | Sí |
| AC-003 | Pantalla Reset Password: solicitud + confirmación token (POST `/auth/forgot-password`, `/auth/reset-password`) | Sí |
| AC-004 | Almacenamiento seguro JWT (Keychain/Keystore); refresh token automático; logout limpia credenciales | Sí |
| AC-005 | Registro token FCM/APNs al login; asociación a `user_id` para push (ver HU-APP-003) | Sí |
| AC-006 | Deep links: `somnguard://event/{id}` abre detalle evento desde notificación | Sí |

### Dependencias

| HU / Artefacto | Tipo | Descripción |
|----------------|------|-------------|
| HU-API-001, HU-API-002 | Bloqueante | Endpoints auth backend |
| RF-SEC-01..05,08,09 | Requisito | Base funcional |

---

## HU-API-001: Gestión de autenticación y sesión de usuario

> **Repo:** API | **Sprint:** 1 | **SP:** 8 | **MoSCoW:** Must
> **Épica:** Seguridad y cuentas | **Feature:** FEA-SEC-AUTH
> **Tracker:** [GitHub Projects](https://github.com/orgs/SomnGuard/projects/1)

### Historia
**Como** usuario de la plataforma (conductor, admin)
**quiero** registrarme, iniciar y cerrar sesión de forma segura
**para** acceder a mis datos y dispositivos protegidos.

### Criterios de Aceptación

| ID | Criterio | Testeable |
|----|----------|-----------|
| AC-001 | POST `/auth/register` crea cuenta con correo/teléfono únicos, hash bcrypt, retorna 201 | Sí |
| AC-002 | POST `/auth/login` valida credenciales, retorna JWT RS256 (access 15min) + refresh token (7d) | Sí |
| AC-003 | POST `/auth/logout` invalida refresh token (blocklist), retorna 204 | Sí |
| AC-004 | POST `/auth/refresh` rota access token usando refresh token válido | Sí |
| AC-005 | Auditoría `audit_login` registra: user_id, IP, user-agent, éxito/fallo, timestamp | Sí |
| AC-006 | Endpoints protegidos requieren `Authorization: Bearer <JWT>`; 401 si inválido/expirado | Sí |

### Notas Técnicas
- JWT firmado con RS256 (claves en `keys/`); JWKS endpoint para verificación distribuida
- Refresh token almacenado en BD con hash; rotación en cada uso
- Rate limiting: 5 req/min en `/auth/*`

### Dependencias

| HU / Artefacto | Tipo | Descripción |
|----------------|------|-------------|
| RF-SEC-01..04,09 | Requisito | Base funcional |
| ADR-001 (JWT RS256) | Decisión | Arquitectura auth |

### Trazabilidad

| Tipo | ID | Referencia |
|------|----|------------|
| Épica | EPC-SEC | Seguridad y cuentas |
| Feature | FEA-SEC-AUTH | Autenticación y sesión |
| RF | RF-SEC-01..04,09 | [functional.md](./functional.md) |
| ADR | ADR-001 | [decisions/records/](../05-architecture/decisions/records/) |

---

## HU-API-002: Recuperación y actualización de cuenta

> **Repo:** API | **Sprint:** 1 | **SP:** 5 | **MoSCoW:** Must
> **Épica:** Seguridad y cuentas | **Feature:** FEA-SEC-ACCOUNT

### Historia
**Como** usuario autenticado
**quiero** recuperar mi contraseña y actualizar mis datos personales
**para** mantener mi acceso y perfil al día.

### Criterios de Aceptación

| ID | Criterio | Testeable |
|----|----------|-----------|
| AC-001 | POST `/auth/forgot-password` envía token temporal (expira 1h) al correo registrado | Sí |
| AC-002 | POST `/auth/reset-password` valida token, actualiza hash, invalida tokens previos | Sí |
| AC-003 | PATCH `/users/me` actualiza nombre, correo, teléfono (validación unicidad) | Sí |
| AC-004 | DELETE `/users/me` inicia soft-delete: marca `deleted_at`, invalida sesiones, ventana 30d recuperación | Sí |

### Dependencias

| HU / Artefacto | Tipo | Descripción |
|----------------|------|-------------|
| HU-API-001 | Bloqueante | Requiere auth funcionando |
| RF-SEC-05,08 | Requisito | Base funcional |

---

## HU-API-003: Control de acceso basado en roles (RBAC)

> **Repo:** API | **Sprint:** 1 | **SP:** 5 | **MoSCoW:** Must
> **Épica:** Seguridad y cuentas | **Feature:** FEA-SEC-RBAC

### Historia
**Como** administrador del sistema
**quiero** definir roles y permisos por feature
**para** restringir funcionalidades según privilegios.

### Criterios de Aceptación

| ID | Criterio | Testeable |
|----|----------|-----------|
| AC-001 | CRUD roles (`role`) y features (`feature`) solo admin | Sí |
| AC-002 | Asignación `user_role` y `role_feature` vía API admin | Sí |
| AC-003 | Middleware valida `role_feature` en cada endpoint protegido; 403 si no autorizado | Sí |
| AC-004 | Roles por defecto: `admin` (todo), `user` (solo sus datos/dispositivos) | Sí |

### Dependencias

| HU / Artefacto | Tipo | Descripción |
|----------------|------|-------------|
| HU-API-001 | Bloqueante | Requiere auth |
| RF-SEC-10 | Requisito | Base funcional |

---

## HU-API-004: CRUD de catálogos de parametrización

> **Repo:** API, DB | **Sprint:** 1 | **SP:** 8 | **MoSCoW:** Must
> **Épica:** Parametrización | **Feature:** FEA-PAR-CATALOGS

### Historia
**Como** administrador técnico
**quiero** gestionar catálogos base (event_type, severity, sound_pattern, media_type, event_category)
**para** configurar reglas de detección y alertas sin deploy.

### Criterios de Aceptación

| ID | Criterio | Testeable |
|----|----------|-----------|
| AC-001 | CRUD completo para cada catálogo (POST, GET, PATCH, DELETE) | Sí |
| AC-002 | `sound_pattern`: frecuencia Hz, duración, repeticiones, patrón (continuo/intermitente/escalado) | Sí |
| AC-003 | `event_type`: umbrales configurables (parpadeo, cierre ojos, bostezo, cabeceo, teléfono, mirada, cinturón) | Sí |
| AC-004 | Versionado: historial de cambios con `created_by`, `updated_by`, timestamp | Sí |
| AC-005 | Seed inicial desde Apéndices 1 y 2 del SRS (AS-01..AS-09, EV-SOM-*, EV-DIS-*, EV-CIN-*) | Sí |

### Dependencias

| HU / Artefacto | Tipo | Descripción |
|----------------|------|-------------|
| HU-DB-001a | Bloqueante | Esquemas BD creados |
| RF-PAR-01..03 | Requisito | Base funcional |

---

## HU-API-005: Configuración remota de dispositivo (device_config)

> **Repo:** API, DB | **Sprint:** 2 | **SP:** 5 | **MoSCoW:** Must
> **Épica:** Gestión de dispositivos | **Feature:** FEA-DEV-CONFIG

### Historia
**Como** administrador
**quiero** enviar configuración JSONB a un dispositivo (umbrales, sound_pattern, volumen, intervalo sync)
**para** ajustar su comportamiento sin intervención física.

### Criterios de Aceptación

| ID | Criterio | Testeable |
|----|----------|-----------|
| AC-001 | PATCH `/devices/{id}/config` valida JSON contra schema, persiste en `device_config` | Sí |
| AC-002 | GET `/devices/{id}/config` retorna config vigente (merge defaults + overrides) | Sí |
| AC-003 | Device pulla config tras sync exitosa (ver HU-DEVICE-004) | Sí |
| AC-004 | Historial de cambios de config con auditoría | Sí |

### Dependencias

| HU / Artefacto | Tipo | Descripción |
|----------------|------|-------------|
| HU-API-006 | Bloqueante | Requiere device registrado |
| HU-DEVICE-004 | Relacionada | Pull config en device |
| RF-DEV-03, RF-TEL-05, RF-EDGE-11 | Requisito | Base funcional |

---

## HU-API-006: Gestión de dispositivos (alta, asociación, estados)

> **Repo:** API, DB | **Sprint:** 1 | **SP:** 8 | **MoSCoW:** Must
> **Épica:** Gestión de dispositivos | **Feature:** FEA-DEV-LIFECYCLE

### Historia
**Como** usuario propietario
**quiero** dar de alta mi dispositivo, asociarlo a mi cuenta y ver su estado
**para** que empiece a enviar telemetría.

### Criterios de Aceptación

| ID | Criterio | Testeable |
|----|----------|-----------|
| AC-001 | POST `/devices` (admin): crea device con `serial_number`, `firmware_version`, `api_key_hash`, estado `Registrado` | Sí |
| AC-002 | POST `/devices/{id}/assign` (user autenticado): asocia device a su cuenta → estado `Asignado` | Sí |
| AC-003 | POST `/devices/{id}/unassign`: libera device → estado `Registrado` | Sí |
| AC-004 | State machine automática: `Asignado` → `Activo` (primer heartbeat) ↔ `Offline` (sin heartbeat 5min) → `Suspendido` (admin) → `Retirado` | Sí |
| AC-005 | GET `/devices` con filtros: estado, fecha asignación, paginación | Sí |
| AC-006 | Heartbeat: PUT `/devices/{id}/heartbeat` actualiza `last_seen`, versión firmware | Sí |

### Dependencias

| HU / Artefacto | Tipo | Descripción |
|----------------|------|-------------|
| HU-API-001 | Bloqueante | Auth usuario |
| HU-DB-001a | Bloqueante | Esquema `device_management` |
| RF-DEV-01..05 | Requisito | Base funcional |

---

## HU-API-007: Ingesta idempotente de eventos y evidencia

> **Repo:** API, DB | **Sprint:** 2 | **SP:** 13 | **MoSCoW:** Must
> **Épica:** Telemetría y sincronización | **Feature:** FEA-TEL-INGEST

### Historia
**Como** dispositivo (edge)
**quiero** enviar lotes de eventos y evidencia multimedia de forma idempotente
**para** que se persistan sin duplicados aunque reintente por red.

### Criterios de Aceptación

| ID | Criterio | Testeable |
|----|----------|-----------|
| AC-001 | POST `/telemetry/events` (auth: API Key device) acepta array de eventos + evidencia | Sí |
| AC-002 | Validación: `device_id` + `api_key` coinciden, `event_id` (UUID) único → 409 si duplicado | Sí |
| AC-003 | Persiste `event` (event_type_id, occurred_at, severity, is_offline_sync, evidence_refs) | Sí |
| AC-004 | Evidencia (imagen/video) → MinIO bucket `somnguard-evidence`; retorna `evidence_id` | Sí |
| AC-005 | Registra `alert_log` con código AS-XX, timestamp, event_id, severidad | Sí |
| AC-006 | Respuesta 201 con array de `event_id` aceptados; ACK para limpieza buffer local | Sí |
| AC-007 | Lote máx 100 eventos; timeout 10s; payload máx 50MB | Sí |

### Dependencias

| HU / Artefacto | Tipo | Descripción |
|----------------|------|-------------|
| HU-DB-001a | Bloqueante | Esquema `telemetry_service` |
| HU-DEVICE-003 | Relacionada | Device envía lote |
| HU-API-008 | Relacionada | Pull config tras ACK |
| RF-TEL-01..03, RF-EDGE-08 | Requisito | Base funcional |

---

## HU-API-008: Consulta de eventos con filtros y paginación

> **Repo:** API, DB | **Sprint:** 2 | **SP:** 5 | **MoSCoW:** Must
> **Épica:** Telemetría y sincronización | **Feature:** FEA-TEL-QUERY

### Historia
**Como** portal/app
**quiero** consultar eventos con filtros (device, tipo, severidad, fechas) y paginación
**para** mostrar timeline y métricas.

### Criterios de Aceptación

| ID | Criterio | Testeable |
|----|----------|-----------|
| AC-001 | GET `/telemetry/events` con query params: `device_id`, `event_type_id`, `severity`, `from`, `to`, `page`, `page_size` | Sí |
| AC-002 | Respuesta: `{data: [], meta: {total, page, page_size, total_pages}}` | Sí |
| AC-003 | JOIN con `event_type`, `severity`, `media_type` para nombres legibles | Sí |
| AC-004 | Índices BD optimizan filtros frecuentes (device_id + occurred_at) | Sí |

### Dependencias

| HU / Artefacto | Tipo | Descripción |
|----------------|------|-------------|
| HU-API-007 | Bloqueante | Datos existiendo |
| RF-TEL-06 | Requisito | Base funcional |

---

## HU-API-009: Notificación push de eventos críticos

> **Repo:** API | **Sprint:** 3 | **SP:** 8 | **MoSCoW:** Must
> **Épica:** Monitoreo y notificaciones | **Feature:** FEA-MON-NOTIFY

### Historia
**Como** usuario de la plataforma
**quiero** recibir notificación push/email cuando mi dispositivo detecte evento crítico
**para** actuar o revisar de inmediato.

### Criterios de Aceptación

| ID | Criterio | Testeable |
|----|----------|-----------|
| AC-001 | Trigger automático al persistir evento con `severity = crítica` (EV-SOM-05, EV-DIS-02, EV-DIS-04, EV-CIN-01/02) | Sí |
| AC-002 | Plantilla por `event_type` + `severity`; canales: push (FCM/APNs), email, in-app | Sí |
| AC-003 | Tracking: `sent` → `delivered` → `read`; reintentos exponenciales (max 3) | Sí |
| AC-004 | Preferencias usuario: habilitar/deshabilitar por canal, horario silencio, severidad mínima | Sí |

### Dependencias

| HU / Artefacto | Tipo | Descripción |
|----------------|------|-------------|
| HU-API-007 | Bloqueante | Eventos persistidos |
| HU-APP-003 | Relacionada | App recibe push |
| RF-MON-01..04 | Requisito | Base funcional |

---

## HU-API-010: Línea de tiempo y métricas de comportamiento (Analytics)

> **Repo:** API, DB | **Sprint:** 4 | **SP:** 8 | **MoSCoW:** Must
> **Épica:** Analítica y reportes | **Feature:** FEA-ANA-TIMELINE

### Historia
**Como** usuario
**quiero** ver timeline cronológico de mis eventos y métricas agregadas
**para** entender mis patrones de conducción.

### Criterios de Aceptación

| ID | Criterio | Testeable |
|----|----------|-----------|
| AC-001 | GET `/analytics/timeline` retorna eventos ordenados desc con filtros (fecha, tipo, severidad) | Sí |
| AC-002 | GET `/analytics/metrics` retorna: freq por event_type, severidad media, tendencia semanal/mensual | Sí |
| AC-003 | Agregaciones calculadas en BD (vistas materializadas o queries optimizadas) | Sí |
| AC-004 | Respuesta < 500ms p95 para 10k eventos | Sí |

### Dependencias

| HU / Artefacto | Tipo | Descripción |
|----------------|------|-------------|
| HU-API-007 | Bloqueante | Datos existiendo |
| HU-DB-002 | Bloqueante | Vistas/índices analytics |
| RF-ANA-01,02 | Requisito | Base funcional |

---

## HU-API-011: Resumen descriptivo IA y reporte consolidado

> **Repo:** API | **Sprint:** 5 | **SP:** 13 | **MoSCoW:** Should
> **Épica:** Analítica y reportes | **Feature:** FEA-ANA-AI-REPORT

### Historia
**Como** usuario
**quiero** un resumen interpretativo generado por IA y un reporte PDF/HTML descargable
**para** tener conclusiones accionables sin analizar datos crudos.

### Criterios de Aceptación

| ID | Criterio | Testeable |
|----|----------|-----------|
| AC-001 | POST `/analytics/summary` invoca LLM (prompt con métricas + eventos) → retorna texto interpretativo | Sí |
| AC-002 | POST `/analytics/report` genera PDF/HTML: timeline + métricas + resumen IA + evidencia (thumbnails) | Sí |
| AC-003 | Descarga directa (Content-Disposition: attachment) y visualización inline | Sí |
| AC-004 | Cache de resumen/reporte 1h por usuario+periodo | Sí |

### Dependencias

| HU / Artefacto | Tipo | Descripción |
|----------------|------|-------------|
| HU-API-010 | Bloqueante | Métricas base |
| RF-ANA-03,04 | Requisito | Base funcional |
| ADR-003 (Object Storage) | Decisión | MinIO para PDFs |

---

## 📦 BACKLOG POST-MVP (Could — Fuera de alcance MVP)

Las siguientes HUs son **Could** (38 SP totales) y se mueven a backlog post-MVP. Se abordarán tras validar el MVP completo.

---

### HU-API-012: Video streaming tiempo real (WebRTC) — POST-MVP

> **Repo:** API, DEVICE | **Sprint:** Post-MVP | **SP:** 13 | **MoSCoW:** Could
> **Épica:** Analítica y reportes | **Feature:** FEA-ANA-STREAM

### Historia
**Como** usuario en la app
**quiero** ver video en vivo de la cámara del dispositivo a demanda
**para** verificar situación en tiempo real.

### Criterios de Aceptación

| ID | Criterio | Testeable |
|----|----------|-----------|
| AC-001 | POST `/devices/{id}/stream/start` negocia WebRTC (offer/answer) via signaling server | Sí |
| AC-002 | Device inicia streaming H.264/VP8 a SFU; app reproduce | Sí |
| AC-003 | POST `/devices/{id}/stream/stop` cierra conexión, libera recursos | Sí |
| AC-004 | Solo bajo demanda; auto-stop si app en background > 30s | Sí |

### Dependencias

| HU / Artefacto | Tipo | Descripción |
|----------------|------|-------------|
| HU-DEVICE-005 (Post-MVP) | Bloqueante | Device implementa WebRTC |
| HU-API-006 | Bloqueante | Device activo |
| RF-ANA-05, RF-EDGE-13 | Requisito | Base funcional |

---

### HU-DEVICE-005: Video streaming WebRTC a demanda — POST-MVP

> **Repo:** DEVICE | **Sprint:** Post-MVP | **SP:** 13 | **MoSCoW:** Could
> **Épica:** Analítica y reportes | **Feature:** FEA-EDGE-STREAM

### Historia
**Como** dispositivo
**quiero** transmitir video WebRTC a demanda desde app/portal
**para** permitir verificación visual en tiempo real.

### Criterios de Aceptación

| ID | Criterio | Testeable |
|----|----------|-----------|
| AC-001 | Recibe oferta SDP via signaling (WebSocket/HTTP), responde answer, inicia streaming H.264 | Sí |
| AC-002 | Bitrate adaptativo según red (min 500kbps, max 2Mbps) | Sí |
| AC-003 | Auto-stop tras 30s sin viewer o comando stop | Sí |
| AC-004 | Solo si dispositivo `Activo` y asociado a usuario autenticado | Sí |

### Dependencias

| HU / Artefacto | Tipo | Descripción |
|----------------|------|-------------|
| HU-API-012 (Post-MVP) | Bloqueante | Signaling server en API |
| HU-DEVICE-002 | Bloqueante | Device activo |
| RF-EDGE-13, RF-ANA-05 | Requisito | Base funcional |

---

### HU-PORTAL-004: Video en vivo embebido — POST-MVP

> **Repo:** PORTAL | **Sprint:** Post-MVP | **SP:** 5 | **MoSCoW:** Could
> **Épica:** Analítica y reportes | **Feature:** FEA-PORTAL-STREAM

### Historia
**Como** usuario en portal
**quiero** ver video en vivo del dispositivo en una ventana modal
**para** verificar situación sin abrir la app.

### Criterios de Aceptación

| ID | Criterio | Testeable |
|----|----------|-----------|
| AC-001 | Botón "Ver en vivo" en detalle dispositivo → modal con player WebRTC | Sí |
| AC-002 | Controles: play/pause, fullscreen, mute, cerrar | Sí |
| AC-003 | Auto-cierre modal si usuario navega fuera | Sí |

### Dependencias

| HU / Artefacto | Tipo | Descripción |
|----------------|------|-------------|
| HU-API-012 (Post-MVP) | Bloqueante | Streaming endpoint |
| RF-ANA-05 | Requisito | Base funcional |

---

### HU-APP-004: Video en vivo en app — POST-MVP

> **Repo:** APP | **Sprint:** Post-MVP | **SP:** 8 | **MoSCoW:** Could
> **Épica:** Analítica y reportes | **Feature:** FEA-APP-STREAM

### Historia
**Como** usuario en app
**quiero** ver video en vivo del dispositivo con un tap
**para** verificar al conductor en tiempo real.

### Criterios de Aceptación

| ID | Criterio | Testeable |
|----|----------|-----------|
| AC-001 | Botón "Ver en vivo" en device card → pantalla fullscreen WebRTC | Sí |
| AC-002 | Controles overlay: mute, flip camera (si dual), cerrar | Sí |
| AC-003 | Indicador calidad red (verde/amarillo/rojo) | Sí |

### Dependencias

| HU / Artefacto | Tipo | Descripción |
|----------------|------|-------------|
| HU-API-012 (Post-MVP) | Bloqueante | Streaming endpoint |
| HU-DEVICE-005 (Post-MVP) | Bloqueante | Device WebRTC |
| RF-ANA-05 | Requisito | Base funcional |

---

## HU-DB-002: Vistas e índices para Analytics

> **Repo:** DB | **Sprint:** 3 | **SP:** 5 | **MoSCoW:** Must
> **Épica:** Analítica y reportes | **Feature:** FEA-DB-ANALYTICS

### Historia
**Como** API de analytics
**quiero** vistas materializadas e índices optimizados
**para** que timeline y métricas respondan < 500ms.

### Criterios de Aceptación

| ID | Criterio | Testeable |
|----|----------|-----------|
| AC-001 | Vista `analytics.v_event_timeline` con JOINs resueltos (event_type, severity, device, user) | Sí |
| AC-002 | Vista `analytics.v_metrics_daily` pre-agregada por device/fecha/tipo/severidad | Sí |
| AC-003 | Índices: `event(device_id, occurred_at)`, `event(event_type_id, severity)`, `evidence(event_id)` | Sí |
| AC-004 | Refresh automático vistas: trigger on insert o pg_cron cada 5min | Sí |

### Dependencias

| HU / Artefacto | Tipo | Descripción |
|----------------|------|-------------|
| HU-DB-001a | Bloqueante | Esquema base |
| HU-API-010 | Relacionada | Consumidor principal |

---

## HU-DEVICE-001: Detección visión: somnolencia, distracción, cinturón

> **Repo:** DEVICE | **Sprint:** 2-3 | **SP:** 21 | **MoSCoW:** Must
> **Épica:** Telemetría y sincronización | **Feature:** FEA-EDGE-VISION

### Historia
**Como** conductor
**quiero** que el dispositivo detecte somnolencia, distracción y cinturón en tiempo real
**para** recibir alertas inmediatas que prevengan accidentes.

### Criterios de Aceptación

| ID | Criterio | Testeable |
|----|----------|-----------|
| AC-001 | Detección PERCLOS, parpadeo anómalo, cierre ojos >2s, bostezo, cabeceo >20°/3s | Sí |
| AC-002 | Clasificación severidad: leve (AS-01), moderada (AS-02), severa (AS-03), crítica (AS-04) | Sí |
| AC-003 | Detección teléfono en mano/rostro >2s (AS-05), mirada fuera eje >3s (AS-06), movimientos anómalos (AS-05) | Sí |
| AC-004 | Detección cinturón: presencia + posición correcta; si no → AS-07 intermitente | Sí |
| AC-005 | Procesamiento < 2s/frame (RNF-1.2); alarma < 1s tras confirmación (RNF-1.3) | Sí |
| AC-006 | Modelo visión cargado en inicialización; fallback si falla (AS-09) | Sí |

### Dependencias

| HU / Artefacto | Tipo | Descripción |
|----------------|------|-------------|
| HU-DEVICE-002 | Bloqueante | Inicialización hw |
| RF-EDGE-04,05,06 | Requisito | Base funcional |
| Apéndice 2 SRS | Especificación | Umbrales y códigos evento |

---

## HU-DEVICE-002: Inicialización, verificación cámara y gestión estados

> **Repo:** DEVICE | **Sprint:** 1 | **SP:** 8 | **MoSCoW:** Must
> **Épica:** Telemetría y sincronización | **Feature:** FEA-EDGE-INIT

### Historia
**Como** dispositivo
**quiero** inicializarme, verificar cámara y gestionar mis estados (activo/espera/offline)
**para** operar de forma autónoma y robusta.

### Criterios de Aceptación

| ID | Criterio | Testeable |
|----|----------|-----------|
| AC-001 | Arranque < 60s (RNF-1.1); verifica cámara → AS-08 (ok) / AS-09 (error) | Sí |
| AC-002 | Verifica campo visual: obstrucción/mala posición → AS-09, pausa detección | Sí |
| AC-003 | Estado `Activo` ↔ `Espera` (sin rostro >30s) ↔ `Activo` (rostro detectado) | Sí |
| AC-004 | Heartbeat cada 30s a API; transición `Activo` ↔ `Offline` por conectividad | Sí |
| AC-005 | Carga `device_config` al iniciar y tras cada sync; aplica umbrales/sound_pattern/volumen | Sí |

### Dependencias

| HU / Artefacto | Tipo | Descripción |
|----------------|------|-------------|
| HU-API-006 | Relacionada | Device registrado |
| RF-EDGE-01,02,03,09,11 | Requisito | Base funcional |

---

## HU-DEVICE-003: Buffer offline, sync automático y limpieza

> **Repo:** DEVICE | **Sprint:** 2 | **SP:** 13 | **MoSCoW:** Must
> **Épica:** Telemetría y sincronización | **Feature:** FEA-EDGE-SYNC

### Historia
**Como** dispositivo
**quiero** guardar eventos localmente sin red, sincronizar cuando haya conexión y limpiar buffer tras ACK
**para** no perder datos y no llenar almacenamiento.

### Criterios de Aceptación

| ID | Criterio | Testeable |
|----|----------|-----------|
| AC-001 | Buffer SQLite local: tabla `pending_events` (event_json, evidence_path, retries, created_at) | Sí |
| AC-002 | Detección conectividad: ping/HEAD a API cada 30s; online → sync automático | Sí |
| AC-003 | Envío lote (máx 100) a POST `/telemetry/events`; reintentos exponenciales (1m, 2m, 4m, max 1h) | Sí |
| AC-004 | Deduplicación: `event_id` UUID único; server rechaza duplicados (409) | Sí |
| AC-005 | Limpieza automática tras ACK 201 (borra confirmados); retención 7d para fallidos | Sí |
| AC-006 | Almacenamiento local < 90% → AS-09; política retención evidencia 7d | Sí |

### Dependencias

| HU / Artefacto | Tipo | Descripción |
|----------------|------|-------------|
| HU-API-007 | Bloqueante | Endpoint ingesta |
| HU-API-008 | Relacionada | Pull config tras sync |
| RF-EDGE-08,10,12, RF-TEL-04,07 | Requisito | Base funcional |

---

## HU-DEVICE-004: Alertas sonoras diferenciadas y escalamiento

> **Repo:** DEVICE | **Sprint:** 2 | **SP:** 5 | **MoSCoW:** Must
> **Épica:** Telemetría y sincronización | **Feature:** FEA-EDGE-ALERTS

### Historia
**Como** conductor
**quiero** alertas sonoras distintas por tipo de riesgo y que escalen si persisto
**para** identificar el peligro sin mirar la pantalla.

### Criterios de Aceptación

| ID | Criterio | Testeable |
|----|----------|-----------|
| AC-001 | Reproduce patrón AS-XX según `event_type` + `severity` (tabla Apéndice 1) | Sí |
| AC-002 | Escalamiento: evento persistente > 10s → siguiente nivel severidad (AS-01→AS-02→AS-03→AS-04) | Sí |
| AC-003 | Volumen configurable via `device_config` (default 80%) | Sí |
| AC-004 | No superposición: una alerta a la vez; cola si múltiples simultáneas | Sí |

### Dependencias

| HU / Artefacto | Tipo | Descripción |
|----------------|------|-------------|
| HU-DEVICE-001 | Bloqueante | Detección genera eventos |
| HU-API-004 | Relacionada | Catálogo sound_pattern |
| RF-EDGE-07, RF-EDGE-11 | Requisito | Base funcional |

---

## HU-PORTAL-002: Dashboard de dispositivos y eventos

> **Repo:** PORTAL | **Sprint:** 3 | **SP:** 8 | **MoSCoW:** Must
> **Épica:** Gestión de dispositivos / Telemetría | **Feature:** FEA-PORTAL-DASHBOARD

### Historia
**Como** usuario en portal web
**quiero** ver mis dispositivos, su estado y timeline de eventos
**para** monitorear mi flota personal.

### Criterios de Aceptación

| ID | Criterio | Testeable |
|----|----------|-----------|
| AC-001 | Lista dispositivos: serial, estado (badge color), último heartbeat, firmware | Sí |
| AC-002 | Timeline eventos: filtro por device, tipo, severidad, fechas; paginación infinita | Sí |
| AC-003 | Detalle evento: evidencia (imagen/video), ubicación, métricas asociadas | Sí |
| AC-004 | Responsive: desktop/tablet/mobile; carga inicial < 2s | Sí |

### Dependencias

| HU / Artefacto | Tipo | Descripción |
|----------------|------|-------------|
| HU-PORTAL-001 | Bloqueante | Auth en portal |
| HU-API-006, HU-API-008 | Bloqueante | APIs dispositivos y eventos |
| RF-DEV-06, RF-TEL-06 | Requisito | Base funcional |

---

## HU-PORTAL-003: Visualización de métricas y tendencias

> **Repo:** PORTAL | **Sprint:** 4 | **SP:** 5 | **MoSCoW:** Must
> **Épica:** Analítica y reportes | **Feature:** FEA-PORTAL-METRICS

### Historia
**Como** usuario
**quiero** ver gráficos de frecuencia, severidad y tendencias temporales
**para** identificar patrones de riesgo en mi conducción.

### Criterios de Aceptación

| ID | Criterio | Testeable |
|----|----------|-----------|
| AC-001 | Gráficos: barras freq por event_type, línea severidad media semanal, heatmap hora/día | Sí |
| AC-002 | Selector periodo: 7d, 30d, 90d, personalizado | Sí |
| AC-003 | Tooltips con valores exactos; exportar PNG/CSV | Sí |

### Dependencias

| HU / Artefacto | Tipo | Descripción |
|----------------|------|-------------|
| HU-API-010 | Bloqueante | Endpoint métricas |
| RF-ANA-01,02 | Requisito | Base funcional |

---

## HU-PORTAL-004: Resumen IA y reporte descargable

> **Repo:** PORTAL | **Sprint:** 5 | **SP:** 8 | **MoSCoW:** Should
> **Épica:** Analítica y reportes | **Feature:** FEA-PORTAL-REPORT

### Historia
**Como** usuario
**quiero** leer un resumen interpretativo y descargar reporte PDF
**para** tener conclusiones claras y compartirlas.

### Criterios de Aceptación

| ID | Criterio | Testeable |
|----|----------|-----------|
| AC-001 | Botón "Generar resumen IA" llama POST `/analytics/summary`, muestra texto en modal | Sí |
| AC-002 | Botón "Descargar reporte" llama POST `/analytics/report`, descarga PDF | Sí |
| AC-003 | Loading states y manejo errores (reintento, notificación) | Sí |

### Dependencias

| HU / Artefacto | Tipo | Descripción |
|----------------|------|-------------|
| HU-API-011 | Bloqueante | Endpoints IA/reporte |
| RF-ANA-03,04 | Requisito | Base funcional |

---

## HU-APP-002: Notificaciones push en móvil

> **Repo:** APP | **Sprint:** 3 | **SP:** 8 | **MoSCoW:** Must
> **Épica:** Monitoreo y notificaciones | **Feature:** FEA-APP-PUSH

### Historia
**Como** usuario en móvil
**quiero** recibir notificaciones push de eventos críticos de mis dispositivos
**para** enterarme aunque no tenga la app abierta.

### Criterios de Aceptación

| ID | Criterio | Testeable |
|----|----------|-----------|
| AC-001 | Registro token FCM/APNs al login; asociación a user_id | Sí |
| AC-002 | Recibe push con título, cuerpo, data (event_id, device_id, severity) | Sí |
| AC-003 | Tap en notificación → abre app en detalle del evento (deep link) | Sí |
| AC-004 | Configuración en app: habilitar/deshabilitar, horario silencio, severidad mínima | Sí |

### Dependencias

| HU / Artefacto | Tipo | Descripción |
|----------------|------|-------------|
| HU-APP-001 | Bloqueante | Auth + token push |
| HU-API-009 | Bloqueante | Backend envía push |
| RF-MON-01,04 | Requisito | Base funcional |

---

## HU-APP-003: Resumen IA y reporte en app

> **Repo:** APP | **Sprint:** 5 | **SP:** 5 | **MoSCoW:** Should
> **Épica:** Analítica y reportes | **Feature:** FEA-APP-REPORT

### Historia
**Como** usuario en app móvil
**quiero** ver resumen IA y generar reporte PDF
**para** tener insights en movimiento.

### Criterios de Aceptación

| ID | Criterio | Testeable |
|----|----------|-----------|
| AC-001 | Pantalla "Analítica" muestra resumen IA (cache 1h) + botón "Actualizar" | Sí |
| AC-002 | Botón "Generar reporte" → descarga PDF en almacenamiento local / share sheet | Sí |
| AC-003 | Offline-first: muestra último resumen cacheado si sin red | Sí |

### Dependencias

| HU / Artefacto | Tipo | Descripción |
|----------------|------|-------------|
| HU-API-011 | Bloqueante | Endpoints IA/reporte |
| RF-ANA-03,04 | Requisito | Base funcional |

---

## Trazabilidad Consolidada HU → RF → Épica → Repo

| HU ID | Título | Repo | RFs Cubiertos | Épica | Sprint | SP | Prioridad |
|-------|--------|------|---------------|-------|--------|----|-----------|
| HU-API-SETUP | Setup proyecto API | API | — | Setup y Foundation | 0 | 8 | Must |
| HU-DB-001a | Esquemas BD + changelog maestro | DB | Todos RF-* | Setup y Foundation | 0 | 8 | Must |
| HU-DB-001b | Seed data, rollback, validación | DB | RF-SEC-*, RF-PAR-* | Setup y Foundation | 0 | 5 | Must |
| HU-API-001 | Auth y sesión | API | RF-SEC-01..04,09 | Seguridad y cuentas | 1 | 8 | Must |
| HU-API-002 | Recuperación/actualización cuenta | API | RF-SEC-05,08 | Seguridad y cuentas | 1 | 5 | Must |
| HU-API-003 | RBAC | API | RF-SEC-10 | Seguridad y cuentas | 1 | 5 | Must |
| HU-API-004 | CRUD catálogos | API, DB | RF-PAR-01..03 | Parametrización | 1 | 8 | Must |
| HU-API-005 | Device config remota | API, DB | RF-DEV-03, RF-TEL-05, RF-EDGE-11 | Gestión disp. | 2 | 5 | Must |
| HU-API-006 | Gestión dispositivos | API, DB | RF-DEV-01..05 | Gestión disp. | 1 | 8 | Must |
| HU-API-007 | Ingesta eventos/evidencia | API, DB | RF-TEL-01..03, RF-EDGE-08 | Telemetría | 2 | 13 | Must |
| HU-API-008 | Consulta eventos | API, DB | RF-TEL-06 | Telemetría | 2 | 5 | Must |
| HU-API-009 | Push eventos críticos | API | RF-MON-01..04 | Monitoreo | 3 | 8 | Must |
| HU-API-010 | Timeline + métricas | API, DB | RF-ANA-01,02 | Analítica | 4 | 8 | Must |
| HU-API-011 | Resumen IA + reporte | API | RF-ANA-03,04 | Analítica | 5 | 13 | Should |
| HU-DB-002 | Vistas/índices analytics | DB | RF-ANA-01,02 | Analítica | 3 | 5 | Must |
| HU-DEVICE-001 | Detección visión | DEVICE | RF-EDGE-04,05,06 | Telemetría | 2-3 | 21 | Must |
| HU-DEVICE-002 | Init + estados + heartbeat | DEVICE | RF-EDGE-01,02,03,09,11 | Telemetría | 1 | 8 | Must |
| HU-DEVICE-003 | Buffer offline + sync | DEVICE | RF-EDGE-08,10,12, RF-TEL-04,07 | Telemetría | 2 | 13 | Must |
| HU-DEVICE-004 | Alertas sonoras + escalamiento | DEVICE | RF-EDGE-07,11 | Telemetría | 2 | 5 | Must |
| HU-PORTAL-001 | Auth portal (login, register, reset) | PORTAL | RF-SEC-01..05,08,09 | Seguridad y cuentas | 1 | 8 | Must |
| HU-PORTAL-002 | Dashboard dispositivos/eventos | PORTAL | RF-DEV-06, RF-TEL-06 | Gestión/Telemetría | 3 | 8 | Must |
| HU-PORTAL-003 | Métricas y tendencias | PORTAL | RF-ANA-01,02 | Analítica | 4 | 5 | Must |
| HU-PORTAL-004 | Resumen IA + reporte | PORTAL | RF-ANA-03,04 | Analítica | 5 | 8 | Should |
| HU-APP-001 | Auth app (login, register, reset) | APP | RF-SEC-01..05,08,09 | Seguridad y cuentas | 1 | 8 | Must |
| HU-APP-002 | Push notifications | APP | RF-MON-01..04 | Monitoreo | 3 | 8 | Must |
| HU-APP-003 | Resumen IA + reporte app | APP | RF-ANA-03,04 | Analítica | 5 | 5 | Should |

---

## Trazabilidad Post-MVP (Could)

| HU ID | Título | Repo | RFs Cubiertos | Épica | Sprint | SP | Prioridad |
|-------|--------|------|---------------|-------|--------|----|-----------|
| HU-API-012 | Video streaming WebRTC | API, DEVICE | RF-ANA-05, RF-EDGE-13 | Analítica | Post-MVP | 13 | Could |
| HU-DEVICE-005 | Video streaming WebRTC | DEVICE | RF-EDGE-13, RF-ANA-05 | Analítica | Post-MVP | 13 | Could |
| HU-PORTAL-004 | Video en vivo embebido | PORTAL | RF-ANA-05 | Analítica | Post-MVP | 5 | Could |
| HU-APP-004 | Video en vivo en app | APP | RF-ANA-05 | Analítica | Post-MVP | 8 | Could |

**Total Story Points MVP: 185** (Must: 148, Should: 37) | **Post-MVP: 39 SP** (Could)

---

## Próximos Pasos

1. **Validar HUs** con PO + Tech Leads (reunión 1h).
2. **Crear issues en GitHub Projects** desde esta tabla (copiar ID, título, ACs, SP, labels).
3. **Actualizar `traceability-matrix.md`** con matriz completa RF ↔ HU ↔ Módulo ↔ Prueba ↔ ADR.
4. **Definir `cross-cutting.md`** (reglas transversales: auth, audit, obs, idempotencia, tz, errores).
5. **Escribir ADRs** (ADR-001..007) en `docs/05-architecture/decisions/records/`.