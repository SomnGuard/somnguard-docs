<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Plan de rollback — SOMNGUARD

**Estado:** Plantilla
**Fecha:** 2026-08-19

</div>

</div>

> **PLANTILLA** — Copiar como `rollback-plan.md` y completar. Eliminar esta línea antes de hacer commit.

> Última actualización: YYYY-MM-DD
> Autor: Por definir | Equipo: DevOps

## Criterios de activación del rollback

| Condición | Acción | Quién decide |
|-----------|--------|--------------|
| Error rate > 5% por más de 5 min | Rollback inmediato | DevOps + Tech Lead |
| Migración de BD fallida | Rollback inmediato | Equipo BD + Tech Lead |
| SLO violado por más de 15 min | Rollback + postmortem | Tech Lead |
| P0 detectado en producción | Rollback inmediato | Tech Lead |

## Procedimiento por componente

### Backend (`backend-java`)

```bash
# Revertir al tag anterior
git checkout v[X.X.X-1] && mvn clean package
# Restaurar la imagen/artefacto anterior según plataforma de despliegue
```

### Base de datos

| Paso | Acción | Responsable |
|------|--------|-------------|
| 1 | Detener tráfico hacia el servicio afectado | DevOps |
| 2 | Restaurar backup del punto anterior a la migración | Equipo BD |
| 3 | Verificar integridad de datos | Equipo BD + QA |
| 4 | Reactivar tráfico | DevOps |

### Frontend (web y móvil)

```bash
# Reactivar versión anterior en CDN/servidor
# [Comandos específicos según plataforma de despliegue]
```

## Verificación post-rollback

- [ ] Healthcheck OK
- [ ] Error rate < 1% por 10 minutos consecutivos
- [ ] Smoke tests pasan en producción
- [ ] Stakeholders notificados

## Comunicación

| Evento | Canal | Responsable |
|--------|-------|-------------|
| Decisión de rollback | [Slack/Teams #incidents] | Tech Lead |
| Rollback en progreso | [Slack/Teams #incidents] | DevOps |
| Rollback completado | [Slack/Teams #incidents + stakeholders] | Tech Lead |

## Post-rollback

- Registrar incidente en [13-operations/_template-incident-postmortem.md](../13-operations/_template-incident-postmortem.md)
- Identificar causa raíz antes de reintentar el despliegue

## Referencias

- [Deployment Plan](./_template-deployment-plan.md)
- [Runbook](../13-operations/_template-runbook.md)
- [Incident Postmortem](../13-operations/_template-incident-postmortem.md)