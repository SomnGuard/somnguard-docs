<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../../../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Modelo de datos — [nombre-modulo]

**Estado:** Plantilla
**Fecha:** 2026-08-19

</div>

</div>

> **PLANTILLA** — Completar con las entidades propias del módulo. Convenciones: ver [modeling-conventions.md](../../../../06-data-architecture/modeling-conventions.md). Eliminar esta línea antes de hacer commit.

> Última actualización: YYYY-MM-DD

## Entidades propias

### [NombreEntidad]

| Campo | Tipo | Nullable | Descripción |
|-------|------|----------|-------------|
| `id` | UUID | No | PK |
| `created_at` | TIMESTAMPTZ | No | Fecha de creación (UTC) |
| `updated_at` | TIMESTAMPTZ | No | Última modificación (UTC) |

**Reglas de negocio aplicables:** [RN-NN]

## Referencias a otras tablas (misma BD)

<!-- Referencias lógicas dentro de la misma base; las FK se cablean en Liquibase 04_alter. -->

| Campo | Tabla referenciada | Acción referencial |
|-------|--------------------|-------------------|
| `[id_externo]` | `[otro_modulo].[tabla]` | RESTRICT |

## Índices relevantes

| Tabla | Campos indexados | Tipo | Motivo |
|-------|-----------------|------|--------|
| | | | |

## Notas de integridad

<!-- Reglas de negocio que no se expresan en constraints de BD (ej. idempotencia de sincronización, RN-08). -->