<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Runbook — SOMNGUARD — [Nombre del componente]

**Estado:** Plantilla
**Fecha:** 2026-08-19

</div>

</div>

> **PLANTILLA** — Copiar como `runbook-<componente>.md` y completar. Eliminar esta línea antes de hacer commit.

> Última actualización: YYYY-MM-DD
> Autor: Por definir | Equipo: DevOps / Operaciones

## Información rápida del componente

| Campo | Valor |
|-------|-------|
| Repositorio | [url] |
| Ambiente `main` | [url / namespace] |
| Dashboard principal | [url Grafana / Datadog] |
| Canal de alertas | [Slack #canal / PagerDuty] |
| Contacto de escalamiento | [nombre / handle] |
| RTO objetivo | [30 min] |
| RPO objetivo | [5 min] |

## Arquitectura operativa (solo lo necesario para operar)

```text
[Edge] → [Backend] → [PostgreSQL]
                 → [MinIO/S3]
                 → [Notificaciones push]
```

**Dependencias críticas:**
- `PostgreSQL`: si cae, la API no puede procesar operaciones
- `MinIO/S3`: si cae, no se puede subir/consultar evidencia multimedia
- `Notificaciones push`: si cae, el flujo de notificación de eventos críticos falla

## Alertas y diagnóstico rápido

### Alta tasa de errores 5xx

**Síntoma:** Error rate > 2% durante 5 minutos.

**Acciones:**
1. Revisar logs recientes del backend (JSON estructurado).
2. Si hay errores de BD: ver sección Base de datos.
3. Si es error de dependencia externa (MinIO/push): verificar conectividad.

### Latencia alta (p95 > SLO)

**Síntoma:** p95 > [200ms] durante 10 minutos.

**Acciones:**
1. Verificar uso de recursos (CPU, memoria, pool de conexiones).
2. Si hay query lento: revisar `pg_stat_activity` y locks.

## Procedimientos por componente

### Base de datos

```bash
docker compose --env-file .env.<ambiente> exec postgres pg_isready -U <user> -d somnguard
# Respuesta esperada: "accepting connections"
```

### Almacenamiento multimedia (MinIO)

```bash
# Verificar salud del bucket
mc alias set somnguard http://localhost:9000 <access> <secret>
mc ls somnguard/<bucket-evidencia>
```

## Operaciones de escala y mantenimiento

### Escalar horizontalmente

```bash
# [Comandos según plataforma de despliegue: Docker Compose / K8s]
```

### Rollback de emergencia

Ver [_template-rollback-plan.md](../10-devops/_template-rollback-plan.md).

## Comunicación durante incidente

| Evento | Canal | Mensaje tipo |
|--------|-------|-------------|
| Detección P0 | [Slack #incidents] | `[P0 INICIO] [componente] degradado. Investigando.` |
| Cada 15 min | [Slack #incidents] | `[P0 UPDATE] Causa: [...]. Acción: [...]. ETA: [...]` |
| Resolución | [Slack #incidents] | `[P0 RESUELTO] Duración: [X min]. Causa raíz: [...]` |

## Checklist post-incidente

- [ ] Servicio estable con métricas en objetivo SLO
- [ ] Incidente registrado en [_template-incident-postmortem.md](./_template-incident-postmortem.md)
- [ ] Stakeholders notificados de resolución
- [ ] Ticket de mejora creado

## Referencias

- [SLA/SLO/SLI](./_template-sla-slo-sli.md)
- [Arquitectura](../05-architecture/architecture-document.md)
- [Rollback Plan](../10-devops/_template-rollback-plan.md)