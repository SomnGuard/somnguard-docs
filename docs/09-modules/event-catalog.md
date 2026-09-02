<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Catálogo de Eventos

**Estado:** Borrador  
**Fecha:** 2026-08-22

</div>

</div>

> **Objetivo:** Inventario centralizado de todos los eventos que circulan entre módulos, device↔API y servicios. Los eventos son **contratos de integración** con formato de envelope estandarizado.
>
> **Convención de naming:** `<entidad>.<accion>` — todo en minúsculas con puntos, en inglés, verbo en pasado.
> Ejemplo: `device.synced`, `event.recorded`, `alert.generated`
>
> **Fuente de verdad:** `docs/02-domain/domain-events.md`, `data-dictionary.md` (codes EV-SOM-*, EV-DIS-*, EV-CIN-*, EV-SYS-*), `cross-cutting.md` (reglas de sincronización), `ADR-005` (offline-first device).

---

## 1. Convención de naming

| Patrón | Ejemplo | Descripción |
|--------|---------|-------------|
| `<entidad>.<accion>` | `device.synced` | Evento detectado recibido, Evento persistido (idempotente) |
| `<módulo>.<entidad>.<accion>` | `telemetry.event.ingested` | Formato extendido con módulo emisor (opcional para traces) |
| **Verbo** | Siempre en pasado (created, synced, generated, started) | Facilita la lectura de "qué ya pasó" |
| **Idioma** | Inglés completo (no abbreviations sin definir) | Los eventos son contratos de integración entre servicios/módulos |

> **Nota:** A diferencia del diseño de microservicios (que usa `<service>.<entity>.<action>`), SomnGuard usa **formato simple** `<entidad>.<accion>` porque el módulo emisor se identifica con `source_module` del envelope (ver sección 3).

---

## 2. Eventos por Publicador (Módulo)

### 2.1 `device_management` (Dispositivo → API)

| Evento | Descripción | Consumidores | Referencia |
|--------|-------------|--------------|------------|
| `device.synced` | Lote de eventos recibido y confirmado por API | `telemetry_service` (persiste events), `monitoring` (actualiza estado device) | RF-EDGE-10, HU-DEVICE-003 |
| `device.config.pulled` | Configuración remota descargada por device | `device_management` (actualiza device_config_cache) | RF-EDGE-11, HU-DEVICE-004 |
| `device.state.changed` | Estado device cambió (ACTIVE↔OFFLINE, etc.) | `monitoring`, `telemetry_service` | ADR-005, cross-cutting.md §5.2 |

### 2.2 `telemetry_service` (Device → API → BD)

| Evento | Descripción | Consumidores | Referencia |
|--------|-------------|--------------|------------|
| `event.recorded` | Evento detectado recibido, persistido en BD (idempotente) | `telemetry_service` (propia tabla), `analytics` (línea de tiempo), `monitoring` (alertas) | RN-TEL-01, HU-API-007, HU-DEVICE-003 |
| `alert.generated` | Evento crítico detectado, registrado en alert_log | `monitoring` (notificaciones), `audit-service` (historial) | RF-TEL-03, HU-API-009 |
| `evidence.uploaded` | Evidencia multimedia (imagen/video) subida a MinIO | `telemetry_service` (guarda minio_key), `monitoring` (thumbnails) | RF-TEL-02, HU-API-007 |
| `event.sync.failed` | Sincronización fallida después de reintentos exponenciales | `device_management` (actualiza buffer retry), `monitoring` (métricas) | RF-EDGE-12, HU-DEVICE-003 |

### 2.3 `monitoring` (Notificaciones)

| Evento | Descripción | Consumidores | Referencia |
|--------|-------------|--------------|------------|
| `notification.sent` | Notificación entregada al usuario (push, email, in-app) | `audit-service` (historial), `app` (confirmación UI) | RF-MON-01, HU-API-009, HU-APP-001 |
| `notification.delivered` | Notificación entregada exitosamente al canal | `audit-service`, `app` (tracking) | RF-MON-03, HU-MON-03 |
| `notification.read` | Notificación leída por usuario (in-app) | `audit-service` (statísticas) | RF-MON-03 (opcional) |

