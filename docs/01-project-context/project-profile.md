<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Perfil del proyecto

**Estado:** En progreso
**Fecha:** 2026-08-19

</div>

</div>

Ficha técnica del proyecto SomnGuard: identificación, dimensiones de complejidad, módulos del backend y trackers. Complementa el contexto narrativo de [overview.md](./overview.md).

## Identificación

| Campo | Valor |
|-------|-------|
| Project key | `SOMNGUARD` |
| Nombre del proyecto | SomnGuard |
| Tipo | `greenfield` |
| Owner | Equipo SomnGuard |
| Fecha de kickoff | 2026-02-10 |

## Dimensiones de complejidad

| Dimensión | Valor | Implicancia |
|-----------|-------|-------------|
| Tiene interfaz de usuario | sí | Activa diseño UX en [`../12-user-experience/`](../12-user-experience/) |
| Canal UI | web + móvil | Portal React JS y app React Native |
| Persistencia | sql | Activa diseño de datos en [`../06-data-architecture/`](../06-data-architecture/) |
| Complejidad de dominio | media | Detección en edge + gestión de eventos y notificaciones |
| Volumen esperado | < 10k usuarios | |
| PII / compliance | básico | Ley 1581/2012: datos personales y evidencia multimedia |
| Integraciones externas | 1-3 | Notificaciones push, almacenamiento multimedia (MinIO/S3) |
| Disponibilidad requerida | 99% | Alertas críticas con baja latencia |
| Multi-tenant | no (single-tenant) | Una sola organización; separación por roles (RBAC) |

## Módulos del backend

| Módulo | Responsabilidad | Estado |
|--------|-----------------|--------|
| `security` | Autenticación, autorización y auditoría | En progreso |
| `parameterization` | Catálogos configurables | En progreso |
| `device-management` | Dispositivos, asignaciones y configuración | En progreso |
| `telemetry-service` | Eventos, evidencia y alertas | En progreso |
| `monitoring` | Notificaciones de eventos críticos | En progreso |
| `analytics` | Línea de tiempo, métricas, resumen IA y reportes | En progreso |

> Ver catálogo y estructura de módulos en [09-modules/module-catalog.md](../09-modules/module-catalog.md) y reglas en [structure-rules.md](../00-documentation-governance/structure-rules.md).

## Trackers externos

| Herramienta | URL | Propósito |
|-------------|-----|-----------|
| GitHub Projects | https://github.com/orgs/SomnGuard/projects/1 | Gestión de HUs y sprints |

## Referencias

- [Contexto del proyecto](./overview.md)
- [Propuesta técnica](./software-technical-proposal.md)
- [Convenciones ágiles](../00-documentation-governance/agile-conventions.md)