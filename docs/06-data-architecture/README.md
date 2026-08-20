<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Arquitectura de datos

**Estado:** En progreso
**Fecha:** 2026-08-19

</div>

</div>

## Contenido

Modelo de datos vigente del sistema. Este modelo es la **fuente de verdad** para entidades, atributos y relaciones; las versiones anteriores están archivadas en `../99-archive/deprecated/data-model-v1/`.

| Archivo | Descripción | Estado |
|---------|-------------|--------|
| [01-entity-relationship-model.mmd](./01-entity-relationship-model.mmd) | Diagrama entidad-relación vigente (20 entidades) | Estable |
| [02-modules-entities.md](./02-modules-entities.md) | Módulos, entidades y atributos con convenciones | Estable |
| [02-relational-model.mmd](./02-relational-model.mmd) | Esquema relacional implementable del modelo vigente | En progreso |
| [03-data-dictionary.md](./03-data-dictionary.md) | Diccionario de datos del modelo vigente | En progreso |
| [modeling-conventions.md](./modeling-conventions.md) | Convenciones de modelado: auditoría, estados, Liquibase | En progreso |

## Convenciones del modelo

- Identificador único: `id UUID` en todas las entidades.
- Auditoría en entidades transaccionales: `created_at`, `updated_at`, `deleted_at`, `created_by`, `updated_by`, `deleted_by`, `is_active`.
- Nombres de entidades y atributos en inglés.
- Contraseñas y API keys se almacenan únicamente como hash.