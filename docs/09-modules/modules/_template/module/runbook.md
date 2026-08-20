<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../../../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Runbook — [nombre-modulo]

**Estado:** Plantilla
**Fecha:** 2026-08-19

</div>

</div>

> **PLANTILLA** — Completar antes del primer despliegue a QA. Eliminar esta línea antes de hacer commit.

> Última actualización: YYYY-MM-DD

## Healthcheck

| Endpoint | Respuesta esperada | SLO |
|----------|--------------------|-----|
| `GET /health` | `200 { "status": "ok" }` | < 200 ms |
| `GET /health/ready` | `200` cuando BD conectada | < 500 ms |

## Alertas críticas

| Alerta | Condición | Severidad | Acción inmediata |
|--------|-----------|-----------|------------------|
| BD no responde | Timeout > 5 s | P0 | Verificar conexión y reiniciar |
| Error rate > 5 % | En ventana de 5 min | P1 | Revisar logs y escalar |

## Reinicio del módulo

```bash
# Pendiente — completar según plataforma de despliegue (Docker Compose / K8s)
```

## Revisión de logs

```bash
# Pendiente — completar según stack de observabilidad (Loki / CloudWatch / otro)
```

## Escalamiento

| Condición | Paso siguiente | Contacto |
|-----------|---------------|----------|
| No se resuelve en 15 min | Escalar a tech lead | @[handle] |
| Incidente de datos | Activar [backup-and-recovery.md](../../../../13-operations/backup-and-recovery.md) | @[handle] |

## Documentos relacionados

- Postmortem: [_template-incident-postmortem.md](../../../../13-operations/_template-incident-postmortem.md)
- SLA/SLO: [_template-sla-slo-sli.md](../../../../13-operations/_template-sla-slo-sli.md)