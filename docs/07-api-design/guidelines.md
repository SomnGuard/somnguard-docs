<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Convenciones de diseño de API REST

**Estado:** En progreso
**Fecha:** 2026-08-19

</div>

</div>

Convenciones REST que rigen las APIs de SomnGuard. Complementa el diseño de API existente en `api-design.md` y aplica a los seis módulos del backend.

## 1. Estilo y formato

- **REST sobre HTTP/JSON.** `Content-Type: application/json; charset=utf-8`.
- Nombres de campos en `snake_case` (coherente con el modelo de datos y el JWT).
- Fechas y horas en **ISO 8601 UTC** (`2026-08-01T14:30:00Z`).
- Identificadores públicos: **UUID** (nunca IDs autoincrementales expuestos).
- Cargas de archivo (evidencia multimedia) vía `multipart/form-data` **solo en `POST /telemetry/events/{eventId}/evidence` (1 archivo por evento)**. El lote `POST /telemetry/events` es **solo JSON** `{"events":[]}` sin archivos inline (descartado `base64 en event_json`: infla 33% y rompe límite 50MB).

## 2. Versionado

- Versión mayor en la **ruta base**: `/api/v1/...`. Un salto de versión (`v2`) solo ante cambios incompatibles.
- Cambios retrocompatibles (nuevos campos opcionales, nuevos endpoints) no cambian la versión.

## 3. Nombres de recursos

- Recursos en **plural, sustantivo, kebab-case**: `/devices`, `/events`, `/notifications`.
- Jerarquía por anidamiento solo cuando el hijo no tiene sentido fuera del padre: `/devices/{device_id}/events`.
- Nada de verbos en la ruta; las acciones se modelan como sub-recurso o transición de estado: `POST /devices/{id}/assign` (no `/assignDevice`).

## 4. Métodos HTTP y semántica

| Método | Uso | Idempotente |
|--------|-----|-------------|
| `GET` | Leer recurso o colección | Sí |
| `POST` | Crear recurso / disparar acción | No |
| `PUT` | Reemplazo completo del recurso | Sí |
| `PATCH` | Actualización parcial | No (por convención) |
| `DELETE` | Eliminación (lógica por defecto) | Sí |

## 5. Códigos de estado HTTP

| Código | Cuándo |
|--------|--------|
| `200 OK` | GET/PUT/PATCH exitoso con cuerpo |
| `201 Created` | POST que crea recurso; incluye `Location` y el recurso creado |
| `202 Accepted` | Trabajo aceptado para proceso asíncrono (p. ej. generación de reportes) |
| `204 No Content` | DELETE o acción sin cuerpo de respuesta |
| `400 Bad Request` | Payload malformado o validación de dominio fallida |
| `401 Unauthorized` | Token ausente, inválido o expirado |
| `403 Forbidden` | Permiso insuficiente (RBAC) |
| `404 Not Found` | Recurso inexistente o fuera del scope del usuario |
| `409 Conflict` | Conflicto de estado (p. ej. evidencia ya existente, asignación inválida). En lote de telemetría NO hay 409: duplicados van en `duplicate_ids` del `201` |
| `422 Unprocessable Entity` | Reglas de negocio no satisfechas con payload sintácticamente válido |
| `429 Too Many Requests` | Límite de tasa superado |
| `500 Internal Server Error` | Error no controlado (nunca filtrar stack traces) |

## 6. Paginación, filtrado y ordenamiento

- Paginación por offset por defecto: `?page=1&page_size=20` (`page_size` máximo 100). Solo los endpoints de alto volumen (`GET /api/v1/events`, `GET /api/v1/analytics/timeline`) podrán usar cursor (`?cursor=<opaco>&limit=50`) cuando se implementen; mientras tanto usan offset.
- Respuesta de colección envuelta con metadatos:

```json
{
  "data": [ ],
  "pagination": {
    "page": 1,
    "page_size": 20,
    "total_items": 137,
    "total_pages": 7
  }
}
```

