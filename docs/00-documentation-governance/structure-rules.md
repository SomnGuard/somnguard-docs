<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Documentación de módulos del backend

**Estado:** En progreso
**Fecha:** 2026-08-16

</div>

</div>

SomnGuard se implementa como un **monolito modular** en Java 21 con Spring Boot 3.x (Maven) con principios de clean architecture (ver `05-architecture/documento-arquitectura.md`). No se orienta a microservicios. Este documento establece los lineamientos para documentar los módulos del backend.

## Regla crítica

No crear carpetas por módulo hasta que el módulo exista en el repositorio de código o su creación esté formalmente aprobada por arquitectura.

No crear módulos ficticios para llenar la estructura.

**Requisito de aprobación:** Todo módulo nuevo debe tener una ADR en `05-architecture/decisions/records/` o una decisión registrada en `15-project-control/open-questions.md` con estado RESUELTA antes de crear su carpeta de documentación. Sin ese artefacto, el PR será rechazado.

## Catálogo de módulos

La fuente de verdad del listado de módulos es `09-modules/module-catalog.md`. Todo módulo del backend debe estar registrado allí antes de documentarse en detalle.

## Ubicación

La documentación detallada de un módulo real se organiza en:

```text
09-modules/modules/<nombre-del-modulo>/
```

El nombre de la carpeta debe coincidir con el nombre del módulo registrado en el catálogo y con el módulo en el código.