# ADR-013: Video en vivo post-MVP (a demanda para app/portal)

**Estado:** Aceptada (implementada fase 1+2 en local; pendiente validación en campo)
**Fecha:** 2026-09-25 (actualizada 2026-10-02)
**Autores:** Equipo SomnGuard
**Equipos involucrados:** Arquitectura, API, Device, Portal, App
**HU afectadas:** HU-API-012, HU-DEVICE-005, HU-PORTAL-005, HU-APP-004 (Post-MVP, Could 39 SP) · **RF base:** RF-ANA-05, RF-EDGE-13 · **RN base:** RN-ANA-05, RN-EDGE-13

---

## Contexto

El MVP es edge-autónomo + cloud por polling (`ADR-005`): el Pi detecta y pita en local, sincroniza eventos por `POST /telemetry/events` y el portal/app consultan por REST. No hay push server→client ni video en backend (`scope-declaration.md`: streaming fuera de alcance MVP).

Las HU post-MVP piden ver la cámara del device en vivo desde app/portal, a demanda y sin mucha latencia (`user-stories.md:900-1010`):

- `HU-API-012 AC-001/003`: `POST /devices/{id}/stream/start|stop` + signaling offer/answer.
- `HU-DEVICE-005 AC-001/002/003`: SDP via signaling (WebSocket/HTTP), H.264, bitrate 500kbps-2Mbps, auto-stop 30s.
- `HU-PORTAL-005`: modal con player.
- `HU-APP-004`: fullscreen + indicador calidad.

Restricciones reales de campo: el Pi no tiene IP pública (NAT/CGNAT móvil), la cámara es exclusiva por proceso y MediaPipe corre en el hilo del loop.

## Decisión

WebRTC a demanda en dos fases, signaling siempre en la API (`WS /ws/stream` nativo + REST):

**Fase 1 — relay por la API (implementada):** el Pi publica MJPEG `640x480 q55 8fps` y/o P2P aiortc H.264; la API reenvía por room de `session_id`. Sin infra nueva. Latencia ~1s en red local.

**Fase 2 — SFU LiveKit self-hosted (implementada en local):** `livekit/livekit-server:v1.13` + `coturn` en compose (perfil `livekit`). La API emite tokens HS256 (`LiveKitTokenService`: viewer solo-suscribe con identidad única por conexión, Pi solo-publica). El Pi publica con SDK `livekit` desde el frame compartido; portal/app suscriben con `livekit-client`. El relay queda como fallback automático.

Parámetros reales adoptados:

| Tema | Valor |
|------|-------|
| Sesión | `POST start → 201`, `POST stop` idempotente, `GET session`; TTL deslizante 5min (cada poll extiende); solo `DEVICE_ACTIVE` + dueño/admin |
| Pi→API | poll `GET /stream/session` cada 3-5s + `GET /stream/detection`; heartbeat inmediato al arrancar (no espera 30s) |
| Video | `640x480`, MJPEG q55@8fps / H.264 ~10fps; QoS por latencia: buena q55@8, media q45@6, mala q35@4 (rango AC-002 500kbps-2Mbps) |
| Cámara compartida | el vivo reutiliza `last_frame` del loop de captura; sin 2ª apertura |
| Pausa detección | `POST/GET /stream/detection {paused}` (memoria API, sin migración); prioritaria sobre presencia; solo la limpia reanudar o reinicio |
| Estado tiempo real | `DeviceStatusChangedEvent` → push WS `{type:status}` a suscritos `{subscribe-status}` (sin esperar heartbeat) |
| Robustez Pi | `detect_async` con timeout 2s + recreación de landmarker, finalizador neutralizado, watchdog con volcado de stacks |

## Consecuencias

### Positivas

- Funciona en red local hoy (relay) y fuera de casa con SFU (fase 2).
- Reutiliza auth existente: JWT dueño + API key Pi, sin credenciales nuevas.
- Pi sigue autónomo: sin red, detección local intacta; streaming best-effort.
- Pausa y estado en vivo sin latencia de polling largo.

### Negativas / Trade-offs

- Fase 2 exige servidor con IP pública + TLS para salir de LAN; en local basta `ws://`.
- Identidad viewer única por conexión (evita que dos pestañas se pateen).
- `HU-APP-004` sin empezar; Expo Go no soporta WebRTC: app requiere dev-client.
- QoS por heurística de latencia, no por RTCP real (fase 3).

### Riesgos

- CGNAT sin TURN configurado = SFU muerto fuera de LAN; TURN necesita IP pública.
- x264 por software compite CPU con MediaPipe: track acotado a 10fps y QoS lo baja más.
- Pausa en memoria API: se pierde si la API reinicia (el portal re-lee cada 15s).

## Alternativas consideradas

| Alternativa | Por qué se descartó |
|-------------|---------------------|
| MJPEG/HLS continuo al backend | Latencia y datos peores; viola offline-first |
| P2P directo sin SFU como única vía | Falla tras CGNAT; queda solo como fallback local |
| mediasoup/Janus propio | Más operación que LiveKit para 39 SP post-MVP |
| Grabar video continuo en backend | Fuera de alcance (`scope-declaration.md:37`) |

## Referencias

- `docs/04-requirements/user-stories.md:900-1010` (HU-API-012, HU-DEVICE-005, HU-PORTAL-005, HU-APP-004)
- `docs/04-requirements/functional.md:118,136,185,198` (RF-ANA-05, RF-EDGE-13)
- `docs/07-api-design/api-design.md` (contrato streaming + pausa + LiveKit)
- `docs/07-api-design/contracts/openapi/streaming.yaml` (TD-003)
- `docs/05-architecture/decisions/records/ADR-005-offline-first-device.md` (polling vs push)
