<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Mapa de Dependencias Hexagonal SomnGuard

**Estado:** Borrador  
**Fecha:** 2026-08-22

</div>

</div>

> **Objetivo:** Visualizar y validar que las dependencias entre los 6 módulos del backend, el dominio central y los repos (API, DB, APP, PORTAL, DEVICE) siguen la arquitectura hexagonal y no hay ciclos de FK ni acoplamiento inverso que bloqueen el desarrollo paralelo.

> **Fuente de verdad:** ADR-002 (arquitectura hexagonal), module-catalog.md, cross-cutting.md, data-dictionary.md, modeling-conventions.md, entities-and-rules.md

---

## 1. Diagrama de Dependencias

```
                          +----------------------+
                          |   platform/transversal|
                          |  (logging, obs, error |
                          |   handling, secrets)  |
                          +----------+------------+
                                     ^
                                     | (puede ser usado por cualquier módulo)
+----------------------+---------+---------+----------------------+
|                      |         |         |                      |
+--------+-------------+---------+---------+--------+---------+
|  security          |parameterization|device_management| telemetry_service | monitoring|
| (módulo base)      | (catálogos)    | (devices)  | (ingesta ev.) | (notific.)|
|  RN-01..03         | event_cat.,    |  device↔   |  events,     |  notifs   |
|  auth, audit       | severity,      |  user,     |  alert_log   |          |
|  JWT, API Key      | media_type,    |  config    |  FKs a device|  RF-MON-*
+--------------------+------------+------------+------------+-----------+
         ^                  ^                  ^                  ^
         |                  |                  |                  |
         |                  |                  |                  |
         +--------+-----------+----------+---------+---------
                  |                   |              |
                  |                   |              |
                  +-----------+-----------+--------+----------
                              |              |
                              |              |
                      +---------+-----------+   V
                      |  analytics          |  vistas/materializadas
                      |  (línea de tiempo, |
                      |   métricas, reportes|
                      +--------------------+

                      ↑
                      |
              +---------+----------+
              |   domain (centro)  |
              |  Entidades RN-*    |
              |  Value objects     |
              |  Reglas deNegocio  |
              +--------------------+

                           ↑
                           |
              +---------+----------+
              |  adapters/ports    |
              |  por cada módulo   |
              +--------------------+
```

### Leyenda de flechas:

| Símbolo | Significado |
|---------|-------------|
| `→` | Dependencia de uso (el módulo de la izquierda usa el de la derecha) |
| `↘` | Dependencia de implementación (adapter implementa port del módulo de abajo) |
| `↓` | Datos fluye hacia abajo (ej. eventos device→API→BD) |
| `↕` | Capacidad de ser usado por (platform/transversal puede usarse de cualquier módulo) |

---

## 2. Matriz de Dependencias por Módulo

| Módulo | Depende de | Para qué | Que depende de él | Por qué |
|--------|------------|----------|-------------------|---------|
| **security** | Ninguna (base) | Autenticación, autorización, auditoría | parameterization, device_management, telemetry_service, monitoring | Proporciona: users, roles, features, JWT, API Keys base |
| **parameterization** | Independiente | Catálogos configurables | device_management, telemetry_service, monitoring | Catálogos autónomos; FKs internos a parameterization (severity, sound_pattern, event_type, status_category, status) |
| **device_management** | `security` + `parameterization` | Alta/asignación de devices, config remota | telemetry_service | devices → user asignation; config con catálogos vigentes |
| **telemetry_service** | `device_management` + `parameterization` | Ingesta events/evidence de devices | monitoring | events → device FK; event_type/severity FKs a catálogos; audit_login para logging |
| **monitoring** | `telemetry_service` | Notificaciones, tracking delivery | analytics | notifications → alert_log FK; notification_delivery tracking; usuario FK a security.user |
| **analytics** | `todos los anteriores` (solo lectura) | Líneas de tiempo, métricas, reportes IA | Ningún módulo (es de solo lectura) | Vistas materializadas sobre tables de security+parameterization+device_management+telemetry+monitoring |
| **platform/transversal** | Ninguna (es independiente) | Errores, logging, observabilidad, secrets | Cualquier módulo | JWT RS256, logs JSON, métricas RED, trazas OTel, X-Request-ID, manejo de errores RFC 7807 |

