<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Diseño de API

**Estado:** En progreso
**Fecha:** 2026-08-19

</div>

</div>

Propuesta inicial de diseño de la API del backend (Java 21 / Spring Boot 4.1.1). Documento de trabajo: los contratos definitivos se validarán con las decisiones registradas en `../05-architecture/decisions/` y las preguntas abiertas de `../15-project-control/open-questions.md`.

## Convenciones generales

| Criterio | Convención |
|----------|------------|
| Estilo | REST |
| Prefijo | `/api/v1` |
| Formato | JSON (`application/json`) |
| Errores | `{"error": {"code", "message", "details", "trace_id"}}` — ver [guidelines.md](./guidelines.md) |
| Paginación | Parámetros `page` (1-based) y `page_size` (máx. 100); respuesta `{ "data": [...], "pagination": { "page", "page_size", "total_items", "total_pages" } }` — ver [guidelines.md](./guidelines.md) |
| Identificadores | UUID en rutas y cuerpo |
| Fechas | ISO 8601 (`yyyy-MM-dd'T'HH:mm:ssXXX`) |
| Autenticación | JWT RS256 + API keys por dispositivo — ver [authentication.md](./authentication.md) |
| Idempotencia | `event_id` en telemetría (`201` con duplicados reportados); header `Idempotency-Key` en `POST /devices`, `/assign`, `/auth/*`, `PATCH /config`, `PATCH /rotate-key` — ver [guidelines.md](./guidelines.md) |
| Documentación en vivo | SpringDoc/OpenAPI (`/swagger-ui.html`) |

### Códigos HTTP

- `200` OK · `201` Created · `204` No Content
- `400` Bad Request · `401` Unauthorized · `403` Forbidden · `404` Not Found · `409` Conflict
- `422` Unprocessable Entity · `500` Internal Server Error

## Módulo security

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| POST | `/api/v1/auth/register` | Registrar cuenta (retorna 201) |
| POST | `/api/v1/auth/login` | Iniciar sesión |
| POST | `/api/v1/auth/logout` | Cerrar sesión |
| POST | `/api/v1/auth/refresh` | Renovar token |
| POST | `/api/v1/auth/verify-email` | Verificar correo con token |
| POST | `/api/v1/auth/forgot-password` | Solicitar recuperación (token 1h al correo) |
| POST | `/api/v1/auth/reset-password` | Confirmar recuperación con token |
| POST | `/api/v1/users` | Crear usuario |
| GET | `/api/v1/users` | Listar usuarios |
| GET | `/api/v1/users/{id}` | Consultar usuario |
| PATCH | `/api/v1/users/{id}` | Actualizar usuario |
| PATCH | `/api/v1/users/me` | Actualizar perfil propio |
| DELETE | `/api/v1/users/{id}` | Eliminar cuenta (con retención) |
| GET | `/api/v1/roles` | Listar roles |
| POST | `/api/v1/roles` | Crear rol |
| PUT | `/api/v1/roles/{id}` | Reemplazar rol |
| DELETE | `/api/v1/roles/{id}` | Desactivar rol |
| GET | `/api/v1/features` | Listar features |
| POST | `/api/v1/features` | Crear feature |
| PUT | `/api/v1/features/{id}` | Reemplazar feature |
| DELETE | `/api/v1/features/{id}` | Eliminar feature |
| GET | `/api/v1/modules` | Listar módulos |
| GET | `/api/v1/modules/{id}/features` | Funcionalidades de un módulo |
| POST | `/api/v1/users/{id}/roles` | Asignar rol a usuario |
| DELETE | `/api/v1/users/{id}/roles/{roleId}` | Quitar rol a usuario |
| POST | `/api/v1/role-features` | Asignar feature a rol (body `roleId + featureId`) |
| DELETE | `/api/v1/role-features/{id}` | Quitar feature a rol |
| GET | `/api/v1/audit-login` | Historial de intentos de autenticación |

## Módulo device-management

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| GET | `/api/v1/devices` | Listar dispositivos |
| POST | `/api/v1/devices` | Registrar dispositivo |
| GET | `/api/v1/devices/{id}` | Consultar dispositivo |
| PUT | `/api/v1/devices/{id}` | Actualizar dispositivo |
| POST | `/api/v1/devices/{id}/assign` | Asociar dispositivo a usuario (REGISTERED->ASSIGNED) |
| POST | `/api/v1/devices/{id}/unassign` | Desasociar dispositivo (->REGISTERED) |
| POST | `/api/v1/devices/{id}/heartbeat` | Saludo periódico del device (ASSIGNED->ACTIVE, ACTIVE<->OFFLINE). Auth: `X-Device-ID + X-API-Key`. Actualiza `last_heartbeat_at, last_seen_ip, firmware_version` |
| GET | `/api/v1/devices/{id}/config` | Consultar configuración (device con API Key o user con JWT) |
| PATCH | `/api/v1/devices/{id}/config` | Actualizar configuración (solo admin JWT) |
| PATCH | `/api/v1/devices/{id}/rotate-key` | Rotar API Key (solo admin JWT). Invalida anterior de inmediato, devuelve nueva key una sola vez. Estado no cambia; device con key vieja recibe `401` hasta reprovisionar |

