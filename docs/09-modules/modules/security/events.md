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

El módulo `security` publica los siguientes eventos de dominio para que otros módulos reaccionen:

| Evento | Cuándo se emite | Payload principal | Consumidores típicos |
|--------|-----------------|-------------------|---------------------|
| `UserCreated` | Tras crear usuario (admin) | `user_id`, `email`, `first_name`, `last_name`, `roles[]` | `device_management` (crear conductor), `monitoring` (prefs notificación) |
| `UserUpdated` | Tras actualizar usuario | `user_id`, `changed_fields{}`, `roles[]` | `device_management` (sync conductor), `analytics` (dimensión usuario) |
| `UserDeactivated` | Tras soft-delete usuario | `user_id`, `deactivated_by`, `deactivated_at` | `device_management` (liberar dispositivos), `monitoring` (cancelar notifs) |
| `UserRoleAssigned` | Asignar rol a usuario | `user_id`, `role_code`, `assigned_by`, `expires_at` | `device_management` (permisos dispositivo), `telemetry_service` (scope ingestión) |
| `UserRoleRevoked` | Revocar rol de usuario | `user_id`, `role_code`, `revoked_by` | `device_management`, `telemetry_service`, `analytics` |
| `PasswordResetRequested` | Solicitud reset password | `user_id`, `email`, `expires_at` | `monitoring` (enviar email) |
| `PasswordResetCompleted` | Reset confirmado | `user_id`, `completed_at` | `audit` (log seguridad) |
| `LoginSucceeded` | Login exitoso | `user_id`, `ip_address`, `user_agent`, `roles[]`, `features[]` | `audit`, `analytics` (sesiones) |
| `LoginFailed` | Login fallido | `email_attempted`, `ip_address`, `reason`, `failed_count` | `audit`, `security` (bloqueo) |

## Eventos suscritos (inbound)

El módulo `security` reacciona a:

| Evento | Origen | Acción |
|--------|--------|--------|
| `DeviceAssigned` | `device_management` | Validar que user existe y está activo; actualizar cache de permisos |
| `DeviceUnassigned` | `device_management` | Invalidar cache de permisos del usuario |
| `DriverCreated` | `device_management` | Verificar/crear user vinculado si no existe |

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

- Nomenclatura: `somnguard.security.<Entidad><Acción>.v<version>`
- Versionado: `v1` inicial; breaking changes → `v2` nuevo topic
- Idempotencia: consumidores deben manejar duplicados (usar `id` del evento)
- Orden: no garantizado; diseñar para eventual consistency
- Trazabilidad: `correlation_id` y `causation_id` en headers Kafka

## Referencias

- [Catálogo de eventos global](../../../event-catalog.md)
- [Modelo de datos](./data-model.md)