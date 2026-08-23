<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Módulos del backend

**Estado:** En progreso
**Fecha:** 2026-08-19

</div>

</div>

## Contenido

Catálogo de módulos del backend. SomnGuard es un monolito modular en Java 21 (Spring Boot 4.1.1, Maven) —no microservicios—; cada módulo agrupa un dominio funcional con sus entidades.

| Archivo | Descripción | Estado |
|---------|-------------|--------|
| [module-catalog.md](./module-catalog.md) | Catálogo de módulos con responsabilidad y entidades | En progreso |
| [dependency-map.md](./dependency-map.md) | Mapa de dependencias hexagonal y matriz de acoplamiento entre módulos | Borrador |
| [event-catalog.md](./event-catalog.md) | Catálogo de eventos de dominio y envelope de integración | Borrador |
| [modules/_template/](./modules/_template/README.md) | Plantilla de documentación de módulo (README, data-model, events, decisions, runbook) | Plantilla |

Reglas para documentar módulos: ver `../00-documentation-governance/structure-rules.md`.