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
  "features": ["DEVICE_READ", "EVENT_READ", "NOTIFICATION_READ"],
  "exp": 1777000000,
  "iat": 1776996400,
  "jti": "uuid-v4"
}
```

## 3. RBAC (roles y permisos)

- Modelo: `user_role` ↔ `role_feature` ↔ `feature` (ver [`../06-data-architecture/02-modules-entities.md`](../06-data-architecture/02-modules-entities.md)).
- El token trae los `features` pre-calculados; cada endpoint declara el permiso que exige (RN-02).
- Ejemplos de features: `ACCOUNT_MANAGE`, `DEVICE_ASSIGN`, `EVENT_WRITE`, `CATALOG_MANAGE`, `REPORT_READ`.

## 4. API keys de dispositivo

- Emitidas al registrar el dispositivo; rotables y revocables por el administrador (mitiga la amenaza T-002 del modelo de amenazas).
- Envío de telemetría: `POST /api/v1/telemetry/events` con header `X-API-Key: <key>`.
- La clave identifica el dispositivo y habilita las reglas RN-03 y RN-08 (validación de dispositivo activo + idempotencia).
- Nunca se expone la clave en respuestas ni logs.

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
| Rate limiting en login | Bloqueo temporal tras intentos fallidos |
| Expiración de sesión | TTL de token + refresh |

## Ver también

- [Documento de arquitectura](../05-architecture/architecture-document.md#10-autenticación-y-autorización)
- [Convenciones REST](./guidelines.md)
- [Modelo de amenazas](../05-architecture/security-threat-model.md)
- [Política de seguridad](../00-documentation-governance/security-policy.md)