## Módulo telemetry-service

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| POST | `/api/v1/telemetry/events` | Ingresar lote de eventos **solo metadata JSON** `{"events":[{event_id UUIDv7, device_id, occurred_at UTC, event_type, severity, metadata, has_evidence}]}`. Auth `X-Device-ID + X-API-Key`. Lote máx 100, timeout 10s. `event_type` y `severity` van por **código de catálogo** (ej. `EV-SOM-01`); la API resuelve a `event_type_id/severity_id`, `422` si desconocido. Idempotencia por `event_id`: el lote **siempre responde `201 {acked_ids[], duplicate_ids[]}`** (los duplicados se reportan, no son error). Device borra local ambos. `REGISTERED` sin assign -> `403` |
| POST | `/api/v1/telemetry/events/{eventId}/evidence` | Subir evidencia de un evento (1 archivo por evento MVP). `multipart/form-data` single file `file` JPG ~50-200KB (integridad v1: `size_bytes` + ETag MinIO; `checksum_sha256` futuro). `201 {evidence_id}`; `409` si el evento ya tiene evidencia; `404` si el evento no existe. Alternativa prod: presigned PUT directo a MinIO (ver ADR-006). Mapeo `event_id -> minio_key {device_id}/{YYYY}/{MM}/{DD}/{event_id}.jpg` |
| GET | `/api/v1/events` | Consultar eventos (filtros por dispositivo, tipo, rango de fechas) |
| GET | `/api/v1/events/{id}` | Consultar detalle de evento |
| GET | `/api/v1/events/{id}/evidence` | Consultar evidencia de un evento |
| GET | `/api/v1/events/{id}/alerts` | Consultar alarmas de un evento |

## Módulo monitoring

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| GET | `/api/v1/notifications` | Listar notificaciones del usuario |
| GET | `/api/v1/notifications/{id}` | Consultar notificación |
| POST | `/api/v1/notifications/{id}/read` | Marcar como leída |

## Módulo parameterization

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| GET | `/api/v1/catalogs/event-categories` | Catálogo de categorías de evento |
| GET | `/api/v1/catalogs/severities` | Catálogo de severidades |
| GET | `/api/v1/catalogs/media-types` | Catálogo de tipos de medio |
| GET | `/api/v1/catalogs/sound-patterns` | Catálogo de patrones de sonido |
| GET | `/api/v1/catalogs/event-types` | Catálogo de tipos de evento |
| POST/PATCH/DELETE | `/api/v1/catalogs/...` | Administración de catálogos (acceso restringido; update parcial con `PATCH`) |

## Módulo analytics

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| GET | `/api/v1/analytics/timeline` | Línea de tiempo de eventos por dispositivo/rango |
| GET | `/api/v1/analytics/metrics` | Métricas de comportamiento (sesiones de riesgo, duración) |
| GET | `/api/v1/analytics/reports` | Reportes generados (resumen IA, descargables) |
| POST | `/api/v1/analytics/reports` | Solicitar generación de reporte (propuesta async, `202 Accepted`) |

## Modelo de respuesta de ejemplo

```json
{
  "error": {
    "code": "EVENT_NOT_FOUND",
    "message": "El evento solicitado no existe",
    "details": [],
    "trace_id": "b3f1c2a4-..."
  }
}
```

## Pendientes (no inventar contratos aún)

- ~~Contrato de sincronización offline del dispositivo (formato de payload y archivos multimedia)~~ Definido: `POST /telemetry/events JSON {"events":[]}` + `POST /telemetry/events/{id}/evidence multipart` (ver arriba + ADR-005 + ADR-006). Descartado `base64 en event_json` (infla 33%) y `multipart` en lote.
- Formato de notificaciones push y estado de lectura.
- Política de retención de datos al eliminar cuenta.
- Respuestas de paginación y filtros definitivos por recurso.

> Autenticación y autorización (JWT RS256 + API keys por dispositivo, RBAC por feature) están definidas en [authentication.md](./authentication.md).