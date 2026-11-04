# ADR-014: Baseline actual de observabilidad

**Estado:** Aceptada
**Fecha:** 2026-11-03
**Autores:** Equipo SomnGuard
**Equipos involucrados:** Arquitectura, Backend, Device, DevOps

---

## Contexto

La ADR-007 definió OTel + LGTM como un stack completo, pero los repositorios revisados no contienen un despliegue de OTel Collector, Tempo, Loki, Prometheus server, Grafana ni Alertmanager. El API sí incluye Actuator, Micrometer y exportación en formato Prometheus. Se necesita registrar la capacidad existente y evitar que la documentación presente como desplegado el stack propuesto.

## Decisión

Para el estado actual del proyecto, la observabilidad se limita a las capacidades verificables del proceso y sus herramientas de ejecución:

- El API expone Actuator en `/actuator`, con `health`, `info`, `prometheus` y `metrics`, configurados en `somnguard-api/src/main/resources/application.yml`.
- El API incluye `micrometer-registry-prometheus` en `somnguard-api/pom.xml`; `/actuator/prometheus` exporta métricas para un scraper compatible cuando se configure.
- Cuando se ejecuta en contenedor, la salida del proceso puede consultarse mediante los logs del runtime. Esto no implica almacenamiento ni búsqueda centralizados.
- No hay backend central de logs, almacenamiento/trazado distribuido, dashboards ni reglas de alertas versionados en los repositorios revisados.

La ADR-007 queda reemplazada como decisión del estado actual. Su diseño OTel + LGTM se conserva únicamente como antecedente; retomarlo requiere una nueva ADR, configuración desplegable, credenciales/retención y evidencia de pruebas.

## Consecuencias

### Positivas

- La documentación distingue entre endpoints implementados y servicios que aún no existen en los despliegues versionados.
- Se puede revisar salud y métricas del API sin afirmar que hay dashboards o alertas centralizados.

### Negativas / Trade-offs

- Sin scraper, dashboards y reglas de alerta, las métricas no se recopilan ni notifican de forma centralizada.
- Sin backend de trazas y logs centralizados, la investigación de incidentes entre componentes sigue siendo manual.

### Pendientes

- Acordar si se implementará un stack centralizado; en ese caso, incorporar sus archivos de configuración a un repositorio y validar recepción, retención, dashboards y alertas.
- Definir autenticación y acceso a endpoints Actuator en cada ambiente antes de exponerlos fuera de la red interna.

## Alternativas consideradas

| Alternativa | Resultado |
|---|---|
| Tratar OTel + LGTM de ADR-007 como stack desplegado | Descartada: no hay configuración de despliegue verificable en los repositorios revisados. |
| Retirar toda referencia a observabilidad | Descartada: el API sí implementa healthchecks y métricas Micrometer/Prometheus. |

## Referencias

- [ADR-007 reemplazada](ADR-007-observability-otel-lgtm.md)
- API: `somnguard-api/src/main/resources/application.yml`
- API: `somnguard-api/pom.xml`
- [Manual técnico de observabilidad](../../../13-operations/observability.md)
