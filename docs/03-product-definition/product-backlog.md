<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Product backlog

**Estado:** En progreso
**Fecha:** 2026-08-19

</div>

</div>

Backlog de producto inicial, derivado de las funcionalidades del sistema (`F-01..F-10` del análisis de software), el catálogo de módulos y el plan de trabajo de la propuesta técnica. Las historias de usuario (HU) se gestionan en GitHub Projects con el formato `HU-<REPO>-NNN - Nombre/Descripción` (ver [convenciones ágiles](../00-documentation-governance/agile-conventions.md)).

## Épicas (agrupaciones de trabajo)

| Épica | Módulo / componente | Funcionalidades |
|-------|---------------------|-----------------|
| Seguridad y cuentas | security | F-09 (gestión de cuentas, roles y permisos) |
| Gestión de dispositivos | device-management | F-07 (alta, asignación y configuración) |
| Telemetría y sincronización | telemetry-service + edge | F-01 (monitoreo), F-02 (detección), F-03 (alertas), F-04 (registro), F-05 (transmisión) |
| Monitoreo y notificaciones | monitoring | F-08 (notificaciones de eventos críticos) |
| Parametrización | parameterization | F-10 (administración de catálogos) |
| Analítica y reportes | analytics | F-06 (análisis de datos y reportes) |

## Priorización MoSCoW (MVP)

| Prioridad | Épicas |
|-----------|--------|
| Must | Seguridad y cuentas, Gestión de dispositivos, Telemetría y sincronización, Monitoreo y notificaciones, Parametrización |
| Should | Analítica y reportes |
| Could | Resumen IA (dentro de Analítica) — se incluye, pero con la menor prioridad: se entrega al final del MVP |
| Won't | Funcionalidades fuera del alcance definido |

## Estado por épica

| Épica | Estado | Nota |
|-------|--------|------|
| Seguridad y cuentas | Pendiente | Base para el resto del sistema; capa de aplicación aún no construida (TD-002) |
| Gestión de dispositivos | Backlog | Depende de Seguridad (roles) |
| Telemetría y sincronización | Backlog | Depende de Gestión de dispositivos |
| Monitoreo y notificaciones | Backlog | Depende de Telemetría (eventos) |
| Parametrización | Backlog | Requisito de Telemetría (catálogos validan eventos) |
| Analítica y reportes | Backlog | Depende de Telemetría y Monitoreo |

## Historias de usuario (ejemplos iniciales)

| HU | Título (GitHub Projects) | Repo |
|----|--------------------------|------|
| `HU-API-###` | Autenticar usuario en la plataforma | API |
| `HU-API-###` | Registrar evento de telemetría de forma idempotente | API |
| `HU-DB-###` | Crear esquema de catálogos de parametrización | DB |
| `HU-DEVICE-###` | Sincronizar lote de eventos con respaldo offline | DEVICE |
| `HU-PORTAL-###` | Consultar línea de tiempo de eventos del conductor | PORTAL |
| `HU-APP-###` | Recibir notificación de evento crítico | APP |

## Ver también

- [Convenciones ágiles](../00-documentation-governance/agile-conventions.md)
- [Matriz de trazabilidad](../04-requeriments/traceability-matrix.md)
- [Análisis del software](../04-requeriments/software-analysis.md)
- [Plan de trabajo](../01-project-context/software-technical-proposal.md#7-plan-de-trabajo-y-cronograma)