# ADR-007: Observabilidad — OpenTelemetry + LGTM Stack (Tempo, Prometheus, Loki, Grafana)

**Estado:** Aceptada
**Fecha:** 2026-08-22
**Autores:** Equipo SomnGuard
**Equipos involucrados:** Arquitectura, Backend, Device, DevOps, SRE

---

## Contexto

SomnGuard es un sistema distribuido con componentes heterogéneos:
- **Backend API** (Java 21 + Spring Boot) — monolito modular, 7 módulos
- **Portal Web** (React + Vite) — SPA servida por Nginx
- **App Móvil** (React Native + Expo) — cliente nativo
- **Device Edge** (Python en Raspberry Pi) — offline-first, sync periódico
- **Infraestructura** — PostgreSQL, MinIO, Redis, Traefik

Requisitos observabilidad (SRS RNF-1, RNF-4, cross-cutting.md):
- **Logs estructurados** correlacionables (trace_id, span_id)
- **Métricas RED** (Rate, Errors, Duration) por endpoint/módulo
- **Trazas distribuidas** end-to-end (device→API→DB→MinIO)
- **Alertas** proactivas (latencia, errores, saturación, device offline)
- **Dashboards** por rol (dev, ops, business)
- **Costo controlado** (open source, self-hosted)

---

## Decisión

### Stack LGTM (Grafana Stack) + OpenTelemetry

| Componente | Tecnología | Versión | Propósito |
|------------|------------|---------|-----------|
| **Instrumentación** | OpenTelemetry (OTel) | 1.30+ | SDKs + Auto-instrumentation; vendor-neutral |
| **Collector** | OTel Collector Contrib | 0.109+ | Recibe (OTLP, Prometheus), procesa, exporta |
| **Traces** | **Tempo** | 2.6+ | Almacena trazas (columnar, barato, integrado Grafana) |
| **Métricas** | **Prometheus** | 2.54+ | Pull model, PromQL, alerting, retención 15d |
| **Logs** | **Loki** | 3.2+ | Indexado por labels (no full-text), barato, multi-tenant |
| **Visualización** | **Grafana** | 11.2+ | Dashboards, alerting UI, exploración unificada |

> **Principio:** **OpenTelemetry como única fuente de telemetría**. Nada de agentes propietarios (Datadog, New Relic, etc.). Exportadores configurables.

---

### 1. Instrumentación por Componente

| Componente | Cómo se instrumenta | Qué emite |
|------------|---------------------|-----------|
| **API (Java)** | **Auto-instrumentation Java Agent** (`opentelemetry-javaagent.jar`) + spans manuales en use cases clave | Traces (HTTP server, JDBC, Redis, MinIO), Métricas (JVM, HTTP server, custom), Logs (logback → OTLP) |
| **Portal (React)** | `@opentelemetry/sdk-trace-web` + `@opentelemetry/exporter-collector` | Traces (navegación, fetch, errores JS), Web Vitals (LCP, FID, CLS) |
| **App (React Native)** | `react-native-otel` (expo compatible) | Traces (navegación, network), crashes, ANR |
| **Device (Python)** | `opentelemetry-sdk` + `opentelemetry-exporter-otlp` | Traces (detección, sync, errores), Métricas (eventos/día, buffer size, sync latency), Logs (JSON → stdout → Loki via Promtail) |
| **PostgreSQL** | `postgres_exporter` (Prometheus) | Métricas: conexiones, queries, cache, replication, tamaño |
| **MinIO** | Built-in Prometheus endpoint (`/minio/v2/metrics/cluster`) | Métricas: requests, bandwidth, disk, replication |
| **Redis** | `redis_exporter` | Métricas: memoria, conexiones, comandos, keyspace |
| **Traefik** | Built-in Prometheus metrics (`/metrics`) | Métricas: requests, latency, certificados, backends |
| **Node/Host** | `node_exporter` | Métricas: CPU, memoria, disco, red, filesystem |

### 2. OTel Collector — Configuración Central

