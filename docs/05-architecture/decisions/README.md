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
| [ADR-004](./records/ADR-004-database-strategy.md) | Database Strategy — 1 PostgreSQL + Esquemas por Módulo + Liquibase | Aceptada | 2026-08-19 |
| [ADR-005](./records/ADR-005-offline-first-device.md) | Offline-First en Device Edge (SQLite Local + Sync Idempotente) | Aceptada | 2026-08-22 |
| [ADR-006](./records/ADR-006-minio-evidence-storage.md) | MinIO Evidence Storage | Aceptada | 2026-08-22 |
| [ADR-007](./records/ADR-007-observability-otel-lgtm.md) | Observabilidad OpenTelemetry + LGTM | Aceptada | 2026-08-22 |
| [ADR-008](./records/ADR-008-traefik-edge-gateway.md) | Traefik Edge Gateway | Aceptada | 2026-08-22 |
| [ADR-009](./records/ADR-009-status-parametrized-audit.md) | Estados Parametrizados + Auditoría Append-Only | Aceptada | 2026-08-22 |