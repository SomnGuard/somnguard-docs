# ADR-005: Offline-First en Device Edge (SQLite Local + Sync Idempotente)

**Estado:** Aceptada
**Fecha:** 2026-08-22
**Autores:** Equipo SomnGuard
**Equipos involucrados:** Arquitectura, Device, Backend

---

## Contexto

El dispositivo SomnGuard (Raspberry Pi + cámara) opera en vehículos con **conectividad intermitente** (túneles, zonas rurales, movilidad). Requisitos del SRS (RNF-2.1, RNF-2.2, RNF-3.1):
- Detección y alertas **siempre operativas** sin red
- **Ningún evento confirmado se pierde** tras reinicio/fallo eléctrico
- Sync automático cuando hay conexión
- Almacenamiento local limitado (SD card 16-32 GB)

El dispositivo genera:
- **Eventos** (somnolencia, distracción, cinturón, sistema) con timestamp, severidad, metadata
- **Evidencia multimedia** (frame JPEG del momento, ~50-200 KB c/u)
- **Alertas locales** (códigos AS-01..AS-09) con timestamp

---

## Decisión

### 1. Buffer Local en SQLite
| Aspecto | Especificación |
|---------|----------------|
| **Archivo** | `data/db/somnguard_local.db` (WAL mode, `PRAGMA journal_mode=WAL`) |
| **Tabla principal** | `pending_events` |
| **Columnas** | `id` (PK UUID v7), `event_json` (TEXT, evento serializado), `evidence_path` (TEXT, nullable), `status` (ENUM: `PENDING`, `SENDING`, `ACKED`, `FAILED`), `retries` (INTEGER DEFAULT 0), `created_at` (TIMESTAMP), `updated_at` (TIMESTAMP) |
| **Índices** | `idx_status_created (status, created_at)`, `idx_event_id (json_extract(event_json, '$.event_id'))` |
| **Tamaño objetivo** | < 50 MB eventos + < 500 MB evidencia (política retención 7 días) |

### 2. Generación de Eventos (Producer)
- Eventos creados en `domain` → serializados a JSON → `INSERT INTO pending_events (event_json, evidence_path, status) VALUES (?, ?, 'PENDING')`
- **`event_id` (UUID v7)** generado en device — **clave de idempotencia** global
- Evidencia guardada en `data/media/{event_id}.jpg` (ruta relativa en `evidence_path`)

### 3. Detección de Conectividad
| Mecanismo | Detalle |
|-----------|---------|
| **Healthcheck** | `HEAD https://api.somnguard.com/actuator/health` cada 30s (configurable via `device_config.sync_interval_seconds`) |
| **Timeout** | 5s connect, 10s read |
| **Estados** | `ONLINE` (2xx) / `OFFLINE` (timeout, 5xx, network error) |
| **Cambio OFFLINE→ONLINE** | Dispara sync inmediato + backoff reset |

### 4. Sync Automático (Consumer)
```python
# Pseudocode del loop de sync
while True:
    if connectivity == ONLINE:
        batch = select_pending(limit=100)  # ORDER BY created_at ASC
        if batch:
            for event in batch:
                event.status = 'SENDING'
                update(event)
            
            response = http_post(
                url=f"{API_URL}/telemetry/events",
                headers={"X-Device-ID": DEVICE_ID, "X-API-KEY": API_KEY},
                json={"events": [json.loads(e.event_json) for e in batch]}
            )
            
            if response.status == 201:
                acked_ids = response.json()['event_ids']
                delete_where_id_in(acked_ids)  # Limpieza inmediata
                pull_config_if_needed()
            elif response.status == 409:  # Duplicate
                delete_where_id_in(batch.event_ids)  # Ya persistido en server
            else:
                increment_retries(batch)
                apply_backoff()
    sleep(SYNC_INTERVAL)
```

### 5. Backoff Exponencial con Jitter
| Parámetro | Valor |
|-----------|-------|
| `base_delay` | 60s (1 min) |
| `max_delay` | 3600s (1 hora) |
| `max_retries` | 10 (luego `FAILED` + AS-09 + log persistente) |
| `jitter` | `random(0, 30s)` |
| **Fórmula** | `min(base_delay * 2^attempt + jitter, max_delay)` |

### 6. Idempotencia en Server (API)
- **Índice único** `telemetry.event(event_id)` → `409 Conflict` si duplicado
- Device **borra local** en `201` (ACK) **y en `409`** (ya existe)
- **Nunca reintenta** evento con `409` — ya persistido