```yaml
# observability/otel-collector.yaml (en somnguard-docker-infra)
receivers:
  otlp:
    protocols:
      grpc: { endpoint: "0.0.0.0:4317" }
      http: { endpoint: "0.0.0.0:4318" }
  prometheus:
    config:
      scrape_configs:
        - job_name: 'api'
          static_configs: [{ targets: ['api:8080'] }]
        - job_name: 'postgres'
          static_configs: [{ targets: ['postgres-exporter:9187'] }]
        - job_name: 'minio'
          static_configs: [{ targets: ['minio:9000'] }]
        - job_name: 'redis'
          static_configs: [{ targets: ['redis-exporter:9121'] }]
        - job_name: 'traefik'
          static_configs: [{ targets: ['traefik:8080'] }]
        - job_name: 'node'
          static_configs: [{ targets: ['node-exporter:9100'] }]

processors:
  batch: { timeout: 10s, send_batch_size: 1000 }
  memory_limiter: { limit_mib: 512 }
  resource:
    attributes:
      - key: "service.namespace"
        value: "somnguard"
        action: "upsert"
      - key: "deployment.environment"
        value: "${ENV:-local}"
        action: "upsert"

exporters:
  otlp/tempo:
    endpoint: "tempo:4317"
    tls: { insecure: true }
  prometheus:
    endpoint: "0.0.0.0:8889"
  loki:
    endpoint: "http://loki:3100/loki/api/v1/push"

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [memory_limiter, batch, resource]
      exporters: [otlp/tempo]
    metrics:
      receivers: [otlp, prometheus]
      processors: [memory_limiter, batch, resource]
      exporters: [prometheus]
    logs:
      receivers: [otlp]
      processors: [memory_limiter, batch, resource]
      exporters: [loki]
```

### 3. Correlación Unificada (Trace ID)

| Flujo | Propagación |
|-------|-------------|
| **Device → API** | Device genera `trace_id` (UUID v7) → header `traceparent: 00-${trace_id}-${span_id}-01` en `POST /telemetry/events` |
| **API internos** | Spring `ObservationFilter` propaga `traceparent` a JDBC, Redis, MinIO, HTTP client |
| **Portal/App → API** | Browser/App envía `traceparent` en cada fetch → API continua trace |
| **Logs** | `logback-spring.xml` + `logstash-logback-encoder` → `trace_id`, `span_id` en cada log JSON |
| **Device logs** | Python `logging` + `OTel LoggingHandler` → stdout → Promtail → Loki (mismo `trace_id`) |

### 4. Métricas Clave (RED + USE + Business)

#### RED por Endpoint (API)
| Métrica | Tipo | Labels |
|---------|------|--------|
| `http_requests_total` | Counter | `method`, `path`, `status`, `module` |
| `http_request_duration_seconds` | Histogram | `method`, `path`, `module` |
| `http_request_size_bytes` | Histogram | `method`, `path` |
| `http_response_size_bytes` | Histogram | `method`, `path` |

#### USE por Recurso
| Recurso | Métricas |
|---------|----------|
| **JVM** | `jvm_memory_used_bytes`, `jvm_gc_pause_seconds`, `jvm_threads_live` |
| **PostgreSQL** | `pg_stat_activity_count`, `pg_database_size_bytes`, `pg_xact_commit` |
| **Redis** | `redis_memory_used_bytes`, `redis_connected_clients`, `redis_keyspace_keys` |
| **MinIO** | `minio_cluster_total_usage_bytes`, `minio_node_disk_free_bytes` |

#### Business (Custom)
| Métrica | Descripción | Origen |
|---------|-------------|--------|
| `somnguard_events_ingested_total` | Eventos recibidos por device_id, event_type, severity | API (span manual) |
| `somnguard_sync_duration_seconds` | Latencia sync device (batch size, retries) | Device + API |
| `somnguard_devices_online` | Gauge: devices con heartbeat < 5 min | API (scheduler) |
| `somnguard_alerts_triggered_total` | Alertas AS-XX por device, tipo, severidad | API (alert_log insert) |
| `somnguard_reports_generated_total` | Reportes PDF/IA por usuario, periodo | Analytics module |

### 5. Dashboards Grafana (Mínimos Viables)

| Dashboard | Paneles Principales | Audiencia |
|-----------|---------------------|-----------|
| **SomnGuard - Overview** | RED global, Devices online, Events/sec, Alertas activas, Error rate | Todos |
| **SomnGuard - API Deep Dive** | RED por módulo, JVM, DB pool, Redis pool, MinIO latency, Traces flamegraph | Dev, SRE |
| **SomnGuard - Device Sync** | Devices online/offline, Sync success/failed rate, Buffer age (P50/P95), Config pull rate | Dev, SRE, Support |
| **SomnGuard - Business** | Events/day por tipo/severidad, Usuarios activos, Reportes generados, Alertas críticas | PO, Business |
| **SomnGuard - Infra** | CPU/Mem/Disco/Red por host, PG/Redis/MinIO/Traefik health | SRE |

### 6. Alertas (PrometheusRule — Ejemplos)

