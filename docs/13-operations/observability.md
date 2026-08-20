<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Observabilidad

**Estado:** En progreso
**Fecha:** 2026-08-19

</div>

</div>

Cómo se observa SomnGuard a través de los tres pilares —logs, métricas y trazas— más healthchecks y alertas. Alineado con la sección 15 del documento de arquitectura y con `platform/observability`.

## Fuentes de observabilidad

| Fuente | Naturaleza | Estado hoy |
|--------|------------|------------|
| Backend (Java/Spring Boot) | Logs, métricas, healthchecks | Modelado; activo con la capa de aplicación |
| PostgreSQL 16 | Logs, `pg_stat_*`, `pg_isready` | Disponible hoy |
| Liquibase (`databasechangelog`) | Trazabilidad de migraciones | Disponible hoy |
| `audit_login` | Auditoría de accesos (append-only) | Modelado; activo con la capa de aplicación |
| `analytics` (métricas y reportes) | Observabilidad de negocio | Modelado |

> **Distinción importante:** `monitoring` y `analytics` son observabilidad del **negocio** (notificaciones, métricas de conducción), no reemplazan la observabilidad **técnica** (latencia, errores 5xx, saturación) de la plataforma. Ambas coexisten.

## Pilar 1 — Logs

Formato de log **estructurado en JSON** con campos mínimos por evento: `timestamp`, `level` (INFO/WARN/ERROR), `service` (ej. `somnguard-backend`), `requestId`, `userId` si aplica, `module` (módulo del monolito), `message` y `context`. Detalle y política de retención por ambiente: ver [_template-observability.md](./_template-observability.md).

## Pilar 2 — Métricas

- **RED** por endpoint (Rate, Errors, Duration p95) para la API.
- **USE** por recurso (Utilización, Saturación, Errores de CPU, memoria y pool de conexiones).
- Métricas de BD: conexiones activas vs. límite del pool, consultas lentas (`pg_stat_activity`), tamaño y disco.
- `analytics` expone métricas de dominio (eventos por severidad, alertas emitidas, notificaciones) que **no** son métricas de plataforma.

## Pilar 3 — Trazas

- Al existir comunicación entre módulos (puertos) y AMQP, se instrumentarán trazas con propagación de contexto (W3C TraceContext) y correlación por `event_id`/`requestId`.
- Herramienta APM: **punto abierto**.

## Healthchecks

| Endpoint | Qué verifica | Respuesta esperada |
|----------|--------------|--------------------|
| `GET /health` | Disponibilidad básica del proceso | `200 { "status": "ok" }` |
| `GET /health/ready` | Disponibilidad + dependencias (BD, MinIO) | `200 { "status": "ready" }` |
| `GET /health/live` | Proceso vivo (sin dependencias) | `200 { "status": "alive" }` |

BD: `docker compose --env-file .env.develop exec postgres pg_isready -U <user> -d <db>` → "accepting connections".

## Alertas

Marco de alertas (umbrales de referencia hasta fijar SLOs reales):

| Alerta | Condición (referencia) | Severidad | Aplica hoy |
|--------|------------------------|-----------|------------|
| PostgreSQL no responde | `pg_isready` falla | P0 | Listo |
| BD sin espacio | Disco > 85% | P1 | Listo |
| Migración fallida | `update` de un módulo termina en error | P1 | Listo |
| Alta tasa de errores 5xx | Error rate > umbral | P1 | Pendiente: con aplicación |
| Latencia alta | p95 > SLO | P2 | Pendiente: con aplicación |

El enrutamiento de alertas y la herramienta de alerting son un **punto abierto** (ver [incident-management.md](./incident-management.md)).

## Puntos abiertos

- Stack concreto de logs/métricas/trazas: sin decisión tomada.
- Umbrales definitivos: dependen de los SLO (usar [_template-sla-slo-sli.md](./_template-sla-slo-sli.md)).
- Dashboards: por crear cuando exista el stack.

## Referencias

- [_template-observability.md](./_template-observability.md)
- [incident-management.md](./incident-management.md)
- [backup-and-recovery.md](./backup-and-recovery.md)
- [Requisitos no funcionales](../04-requeriments/non-functional.md)
- [Documento de arquitectura](../05-architecture/architecture-document.md#15-observabilidad-y-auditoría)