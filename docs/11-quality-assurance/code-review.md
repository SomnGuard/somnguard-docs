<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Revisión de código

**Estado:** En progreso
**Fecha:** 2026-08-19

</div>

</div>

Criterios y flujo para revisar los cambios de SomnGuard (somnguard-docs y somnguard-api). La revisión es obligatoria: ningún cambio entra a una rama padre sin al menos una aprobación (ver [Definition of Done](../00-documentation-governance/definition-of-done.md)).

## Flujo de revisión

Alineado con las [convenciones de Git](../00-documentation-governance/git-conventions.md) (`develop → qa → main`):

1. El autor abre PR desde su rama hija (`hu-###-<ambiente>`, `feat/*`, `fix/*`, etc.) hacia la rama padre correspondiente.
2. El PR cumple la [Definition of Ready](../00-documentation-governance/definition-of-ready.md) antes de solicitar revisión.
3. Se asignan revisores manualmente según el área del cambio (gobernanza → Arquitectura, módulos/API → Backend, etc.). Mínimo **1 aprobación**; para cambios en contratos compartidos (API, modelo de datos, eventos, convenciones) se recomiendan **2**.
4. El revisor aplica el checklist correspondiente y deja comentarios accionables.
5. El autor resuelve los comentarios; los hilos se cierran antes del merge.
6. Merge solo con checks verdes y aprobación. El autor actualiza el `CHANGELOG` si el cambio toca gobernanza, estructura de carpetas o un contrato compartido.

## Checklist base (todo PR)

- [ ] El commit sigue **Conventional Commits**: `type(scope): descripción en inglés` (ver git-conventions).
- [ ] El cambio es pequeño, atómico y trazable a una HU o tarea.
- [ ] Los enlaces relativos funcionan y el archivo está enlazado desde el `README.md` de su sección (para docs).
- [ ] No hay **secretos** ni datos sensibles: sin credenciales, tokens, `.env` con valores reales, `.pem`/`.key`, ni PII (ver [política de seguridad](../00-documentation-governance/security-policy.md)).
- [ ] `CHANGELOG` actualizado si aplica.
- [ ] Los checks automáticos del PR pasan.

## Checklist de documentación

- [ ] Encabezado estándar SomnGuard presente (logo, título, estado, fecha).
- [ ] IDs coherentes con la convención (HU-<REPO>###/AC-/TC-/RN-/NFR-/ADR-) — ver [agile-conventions.md](../00-documentation-governance/agile-conventions.md).
- [ ] Fechas en `YYYY-MM-DD`; referencias a módulos y ADRs vigentes.

## Checklist de cambios de datos (Liquibase / DDL)

Complementa [modeling-conventions.md](../06-data-architecture/modeling-conventions.md):

- [ ] Cada **changeset** declara `id` + `author` únicos y su **rollback** espejo en `05_rollbacks/`.
- [ ] Orden de aplicación respetado: tablas sin FK en `03_tables`, FKs en `04_alter`, índices en `10_indexes`.
- [ ] Cada FK declara acción `ON UPDATE`/`ON DELETE` acorde a la convención.
- [ ] Nomenclatura de constraints: `pk_`, `uq_`, `fk_`, `ck_`, `ix_`.
- [ ] El módulo escribe solo en su schema (nunca en `public`).
- [ ] Seeds idempotentes (`ON CONFLICT DO NOTHING`); datos de prueba aislados con `context`/`labels`.
- [ ] Migración probada en local: `update` + `status` verdes y rollback reversible.

## Checklist de código (backend Java — futuro)

- [ ] Respeta las fronteras hexagonales: dependencias hacia el centro, sin lógica de negocio en `adapter/*` (ADR-002, structure-rules).
- [ ] Manejo de errores vía `platform/error-handling`; sin stack traces en respuestas.
- [ ] Contratos de API y eventos coherentes con `07-api-design/` y `02-domain/domain-events.md`.
- [ ] Consumo de eventos idempotente (RN-08).
- [ ] Pruebas unitarias/integración acompañan al cambio y pasan.

## Asignación de revisores

Como el equipo es pequeño, los revisores se asignan **manualmente** en cada PR según el área del cambio:

- **Gobernanza** (`00-documentation-governance/`), **datos** (`06-data-architecture/`) y **arquitectura** (`05-architecture/`, ADRs) → Arquitectura.
- **Módulos** (`09-modules/`) y **API** (`07-api-design/`) → Backend.

## Tono de la revisión

- Comentarios accionables y específicos; distinguir bloqueante de sugerencia.
- Preferir preguntas a imperativos cuando haya duda de intención.
- Revisar la **intención del cambio** (¿resuelve la HU?), no solo la sintaxis.

## Ver también

- [Estrategia de pruebas](./test-strategy.md)
- [Convenciones de Git](../00-documentation-governance/git-conventions.md)
- [Definition of Done](../00-documentation-governance/definition-of-done.md)