<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Convenciones ágiles

**Estado:** En progreso
**Fecha:** 2026-08-19

</div>

</div>

Convenciones para la gestión de requerimientos y trabajo ágil del proyecto: identificadores, prioridades, estados y trazabilidad.

## Identificadores

| Tipo | ID | Descripción | Dónde vive |
|------|----|-------------|------------|
| Historia de usuario | `HU-<REPO>-NNN` | Requerimiento accionable con criterios de aceptación | **GitHub Projects** (board del proyecto) |
| Criterio de aceptación | `AC-###` | Condición verificable de una HU | Dentro de la HU |
| Caso de prueba | `TC-###` | Prueba trazada a una HU/AC | `../11-quality-assurance/test-strategy.md` |
| Defecto | `BUG-###` | Defecto registrado en reporte/evidencia de QA | `../11-quality-assurance/_template-qa-report.md` |
| Regla de negocio | `RN-##` | Regla del negocio (las ya publicadas usan `RB-##`) | `../02-domain/entities-and-rules.md` |
| Requisito no funcional | `NFR-##` | Requisito de calidad/operación | `../04-requeriments/non-functional.md` |
| Decisión de arquitectura | `ADR-###` | Decisión registrada | `../05-architecture/decisions/records/` |

> **IDs ya publicados:** `F-01..F-10` (funcionalidades del sistema) y `RB-01..RB-10` (reglas de negocio) se conservan sin renumerar. `RB-*` equivale a `RN-*`.

## Historias de usuario (HU)

Las HU se gestionan exclusivamente en **GitHub Projects**:

- **Board:** [github.com/orgs/SomnGuard/projects/1](https://github.com/orgs/SomnGuard/projects/1)
- **Formato del título:** `HU-<REPO>-NNN - Nombre/Descripción`
- **Repo del prefijo** según el componente afectado:

| Prefijo | Repo / componente |
|---------|-------------------|
| `HU-DEVICE-###` | Dispositivo edge (Raspberry Pi + cámara, Python) |
| `HU-API-###` | Backend (Java 21 + Spring Boot 4.1.1, Maven) |
| `HU-DB-###` | Base de datos (PostgreSQL + Liquibase) |
| `HU-APP-###` | App móvil (React Native) |
| `HU-PORTAL-###` | Portal web (React JS) |

Ejemplos:

| Mal | Bien |
|-----|------|
| `Login del sistema` | `HU-API-012 - Autenticar usuario en la plataforma` |
| `Reportes` | `HU-PORTAL-013 - Generar reporte de eventos por rango de fechas` |

> Una HU puede tocar más de un repo (p. ej. edge + API): se identifica con el prefijo del repo principal y los demás se indican como dependencias o subtareas.

## Criterios de aceptación (AC)

Cada HU declara sus criterios `AC-###` (numerados por HU) en el campo de descripción del item de GitHub Projects. Ver la plantilla en `../04-requeriments/_template-hu.md`.

## Casos de prueba (TC)

Los casos de prueba se identifican `TC-###` y se registran en `../11-quality-assurance/` (estrategia, evidencia y reportes), trazados a HU → AC → TC.

## Severidades de incidentes y defectos

| Severidad | Descripción | Ejemplo |
|-----------|-------------|---------|
| **P0** | Bloqueante: sistema inoperante, datos comprometidos | API caída, pérdida de evidencia |
| **P1** | Crítico: función principal degradada sin workaround | No llegan notificaciones de eventos críticos |
| **P2** | Medio: función con workaround razonable | Error de formato en un reporte |
| **P3** | Bajo: cosmético, no bloquea | Texto de UI incorrecto |

## Estados de una historia de usuario

Los estados se manejan en GitHub Projects:

| Estado | Significado |
|--------|-------------|
| Backlog | Aceptada por producto, aún no planificada |
| Ready | Cumple Definition of Ready |
| In Progress | Desarrollo en curso (rama `hu-<repo>-###-dev`) |
| In Review | PR abierto o revisión de código |
| In QA | Validación en ambiente `qa` |
| Approved | Validada por QA y producto |
| Done | Desplegada a `main` / producción |

## Trazabilidad

Cada HU debe trazar: Funcionalidad (`F-*`) → Módulo → Reglas de negocio (`RN-*`) → Pruebas (`TC-*`) → ADR cuando aplique.

Fuente de verdad de la matriz: `../04-requeriments/traceability-matrix.md`.

## Ver también

- [Definition of Ready](definition-of-ready.md)
- [Definition of Done](definition-of-done.md)
- [Metodología adoptada](adopted-methodology.md)