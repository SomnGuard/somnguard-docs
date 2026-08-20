<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Diseño de API

**Estado:** En progreso
**Fecha:** 2026-08-16

</div>

</div>

Propuesta inicial de diseño de la API del backend (Java 21 / Spring Boot 3.x). Documento de trabajo: los contratos definitivos se validarán con las decisiones registradas en `../05-architecture/decisions/` y las preguntas abiertas de `../15-project-control/open-questions.md`.

## Convenciones generales

| Criterio | Convención |
|----------|------------|
| Estilo | REST |
| Prefijo | `/api/v1` |
| Formato | JSON (`application/json`) |
| Errores | `{"code": "<codigo>", "message": "<descripcion>", "details": [...]}` |
| Paginación | Parámetros `page` (1-based) y `page_size` (máx. 100); respuesta `{ "data": [...], "pagination": { "page", "page_size", "total_items", "total_pages" } }` — ver [guidelines.md](./guidelines.md) |
| Identificadores | UUID en rutas y cuerpo |
| Fechas | ISO 8601 (`yyyy-MM-dd'T'HH:mm:ssXXX`) |
| Autenticación | JWT RS256 + API keys por dispositivo — ver [authentication.md](./authentication.md) |
| Documentación en vivo | SpringDoc/OpenAPI (`/swagger-ui.html`) |

### Códigos HTTP

- `200` OK · `201` Created · `204` No Content
- `400` Bad Request · `401` Unauthorized · `403` Forbidden · `404` Not Found · `409` Conflict
- `422` Unprocessable Entity · `500` Internal Server Error

## Módulo security

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| POST | `/api/v1/auth/login` | Iniciar sesión |
| POST | `/api/v1/auth/logout` | Cerrar sesión |
| POST | `/api/v1/auth/refresh` | Renovar token |
| POST | `/api/v1/auth/password-reset` | Solicitar recuperación de contraseña |
| POST | `/api/v1/auth/password-reset/confirm` | Confirmar recuperación con token |
| POST | `/api/v1/users` | Crear usuario |
| GET | `/api/v1/users/{id}` | Consultar usuario |
| PUT | `/api/v1/users/{id}` | Actualizar usuario |
| DELETE | `/api/v1/users/{id}` | Eliminar cuenta (con retención) |
| GET | `/api/v1/roles` | Listar roles |
| POST | `/api/v1/roles` | Crear rol |
| GET | `/api/v1/modules` | Listar módulos |
| GET | `/api/v1/modules/{id}/features` | Funcionalidades de un módulo |
| POST | `/api/v1/users/{id}/roles` | Asignar rol a usuario |
| DELETE | `/api/v1/users/{id}/roles/{roleId}` | Quitar rol a usuario |
| GET | `/api/v1/audit-login` | Historial de intentos de autenticación |

## Módulo device-management

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| GET | `/api/v1/devices` | Listar dispositivos |
| POST | `/api/v1/devices` | Registrar dispositivo |
| GET | `/api/v1/devices/{id}` | Consultar dispositivo |
| PUT | `/api/v1/devices/{id}` | Actualizar dispositivo |
| POST | `/api/v1/devices/{id}/assign` | Asociar dispositivo a usuario |
| POST | `/api/v1/devices/{id}/unassign` | Desasociar dispositivo |
| GET | `/api/v1/devices/{id}/config` | Consultar configuración |
| PUT | `/api/v1/devices/{id}/config` | Actualizar configuración |

## Módulo telemetry-service

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| POST | `/api/v1/telemetry/events` | Ingresar eventos y evidencia desde el dispositivo |
| GET | `/api/v1/events` | Consultar eventos (filtros por dispositivo, tipo, rango de fechas) |
| GET | `/api/v1/events/{id}` | Consultar detalle de evento |
| GET | `/api/v1/events/{id}/evidence` | Consultar evidencia de un evento |
| GET | `/api/v1/events/{id}/alerts` | Consultar alarmas de un evento |

## Módulo monitoring

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| GET | `/api/v1/notifications` | Listar notificaciones del usuario |
| GET | `/api/v1/notifications/{id}` | Consultar notificación |
| PATCH | `/api/v1/notifications/{id}/read` | Marcar como leída |

## Módulo parameterization

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| GET | `/api/v1/catalogs/event-categories` | Catálogo de categorías de evento |
| GET | `/api/v1/catalogs/severities` | Catálogo de severidades |
| GET | `/api/v1/catalogs/media-types` | Catálogo de tipos de medio |
| GET | `/api/v1/catalogs/sound-patterns` | Catálogo de patrones de sonido |
| GET | `/api/v1/catalogs/event-types` | Catálogo de tipos de evento |
| POST/PUT/DELETE | `/api/v1/catalogs/...` | Administración de catálogos (acceso restringido) |

## Modelo de respuesta de ejemplo

```json
{
  "code": "EVENT_NOT_FOUND",
  "message": "El evento solicitado no existe",
  "details": []
}
```

## Pendientes (no inventar contratos aún)

- Mecanismo de autenticación y autorización (JWT vs sesiones; roles/perfiles).
- Contrato de sincronización offline del dispositivo (formato de payload y archivos multimedia).
- Formato de notificaciones push y estado de lectura.
- Política de retención de datos al eliminar cuenta.
- Respuestas de paginación y filtros definitivos por recurso.