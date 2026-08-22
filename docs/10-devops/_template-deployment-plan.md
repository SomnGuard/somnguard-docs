<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Plan de despliegue — SOMNGUARD

**Estado:** Plantilla
**Fecha:** 2026-08-19

</div>

</div>

> **PLANTILLA** — Copiar como `deployment-plan.md` y completar. Eliminar esta línea antes de hacer commit.

> Última actualización: YYYY-MM-DD
> Autor: Por definir | Equipo: DevOps

## Metadata del despliegue

| Campo | Valor |
|-------|-------|
| Versión / Release | |
| Ambiente destino | develop / qa / main |
| Fecha planificada | YYYY-MM-DD HH:MM |
| Responsable | |
| Ventana de despliegue | [duración estimada] |
| Requiere downtime | Sí / No |

## Pre-requisitos

- [ ] Gate de QA aprobado
- [ ] Gate de seguridad aprobado
- [ ] Backups verificados
- [ ] Variables de entorno configuradas en ambiente destino
- [ ] Plan de rollback preparado

## Componentes a desplegar

| Componente | Versión actual | Versión nueva | Cambios |
|------------|----------------|---------------|---------|
| `backend-java` | v1.0.0 | v1.1.0 | |
| Liquibase (migraciones) | — | migration V00X | |

## Pasos de despliegue

| Orden | Paso | Comando / Acción | Responsable | Verificación |
|-------|------|-----------------|-------------|--------------|
| 1 | Backup de BD | | DBA | Backup confirmado |
| 2 | Aplicar migraciones | `docker compose --env-file .env.<ambiente> run --rm --build liquibase update` (repo `somnguard-db`) | DevOps | `db-status` OK |
| 3 | Desplegar backend | | DevOps | Healthcheck OK |
| 4 | Smoke test | | QA | Tests pasan |
| 5 | Activar tráfico | | DevOps | Métricas estables |

## Verificación post-despliegue

- [ ] Healthcheck respondiendo
- [ ] Métricas de error < 1% en los primeros 15 minutos
- [ ] Pruebas de smoke en ambiente destino
- [ ] Dashboard de observabilidad sin anomalías

> **Nota:** este es un gate de éxito del despliegue; el umbral de alerta operativa es error rate > 1% por 5 min (ver `../13-operations/_template-observability.md`).

## Plan de rollback

| Condición | Acción | Tiempo estimado |
|-----------|--------|-----------------|
| Error en migración de BD | Restaurar backup | [X min] |
| Error en despliegue del backend | Revertir versión anterior | [X min] |
| Smoke tests fallando | Rollback completo | [X min] |

> Procedimiento detallado en [_template-rollback-plan.md](./_template-rollback-plan.md).

## Comunicación

| Evento | Canal | Mensaje |
|--------|-------|---------|
| Inicio del despliegue | [Slack/Teams #deploys] | `[DEPLOY INICIO] v1.1.0 en <ambiente>` |
| Despliegue exitoso | [Slack/Teams #deploys] | `[DEPLOY OK] v1.1.0 estable` |
| Rollback | [Slack/Teams #incidents] | `[ROLLBACK] Motivo: [...]. ETA: [...]` |

## Referencias

- [Release Checklist](./_template-release-checklist.md)
- [Rollback Plan](./_template-rollback-plan.md)
- [Runbook](../13-operations/_template-runbook.md)