<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## UI Spec — SOMNGUARD

**Estado:** Plantilla
**Fecha:** 2026-08-19

</div>

</div>

> **PLANTILLA** — Copiar como `ui-spec.md` y completar. Eliminar esta línea antes de hacer commit.

> Última actualización: YYYY-MM-DD
> Autor: Por definir | Equipo: UX/UI

## Pantallas

### Pantalla: [Nombre]

**HU relacionada:** HU-<REPO>-NNN
**Ruta:** `/[ruta-de-la-pantalla]`
**Descripción:** [qué hace el usuario en esta pantalla]

#### Componentes

| Componente | Tipo | Comportamiento | Estado vacío | Estado error |
|------------|------|----------------|--------------|--------------|
| [Botón Guardar] | Button | Enviar formulario | — | Deshabilitado si form inválido |
| [Campo email] | Input text | Validar formato email | Placeholder | Borde rojo + mensaje |

#### Estados de la pantalla

| Estado | Descripción | Componentes afectados |
|--------|-------------|----------------------|
| Cargando | Spinner visible, interacción bloqueada | Todos |
| Éxito | Mensaje de confirmación | Toast/Banner |
| Error | Mensaje de error descriptivo | Campo o banner |
| Vacío | Sin datos (p. ej. sin eventos registrados) | Lista vacía + CTA |

#### Accesibilidad

- [ ] Contraste mínimo AA (4.5:1 texto normal, 3:1 texto grande)
- [ ] Navegable por teclado (Tab order lógico)
- [ ] Etiquetas ARIA en campos de formulario
- [ ] Alt text en imágenes

## Componentes globales

| Componente | Descripción | Reutilizado en |
|------------|-------------|----------------|
| Header | Navegación principal | Todas las pantallas |
| Footer | Links institucionales | Todas las pantallas |
| AlertBanner | Aviso de evento crítico | Dashboard, notificaciones |

## Referencias

- [UX Flows](./_template-ux-flows.md)
- [Design System](./_template-design-system.md)
- [Backlog](../03-product-definition/product-backlog.md)