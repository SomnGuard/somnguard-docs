<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Autenticación y autorización de la API

**Estado:** En progreso
**Fecha:** 2026-08-19

</div>

</div>

Mecanismos de autenticación y autorización de la API de SomnGuard. Alineado con la sección 10 del documento de arquitectura y con el módulo `security`.

## 1. Mecanismos

| Actor | Mecanismo | Scope |
|-------|-----------|-------|
| Usuario de plataforma (web/móvil) | JWT Bearer (RS256) | Sesiones de usuario |
| Dispositivo edge | API key por dispositivo | Envío de telemetría y consulta de configuración |
| Administrador | JWT con rol `admin` | Gestión de cuentas, dispositivos y catálogos |

## 2. JWT de usuario

- Emitido por el módulo `security` tras login exitoso (`POST /api/v1/auth/login`).
- Firma RS256; los módulos verifican localmente con la clave pública (JWKS).
- Claims mínimos: `sub` (user id), `email`, `roles`, `features` (permisos pre-calculados), `exp`, `iat`, `jti`.
- TTL: acceso corto (p. ej. 15-60 min) + refresh token de mayor duración.
- Revocación: por `jti` o por invalidación de sesión (auditoría en `audit_login`).

```json
{
  "sub": "uuid-v4",
  "email": "conductor@example.com",
  "roles": ["user"],
  "features": ["device.read", "event.read", "notification.read"],
  "exp": 1777000000,
  "iat": 1776996400,
  "jti": "uuid-v4"
}
```

## 3. RBAC (roles y permisos)

- Modelo: `user_role` ↔ `role_feature` ↔ `feature` (ver [`../06-data-architecture/02-modules-entities.md`](../06-data-architecture/02-modules-entities.md)).
- El token trae los `features` pre-calculados; cada endpoint declara el permiso que exige (RN-02).
- Ejemplos de features: `user.read`, `device.write`, `event.write`, `catalog.write`, `analytics.report` (formato `recurso.accion` en minúscula, ver seeds).

## 4. API keys de dispositivo

- Emitidas al registrar el dispositivo (manual `POST /devices` admin o `POST /devices/self-register`, estado `REGISTERED`); rotables y revocables por el administrador (mitiga la amenaza T-002 del modelo de amenazas).
- Envío de telemetría: `POST /api/v1/telemetry/events` con headers `X-Device-ID: <uuid>` + `X-API-Key: <key>`. Ambos obligatorios.
- Saludo/heartbeat: `POST /api/v1/devices/{id}/heartbeat` con los mismos headers. Primer heartbeat válido provoca `ASSIGNED->ACTIVE`; sin heartbeat 5min el device se considera `OFFLINE` (`last_heartbeat_at`).
- La clave identifica el dispositivo y habilita las reglas RN-03 y RN-08 (validación de dispositivo asignado/activo + idempotencia).
- Nunca se expone la clave en respuestas ni logs. En `REGISTERED` sin `assign` la API responde `403` en `/telemetry/events` aunque la key sea válida.

## 4b. Tres credenciales distintas (ver ADR-010)

| Credencial | Para qué | Scope | Vida |
|------------|----------|-------|------|
| Provisioning Token (`X-Provision-Token`) | Nacimiento: solo `POST /devices/self-register` | Un endpoint, `max_uses` (1), expira (7d), revocable, solo hash en BD | Muere al usarse |
| Device API Key (`X-Device-ID + X-API-Key`) | Operación continua: heartbeat, telemetría, evidencia, config | Endpoints operativos, sin `max_uses`, rotable/revocable | Hasta revocación/rotación |
| Claim Code (body `POST /devices/claim`) | Reclamo por el usuario (crea `device_assignment`) | Un uso, hash en BD, se invalida al reclamar | Hasta el claim |

Un token comprometido **no** opera ni telemetra; una key comprometida **no** registra devices; el claim solo asigna.

## 5. Flujos críticos

| Flujo | Referencia |
|-------|------------|
| Login y emisión de JWT | `sd-authentication` (08-uml) |
| Restablecimiento de contraseña | `sd-password-reset` (08-uml) |
| Sincronización del dispositivo | `sd-offline-sync` (08-uml) |

## 6. Controles de seguridad

| Control | Valor |
|---------|-------|
| Cifrado en tránsito | TLS 1.2+ |
| Hash de contraseñas | bcrypt (nunca plano, RN-01) |
| Rate limiting en login | 5 req/min por IP en API + bloqueo tras 5 fallos (`locked_until`, ver `cross-cutting.md`) |
| Expiración de sesión | TTL de token + refresh |

## Ver también

- [Documento de arquitectura](../05-architecture/architecture-document.md#10-autenticación-y-autorización)
- [Convenciones REST](./guidelines.md)
- [Modelo de amenazas](../05-architecture/security-threat-model.md)
- [Política de seguridad](../00-documentation-governance/security-policy.md)