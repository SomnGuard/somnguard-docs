# Observabilidad — estado implementado

**Estado:** Baseline parcial implementado; centralización pendiente
**Fecha de actualización:** 2026-11-03

Este documento registra las capacidades que existen en los repositorios. La decisión vigente es [ADR-014](../05-architecture/decisions/records/ADR-014-observability-current-baseline.md). El stack completo OTel + LGTM descrito anteriormente fue reemplazado; su propuesta histórica permanece en [ADR-007](../05-architecture/decisions/records/ADR-007-observability-otel-lgtm.md).

## Capacidades actuales

| Componente | Capacidad verificable | Repositorio / fuente | Límite actual |
|---|---|---|---|
| API | Spring Actuator habilita `health`, `info`, `prometheus` y `metrics` bajo `/actuator`. | `somnguard-api/src/main/resources/application.yml` | No se encontró scraper Prometheus ni backend central desplegable en los repositorios revisados. |
| Métricas API | `micrometer-registry-prometheus` exporta métricas en formato Prometheus. | `somnguard-api/pom.xml` | No hay dashboards ni reglas Alertmanager versionados. |
| PostgreSQL | Docker healthcheck con `pg_isready`; Liquibase conserva historial de migraciones. | `somnguard-db/docker-compose.yml`, changelog | No sustituye métricas centralizadas de base de datos. |
| Logs de proceso | Se pueden consultar desde el runtime del servicio (por ejemplo, `docker logs` para contenedores). | Compose de cada servicio | No hay colector y almacenamiento central de logs configurados. |
| Trazas distribuidas | Sin implementación de exportación OTel/OTLP identificada en los repositorios revisados. | — | Propagación, collector y backend de trazas pendientes. |

## Endpoints del API

La configuración actual declara los siguientes endpoints Actuator:

| Endpoint | Uso |
|---|---|
| `GET /actuator/health` | Estado general del API y dependencias registradas por Spring |
| `GET /actuator/health/liveness` | Probe de liveness de Spring |
| `GET /actuator/health/readiness` | Probe de readiness de Spring |
| `GET /actuator/info` | Información de la aplicación |
| `GET /actuator/prometheus` | Métricas en formato Prometheus |
| `GET /actuator/metrics` | Métricas Micrometer |

La exposición se configura en `somnguard-api/src/main/resources/application.yml`. Antes de publicar estos endpoints fuera de la red interna, QA/DevOps debe verificar autenticación, reglas del proxy y datos que expone cada endpoint para el perfil del ambiente.

## Desarrollo local

- Revisar la salud del API en `http://localhost:8080/actuator/health`.
- Consultar métricas en `http://localhost:8080/actuator/metrics` y `http://localhost:8080/actuator/prometheus` cuando el API esté iniciado.
- Consultar logs según el modo de ejecución: consola del proceso Maven o `docker logs <contenedor>`.
- No se requiere ni se inicia Grafana, Loki, Tempo u OTel Collector en el setup local actual.

## Estado de la decisión LGTM

La ADR-007 había propuesto OpenTelemetry Collector, Tempo, Prometheus, Loki, Grafana y Alertmanager. Esos servicios no están configurados en los repositorios revisados, por lo que no se presentan como parte del runtime actual. Para implementarlos, primero se debe acordar el alcance y después versionar la configuración, retención, controles de acceso, dashboards, alertas y pruebas de extremo a extremo en un repositorio de infraestructura.

## Referencias

- [ADR-014: baseline actual de observabilidad](../05-architecture/decisions/records/ADR-014-observability-current-baseline.md)
- [ADR-007 reemplazada: propuesta OTel + LGTM](../05-architecture/decisions/records/ADR-007-observability-otel-lgtm.md)
- [Actuator y métricas del API](https://github.com/SomnGuard/somnguard-api/blob/develop/README.md)
- [Arquitectura](../05-architecture/architecture-document.md)