### 2.1 Regla de Oro (ADR-002)

```
domain NUNCA depende de:
- Spring / JPA / Hibernate
- HTTP / frameworks web
- Controladoras REST (controllers)
- Configuraciones de despliegue

SIEMPRE depende de:
- MISMO dominio (mismo módulo)
- Ningún módulo externo (es el origen)
```

### 2.2 Dirección de Dependencias por Capa

```
Capa interna (más cercana a domain):
  domain (entidades RN-*, PK UUID, auditoría estándar)
  
  ↓ depende de
  
application/port/in (use case interfaces por módulo):
  - security: login, logout, RBAC
  - parameterization: CRUD catálogos, obtener estado device
  - device_management: alta device, asociar usuario, heartbeat
  - telemetry_service: ingestar eventos, pull config
  - monitoring: enviar notificación, tracking delivery

  ↓ depende de

application/port/out (output ports/Repositories por módulo):
  - security: user repository, role repository, password reset
  - parameterization: event_category repository, severity lookup
  - device_management: device repository, config repository
  - telemetry_service: event repository, evidence repository, alert_log repository
  - monitoring: notification repository, notification_delivery repository

  ↓ depende de

adapter/out (implementaciones concretas):
  - security: JPA/Spring Data repos, JWT validator, API Key validator
  - parameterization: Liquibase changesets, catálogo seeds
  - device_management: SQLite repos (device), API client for config sync
  - telemetry_service: Liquibase tables, MinIO client for evidence, PostgreSQL repos
  - monitoring: PostgreSQL repos, Redis cache, email/push SDKs

Capa externa:
  platform/transversal (puede ser usado por TODAS las capas anteriores):
  - error handling (RFC 7807 Problem Details)
  - observabilidad (OpenTelemetry, logs JSON, métricas RED)
  - seguridad (JWT RS256 verification, API Key validation)
  - gestión de errores y excepciones
```

---

## 3. Mapa de Dependencias por Repos (5 repos del proyecto)

### 3.1 `somnguard-api` (backend Java + Spring Boot)

**Depende de (todas sus capas internas):**

| Capa | Dependencias concretas |
|------|-----------------------|
| `domain` | RN-01..RN-10 (solo lectura, no código) |
| `application/port/in` | Use cases: auth, device mgmt, telemetry ingestion, notifications |
| `application/port/out` | Repositories: user repo, device repo, event repo, notification repo |
| `adapter/in/web` | Controllers REST, DTOs, mappers (usando ports de arriba) |
| `platform/transversal` | Spring Security (JWT RS256 validator, API Key validator), OpenTelemetry auto-instrumentation, Micrometer RED metrics, global exception handler (RFC 7807) |

**No debe depender de:**
- Módulos que no le correspondan (ej. no debería importar lógica de device_management si solo expone auth)
- Tables de otros módulos vía JPA directas (usar ports/adapters)

### 3.2 `somnguard-db` (PostgreSQL + Liquibase)

**Depende de (orden de ejecución obligado):**

| Módulo | Changelog | Por qué |
|--------|-----------|---------|
| `security` | `01_ddl/03_tables` + `02_dml/00_inserts` | Tabla base: users, roles, features - debe aplicarse primero |
| `parameterization` | `01_ddl/03_tables` + `02_dml/00_inserts` | Catálogos con FK a security.user/role - debe aplicarse después |
| `device_management` | `01_ddl/03_tables` + `02_dml/00_inserts` | Devices → user FK, config con catálogos - después de security + parameterization |
| `telemetry_service` | `01_ddl/03_tables` + `02_dml/00_inserts` | Events → device FK, event_type/severity FKs - después de los 3 anteriores |
| `monitoring` | `01_ddl/03_tables` + `02_dml/00_inserts` | Notifications → alert_log FK, user FK - después de telemetry_service |
| `analytics` | `01_ddl/03_tables` + `02_dml/00_inserts` (solo vistas) | Vistas materializadas sobre tables de todos los anteriores - último |