- Filtrado por query params explícitos: `?device_id=<uuid>&severity=critical` (códigos en minúscula, ver catálogo `severity`).
- Ordenamiento: `?sort=created_at:desc` (campo permitido por endpoint, no arbitrario). En `cross-cutting.md` la notación `-occurred_at` equivale a `occurred_at:desc`.

## 7. Formato de errores

Cuerpo de error uniforme en toda la plataforma (alineado con `platform/error-handling`):

```json
{
  "error": {
    "code": "DEVICE_NOT_ACTIVE",
    "message": "El dispositivo no está activo para recibir eventos.",
    "details": [
      { "field": "device_id", "issue": "INACTIVE_DEVICE" }
    ],
    "trace_id": "b3f1c2a4-..."
  }
}
```

- `code`: identificador estable en `SCREAMING_SNAKE_CASE`, independiente del idioma.
- `message`: legible para humanos, en español; no expone internals.
- `details`: opcional; errores de validación por campo.
- `trace_id`: correlación con logs y observabilidad (ver `platform/logging`).

## 8. Autenticación y autorización

- Autenticación vía **JWT Bearer** (RS256, clave pública JWKS) para usuarios de plataforma y clientes web/móvil.
- **API keys** por dispositivo para el envío de telemetría desde el edge (sin JWT de usuario): headers obligatorios `X-Device-ID + X-API-Key` (ver `authentication.md`).
- Autorización **RBAC por `feature`**: el token trae los permisos pre-calculados; cada endpoint declara el permiso que exige.
- Detalle en [authentication.md](./authentication.md).

## 9. Endpoints de referencia (por módulo)

| Módulo | Ejemplos de recursos |
|--------|----------------------|
| security | `POST /api/v1/auth/login`, `POST /api/v1/auth/forgot-password`, `GET /api/v1/users`, roles y features |
| device-management | `POST /api/v1/devices`, `POST /api/v1/devices/{id}/assign`, `GET /api/v1/devices/{id}/config` |
| telemetry-service | `POST /api/v1/telemetry/events` (API key, lote), `GET /api/v1/events`, evidencia multimedia |
| monitoring | `GET /api/v1/notifications`, `POST /api/v1/notifications/{id}/read` |
| parameterization | CRUD de catálogos: `/api/v1/catalogs/event-categories`, `/severities`, `/media-types`, `/sound-patterns`, `/event-types` |
| analytics | `GET /api/v1/analytics/timeline`, `/metrics`, `/reports` |

> Los ejemplos son ilustrativos; el contrato definitivo por módulo se publica como OpenAPI en [`./contracts/openapi/`](./contracts/openapi/) (ver nota en `contracts/README.md`).

## 10. Convenciones transversales

- **Idempotencia (dos mecanismos, no mezclados):** telemetría usa `event_id UUIDv7` único: el lote `POST /telemetry/events` **siempre responde `201 {acked_ids[], duplicate_ids[]}`** y el device borra local ambos (RN-08). `409` solo en `POST /telemetry/events/{id}/evidence` si ya existe evidencia. Header `Idempotency-Key` **solo** en `POST` no-telemetría (`/devices`, `/{id}/assign`, `/auth/*`, `/devices/self-register` con serial como clave natural) y en `PATCH /devices/{id}/config` y `PATCH /rotate-key` (reintentos del portal). No usar header en `/telemetry/events`. En `self-register` repetido con mismo token+serial: `200 {device_id, status}` sin reexponer la key.
- **Correlación**: propagar `trace_id` entre módulos y hacia logs/eventos.
- **Rate limiting**: respuesta `429` con `Retry-After`; límites en `cross-cutting.md` (auth 5/min API, device 1000/min gateway + 60/2/10 por endpoint).
- **Compatibilidad**: nunca romper un contrato publicado sin subir versión mayor.

## Ver también

- [Diseño de API](./api-design.md)
- [Autenticación](./authentication.md)
- [Modelo de amenazas](../05-architecture/security-threat-model.md)