<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Estructura por módulo y entidad

**Estado:** En progreso
**Fecha:** 2026-05-03

</div>

</div>

> **Nota:** la versión vigente del modelo de datos (con atributos) está en [06-data-architecture/02-modules-entities.md](../06-data-architecture/02-modules-entities.md). Este documento es la versión preliminar sin atributos.

## 1. Seguridad, cuenta y autorizacion (Security)

- person
- user
- role
- permission
- module
- form
- role_user
- form_module
- role_form_permission

## 2. Gestion de dispositivos asociados (DeviceManagement)

- device
- device_assignment
- device_config


## 3. Ingesta y sincronizacion de datos (EventIngestion)

- event
- evidence

## 4. Monitoreo de eventos y alertamiento (Monitoring)

- alert
- notification

## 5. Parametrizacion y catalogos (Parameterization)

- status_catalog
- media_type_catalog
- severity_catalog
- event_category_catalog
- sound_pattern
- event_type

---