### 2.4 `analytics` (Línea de tiempo y reportes)

| Evento | Descripción | Consumidores | Referencia |
|--------|-------------|--------------|------------|
| `analytics.metrics.updated` | Métricas agregadas actualizadas (freq, severidad, tendencia) | `portal`, `app` (dashboard) | RF-ANA-02, HU-PORTAL-002, HU-APP-002 |
| `analytics.summary.generated` | Resumen IA generado para un usuario/periodo | `portal`, `app` (botón "Resumen IA") | RF-ANA-03, HU-PORTAL-003, HU-APP-002 |

### 2.5 `security` (Autenticación y autorización)

| Evento | Descripción | Consumidores | Referencia |
|--------|-------------|--------------|------------|
| `auth.login.attempt` | Intento de login (éxito o fallo, con IP, user-agent) | `audit-service` (persiste audit_login) | RF-SEC-09, cross-cutting.md §7.2 |
| `auth.logout` | Cierre de sesión de usuario | `audit-service`, `monitoring` (tracking sesiones) | RF-SEC-03, cross-cutting.md §7.2 |

---

## 3. Estructura del Envelope de Evento

Todos los eventos siguen un **envelope estándar** con campos de cabecera obligatorios. El `payload` varía según el tipo de evento.

### 3.1 Campos obligatorios (envelope header)

| Campo | Tipo | Descripción | Ejemplo |
|-------|------|-------------|---------|
| `event_id` | UUID v7 | Identificador único, generado en device, **único e idempotente** | `a1b2c3d4-e5f6-7890-abcd-ef1234567890` |
| `event_type` | VARCHAR | Código del evento (EV-SOM-01, EV-DIS-02, etc.) | `EV-SOM-05` |
| `timestamp` | TIMESTAMPTZ | Momento del evento en UTC | `2026-08-22T14:30:00.123Z` |
| `source_module` | VARCHAR | Módulo que publicó el evento | `device_management` o `telemetry_service` |
| `correlation_id` | UUID (opcional) | Agrupa eventos de una misma operación (lote sync) | `d4c3b2a1-0000-1111-2222-333344445555` |

### 3.2 Ejemplo de envelope completo

```json
{
  "envelope": {
    "event_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "event_type": "EV-SOM-05",
    "timestamp": "2026-08-22T14:30:00.123Z",
    "source_module": "device_management",
    "correlation_id": "d4c3b2a1-0000-1111-2222-333344445555"
  },
  "payload": {
    "device_id": "dev-001",
    "occurred_at": "2026-08-22T14:29:00.000Z",
    "severity": "critical",
    "event_data": {
      "type": "microsueño detectado",
      "confidence": 0.95
    }
  }
}
```

> **Nota:** El `payload` es libre según el `event_type`; la única obligatoriedad es que todo evento debe poder ser identificado por su `event_id` para deduplicación.

---

## 4. Eventos Internos de Servicio (No cruzan frontera)

Estos eventos son de **orquestación interna** de un módulo (worker↔worker, scheduler/cron) y **no son contratos de integración** que consume otro módulo. Se documentan aquí para trazabilidad.

| Evento interno | Servicio | Propósito |
|----------------|----------|-----------|
| `document.render.requested` | document-service | pdf-renderer: solicitud de render encolada |
| `document.render.completed` | document-service | pdf-renderer: render finalizado |
| `document.render.failed` | document-service | pdf-renderer: render fallido → reintento/DLQ |
| `document.lifecycle.tick` | document-service | document-lifecycle-worker: pulso de cron |
| `monitoring.kpi.tick` | monitoring-service | cron de recálculo de KPIs cada 5 min |
| `telemetry.batch.processed` | telemetry_service | Lote de events sincronizados exitosamente (post-ACK) |
| `config.sync.applied` | device_management | Configuración device_config aplicada en runtime after sync |