**No debe tener cambios inline en yaml:** Todo vía `sqlFile` a archivos `.sql` externos (validado en migration-strategy.md creado previamente).

### 3.3 `somnguard-portal` (frontend Vite + Nginx)

**Depende solo de endpoints API:**

| Funcionalidad | Endpoint API | Módulo origen |
|---------------|--------------|---------------|
| Listar devices | `GET /devices` | device_management |
| Timeline eventos | `GET /telemetry/events` | telemetry_service |
| Métricas y filtros | `GET /analytics/timeline` | analytics |
| Config device | `GET /devices/{id}/config` | telemetry_service + device_management |
| Notificaciones | `GET /notifications` | monitoring |
| Auth (JWT) | `POST /auth/login` | security |

**No depende directamente de BD ni de módulos internos:** Todo por API. Si necesita datos nuevos, pide nuevo endpoint al equipo API.

### 3.4 `somnguard-app` (móvil React Native / Expo)

**Depende solo de endpoints API (igual que portal):**

| Funcionalidad | Endpoint API | Módulo origen |
|---------------|--------------|---------------|
| Login / JWT | `POST /auth/login` | security |
| Recibir push notifications | `GET /notifications` | monitoring |
| Ver timeline eventos | `GET /analytics/timeline` | analytics |
| Streaming WebRTC | `GET /devices/{id}/stream` | API + device |
| Config device | `GET /devices/{id}/config` | telemetry_service |

**Modo offline-first:** Al no tener red, usa SQLite local (propio al app) y sincroniza cuando hay conexión. No depende de SQLite del device.

### 3.5 `somnguard-device` (Python en Raspberry Pi/Edge)

**Arquitectura particular: Offline-first (ADR-005)**

| Componente | Dependencia | Descripción |
|------------|-------------|-------------|
| `SQLite local` | Ninguna (autónoma) | Buffer de eventos pending_events, device_config, evidence local |
| `API Key` | `security` (validación central) | API Key generada por security, validada HMAC-SHA256 en device |
| `Heartbeat → API` | `telemetry_service` (endpoint POST) | Cada 30s envía status, último evento, buffer stats |
| `Config pull → API` | `telemetry_service` (GET /devices/{id}/config) | Solicita device_config JSONB, thresholds, sound_pattern, volumen |
| `Event sync → API` | `telemetry_service` (POST /telemetry/events) | Envía lote de eventos offline, deduplicación por event_id UUID v7 |
| `Local DB esquema` | Mismo patrón que BD `security + parameterization + device_management` (tabla mínima) | Solo lo necesario: device, pending_events, device_config (recorta las 6 esquemas completos) |

**No depende de:** JPA, Spring, HTTP frameworks. Usa `httpx` para calls HTTP simples a API gateway (Traefik en puerto 8080).

---

## 4. Ciclo de Desarrollo y Validación de Dependencias

### 4.1 Checklist por PR (nuevo módulo o cambio)

| # | Validación | Quien la firma | Comentario |
|---|------------|----------------|------------|
| 1 | Nuevo port/adapter no rompe dependencias existentes | Tech Lead módulo afectado | Revisar que no haya `import` circulares |
| 2 | FKs cruzadas entre módulos están documentadas | DBA | Agregar a `migration-strategy.md` sección 4 |
| 3 | Nuevo endpoint API no viola contrato hexagonal | API Team | Revisar en `cross-cutting.md` y `guidelines.md` |
| 4 | Cambio en entidad domain (RN-*) no requiere BD migration innecesaria | Backend Lead | Verificar en `data-dictionary.md` y `modeling-conventions.md` |
| 5 | Modificación en plataforma/transversal no rompe módulos existentes | Platform Team | Revisar `cross-cutting.md` sección 11 |
| 6 | Device mock sync es idempotente con API real | Device Team | Probar en `somnguard-device` `--profile device-mock` |
| 7 | Analytics views reflejan cambios en tablas base | DBA + Analytics Team | `refresh` vistas materializadas después de migración |

