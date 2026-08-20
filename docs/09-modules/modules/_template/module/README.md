<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../../../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Módulo: [nombre-modulo] — [descripción en una línea]

**Estado:** Plantilla
**Fecha:** 2026-08-19

</div>

</div>

> **PLANTILLA** — Copiar esta carpeta a `modules/<nn>-<nombre-modulo>/` y completar. Ver instrucciones en [la guía de uso](../README.md). Eliminar esta línea antes de hacer commit.

> Última actualización: YYYY-MM-DD
> Autor: Por definir | Equipo: [backend]

## Responsabilidad

<!-- Una sola oración: qué hace este módulo y qué NO hace. -->

## Entidades propias

<!-- Entidades que este módulo POSEE. Ningún otro módulo escribe directamente en estas tablas. -->

| Entidad | Descripción |
|---------|-------------|
| `[entidad]` | [descripción] |

## Dependencias

| Módulo / recurso | Tipo | Motivo |
|------------------|------|--------|
| `platform` | transversal | Logging, observabilidad, errores |
| `[otro módulo]` | [port in] | [motivo] |

## Puertos (interfaces)

### Puertos de entrada (consumidos por adapters in)

| Puerto | Caso de uso típico |
|--------|--------------------|
| `[NombreUseCasePort]` | [cuándo se invoca] |

### Puertos de salida (implementados por adapters out)

| Puerto | Implementación |
|--------|----------------|
| `[EntidadRepository]` | `adapter/out/persistence` (PostgreSQL) |
| `[MediaStorage]` | `adapter/out/storage` (MinIO/S3) |

## Base de datos

- Nombre lógico: `somnguard` (una sola BD; esquema por módulo)
- Motor: PostgreSQL 16 + Liquibase
- Esquema: [snake_case del módulo]

## API expuesta

| Recurso | Nota |
|---------|------|
| `[GET /api/v1/...]` | Ver contrato en `07-api-design/contracts/` |

## Links

- Data model: [data-model.md](./data-model.md)
- Eventos: [events.md](./events.md)
- Runbook: [runbook.md](./runbook.md)
- Decisiones internas: [decisions.md](./decisions.md)
- Repo (código): `somnguard-api/backend-java` → `com.somnguard.<nombre-modulo>`