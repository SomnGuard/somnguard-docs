<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Decisiones de arquitectura (ADR)

**Estado:** En progreso
**Fecha:** 2026-08-19

</div>

</div>

Las decisiones de arquitectura (Architecture Decision Records) se registran aquí para dejar trazada la justificación de las opciones técnicas del proyecto.

## Cómo crear una ADR

1. Copia la plantilla [_template-adr.md](./_template-adr.md) como `ADR-NNN-titulo-corto.md`.
2. Regístrala en `records/` con el estado correspondiente.
3. Enlázala en la tabla de este README.
4. Actualiza el `CHANGELOG.md` de la raíz del repositorio.

## Registro de ADRs

| ADR | Título | Estado | Fecha |
|-----|--------|--------|-------|
| [ADR-001](./records/ADR-001-backend-java-spring-boot.md) | Backend en Java Spring Boot (migración desde C#/.NET) | Aceptada | 2026-08-16 |
| [ADR-002](./records/ADR-002-hexagonal-architecture.md) | Arquitectura hexagonal (puertos y adaptadores) en el backend | Aceptada | 2026-08-19 |
| [ADR-003](./records/ADR-003-analytics-module.md) | Módulo analítico (analytics) en el backend | Aceptada | 2026-08-19 |