### 4.2 Herramientas de validación

```bash
# 1. Verificar que no hay import circulares en el código (Java Spring)
#    - Revisar annotations @Repository, @Service en paquetes correctos
#    - Ejecutar: mvn dependency:tree -Dincludes=org.springframework

# 2. Validar orden de migraciones Liquibase
docker compose --profile tooling run --rm liquibase update --fail-on-missing-liker

# 3. Revisar que todos los módulos tengan su contexto (contextFilter) en changelog.yaml
grep -R "contextFilter" somnguard-db/01_ddl/ somnguard-db/02_dml/

# 4. Verificar que domain no tiene dependencias a frameworks
#    - Revisar que ningún archivo .java en domain/ importe org.springframework.*
#    - Esta revisión se hace en el repo somnguard-api, no en somnguard-db
```

### 4.3 Diagrama Resumido para README

```
somnguard dependency map (resumido):

security ──▶ parameterization ──▶ device_management ──▶ telemetry_service ──▶ monitoring ──▶ analytics
     │                │                  │                      │                │
     ▼                ▼                  ▼                      ▼                ▼
  JWT, API Key   catálogos      device↔user         events, evidence  notifications  métricas/Timeline
  auth, audit                      config                alert_log         RF-MON-*     RF-ANA-*
                                                   ↑
                                                   │
                                              analytics
                                              (vistas materializadas)
```

---

## 5. Referencias Cruzadas

| Documento | Sección | Qué aporta al mapa |
|-----------|---------|-------------------|
| `ADR-002` | `05-architecture/decisions/records/ADR-002-hexagonal-architecture.md` | Define regla de dependencias hacia adentro (domain es centro) |
| `module-catalog.md` | `09-modules/module-catalog.md` | Lista los 6 módulos y su responsabilidad |
| `cross-cutting.md` | `05-architecture/cross-cutting.md` | Estándares transversales que aplican a todos los módulos |
| `data-dictionary.md` | `06-data-architecture/data-dictionary.md` | Convenciones de naming, PK, auditoría usadas en todas tablas |
| `modeling-conventions.md` | `06-data-architecture/modeling-conventions.md` | Estructura DDL, orden Liquibase, FKs, índices por módulo |
| `entities-and-rules.md` | `02-domain/entities-and-rules.md` | Reglas de negocio RN-* que definen qué entidades existen y sus relaciones |
| `guidelines.md` | `07-api-design/guidelines.md` | Contratos API por módulo y estándares de petición/respuesta |
| `ADR-001` | `05-architecture/decisions/records/ADR-001-backend-java-spring-boot.md` | JWT RS256 + API Keys (usado por security y device) |
| `ADR-009` | `05-architecture/decisions/records/ADR-009-status-parametrized-audit.md` | `status_category` + `status` en todas las entidades (regla transversal) |
| `ci-cd-strategy.md` | `10-devops/ci-cd-strategy.md` | Validación de dependencias en PRs y pipelines |

---

## Próximos Pasos

1. **Validar** este mapa con Architecture Team y DBA (revisión 45 min) - asegurar que no falten módulos o dependencias ocultas
2. **Añadir** a `LISTA_DOCS_OTRO-PROJECT-PARA-SOMNGUARD.md` como entregable de PRIORIDAD 2 (junto con `migration-strategy.md`)
3. **Integrar** en la `ci-cd-strategy.md` validación automática de dependencias en cada PR merged
4. **Revisar** con cada equipo de repos (API, DB, PORTAL, APP, DEVICE) que el mapa se ajusta a su realidad actual
5. **Actualizar** cuando haya nuevos módulos, ADRs o cambios en la arquitectura hexagonal (verificar que no se introduzcan ciclos de dependencia)
6. **Usar** este mapa como guía para definir nuevos ports/adapters sin romper la arquitectura hexagonal existente