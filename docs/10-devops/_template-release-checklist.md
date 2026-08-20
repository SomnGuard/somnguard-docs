<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Release Checklist — SOMNGUARD — v[X.X.X]

**Estado:** Plantilla
**Fecha:** 2026-08-19

</div>

</div>

> **PLANTILLA** — Copiar como `release-checklist-vX.X.X.md` y completar. Eliminar esta línea antes de hacer commit.

> Última actualización: YYYY-MM-DD
> Autor: Por definir | Equipo: DevOps + QA

## Identificación del release

| Campo | Valor |
|-------|-------|
| Versión | v[X.X.X] |
| Fecha objetivo | YYYY-MM-DD |
| Release manager | |

## Gate de código

- [ ] Todas las HUs comprometidas están en estado `Hecho`
- [ ] Sin PRs pendientes de merge para este release
- [ ] Rama `release/vX.X.X` creada y estable
- [ ] Code review completado en todos los cambios

## Gate de calidad (QA)

- [ ] Pruebas unitarias: cobertura ≥ [80%]
- [ ] Pruebas de integración: todas en verde
- [ ] Pruebas funcionales (smoke): todas pasan en QA
- [ ] Regresión: sin nuevos defectos P0 o P1
- [ ] Reporte QA firmado: [enlace]

## Gate de seguridad

- [ ] Escaneo de dependencias: sin vulnerabilidades críticas
- [ ] SAST/DAST ejecutado: sin hallazgos críticos
- [ ] Secrets scan: sin credenciales en código
- [ ] Reporte AppSec firmado: [enlace]

## Gate de infraestructura

- [ ] Migraciones de BD probadas en `qa`
- [ ] Variables de entorno configuradas en producción
- [ ] Backups verificados
- [ ] Plan de rollback preparado y probado
- [ ] Runbook actualizado

## Gate de documentación

- [ ] CHANGELOG actualizado
- [ ] Documentación de API actualizada
- [ ] Guías de usuario actualizadas (si aplica)

## Aprobaciones requeridas

| Rol | Nombre | Fecha | Firma |
|-----|--------|-------|-------|
| Tech Lead | | | |
| QA Lead | | | |
| Product Owner | | | |

## Referencias

- [Deployment Plan](./_template-deployment-plan.md)
- [QA Report](../11-quality-assurance/_template-qa-report.md)
- [Rollback Plan](./_template-rollback-plan.md)