# ADR-011: Configuración global versionada (global_config.version + pull manual)

**Estado:** Aceptada (implementada)
**Fecha:** 2026-09-14
**Enmienda:** 2026-09-15 — pull manual-only (sin lazy OR), `GET /config` del device persiste `device_config` + `device_config_history`, device restaura caché al arrancar y fusiona `detection_thresholds` (vacío no borra base)
**Autores:** Equipo SomnGuard
**Equipos involucrados:** Arquitectura, Backend, Device, DBA, App/Portal

---

## Contexto

El modelo vigente (`HU-API-005`) es merge en lectura con overrides por device:

- `device_config.configuration` guarda solo deltas por device; `GET /devices/{id}/config` mergea defaults vivos de catálogo (`sound_pattern` activos + `event_type.threshold_config`) + overrides (precedencia override > catálogo).
- `PATCH /devices/{id}/config` (solo admin) persiste overrides, `version++`, historial en `device_config_history`, `pending_config_update=true`.
- `POST /devices/{id}/config/refresh` marca `pending=true`; el próximo `heartbeat` responde `config_pending=true`; el device hace `GET /config` (limpia flag). Sin pull tras cada heartbeat/sync.
- Cambios de catálogo (`HU-API-004`) no generan filas en `device_config_history`; se versionan por entidad (`event_type.version`, `updated_at`) y se propagan solo con Actualizar manual.

Problemas detectados:

1. No hay forma barata de saber "¿estoy desactualizado?": hay que comparar `sources.catalog_updated_at` (timestamp) y no hay contador monotónico.
2. La app no puede mostrar `Versión actual: 15 / Disponible: 16 [Actualizar]` porque no existe versión global.
3. Los overrides por device impiden razonar sobre "la" configuración del sistema; cada device puede tener una vista distinta imposible de auditar como versión única.
4. `DetectionThreshold` y `camera` no existen como tablas: son `event_type.threshold_config JSONB` y passthroughs (`camera_resolution/fps`) + default local del device.

Se necesita: DB como fuente de verdad, una versión global entera y monotónica, cada device guarda la última aplicada, y el flujo manual `refresh -> heartbeat -> GET` ya existente se reutiliza.

---

## Decisión

Se decide **configuración 100% global, sin overrides por device** (opción Solo global), con bump en API Java y pull **manual-only** (sin lazy OR: el device solo pulla tras `POST /refresh`).

### 1. Fuente de verdad y versión global

Nueva tabla singleton en `parameterization`:

```sql
CREATE TABLE parameterization.global_config (
    id          SMALLINT PRIMARY KEY DEFAULT 1 CHECK (id = 1),
    version     INTEGER NOT NULL DEFAULT 1 CHECK (version > 0),
    updated_at  TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_by  UUID,
    CONSTRAINT ck_global_config_singleton CHECK (id = 1)
);

CREATE TABLE parameterization.global_config_history (
    id              UUID PRIMARY KEY,
    version         INTEGER NOT NULL,
    snapshot_json   JSONB NOT NULL,   -- config efectiva completa de esa versión
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by      UUID
);
```

Regla: **cualquier `POST/PATCH/DELETE` efectivo sobre `sound_pattern` o `event_type` (únicas fuentes de la config) incrementa `global_config.version++` en la misma transacción y escribe una fila en `global_config_history`**. El bump vive en **API Java** (`ParameterizationController` / caso de uso), no en trigger PG, para conservar `created_by` (auditoría de usuario) y validaciones de negocio. `severity/media_type/event_category` no bumpean (no forman parte del JSON entregado al device).

El JSON entregado al device **se genera siempre desde DB** (nunca se edita un `config.json` a mano):

```json
{
  "version": 16,
  "thresholds": {"EV-SOM-01": {...}},
  "event_sound_map": {"EV-SOM-01": "AS-02"},
  "detection_thresholds": {},
  "sound_patterns": {"AS-02": {"frequency_hz": 880, "duration_sec": 0.5, "...": "..."}},
  "volume_pct": 80,
  "volume_scale": 0.8,
  "schema_version": 1,
  "sync_interval_sec": 30,
  "sync_interval_seconds": 30,
  "heartbeat_interval_sec": 30,
  "retention_days": 7
}
```

