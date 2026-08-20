<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## SLA / SLO / SLI — SOMNGUARD

**Estado:** Plantilla
**Fecha:** 2026-08-19

</div>

</div>

> **PLANTILLA** — Copiar como `sla-slo-sli.md` y completar. Eliminar esta línea antes de hacer commit.

> Última actualización: YYYY-MM-DD
> Autor: Por definir | Equipo: DevOps / Arquitectura

## Definiciones

| Concepto | Definición |
|----------|------------|
| SLA | Acuerdo de nivel de servicio con el cliente/usuario — consecuencias si se incumple |
| SLO | Objetivo interno de nivel de servicio — meta técnica que el equipo se compromete a mantener |
| SLI | Indicador de nivel de servicio — métrica real medida que evalúa el SLO |

## SLOs por componente

### Componente: [backend / edge / portal / app móvil]

| SLI | SLO | Ventana de medición | SLA asociado |
|-----|-----|---------------------|--------------|
| Disponibilidad (% uptime) | ≥ 99% | Rolling 30 días | Si cae < 99%: [compensación] |
| Latencia p95 | < [200ms] | Rolling 7 días | — |
| Latencia p99 | < [500ms] | Rolling 7 días | — |
| Tasa de errores | < 0.5% | Rolling 24 horas | — |
| Tiempo de recuperación (MTTR) | < [30 min] | Por incidente | — |

## Error budget

| Componente | SLO disponibilidad | Error budget mensual | Consumido (mes actual) |
|------------|--------------------|----------------------|------------------------|
| [backend] | 99% | 7.2 h/mes | [X min] |

> Si el error budget se agota, se congela el desarrollo de nuevas features hasta que se recupere.

## Políticas

| Condición | Acción |
|-----------|--------|
| Error budget < 50% restante | Alerta al Tech Lead y congelamiento de deploys no críticos |
| Error budget agotado | Feature freeze + postmortem obligatorio |
| SLO incumplido 2 meses consecutivos | Revisión de arquitectura |

## Métricas y herramientas

| Métrica | Herramienta | Dashboard |
|---------|-------------|-----------|
| Uptime | [UptimeRobot / Datadog / Grafana] | [URL] |
| Latencia | [Prometheus / Datadog] | [URL] |
| Error rate | [ELK / Datadog / Grafana] | [URL] |

## Referencias

- [Runbook](./_template-runbook.md)
- [NFR](../04-requeriments/non-functional.md)
- [Arquitectura](../05-architecture/architecture-document.md)