```yaml
groups:
- name: somnguard.rules
  rules:
  - alert: APIHighErrorRate
    expr: |
      sum(rate(http_requests_total{status=~"5.."}[5m])) by (module)
      /
      sum(rate(http_requests_total[5m])) by (module)
      > 0.05
    for: 2m
    labels: {severity: "critical"}
    annotations:
      summary: "High 5xx rate in {{ $labels.module }}"
      description: "{{ $value | humanizePercentage }} of requests failing"

  - alert: DeviceOffline
    expr: |
      time() - somnguard_devices_online > 300
    for: 5m
    labels: {severity: "warning"}
    annotations:
      summary: "Device {{ $labels.device_id }} offline > 5 min"

  - alert: SyncFailureRateHigh
    expr: |
      sum(rate(somnguard_sync_failed_total[15m])) by (device_id)
      /
      sum(rate(somnguard_sync_attempted_total[15m])) by (device_id)
      > 0.2
    for: 5m
    labels: {severity: "warning"}

  - alert: DiskSpaceCritical
    expr: |
      (node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"}) < 0.1
    for: 1m
    labels: {severity: "critical"}
```

### 7. Retención y Costos

| Dato | Retención | Estimación Tamaño (MVP) |
|------|-----------|-------------------------|
| **Traces (Tempo)** | 7 días | ~2 GB/día (100k spans/día × 20 KB) |
| **Métricas (Prometheus)** | 15 días | ~500 MB (scrape 15s, 200 series) |
| **Logs (Loki)** | 30 días | ~1 GB/día (JSON estructurado, compresión) |
| **Dashboards/Alerts** | Para siempre | Negligible |

---

## Consecuencias

### Positivas
- **Vendor-neutral:** OTel estándar CNCF; migrable a cualquier backend (Datadog, Honeycomb, Jaeger, etc.)
- **Costo predecible:** Self-hosted, open source; sin per-host/per-span pricing
- **Correlación real:** Un `trace_id` une device→API→DB→MinIO→Portal
- **Escalabilidad:** Tempo (object storage backend), Prometheus (remote write), Loki (boltdb-shipper) escalan horizontal
- **Ecosistema maduro:** Grafana dashboards, PrometheusRule, OTel Collector processors (transform, filter, batch)

### Negativas / Trade-offs
- **Complejidad operacional:** 5 servicios (OTel, Tempo, Prometheus, Loki, Grafana) + exporters
- **Curva de aprendizaje:** PromQL, LogQL, TraceQL, OTel Collector config
- **Recursos:** ~2-4 GB RAM + 2-4 CPU para stack completo en local (Docker Compose)
- **Sampling necesario:** 100% traces = costo alto → Sampling 10% + 100% errors (config en Collector)

### Riesgos
- **Cardinalidad explosiva en Prometheus** (labels: `device_id`, `user_id`, `event_id`) → Mitigación: **NO** usar high-cardinality labels en métricas; usar logs/traces para detalle
- **Tempo requiere object storage** (S3/MinIO/GCS) para producción → Mitigación: Local usa filesystem; prod usa MinIO/S3
- **Loki no indexa full-text** → Queries por label; diseñar labels bien (`service`, `module`, `level`, `device_id`)

---

## Alternativas consideradas

| Alternativa | Por qué se descartó |
|-------------|---------------------|
| **ELK/EFK (Elasticsearch, Fluentd, Kibana)** | Elasticsearch caro (RAM, licencia), complejo; Loki más barato para logs |
| **Jaeger (traces) + Prometheus + Loki** | Jaeger no escala tan bien como Tempo; Tempo + Grafana = integración nativa |
| **Datadog / New Relic / Dynatrace** | Coste prohibitivo para startup/académico; vendor lock-in |
| **Solo logs (sin traces/métricas)** | No permite debugging distribuido ni alertas proactivas RED |
| **Zipkin (traces)** | Menos features que Tempo; sin multi-tenancy nativo; comunidad menor |
| **Custom metrics en DB** | No time-series; no alerting nativo; no dashboards unificados |

---

## Referencias

- [cross-cutting.md](../cross-cutting.md#3-observabilidad-logs-métricas-trazas)
- [functional.md](../../../04-requeriments/functional.md) → RF-TEL-*, RF-MON-*, RF-ANA-*
- [architecture-document.md](../../architecture-document.md) §Observabilidad
- [local-setup.md](../../../10-devops/local-setup.md) (Docker Compose profiles)
- **OpenTelemetry:** https://opentelemetry.io/docs/
- **Grafana LGTM:** https://grafana.com/oss/lgtm/
- **OTel Collector Contrib:** https://github.com/open-telemetry/opentelemetry-collector-contrib
- **Spring Boot OTel:** https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.opentelemetry