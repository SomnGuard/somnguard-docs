<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Índice de diagramas

**Estado:** En progreso
**Fecha:** 2026-08-19

</div>

</div>

Catálogo de diagramas del proyecto. Todo diagrama debe tener fuente editable (`source/`) y su exportación (`exports/`).

## Casos de uso

| Diagrama | Fuente | Exportación | Descripción |
|----------|--------|-------------|-------------|
| Cuenta y dispositivo | [source/cu-account-device.mmd](./diagrams/source/cu-account-device.mmd) | [exports/cu-account-device.png](./diagrams/exports/cu-account-device.png) | Registro, validación, login y asociación de dispositivos |
| Dispositivo | [source/cu-device.mmd](./diagrams/source/cu-device.mmd) | [exports/cu-device.png](./diagrams/exports/cu-device.png) | Ciclo de vida del dispositivo y alertas locales |
| General | [source/cu-general.mmd](./diagrams/source/cu-general.mmd) | [exports/cu-general.png](./diagrams/exports/cu-general.png) | Visión general del sistema |
| Gestión local | [source/cu-local-management.mmd](./diagrams/source/cu-local-management.mmd) | [exports/cu-local-management.png](./diagrams/exports/cu-local-management.png) | Operación offline del dispositivo |
| Plataforma | [source/cu-plataforma.mmd](./diagrams/source/cu-plataforma.mmd) | [exports/cu-plataforma.png](./diagrams/exports/cu-plataforma.png) | Funcionalidades de la plataforma |
| Visualización analítica | [source/cu-analytics-visualization.mmd](./diagrams/source/cu-analytics-visualization.mmd) | [exports/cu-analytics-visualization.png](./diagrams/exports/cu-analytics-visualization.png) | Módulo analítico y reportes |

## Diagramas de secuencia

| Diagrama | Fuente | Exportación | Descripción |
|----------|--------|-------------|-------------|
| Detección y alerta | [source/sd-detection-alert.mmd](./diagrams/source/sd-detection-alert.mmd) | [exports/sd-detection-alert.png](./diagrams/exports/sd-detection-alert.png) | Captura, análisis de visión y alerta sonora en el edge |
| Sincronización offline | [source/sd-offline-sync.mmd](./diagrams/source/sd-offline-sync.mmd) | [exports/sd-offline-sync.png](./diagrams/exports/sd-offline-sync.png) | Cola local, envío de lote a la API y confirmación |
| Autenticación | [source/sd-authentication.mmd](./diagrams/source/sd-authentication.mmd) | [exports/sd-authentication.png](./diagrams/exports/sd-authentication.png) | Login JWT y auditoría de accesos |
| Restablecimiento de contraseña | [source/sd-password-reset.mmd](./diagrams/source/sd-password-reset.mmd) | [exports/sd-password-reset.png](./diagrams/exports/sd-password-reset.png) | Solicitud, token y cambio de contraseña |
| Alta y asignación de dispositivo | [source/sd-device-registration.mmd](./diagrams/source/sd-device-registration.mmd) | [exports/sd-device-registration.png](./diagrams/exports/sd-device-registration.png) | Registro, asignación a cuenta y configuración inicial |
| Consulta de eventos | [source/sd-events-query.mmd](./diagrams/source/sd-events-query.mmd) | [exports/sd-events-query.png](./diagrams/exports/sd-events-query.png) | Línea de tiempo con filtros, detalle y evidencia |
| Notificación de evento crítico | [source/sd-critical-notification.mmd](./diagrams/source/sd-critical-notification.mmd) | [exports/sd-critical-notification.png](./diagrams/exports/sd-critical-notification.png) | Evaluación de severidad, notificación push y lectura |
| Generación de reporte | [source/sd-report-generation.mmd](./diagrams/source/sd-report-generation.mmd) | [exports/sd-report-generation.png](./diagrams/exports/sd-report-generation.png) | Reporte consolidado con resumen descriptivo IA |

## Diagramas de actividades

| Diagrama | Fuente | Exportación | Descripción |
|----------|--------|-------------|-------------|
| Detección de evento | [source/ac-event-detection.mmd](./diagrams/source/ac-event-detection.mmd) | [exports/ac-event-detection.png](./diagrams/exports/ac-event-detection.png) | Flujo de captura, análisis y alerta en el edge |
| Sincronización offline | [source/ac-offline-sync.mmd](./diagrams/source/ac-offline-sync.mmd) | [exports/ac-offline-sync.png](./diagrams/exports/ac-offline-sync.png) | Envío de lote y confirmación desde la cola local |
| Generación de reporte | [source/ac-report-generation.mmd](./diagrams/source/ac-report-generation.mmd) | [exports/ac-report-generation.png](./diagrams/exports/ac-report-generation.png) | Consulta de eventos, resumen IA y compilación |
| Registro de cuenta | [source/ac-account-registration.mmd](./diagrams/source/ac-account-registration.mmd) | [exports/ac-account-registration.png](./diagrams/exports/ac-account-registration.png) | Validación de datos y activación de la cuenta |

## Diagramas de estados

| Diagrama | Fuente | Exportación | Descripción |
|----------|--------|-------------|-------------|
| Dispositivo | [source/es-device.mmd](./diagrams/source/es-device.mmd) | [exports/es-device.png](./diagrams/exports/es-device.png) | Ciclo de vida del dispositivo |
| Evento | [source/es-event.mmd](./diagrams/source/es-event.mmd) | [exports/es-event.png](./diagrams/exports/es-event.png) | Ciclo de vida del evento detectado |
| Cuenta | [source/es-account.mmd](./diagrams/source/es-account.mmd) | [exports/es-account.png](./diagrams/exports/es-account.png) | Ciclo de vida de la cuenta de usuario |

## Diagramas de clases

| Diagrama | Fuente | Exportación | Descripción |
|----------|--------|-------------|-------------|
| Dominio | [source/cd-domain.mmd](./diagrams/source/cd-domain.mmd) | [exports/cd-domain.png](./diagrams/exports/cd-domain.png) | Entidades del dominio y sus relaciones principales |

## Modelo de datos

| Diagrama | Fuente | Ubicación |
|----------|--------|-----------|
| Modelo entidad-relación vigente | [06-data-architecture/01-entity-relationship-model.mmd](../06-data-architecture/01-entity-relationship-model.mmd) | Sección de datos |