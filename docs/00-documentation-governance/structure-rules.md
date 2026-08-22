<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Reglas de estructura de la documentación de módulos

**Estado:** En progreso
**Fecha:** 2026-08-19

</div>

</div>

SomnGuard se implementa como un **monolito modular** en Java 21 con Spring Boot 4.1.1 (Maven) con arquitectura limpia formalizada como **hexagonal (puertos y adaptadores)** por módulo, con `platform` transversal fuera de los módulos (ver `../05-architecture/architecture-document.md` y ADR-002). No se orienta a microservicios. Este documento establece los lineamientos para documentar los módulos del backend.

## Regla crítica

No crear carpetas por módulo hasta que el módulo exista en el repositorio de código o su creación esté formalmente aprobada por arquitectura.

No crear módulos ficticios para llenar la estructura.

**Requisito de aprobación:** Todo módulo nuevo debe tener una ADR en `../05-architecture/decisions/records/` o una decisión registrada en `../15-project-control/open-questions.md` con estado RESUELTA antes de crear su carpeta de documentación. Sin ese artefacto, el PR será rechazado.

## Catálogo de módulos

La fuente de verdad del listado de módulos es `../09-modules/module-catalog.md`. Todo módulo del backend debe estar registrado allí antes de documentarse en detalle.

## Ubicación

La documentación detallada de un módulo real se organiza en:

```text
09-modules/modules/<nombre-del-modulo>/
```

El nombre de la carpeta debe coincidir con el nombre del módulo registrado en el catálogo y con el módulo en el código.