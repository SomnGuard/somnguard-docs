<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Catálogo de módulos

**Estado:** En progreso
**Fecha:** 2026-08-19

</div>

</div>

Catálogo de los módulos del backend de SomnGuard. Fuente de verdad del listado de módulos; el detalle de entidades y atributos vive en `../06-data-architecture/02-modules-entities.md`.

| Módulo | Responsabilidad | Entidades |
|--------|-----------------|-----------|
| **security** | Gestiona la autenticación, autorización y auditoría de usuarios | user, role, module, feature, role_feature, user_role, password_reset_request, audit_login |
| **parameterization** | Gestiona los catálogos configurables del sistema | event_category, severity, media_type, sound_pattern, event_type |
| **device-management** | Gestiona dispositivos, asignaciones y configuración | device, device_assignment, device_config |
| **telemetry-service** | Recibe y almacena la información generada por los dispositivos | event, evidence, alert_log |
| **monitoring** | Gestiona el envío y seguimiento de notificaciones | notification |
| **analytics** | Módulo analítico: línea de tiempo de eventos, métricas de comportamiento, resumen descriptivo con IA y reportes (ver [ADR-003](../05-architecture/decisions/records/ADR-003-analytics-module.md)) | vistas/reportes derivados (no agrega entidades transaccionales) |

## Reglas

- No documentar módulos ficticios: un módulo solo se registra aquí cuando existe en el código o tiene ADR aceptada / decisión RESUELTA registrada (ver `../00-documentation-governance/structure-rules.md`).
- Los cinco módulos base (security, parameterization, device-management, telemetry-service, monitoring) quedaron aprobados por decisión de arquitectura en [QR-006](../15-project-control/open-questions.md); `analytics` se incorporó por [ADR-003](../05-architecture/decisions/records/ADR-003-analytics-module.md).
- La documentación detallada de un módulo se ubica en `modules/<nombre-del-modulo>/`.