<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Documentación de microservicios

**Estado:** En progreso
**Fecha:** 2026-06-26

</div>

</div>
Este documento establece los lineamientos para documentar los microservicios del proyecto, definiendo la información mínima que debe registrarse para garantizar consistencia, claridad, trazabilidad y facilidad de mantenimiento.

## Regla crítica

No crear carpetas en `09-microservices/services/` hasta que el servicio exista en el repositorio de código o su creación esté formalmente aprobada por arquitectura.

No crear microservicios ficticios para llenar la estructura.

**Requisito de aprobación:** Todo servicio nuevo debe tener una ADR en `05-architecture/decisions/records/` o una decisión registrada en `15-project-control/open-questions.md` con estado RESUELTA antes de crear la carpeta en `09-microservices/services/`. Sin ese artefacto, el PR será rechazado.

## Ubicación

Cada servicio real se documenta en:

```text
09-microservices/services/<nombre-del-servicio>/
```

El nombre de la carpeta debe coincidir con el nombre del repositorio de código.