> Nota device: `thresholds`/`event_sound_map` son informativos (el firmware los ignora); solo aplican `detection_thresholds` (fusionado, vacío no borra base), `sound_patterns`, `volumen` e intervalos. Ver `somnguard-device/app/common/config.py:merge_configs`.

### 2. Estado por device

En `device_management.device`:

```sql
ALTER TABLE device_management.device
    ADD COLUMN IF NOT EXISTS applied_config_version INTEGER NOT NULL DEFAULT 0,
    ADD COLUMN IF NOT EXISTS pending_config_update BOOLEAN NOT NULL DEFAULT FALSE;
-- pending_config_update ya existe en API/docs; falta en DDL: se regulariza aquí.
```

- `applied_config_version`: última versión global aplicada por el device (la que devolvió el último `GET /config` con API Key).
- Regla principal: **`device.applied_config_version < global_config.version` ⇒ desactualizado** (la app lo muestra vía `GET /devices/{id}/config/status{outdated}` o `applied vs available`).
- `pending_config_update`: semántica manual (`POST /refresh` lo pone en `true`; el `GET` del device lo limpia). El `heartbeat` expone **solo el flag manual** (sin lazy OR) para no auto-pullar el device (ver §3).

`tablas `device_config` / `device_config_history` quedan **REACTIVADAS como registro del pull**: cada `GET /config` con API Key hace upsert en `device_config` (snapshot global aplicado, `version` por optimistic locking) + INSERT en `device_config_history` (`change_reason="Pull manual tras refresh (heartbeat pending)"`, actor = `device.created_by` o `SYSTEM_ID` si es NULL por self-register). `PATCH /devices/{id}/config` queda **DEPRECATED** (responde `410 Gone` con puntero a catálogos; se elimina en la siguiente mayor). `RF-PAR-04` (overrides) queda derogada.

### 3. Flujo (reutiliza HU-API-005, cambia el origen del cambio)

```text
DB (sound/event cambia) -> global 15->16 + history
APP: GET /devices/{id}/config/status => {applied: 15, available: 16, pending: false, outdated: true} => "Desactualizado [Actualizar]"
APP: POST /devices/{id}/config/refresh -> pending=true
DEVICE: heartbeat c/30s -> BACKEND responde configPending = pending (solo manual)
DEVICE: GET /devices/{id}/config (API Key) -> BACKEND genera JSON v16, upsert device_config + history, applied=16, pending=false, last_config_pull_at=now
DEVICE: aplica en caliente y cachea data/device_config.cache.json (sobrevive reinicios)
RESULTADO: DEVICE A applied=16 ✅, DEVICE B applied=15 ⚠
```

Contratos:

- `POST /devices/{id}/heartbeat` → `{..., configPending: bool, configVersionAvailable: int}`. `configPending` es **solo flag manual** (sin fan-out masivo y sin auto-pull).
- `GET /devices/{id}/config` → siempre incluye `version` (global) en raíz + `sources={global_version, global_updated_at}`. Con API Key persiste `applied_config_version` + upsert `device_config` + INSERT `device_config_history`; con JWT es solo lectura (para la app).
- `GET /devices/{id}/config/status` → `{applied, available, pending, outdated}` (la app lo usa para el botón Actualizar).
- `POST /devices/{id}/config/refresh` → `pending=true` (gesto de usuario; idempotente).
- `PATCH /devices/{id}/config` → `410 Gone` (solo global).

Sin fan-out: ningún cambio de catálogo hace `UPDATE device SET pending=true` masivo. La detección es por `GET /config/status` y `applied vs available` en la app; el `heartbeat` solo transporta el gesto manual.

### 4. Alcance explícito (no-tablas)

- No se crea tabla `DetectionThreshold`: sigue siendo `event_type.threshold_config JSONB`.
- No se crea tabla `camera`: `camera_resolution/fps` siguen siendo default local del device (`config/device.default.json`) + valores globales informativos si se publican (p. ej. `heartbeat_interval_sec`); el device sanitiza rangos y nunca rompe el arranque.
- `volume_pct/sync/heartbeat/retention` con defaults `80/30/30/7` viven como defaults de generación (no como tabla propia) hasta que se justifique un catálogo de `system_defaults`.

---

## Consecuencias

### Positivas

- Una sola versión auditable (`global_config_history.snapshot_json` por versión).
- La app puede mostrar `actual vs disponible` sin diff de JSON.
- Se elimina la complejidad de overrides por device (merge, normalizer de alias, validación 422 por device).
- Sin fan-out: el bump es `O(1)` (una fila), la detección es por comparación (`GET /config/status`, app) + flag manual (`heartbeat`).
- Reutiliza `heartbeat -> GET` ya implementado en API y device.

### Negativas / Trade-offs

- Se pierde personalización por device (comportamiento intencional de esta ADR; si vuelve a necesitarse, requiere nueva ADR y redefine `versión efectiva = f(global, override)`). Excepción vigente: `data/device_config.override.json` local en campo (gana sobre la caché).
- `PATCH /config` queda como deuda de migración hasta su borrado; `device_config*` son registro activo del pull (no deuda).
- Concurrencia en bump: dos admins editando catálogo a la vez deben serializar `version++` (transacción + `SELECT ... FOR UPDATE` sobre la fila singleton).
- `GET /config` pasa a depender de `global_config` en cada llamada (lectura de una fila; cacheable en memoria con invalidación por versión).

### Riesgos

- DDL `device` sin `pending_config_update` en producción → la regularización (`ADD COLUMN IF NOT EXISTS`) debe correr antes que la API que lo lee. Mitigación: migración Liquibase dedicada + backfill `applied_config_version=0` (fuerza un pull inicial).
- Devices antiguos que ignoran `version` y solo aplican claves conocidas → siguen funcionando (ignoran `version`), pero no reportan `applied`. Mitigación: el backend considera `applied=0` como desactualizado hasta el primer `GET` nuevo.
- Borrado prematuro de `device_config*` rompería rollback → Mitigación: mantener 2 releases en modo solo-lectura.

---

## Alternativas consideradas

| Alternativa | Por qué se descartó |
|-------------|---------------------|
| Híbrido (global + overrides por device, `efectiva = f(global, override)`) | Mantiene merge/normalizer/422 por device; impide "una versión = un snapshot"; se reevaluará solo si vuelve la necesidad de personalización |
| Bump en trigger PG | Pierde `created_by` de negocio y validaciones; más difícil de testear; se prefiere transacción en caso de uso Java |
| Fan-out (`UPDATE device SET pending=true` en cada bump) | `O(N)` por cambio de catálogo, bloqueos masivos en flotas; manual-only lo evita |
| `max(updated_at)` como versión lógica (sin tabla) | Sin monotonicidad entera, sin snapshot por versión, comparar timestamps es frágil (reloj, empates); útil como interino, no como modelo final |
| Tabla por cada dominio (`detection_threshold`, `camera_config`) | Sobremodelado para el MVP: los datos ya viven en `threshold_config` y defaults; se difiere |

---

## Referencias

- `../../../04-requirements/functional.md` → RF-PAR-04 (derogada), RF-PAR-05/06, RF-DEV-03, RF-TEL-05
- `../../../02-domain/entities-and-rules.md` → RN-PAR-04 (derogada), RN-PAR-05/06, RN-DEV-04, RN-TEL-05
- `../../../04-requirements/user-stories.md` → HU-API-004 (bump), HU-API-005 (solo global)
- `../../../04-requirements/traceability-matrix.md` → ADR-011
- `../../../06-data-architecture/data-dictionary.md` → `global_config`, `device.applied_config_version`
- `../../../07-api-design/api-design.md` → heartbeat manual-only, `GET` versionado (+persiste pull), `GET /config/status`, `PATCH` 410, `POST /refresh`
- `../../../02-domain/domain-events.md` → `config.global_version_incremented`
