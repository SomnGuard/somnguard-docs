<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Matriz de trazabilidad

**Estado:** En progreso
**Fecha:** 2026-08-19

</div>

</div>

Matriz que relaciona funcionalidades (`F-*`), casos de uso, vistas dinámicas, módulos del backend, reglas de negocio (`RN-*`) y pruebas. Base: tabla de trazabilidad del análisis de software, ampliada con reglas de negocio y NFR.

## Matriz

| Funcionalidad | Caso de uso | Vistas dinámicas | Módulo backend | Reglas de negocio | NFR | Pruebas (estrategia) |
|---------------|-------------|------------------|----------------|-------------------|-----|----------------------|
| F-01 Monitoreo continuo | cu-device | sd-detection-alert, ac-event-detection | Telemetry Service + Edge | — | NFR-03, NFR-04 | Unitarias del detector; integración del agente |
| F-02 Detección de riesgo | cu-device | sd-detection-alert, es-event | Edge + Telemetry Service | RN-08 | NFR-04 | Validación con datos de campo |
| F-03 Alertas sonoras | cu-device | sd-detection-alert | Edge | RN-05 | NFR-04 | Pruebas en prototipo |
| F-04 Registro de eventos | cu-local-management | sd-offline-sync, es-event | Telemetry Service | RN-04, RN-05, RN-06 | NFR-06, NFR-07 | Unitarias de repositorio; integración |
| F-05 Transmisión a plataforma | cu-general | sd-offline-sync, ac-offline-sync | Telemetry Service | RN-08 | NFR-03 | Integración con API simulada |
| F-06 Análisis y reportes | cu-analytics-visualization | sd-report-generation, ac-report-generation | Analítico | — | — | Pruebas de API y E2E del portal |
| F-07 Gestión de dispositivos | cu-device, cu-plataforma | sd-device-registration, es-device | Device Management | RN-03, RN-10 | NFR-01 | Unitarias e integración |
| F-08 Notificaciones | cu-plataforma | sd-critical-notification | Monitoring | RN-07 | — | Pruebas de integración del push |
| F-09 Cuentas y roles | cu-account-device | sd-authentication, sd-password-reset, es-account | Security | RN-01, RN-02 | NFR-01 | Unitarias de seguridad; integración |
| F-10 Catálogos | cu-plataforma | — | Parameterization | RN-04, RN-09 | — | CRUD por API |

## Otras trazabilidades

| Origen | Destino | Referencia |
|--------|---------|------------|
| HU (nueva) | F-* / RN-* / NFR / ADR / TC | Plantilla `04-requeriments/_template-hu.md` |
| Épica | Funcionalidades | `03-product-definition/product-backlog.md` |
| NFR | Componentes | `04-requeriments/non-functional.md` |

## Ver también

- [Análisis del software](./software-analysis.md#8-trazabilidad)
- [Estrategia de pruebas](../11-quality-assurance/test-strategy.md)
- [Convenciones ágiles](../00-documentation-governance/agile-conventions.md)