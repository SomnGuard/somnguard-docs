<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Registro de riesgos

**Estado:** En progreso
**Fecha:** 2026-08-19

</div>

</div>

Riesgos del proyecto SomnGuard: técnicos, de alcance, de equipo, externos y de seguridad.

## Cómo se calcula la exposición

**Exposición = Probabilidad × Impacto** (escala Baja=1 / Media=2 / Alta=3).

- Baja: producto < 3
- Media: producto 3–6
- Alta: producto > 6

## Riesgos activos

| ID | Categoría | Riesgo | Prob. | Impacto | Exposición | Mitigación | Owner | Estado |
|----|-----------|--------|-------|---------|------------|------------|-------|--------|
| R-001 | Seguridad | Secretos `.env.*` versionados pueden filtrar credenciales de base de datos. | Alta | Alto | Alta | Purgar del historial, rotar credenciales, mover a Secret Manager y bloquear vía `.gitignore`. | PM/Arquitecto | Abierto |
| R-002 | Técnico | Migraciones Liquibase no reproducibles impiden levantar el esquema desde cero. | Media | Alto | Media | Validar `liquibase update` sobre base vacía en CI (ver [convenciones de modelado](../06-data-architecture/modeling-conventions.md)). | Equipo Backend | Abierto |
| R-003 | Alcance | La capa de aplicación (backend, portal, app, edge) no está construida; subestimar su esfuerzo pone en riesgo el cronograma. | Alta | Alto | Alta | Plan por épicas con DoR/DoD (ver [product backlog](../03-product-definition/product-backlog.md)), comprometer alcance por iteración. | PM/Arquitecto | Abierto |
| R-004 | Técnico | Precisión de la detección de fatiga/somnolencia en edge depende de datos de campo; validación tardía puede retrasar el MVP. | Media | Alto | Media | Validar con datos de campo desde el prototipo; umbrales configurables vía `parameterization`. | Equipo Edge | Abierto |
| R-005 | Técnico | Sincronización offline sin idempotencia puede duplicar eventos y corromper métricas. | Media | Medio | Media | ID único por evento + índice de idempotencia (RN-08); pruebas de integración. | Equipo Backend | Abierto |
| R-006 | Seguridad | Exposición de evidencia multimedia (PII) por control de acceso débil. | Media | Alto | Media | URLs firmadas, RBAC por rol (ver [modelo de amenazas](../05-architecture/security-threat-model.md)). | Arquitectura | Abierto |
| R-007 | Equipo | Concentración de conocimiento: una sola persona figura como owner de la mayoría de los documentos; ausencia o rotación frena el proyecto (factor bus = 1). | Media | Alto | Media | Documentar decisiones (ADRs, runbooks), mantener este repo como fuente de verdad, formar equipo por módulo. | PM | Abierto |
| R-008 | Externo | Proveedor de notificaciones push (FCM/APNs) no definido (Q-008) retrasa el módulo monitoring. | Media | Medio | Media | Cerrar pregunta abierta; diseñar el puerto de salida desacoplado del proveedor. | Arquitectura | Abierto |
| R-009 | Alcance | Contratos de API aún no implementados: el diseño podría divergir del modelo de datos real al construir la capa app. | Media | Medio | Media | Contratos por módulo en `07-api-design/contracts/`; publicar OpenAPI solo al estabilizar. | Arquitectura | Abierto |
| R-010 | Técnico | `analytics` (resumen IA) depende de un modelo externo; costo/latencia o indisponibilidad afectan el módulo. | Media | Medio | Media | Diseñar con puerto de salida desacoplado y fallback a resumen sin IA. | Arquitectura | Abierto |

## Bloqueantes activos

| ID | Bloqueante | Condición de desbloqueo | Responsable |
|----|-----------|-------------------------|-------------|
| B-001 | No se puede construir la capa de aplicación con seguridad hasta sanear secretos. | Cerrar R-001 (rotación + Secret Manager). | PM/Arquitecto |
| B-002 | El módulo monitoring no puede diseñarse en detalle sin decisión del canal push. | Cerrar Q-008 (FCM/APNs). | Arquitectura |
| B-003 | La validación del MVP depende de datos de campo del detector. | Completar prototipo edge con datos reales. | Equipo Edge |

## Riesgos aceptados

| ID | Riesgo | Justificación |
|----|--------|---------------|
| RA-001 | En el MVP, el edge envía telemetría con API key del dispositivo en lugar de un flujo M2M más robusto. | Simplicidad del MVP; la clave es rotable y revocable (amenaza T-002 del modelo de amenazas, mitigada). |

## Riesgos resueltos

| ID | Riesgo | Resolución | Estado |
|----|--------|------------|--------|
| — | (Sin cierres registrados aún) | — | — |

## Ver también

- [Preguntas abiertas](./open-questions.md)
- [Modelo de amenazas](../05-architecture/security-threat-model.md)
- [Backlog técnico](./technical-backlog.md)