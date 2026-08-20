<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Gestión de incidentes

**Estado:** En progreso
**Fecha:** 2026-08-19

</div>

</div>

Cómo se clasifica, responde, comunica y aprende de los incidentes del sistema SomnGuard. El marco aplica a base de datos, migraciones y, cuando exista, a la capa de aplicación (backend, MinIO, notificaciones).

## Severidades

| Sev | Definición | Ejemplos | Respuesta |
|-----|------------|----------|-----------|
| **P0** | Pérdida total o corrupción de datos; servicio caído | PostgreSQL no responde; corrupción de datos; pérdida de backups | Atención inmediata; se moviliza el equipo |
| **P1** | Degradación grave o riesgo alto sin pérdida aún | Migración fallida; disco > 85%; fallo de MinIO que impide subir evidencia | Atención el mismo día |
| **P2** | Impacto acotado, con workaround | Consultas lentas sostenidas; notificaciones con retraso | Priorizable en el sprint |
| **P3** | Impacto menor / cosmético | Discrepancia menor en datos de prueba | Deuda menor, se agenda |

## Roles

| Rol | Responsabilidad |
|-----|-----------------|
| **Incident Commander (IC)** | Coordina la respuesta, decide mitigaciones, mantiene la línea de tiempo. Un único responsable a la vez. |
| **Responsable técnico** | Ejecuta el diagnóstico y las acciones sobre BD/migraciones/servicio afectado. |
| **Comunicaciones** | Informa a stakeholders y mantiene el registro de actualizaciones. |
| **Escalamiento** | Arquitectura para decisiones de esquema; seguridad si hay exposición de datos (ver [política de seguridad](../00-documentation-governance/security-policy.md)). |

> **Punto abierto:** la rotación de guardia (on-call) y los canales reales de cada rol están por definir.

## Flujo de respuesta

1. **Detección.** Alerta automática (ver [observability.md](./observability.md)) o reporte manual.
2. **Triaje.** Asignar severidad y designar IC. Para P0/P1 se abre canal dedicado del incidente.
3. **Contención.** Frenar el daño antes de arreglar la causa:
   - Migración fallida: **no** reintentar a ciegas; revisar `databasechangelog` y, en local, revertir con el rollback del changeset (en `qa`/`main` las migraciones son forward-only: se prepara un forward-fix).
   - Corrupción/pérdida: aislar la BD y preparar restauración (ver [backup-and-recovery.md](./backup-and-recovery.md)).
   - Disco lleno: liberar espacio/rotar logs antes de reanudar escrituras.
4. **Diagnóstico.** Identificar causa inmediata y causa raíz.
5. **Mitigación / resolución.** Aplicar el fix y verificar con healthchecks.
6. **Verificación y cierre.** Confirmar integridad de datos y estabilidad; comunicar resolución.
7. **Postmortem** para P0/P1 (obligatorio).

## Comunicación

| Momento | Contenido |
|---------|-----------|
| Inicio (P0/P1) | Qué falla, severidad, impacto conocido, IC asignado |
| Cada intervalo acordado | Causa (si se conoce), acción en curso, próximo paso, ETA |
| Resolución | Qué se restauró, duración, causa raíz preliminar, impacto en datos |

Regla de seguridad: si el incidente involucra exposición de datos sensibles o una credencial, **rotar primero, avisar después** y seguir el procedimiento de [política de seguridad](../00-documentation-governance/security-policy.md). No pegar datos reales ni secretos en los canales del incidente.

## Postmortem

Obligatorio para P0/P1, **sin culpables** (blameless). Se documenta con [_template-incident-postmortem.md](./_template-incident-postmortem.md): resumen ejecutivo, línea de tiempo, causa inmediata/raíz/sistémica, impacto, qué funcionó y qué no, y **acciones correctivas** con responsable y fecha.

Regla de mejora continua: toda acción correctiva se convierte en un ticket rastreable; un incidente sin acciones correctas registradas no se considera cerrado.

## Puntos abiertos

- Guardia on-call, canales reales y herramienta de alerting/paging.
- Umbrales de severidad ligados a SLO por servicio.

## Referencias

- [_template-incident-postmortem.md](./_template-incident-postmortem.md)
- [_template-runbook.md](./_template-runbook.md)
- [observability.md](./observability.md)
- [backup-and-recovery.md](./backup-and-recovery.md)
- [Política de seguridad](../00-documentation-governance/security-policy.md)