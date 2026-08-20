<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Eventos de dominio

**Estado:** En progreso
**Fecha:** 2026-08-19

</div>

</div>

Vista de negocio de los eventos de dominio del sistema: disparador, cambio de estado y consumidores. Complementa el registro por módulo (ver [`../09-modules/`](../09-modules/)). El formato técnico del envelope se define en la plantilla de módulo ([`events.md`](../09-modules/modules/_template/module/events.md)).

## Catálogo de eventos

| Evento | Disparador | Cambio de estado | Consumidores |
|--------|------------|------------------|--------------|
| `account.registered` | Registro de cuenta | Cuenta pendiente → activa | security (auditoría) |
| `account.password_reset_requested` | Solicitud de reseteo | Se emite token temporal | security |
| `device.registered` | Alta de dispositivo | Dispositivo registrado | device-management |
| `device.assigned` | Asignación a cuenta | Dispositivo asignado | device-management, telemetry-service |
| `device.config_updated` | Cambio de configuración | Nueva configuración vigente | device-management (sincronización) |
| `device.synced` | Lote de eventos recibido | Sincronización confirmada | telemetry-service |
| `event.recorded` | Evento detectado recibido | Evento persistido (idempotente) | telemetry-service, analytics |
| `evidence.uploaded` | Evidencia multimedia subida | Referencia disponible | telemetry-service, analytics |
| `alert.generated` | Evento crítico detectado | Registro en `alert_log` | telemetry-service |
| `notification.sent` | Evento crítico evaluado | Notificación emitida | monitoring |
| `notification.read` | Usuario abre notificación | Emitida → leída | monitoring |

## Envelope estándar

```json
{
  "event_id": "uuid-v4",
  "event_type": "<modulo>.<entidad>.<accion>",
  "version": "1.0",
  "timestamp": "2026-01-01T00:00:00Z",
  "source_module": "<nombre-modulo>",
  "correlation_id": "uuid-v4",
  "payload": {}
}
```

El envelope es el **sobre** y el `payload` el **contenido**: los campos de cabecera (`event_id`, `event_type`, `timestamp`, `source_module`, `correlation_id`) son iguales en todos los eventos; el `payload` cambia por evento y lleva los datos propios del hecho ocurrido.

## Ejemplo: `event.recorded` enviado por el dispositivo

```json
{
  "event_id": "a1b2c3d4-1111-2222-3333-444455556666",
  "event_type": "telemetry.event.recorded",
  "version": "1.0",
  "timestamp": "2026-01-01T00:00:00Z",
  "source_module": "telemetry-service",
  "correlation_id": "9f8e7d6c-1111-2222-3333-444455556666",
  "payload": {
    "device_id": "raspberry-0001",
    "event_category": "fatiga",
    "event_type": "microsueño",
    "severity": "alta",
    "captured_at": "2026-01-01T00:00:01Z",
    "evidence_ref": "minio://somnguard/evidencia/2026-01-01/a1b2.mp4"
  }
}
```

El `correlation_id` une los eventos de una misma operación (p. ej. el lote sincronizado por el dispositivo comparte el mismo valor en `device.synced`, `event.recorded` y `evidence.uploaded`).

## Reglas

- Los eventos se nombran `<modulo>.<entidad>.<accion>` en inglés.
- La idempotencia se garantiza con `event_id` (RN-08).
- **Hoy el payload viaja por HTTP** (dispositivo → API, POST /events). **Entre módulos no hay mensajería**: el monolito hexagonal se comunica por puertos (interfaces de entrada/salida en Java), no por eventos. El envelope solo se materializa como mensaje si se adopta una cola (p. ej. `adapter/in/amqp`) para desacoplar procesos como analytics.

## Ver también

- [Mapa de dominio](domain-map.md)
- [Entidades y reglas de negocio](entities-and-rules.md)
- [Análisis del software](../04-requeriments/software-analysis.md)