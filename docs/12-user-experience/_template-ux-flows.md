<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## UX Flows — SOMNGUARD

**Estado:** Plantilla
**Fecha:** 2026-08-19

</div>

</div>

> **PLANTILLA** — Copiar como `ux-flows.md` y completar. Eliminar esta línea antes de hacer commit.

> Última actualización: YYYY-MM-DD
> Autor: Por definir | Equipo: UX/UI

## Usuarios y contexto

| Rol | Dispositivo principal | Nivel técnico | Necesidad principal |
|-----|-----------------------|---------------|---------------------|
| [Conductor] | Móvil (app React Native) | Básico | [Recibir alertas y consultar su historial] |
| [Administrador] | Desktop (portal React JS) | Medio | [Gestionar cuentas, dispositivos y catálogos] |

## Flujos principales

### Flujo: [Nombre del flujo]

**HU relacionada:** HU-<REPO>-NNN
**Actor:** [Conductor / Administrador]
**Objetivo:** [qué logra el usuario al completar este flujo]

```
[Pantalla inicial]
  ↓ [acción del usuario]
[Pantalla 2]
  ↓ [condición: si / no]
  ├─ [Camino feliz] → [Pantalla de éxito]
  └─ [Camino de error] → [Mensaje de error]
```

**Estados del flujo:**
- Inicio: [descripción]
- Happy path: [descripción]
- Error: [descripción del manejo de error]
- Fin: [estado final del sistema]

## Flujos secundarios

| Flujo | Actor | Trigger | Resultado |
|-------|-------|---------|-----------|
| | | | |

## Diagrama

> Diagrama fuente en `../08-uml/diagrams/source/` — exportación en `../08-uml/diagrams/exports/`.

## Referencias

- [UI Spec](./_template-ui-spec.md)
- [PRD](../03-product-definition/product-backlog.md)
- [HU template](../04-requeriments/_template-hu.md)