### 7. Pull de Configuración Remota
- Tras **sync exitoso (201)**: `GET /devices/{id}/config` → merge con defaults → guarda en `device_config_cache` + aplica en runtime
- Config incluye: umbrales detección, `sound_pattern` (AS-XX), volumen, `sync_interval_seconds`

### 8. Limpieza y Retención
| Política | Acción |
|----------|--------|
| **ACK confirmado** | `DELETE` inmediato de `pending_events` + archivo evidencia |
| **Fallidos persistentes** (`retries >= 10`) | `status = 'FAILED'`; se quedan en BD; log persistente; AS-09 cada 24h |
| **Retención por tiempo** | Job diario: `DELETE FROM pending_events WHERE created_at < now() - 7 days AND status != 'PENDING'` |
| **Espacio crítico** (`> 90%` disk) | AS-09; borra `FAILED` más antiguos; si persiste, borra `ACKED` más antiguos (should not happen) |

### 9. Recuperación ante Fallos
| Escenario | Comportamiento |
|-----------|----------------|
| **Power loss / Crash** | WAL de SQLite garantiza durabilidad; al reiniciar, eventos `PENDING`/`SENDING` reintentan |
| **SD card corruption** | Eventos no sincronizados se pierden (aceptado: RNF-2.2 dice "evento confirmado no se pierde"; confirmado = ACK del server) |
| **Factory reset** | Borra `data/db/` y `data/media/`; device vuelve a estado `Registrado` |

---

## Consecuencias

### Positivas
- **Disponibilidad 100% offline:** Detección, alerta, registro funcionan sin red
- **Cero pérdida de eventos confirmados:** ACK del server = borrado local; power loss no pierde `PENDING` (WAL)
- **Idempotencia real:** `event_id` UUID v7 + índice único server = reintentos seguros
- **Auto-limpieza:** Buffer no crece indefinidamente; retención 7d + espacio crítico
- **Config dinámica:** Device se autoconfigura tras cada sync

### Negativas / Trade-offs
- **Complejidad en device:** Estado machine (PENDING/SENDING/ACKED/FAILED), backoff, retención
- **Eventos `FAILED` invisibles para usuario** hasta sync manual o admin → Mitigación: métrica `device.sync.failed_count` en telemetría
- **Evidencia multimedia consume espacio** → Mitigación: compresión JPEG 80%; max 1 frame/evento; retención 7d

### Riesgos
- **Clock drift en device** → `occurred_at` incorrecto → Mitigación: NTP sync en boot + cada 24h; `event_id` UUID v7 incluye timestamp
- **API Key comprometida** → Mitigación: Rotación via admin (`rotate-key`); rate limit 1000/min; device solo escribe `/telemetry/events`
- **Schema evolution** → `event_json` v1 vs v2 → Mitigación: Versionar en `event_json` (`{"version": 1, ...}`); server acepta múltiples versiones

---

## Alternativas consideradas

| Alternativa | Por qué se descartó |
|-------------|---------------------|
| **Solo online (sin buffer)** | Violenta RNF-2.1, RNF-3.1; túneles/zonas rurales = pérdida eventos críticos |
| **Buffer en memoria (RAM)** | Power loss = pérdida total; viola RNF-2.2 |
| **Archivos JSONL (append-only) en FS** | Sin índices, sin consultas eficientes, sin transacciones; SQLite es ACID + ligero |
| **Message broker local (MQTT broker en Pi)** | Overkill; añade proceso, memoria, complejidad; SQLite + HTTP POST es suficiente |
| **Sync push (server → device via WebSocket/MQTT)** | Device no tiene IP pública; NAT/firewall; polling HTTP es más robusto en redes móviles |
| **UUID v4 (random) para event_id** | v7 (time-ordered) permite ordenamiento natural + debugging + particionado temporal en server |

---

## Referencias

- [cross-cutting.md](../cross-cutting.md#4-idempotencia-y-consistencia-en-sincronización)
- [functional.md](../../../04-requeriments/functional.md) → RF-EDGE-08,10,12, RF-TEL-04,07
- [software-analysis.md](../../../04-requeriments/software-analysis.md) §6.1 (sd-offline-sync, ac-offline-sync)
- [entities-and-rules.md](../../../02-domain/entities-and-rules.md) → RN-TEL-04, RN-EDGE-08,10,12
- [SRS RNF-2.1, 2.2, 3.1](../../../04-requeriments/01-srs/) (offline operation, persistence, availability)
- SQLite WAL mode: https://www.sqlite.org/wal.html
- UUID v7: https://datatracker.ietf.org/doc/html/draft-peabody-dispatch-new-uuid-format