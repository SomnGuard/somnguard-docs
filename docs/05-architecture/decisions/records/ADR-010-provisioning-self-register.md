# ADR-010: Aprovisionamiento y auto-registro de dispositivos (Provisioning Token + API Key + Claim)

**Estado:** Aceptada con enmienda 2026-09-07 (claim reutilizable público, ver § Enmienda)
**Fecha:** 2026-09-05
**Autores:** Equipo SomnGuard
**Equipos involucrados:** Arquitectura, Backend, Device, Seguridad

---

## Contexto

El alta de dispositivos exige hoy `POST /devices` con JWT de admin (`HU-API-006 AC-001`): un humano crea cada device, copia su API key y la inyecta en el equipo. No escala a flotas y acopla logística con credenciales operativas.

Se necesita que el dispositivo se registre solo en su primer arranque, con seguridad equivalente: secreto de un solo uso para nacer, credencial operativa permanente y código para que un usuario reclame el equipo. Las tres piezas deben tener responsabilidades separadas (un secreto comprometido no debe servir para las otras dos funciones).

Requisitos: TLS, hash en BD (nunca plano), expiración, revocación, rate limiting, auditoría e idempotencia ante pérdida de respuesta.

---

## Decisión

Se decide un flujo en tres credenciales con dos endpoints nuevos y uno de reclamo, conviviendo con el alta manual (`POST /devices` sigue vigente para laboratorio y reposición):

### 1. Provisioning Token (nacimiento, un solo uso)

- El admin lo genera vía `POST /devices/provisioning-tokens` (JWT + feature `device.provision`). Respuesta `201` con el token **en claro una sola vez** + `token_id`, `expires_at` (default 7 días), `max_uses` (default 1).
- Se guarda **solo el hash** (SHA-256 + sal por fila); expiración, `max_uses`, `uses_count`, `revocado`, `device_id` asociado tras el uso.
- Scope exclusivo: `POST /devices/self-register`. No abre telemetría, heartbeat, evidencia ni config.
- Entropía 256 bits, TLS obligatorio, nunca en logs, rate limit estricto (`5/min` por IP + por token), auditoría de creación/uso/revocación.

### 2. Auto-registro (self-register)

- El device lee `serial_number` (hardware) y `firmware_version` (software); recibe el token por variable de entorno o configuración segura. El humano **no** teclea serial ni firmware.
- `POST /devices/self-register` con `X-Provision-Token` + `Idempotency-Key` (o serial como clave natural) + body `{serialNumber, firmwareVersion}`.
- El backend valida token (existe, no expirado, no revocado, con usos disponibles), valida serial, rechaza serial duplicado, crea el device en `REGISTERED`, genera su **Device API Key** (32 B) y su **claim_code**, asocia y consume el token, audita. Responde `201 {device_id, api_key, claim_code}` **una sola vez**.
- Reintento tras pérdida de respuesta: mismo token + mismo serial → `200 {device_id, status}` **sin** reexponer la `api_key`. Si el device perdió su key, solo queda rotación/recovery administrativo (`PATCH /rotate-key`).
- Tras guardar `device_id + api_key` en almacenamiento seguro (`chmod 600`, luego `/etc/somnguard/` en Pi), el device deja de usar el token.

### 3. Device API Key (operación continua)

- Credencial permanente del equipo: heartbeat, `POST /telemetry/events`, evidencia, `GET /config`. Sin `max_uses`; rotable/revocable (`PATCH /rotate-key`); scope idéntico al actual.
- Una API Key comprometida **no** permite registrar devices; un token comprometido **no** permite operar ni telemetría.

### 4. Claim Code (reclamo por el usuario)

- Código permanente por device generado en el registro (manual o self), guardado **en claro** (`device.claim_code`, UNIQUE) y visible en `GET /devices` (enmienda 2026-09-07; antes hash de un solo uso, derogado por migración `021`). El admin puede imprimirlo/entregarlo con el equipo.
- Solo reclamable en `REGISTERED` (sin asignación activa): el usuario lo introduce y llama `POST /devices/claim` (JWT + feature `device.claim`) → crea `device_assignment`, `REGISTERED → ASSIGNED`, marca `claimed_at`. Si el device está asignado, el claim responde `409` hasta que se libere.
- `unassign` → `REGISTERED` limpia `claimed_at` y **el mismo código vuelve a servir** (ciclo reclamar ↔ liberar). El `assign` directo de admin sigue disponible.
- Primer heartbeat válido posterior: `ASSIGNED → ACTIVE`. `ACTIVE ↔ OFFLINE` por heartbeat como hoy; `UNREGISTERED` es solo el estado conceptual previo a existir en BD.

