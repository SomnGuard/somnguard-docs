<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../../../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Eventos — [nombre-modulo]

**Estado:** Plantilla
**Fecha:** 2026-08-19

</div>

</div>

> **PLANTILLA** — Registrar eventos de dominio publicados y consumidos. El catálogo general está en [02-domain/domain-events.md](../../../../02-domain/domain-events.md). Eliminar esta línea antes de hacer commit.

> Última actualización: YYYY-MM-DD

## Eventos publicados

| Evento | Canal / destino | Payload principal | Consumidores conocidos |
|--------|-----------------|-------------------|------------------------|
| `[entidad].[accion]` | [puerto in de otro módulo / AMQP] | `{ id, ... }` | `[modulo-consumidor]` |

## Eventos consumidos

| Evento | Módulo fuente | Acción que dispara |
|--------|---------------|--------------------|
| `[entidad].[accion]` | `[modulo]` | [qué hace este módulo al recibirlo] |

## Formato de envelope

Todos los eventos siguen el envelope estándar:

```json
{
  "event_id": "uuid-v4",
  "event_type": "<entidad>.<accion>",
  "version": "1.0",
  "timestamp": "2026-01-01T00:00:00Z",
  "source_module": "<nombre-modulo>",
  "correlation_id": "uuid-v4",
  "payload": {}
}
```

## Política de reintentos

<!-- Circuit breaker, dead letter queue, retries — completar si se adopta mensajería (adapter/in/amqp). Mientras sea monolito, la comunicación entre módulos usa puertos de entrada directos. -->