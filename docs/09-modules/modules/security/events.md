<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Eventos — security

**Estado:** En progreso
**Fecha:** 2026-09-01

</div>

</div>

> Última actualización: 2026-09-01

## Eventos publicados (outbound)

El módulo `security` publica los siguientes eventos de dominio para que otros módulos reaccionen. Nombres en forma CloudEvents `Type` (transporte futuro; hoy la comunicación es por puertos, sin broker — ver `domain-events.md`). Equivalentes de catálogo en `../../event-catalog.md` §2.5:

| Evento | Cuándo se emite | Payload principal | Consumidores típicos |
|--------|-----------------|-------------------|---------------------|
| `UserCreated` (`user.created`) | Tras crear usuario (admin) | `user_id`, `email`, `first_name`, `last_name`, `roles[]` | `device_management` (validar asignaciones), `monitoring` (preferencias de notificación) |
| `UserUpdated` (`user.updated`) | Tras actualizar usuario | `user_id`, `changed_fields{}`, `roles[]` | `device_management`, `analytics` (dimensión usuario) |
| `UserDeactivated` (`user.deactivated`) | Tras soft-delete usuario | `user_id`, `deactivated_by`, `deactivated_at` | `device_management` (liberar dispositivos), `monitoring` (cancelar notifs) |
| `UserRoleAssigned` (`user.role.assigned`) | Asignar rol a usuario | `user_id`, `role_code`, `assigned_by`, `expires_at` | `device_management` (permisos dispositivo), `telemetry_service` (scope ingestión) |
| `UserRoleRevoked` (`user.role.revoked`) | Revocar rol de usuario | `user_id`, `role_code`, `revoked_by` | `device_management`, `telemetry_service`, `analytics` |
| `PasswordResetRequested` (`auth.password.reset.requested`) | Solicitud reset password | `user_id`, `email`, `expires_at` | `monitoring` (enviar email) |
| `PasswordResetCompleted` (`auth.password.reset.completed`) | Reset confirmado | `user_id`, `completed_at` | `security` (log en `audit_login`) |
| `LoginSucceeded` (`auth.login.succeeded`) | Login exitoso | `user_id`, `ip_address`, `user_agent`, `roles[]`, `features[]` | `security` (`audit_login`), `analytics` (sesiones) |
| `LoginFailed` (`auth.login.failed`) | Login fallido | `email_attempted`, `ip_address`, `reason`, `failed_count` | `security` (`audit_login`, bloqueo) |

## Eventos suscritos (inbound)

El módulo `security` reacciona a:

| Evento | Origen | Acción |
|--------|--------|--------|
| `device.assigned` | `device_management` | Validar que user existe y está activo; actualizar cache de permisos |
| `device.unassigned` | `device_management` | Invalidar cache de permisos del usuario |

## Formato de evento (CloudEvents 1.0)

```json
{
  "specversion": "1.0",
  "id": "uuid-v7",
  "source": "somnguard.security",
  "type": "somnguard.security.UserCreated.v1",
  "time": "2026-09-01T15:30:00Z",
  "datacontenttype": "application/json",
  "data": {
    "user_id": "uuid",
    "email": "conductor@flota.com",
    "first_name": "Juan",
    "last_name": "Pérez",
    "roles": ["user"]
  }
}
```

## Convenciones

- Nomenclatura: `somnguard.security.<Entidad><Acción>.v<version>` en transporte (futuro); nombre lógico de catálogo `<entidad>.<acción>` en minúscula entre paréntesis
- Versionado: `v1` inicial; breaking changes → `v2` nuevo topic
- Idempotencia: consumidores deben manejar duplicados (usar `id` del evento)
- Orden: no garantizado; diseñar para eventual consistency
- Trazabilidad: `correlation_id` y `causation_id` en headers (futuro broker; hoy sin mensajería entre módulos)

## Referencias

- [Catálogo de eventos global](../../event-catalog.md)
- [Modelo de datos](./data-model.md)