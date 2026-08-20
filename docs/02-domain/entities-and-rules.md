<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Entidades y reglas de negocio

**Estado:** En progreso
**Fecha:** 2026-08-19

</div>

</div>

Reglas de negocio del sistema. Fuente de verdad de las reglas `RN-*`; consolidan y equivalen a las reglas `RB-*` del informe de diseño (no se renumeran las ya publicadas; los nuevos identificadores usan `RN-`).

## Reglas de negocio

| ID | Módulo | Regla |
|----|--------|-------|
| RN-01 | security | Una cuenta es única por correo electrónico; las contraseñas se almacenan solo como hash |
| RN-02 | security | El acceso a funcionalidades se controla por roles y permisos (`role_feature`); un usuario hereda los permisos de sus roles |
| RN-03 | device-management | Un dispositivo solo puede pertenecer a la cuenta a la que fue asignado (`device_assignment`); el envío de eventos exige una API key válida |
| RN-04 | parameterization | Todo evento debe clasificarse con una categoría, un tipo y una severidad vigentes (catálogos de parameterization) |
| RN-05 | telemetry-service | Las alertas emitidas por el dispositivo se registran en `alert_log` asociadas al evento que las originó y al patrón de sonido reproducido |
| RN-06 | telemetry-service | La evidencia multimedia se conserva por un período de retención definido; los metadatos en `evidence` incluyen el tipo de medio y su referencia |
| RN-07 | monitoring | Las notificaciones de eventos críticos se generan automáticamente y se dirigen a la cuenta propietaria del dispositivo |
| RN-08 | telemetry-service | La sincronización offline no debe duplicar eventos: el dispositivo envía un identificador único (idempotencia) y la API descarta envíos repetidos |
| RN-09 | parameterization | El catálogo de sonidos (`sound_pattern`) es gestionado exclusivamente por el administrador técnico |
| RN-10 | device-management | La configuración básica del dispositivo (`device_config`) se descarga del backend y se aplica localmente antes de iniciar el monitoreo |

> Equivalencias: RN-01..RN-10 ↔ RB-01..RB-10 del [informe de diseño](../05-architecture/software-design-report.md).

## Modelo de dominio por módulo

| Módulo | Entidades (agregados y tablas) |
|--------|--------------------------------|
| security | user, role, module, feature, role_feature, user_role, password_reset_request, audit_login |
| parameterization | event_category, severity, media_type, sound_pattern, event_type |
| device-management | device, device_assignment, device_config |
| telemetry-service | event, evidence, alert_log |
| monitoring | notification |
| analytics | vistas/reportes derivados (sin entidades transaccionales) |

Detalle de atributos en [`../06-data-architecture/02-modules-entities.md`](../06-data-architecture/02-modules-entities.md).

## Ver también

- [Mapa de dominio](domain-map.md)
- [Eventos de dominio](domain-events.md)
- [Reglas de negocio del diseño](../05-architecture/software-design-report.md#9-reglas-de-negocio)