---

## 5. Normalización Pendiente

| Evento | Conflito | Propuesta | Decisión pendiente |
|--------|----------|-----------|--------------------|
| `monitoring.alert.generated` vs `monitoring.alert.triggered` | Usado en alert-worker/notification-worker ≡ `monitoring.alert.triggered` | Unificar al nombre canónico `monitoring.alert.triggered` | **Pendiente**: definir nombre canónico único |
| `device.synced` (device_management) vs `event.sync.failed` (telemetry_service) | Mismos orígenes pero vistas diferentes | `device.synced` es confirmación de sincronización completa; `event.sync.failed` es fallo después de reintentos | **Pendiente**: definir si son eventos separados o uno con status |

---

## 6. Nota de Migración

Los nombres de evento en SomnGuard siguen la convención `<entidad>.<accion>` en inglés, basada en el catálogo de dominio (`docs/02-domain/domain-events.md`). Cambios previos de nombres (ej. en versiones anteriores del proyecto) deben ser_trackeados en este catálogo para:

1. **Actualizar consumidores**: Cuando se renombra un evento, todos los módulos que lo suscriban deben actualizarse en paralelo
2. **Compatibilidad hacia atrás**: El `event_id` UUID v7 permite correlación incluso si el nombre cambia
3. **Wildcard en audit-worker**: El `audit-service` suscribe por wildcard, por lo que cambios de naming no afectan su lógica de consumo
4. **Contratos API**: Cualquier endpoint que acepte/retorne nombres de eventos (ej. `GET /telemetry/events?event_type=XX`) debe actualizarse en la misma PR

---

## 7. Referencias Cruzadas

| Documento | Sección | Qué aporta |
|-----------|---------|------------|
| `domain-events.md` | `02-domain/domain-events.md` | Catálogo base con 4 eventos dominio (`device.synced`, `event.recorded`, `alert.generated`, `notification.sent`) |
| `domain-events.md` | `02-domain/domain-events.md` | Eventos de negocio; complementa el catálogo por módulo |
| `data-dictionary.md` | `06-data-architecture/data-dictionary.md` | Codes EV-SOM-*, EV-DIS-*, EV-CIN-*, EV-SYS-* y estructura de event_type |
| `cross-cutting.md` | `05-architecture/cross-cutting.md` | Reglas de sincronización, IDs únicos, formatos de envelope, idempotencia |
| `ADR-005` | `05-architecture/decisions/records/ADR-005-offline-first-device.md` | Offline-first, sync automático, backoff exponencial, UUID v7 para event_id |
| `ADR-009` | `05-architecture/decisions/records/ADR-009-status-parametrized-audit.md` | `status_category` + `status` en eventos (DETECTED, REGISTERED, SYNCHRONIZED, ANALYZED, ARCHIVED) |
| `guidelines.md` | `07-api-design/guidelines.md` | Contratos API para `/telemetry/events` y filtros/frecuentes |
| `module-catalog.md` | `09-modules/module-catalog.md` | Módulos que publican/consumen cada evento |

---

## Próximos Pasos

1. **Validar** este catálogo con el equipo de arquitectura y el team de device (revisión 30 min)
2. **Añadir** a `LISTA_DOCS_OTRO-PROJECT-PARA-SOMNGUARD.md` como entregable de PRIORIDAD 2
3. **Integrar** en la `ci-cd-strategy.md` validación de nombres de eventos en PRs (validar que nuevos events sigan convención `<entidad>.<accion>`)
4. **Crear** `_template/service/events.md` plantilla estándar para nuevos eventos por módulo
5. **Actualizar** `domain-events.md` con los eventos nuevos que añada este catálogo
6. **Definir** nombres canónicos en la sección "Normalización Pendiente" tras decisión del equipo
7. **Documentar** en los `ADR` correspondientes decisiones sobre nombres de eventos que tengan impacto transversal