---

## Enmienda 2026-09-07 — claim_code público reutilizable (RF-DEV-12, RN-DEV-09)

**Motivo:** con claim de un solo uso, un `unassign` + re-asignación dejaba el device sin código válido y sin forma de reclamarlo de nuevo.

**Cambio:** el claim pasa de secreto hash de un uso a identificador público permanente por device (`VARCHAR(20)` UNIQUE, formato `XXXX-XXXX-XXXX`; filas legacy `CLM-XXXXXXXXXXXX` vía backfill en migración `021`, que elimina `claim_code_hash`). Reglas: solo sirve en `REGISTERED`; `409` si asignado; `unassign` lo libera; visible en `GET /devices` y `GET /devices/{id}`.

**Riesgo aceptado:** quien vea el código impreso puede reclamar el equipo whenever esté liberado. Se mitiga con: solo-`REGISTERED`, rate limit en `claim`, auditoría `CLAIMED`, y asignación 1:1 vigente (RN-DEV-01). Alternativa descartada "Claim sin hash" de § Alternativas queda invertida por esta enmienda.

---

## Consecuencias

### Positivas

- **Escala:** flotas sin alta manual por equipo; logística (claim impreso) separada de seguridad (token de un uso).
- **Menor blast radius:** tres secretos con scopes disjuntos; el token muere al nacer el device.
- **Idempotencia real:** reintentos de primer arranque no duplican devices ni reexponen keys.
- **Trazable:** cada paso auditado; estados y transiciones reutilizan ADR-009.

### Negativas / Trade-offs

- **Más superficie:** dos endpoints nuevos + reclamo; más seeds de features (`device.provision`, `device.claim`) y dos tablas (`device_provisioning_token`, `device_provisioning_audit`) + columnas de claim en `device`.
- **Distribución del token:** hay que llevarlo al device por canal seguro (imagen, USB, QR efímero); si se filtra antes del uso, hay que revocarlo.
- **Claim impreso:** si se pierde el papel, se necesita reemisión administrativa.

### Riesgos

- **Token interceptado antes del uso** → Mitigación: expiración corta, un uso, revocación inmediata, alerta de uso inesperado.
- **Enumeración de seriales/claims** → Mitigación: rate limit agresivo en `self-register` y `claim`, respuestas sin oráculo (mismo tiempo y mensaje genérico).
- **Pérdida de API Key en campo** → Mitigación: solo recovery vía `rotate-key` autenticado como admin; el token original ya no sirve.
- **Reloj del device** → `occurred_at` y expiración se validan con hora de servidor.

---

## Alternativas consideradas

| Alternativa | Por qué se descartó |
|-------------|---------------------|
| Bootstrap con clave compartida por ambiente (`X-Bootstrap-Key`) | Un secreto global registra infinitos devices si se filtra; sin trazabilidad por equipo |
| Solo alta manual por admin | No escala; acopla logística y credenciales |
| mTLS con certificados por device | Complejidad de PKI/renovación para el MVP; API Key + token es suficiente |
| Claim sin hash (código en claro en BD) | Expone todos los reclamos ante lectura de BD; el hash lo evita |
| Reexponer la API Key en reintentos | Rompe la regla de "mostrar una sola vez"; se devuelve solo `device_id` + estado |

---

## Referencias

- [`../../../07-api-design/api-design.md`](../../../07-api-design/api-design.md) (endpoints `provisioning-tokens`, `self-register`, `claim`)
- [`../../../07-api-design/authentication.md`](../../../07-api-design/authentication.md) (token vs key vs claim)
- [`../../cross-cutting.md`](../../cross-cutting.md#1-autenticación-y-autorización) (scopes y rate limits)
- [`../../../06-data-architecture/data-dictionary.md`](../../../06-data-architecture/data-dictionary.md) (`device_provisioning_token`, claim en `device`)
- [`../../../02-domain/entities-and-rules.md`](../../../02-domain/entities-and-rules.md) (RN-DEV-08..10)
- [`./ADR-005-offline-first-device.md`](./ADR-005-offline-first-device.md) (idempotencia y reintentos)
- [`./ADR-009-status-parametrized-audit.md`](./ADR-009-status-parametrized-audit.md) (estados y transiciones)
