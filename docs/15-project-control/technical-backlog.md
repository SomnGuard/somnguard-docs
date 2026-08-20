<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Backlog técnico

**Estado:** En progreso
**Fecha:** 2026-08-19

</div>

</div>

Deuda técnica conocida y pendientes de construcción del proyecto SomnGuard. El sistema hoy tiene documentación y el esqueleto del backend (estructura hexagonal en `somnguard-api`); la capa de aplicación está en construcción.

## Leyenda

- **Impacto**: Bajo / Medio / Alto — efecto sobre despliegue, integridad o velocidad del equipo.
- **Prioridad**: P0 (bloqueante) · P1 (alta) · P2 (media) · P3 (baja).
- **Estado**: Abierto · En curso · Resuelto.

## Deuda técnica activa

| ID | Descripción | Impacto | Prioridad | Módulo(s) afectado(s) | Estado |
|----|-------------|---------|-----------|----------------------|--------|
| TD-001 | **Secretos `.env.*` versionados**: los archivos de entorno con credenciales de conexión están bajo control de versiones. Requiere purga del historial, rotación de credenciales y mover secretos a un gestor (Secret Manager). | Alto | P0 | Todos | Abierto |
| TD-002 | **Capa de aplicación no construida**: no existe aún la implementación de los módulos en Java (solo estructura de carpetas). Es la mayor pieza pendiente; condiciona la validación real de contratos, RBAC y eventos. Se aborda por épicas (ver [product backlog](../03-product-definition/product-backlog.md)). | Alto | P1 | Todos los módulos | Abierto |
| TD-003 | **Contratos OpenAPI no publicados**: la API se documenta en `07-api-design/` pero sin especificación OpenAPI versionable. | Medio | P2 | Todos los módulos | Abierto |
| TD-004 | **Plantilla de módulo sin instanciar**: `09-modules/modules/_template/` está lista pero ningún módulo tiene aún su documentación por módulo. | Medio | P2 | 09-modules | Abierto |
| TD-005 | **Catálogos sin semillas definidas**: los valores iniciales de `parameterization` (categorías, severidades, sonidos) no están fijados. | Medio | P2 | parameterization | Abierto |
| TD-006 | **Estados de negocio sin decisión**: si usar catálogo parametrizable genérico o `VARCHAR + CHECK` para estados de dispositivo/evento (ver [modeling-conventions](../06-data-architecture/modeling-conventions.md) y Q en open-questions). | Medio | P2 | telemetry-service, device-management | Abierto |
| TD-007 | **Driver JDBC / dependencias de build sin fijar**: el `pom.xml` del backend aún no está definido. | Medio | P1 | backend-java | Abierto |

## Trabajo estructural pendiente (capa de aplicación)

Desglose de TD-002 en épicas relativas. No se comprometen fechas hasta cerrar los bloqueantes P0.

| Fase | Alcance | Depende de |
|------|---------|------------|
| Fase A — Fundaciones | Cerrar TD-001, TD-007; `pom.xml`, `platform` (errores, logging, observabilidad), CI básico. | — |
| Fase B — Identidad y catálogos | Módulos `security` (login, JWT, RBAC) y `parameterization` (CRUD de catálogos). | Fase A |
| Fase C — Dispositivos | Módulo `device-management` (alta, asignación, configuración, API keys). | Fase B |
| Fase D — Telemetría | Módulo `telemetry-service` (ingesta idempotente, evidencia, alertas). | Fase C |
| Fase E — Transversales | Módulos `monitoring` (notificaciones) y `analytics` (métricas, reportes, resumen IA). | Fase D |

## Criterio de ingreso al backlog

Un ítem entra aquí cuando: (1) está identificado con módulo y evidencia concreta, (2) tiene impacto y prioridad asignados, y (3) no es una tarea de contenido puramente documental (esas van a las ramas `docs/*`). Al resolverse se marca como Resuelto y se mueve al histórico en el cierre de iteración.

## Ver también

- [Product backlog](../03-product-definition/product-backlog.md)
- [Riesgos](./risks.md)
- [Dependencias](./dependencies.md)