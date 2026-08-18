<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Catálogo de módulos

**Estado:** En progreso
**Fecha:** 2026-08-16

</div>

</div>

Catálogo de los módulos del backend de SomnGuard. Fuente de verdad del listado de módulos; el detalle de entidades y atributos vive en `06-data-architecture/02-modulos-entidades.md`.

| Módulo | Responsabilidad | Entidades |
|--------|-----------------|-----------|
| **security** | Gestiona la autenticación, autorización y auditoría de usuarios | user, role, module, feature, role_feature, user_role, password_reset_request, audit_login |
| **parameterization** | Gestiona los catálogos configurables del sistema | event_category, severity, media_type, sound_pattern, event_type |
| **device-management** | Gestiona dispositivos, asignaciones y configuración | device, device_assignment, device_config |
| **telemetry-service** | Recibe y almacena la información generada por los dispositivos (corresponde al módulo EventIngestion del documento de arquitectura) | event, evidence, alert_log |
| **monitoring** | Gestiona el envío y seguimiento de notificaciones | notification |

## Reglas

- No documentar módulos ficticios: un módulo solo se registra aquí cuando existe en el código o tiene ADR aprobada (ver `structure-rules.md`).
- La documentación detallada de un módulo se ubica en `modules/<nombre-del-modulo>/`.