<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Matriz de trazabilidad

**Estado:** Estable
**Fecha:** 2026-08-23

</div>

</div>

Matriz completa que relaciona **Requisitos Funcionales (RF)** ↔ **Historias de Usuario (HU)** ↔ **Módulo Backend** ↔ **Reglas de Negocio (RN)** ↔ **NFR** ↔ **ADR** ↔ **Pruebas**. Fuente: `functional.md`, `user-stories.md`, `entities-and-rules.md`, `non-functional.md`, ADRs.

---

## Matriz Principal: RF → HU → Módulo → RN → NFR → ADR → Pruebas

| RF ID | Descripción | HU(s) Asociadas | Módulo Backend | RN Origen | NFR | ADR | Estrategia de Pruebas |
|-------|-------------|-----------------|----------------|-----------|-----|-----|----------------------|
| RF-SEC-01 | Registro cuenta (email/tel único, hash bcrypt) | HU-API-001, HU-PORTAL-001, HU-APP-001 | Security | RN-SEC-01, RN-SEC-02 | NFR-01 | ADR-001 | Unitarias: validador unicidad, hash bcrypt. Integración: POST /auth/register 201/409 |
| RF-SEC-02 | Login JWT RS256 + refresh token (15min/7d) | HU-API-001, HU-PORTAL-001, HU-APP-001 | Security | RN-SEC-03, RN-SEC-04 | NFR-01, NFR-04 | ADR-001 | Unitarias: generación/validación JWT, rotación refresh. Integración: login 200, token válido 15min |
| RF-SEC-03 | Logout con invalidación refresh token (blocklist) | HU-API-001, HU-PORTAL-001, HU-APP-001 | Security | RN-SEC-05 | NFR-01 | ADR-001 | Integración: logout 204, refresh posterior 401 |
| RF-SEC-04 | Recuperación contraseña via token temporal (1h) | HU-API-002, HU-PORTAL-001, HU-APP-001 | Security | RN-SEC-06 | NFR-01 | ADR-001 | Integración: forgot→reset flow, token expira 1h, un solo uso |
| RF-SEC-05 | Actualización datos personales (unicidad email/tel) | HU-API-002, HU-PORTAL-001, HU-APP-001 | Security | RN-SEC-07 | NFR-01 | — | Unitarias: validador PATCH /users/me. Integración: 200/409 |
| RF-SEC-06 | Asociación device ↔ user (1:1, device_assignment) | HU-API-006, HU-PORTAL-002 | Device Management | RN-DEV-01 | NFR-01 | — | Integración: assign/unassign, state machine |
| RF-SEC-07 | Desasociación device (libera para otro user) | HU-API-006 | Device Management | RN-DEV-02 | NFR-01 | — | Integración: unassign → estado Registrado |
| RF-SEC-08 | Eliminación cuenta soft-delete (ventana 30d) | HU-API-002 | Security | RN-SEC-08 | NFR-01, NFR-06 | — | Integración: DELETE /users/me → deleted_at, recuperación 30d |
| RF-SEC-09 | Auditoría login (IP, user-agent, éxito/fallo, ts) | HU-API-001 | Security | RN-SEC-09 | NFR-04 | — | Unitarias: audit_login insert. Integración: login fallido/éxito registra |
| RF-SEC-10 | RBAC: roles, features, role_feature, middleware 403 | HU-API-003 | Security | RN-SEC-10 | NFR-01, NFR-04 | ADR-001 | Unitarias: middleware authz. Integración: 403 sin feature, 200 con feature |
| RF-PAR-01 | CRUD catálogos base (event_category, severity, media_type, sound_pattern, event_type) | HU-API-004 | Parameterization | RN-PAR-01 | — | — | CRUD por API: 201/200/204/404. Seed SRS verificado |
| RF-PAR-02 | Gestión sound_pattern (Hz, duración, repeticiones, patrón) | HU-API-004 | Parameterization | RN-PAR-02 | — | — | Unitarias: validador sound_pattern. Integración: CRUD completo |
| RF-PAR-03 | Gestión event_type con umbrales configurables | HU-API-004 | Parameterization | RN-PAR-03 | — | — | Unitarias: validador umbrales JSONB. Integración: CRUD + seed Apéndice 2 |
| RF-PAR-04 | Umbrales por defecto + override por device_config (JSONB) | HU-API-005, HU-DEVICE-004 | Parameterization | RN-PAR-04 | — | — | Integración: merge defaults+override, device pulla config |
| RF-PAR-05 | Versionado catálogos (histórico cambios, auditoría) | HU-API-004 | Parameterization | RN-PAR-05 | — | — | Unitarias: created_by/updated_by/timestamp en cada cambio |
| RF-DEV-01 | Alta device: serial, firmware, api_key_hash, estados | HU-API-006 | Device Management | RN-DEV-03 | NFR-01 | — | Integración: POST /devices (admin), unique serial/api_key |
| RF-DEV-02 | Asociación/desasociación device ↔ user | HU-API-006 | Device Management | RN-DEV-01, RN-DEV-02 | NFR-01 | — | Integración: assign/unassign, state machine 6 estados |
| RF-DEV-03 | Config remota device_config (JSONB: umbrales, sound, volumen, sync) | HU-API-005, HU-DEVICE-004 | Device Management | RN-DEV-04 | NFR-03 | — | Integración: PATCH/GET config, merge defaults, historial |
| RF-DEV-04 | Heartbeat device: last_seen, firmware, conectividad | HU-API-006, HU-DEVICE-002 | Device Management | RN-DEV-05 | NFR-03 | — | Integración: PUT heartbeat, transición Activo↔Offline 5min |
| RF-DEV-05 | State machine device (Registrado→Asignado→Activo↔Offline→Suspendido→Retirado) | HU-API-006, HU-DEVICE-002 | Device Management | RN-DEV-06 | NFR-03 | — | Integración: transiciones automáticas + admin manual |
| RF-DEV-06 | Consulta devices por user (filtros estado, fecha) | HU-API-006, HU-PORTAL-002 | Device Management | RN-DEV-07 | — | — | Integración: GET /devices con filtros + paginación |
| RF-TEL-01 | Ingesta eventos idempotente (device+API key, event_id único) | HU-API-007, HU-DEVICE-003 | Telemetry Service | RN-TEL-01 | NFR-03, NFR-07 | — | Unitarias: validador idempotencia. Integración: duplicado → 409, ACK limpia buffer |
| RF-TEL-02 | Ingesta evidencia multimedia → MinIO bucket somnguard-evidence | HU-API-007 | Telemetry Service | RN-TEL-02 | NFR-02, NFR-06 | ADR-003 | Integración: upload MinIO, retorna evidence_id, checksum SHA256 |
| RF-TEL-03 | Registro alert_log (código AS-XX, ts, event_id, severidad) | HU-API-007 | Telemetry Service | RN-TEL-03 | NFR-04 | — | Integración: alert_log creado tras ingesta evento crítico |
| RF-TEL-04 | Sync offline-first: buffer SQLite local, reintentos backoff, deduplicación | HU-DEVICE-003, HU-API-007 | Telemetry Service + Edge | RN-TEL-04 | NFR-03, NFR-07 | — | Integración: offline 1h → online sync → ACK → limpieza buffer |
| RF-TEL-05 | Pull device_config desde device (GET /devices/{id}/config) | HU-API-005, HU-DEVICE-002, HU-DEVICE-003 | Telemetry Service | RN-TEL-05 | NFR-03 | — | Integración: device pulla config tras sync, aplica umbrales |
| RF-TEL-06 | Consulta eventos con filtros (device, tipo, severidad, fechas, paginación) | HU-API-008, HU-PORTAL-002 | Telemetry Service | RN-TEL-06 | NFR-03 | — | Integración: GET /telemetry/events con joins, índices device_id+occurred_at |
| RF-TEL-07 | Limpieza buffer local tras ACK sync (retención 7d fallidos) | HU-DEVICE-003 | Telemetry Service | RN-TEL-07 | NFR-02, NFR-06 | — | Integración: ACK 201 → borra confirmados, retención 7d fallidos |
| RF-MON-01 | Notificación push/email/webhook evento crítico (EV-SOM-05, EV-DIS-02, EV-DIS-04, EV-CIN-01/02) | HU-API-009, HU-APP-002 | Monitoring | RN-MON-01 | NFR-04 | — | Integración: trigger auto severity=crítica, canales push/email/in-app |
| RF-MON-02 | Plantillas notificación por event_type + severity | HU-API-009 | Monitoring | RN-MON-02 | — | — | Unitarias: template renderer. Integración: plantillas por tipo/severidad |
| RF-MON-03 | Tracking delivery: sent→delivered→read, reintentos backoff | HU-API-009, HU-APP-002 | Monitoring | RN-MON-03 | NFR-04 | — | Integración: estados notificación, reintentos max 3 exponenciales |
| RF-MON-04 | Preferencias notificación por user (canales, horarios, severidad mín) | HU-API-009, HU-APP-002, HU-PORTAL-001 | Monitoring | RN-MON-04 | — | — | Unitarias: filtro preferencias. Integración: respeta silencio/severidad |
| RF-ANA-01 | Timeline cronológico eventos (filtros fecha, tipo, severidad, device) | HU-API-010, HU-PORTAL-003 | Analytics | RN-ANA-01 | NFR-03 | — | Integración: GET /analytics/timeline <500ms p95 10k eventos |
| RF-ANA-02 | Métricas agregadas: freq por tipo, severidad media, tendencia temporal | HU-API-010, HU-PORTAL-003 | Analytics | RN-ANA-02 | NFR-03 | — | Integración: GET /analytics/metrics, vistas materializadas |
| RF-ANA-03 | Resumen descriptivo IA (patrones, tendencia, conclusiones) | HU-API-011, HU-PORTAL-004, HU-APP-003 | Analytics | RN-ANA-03 | NFR-05 | ADR-003 | Integración: POST /analytics/summary → LLM prompt → texto |
| RF-ANA-04 | Reporte consolidado PDF/HTML (timeline + métricas + IA + evidencia) | HU-API-011, HU-PORTAL-004, HU-APP-003 | Analytics | RN-ANA-04 | NFR-02, NFR-05 | ADR-003 | Integración: POST /analytics/report → PDF/HTML, cache 1h, MinIO |
| RF-ANA-05 | Video tiempo real WebRTC a demanda (Post-MVP) | HU-API-012, HU-DEVICE-005, HU-PORTAL-004, HU-APP-004 | Analytics + Edge | RN-ANA-05 | NFR-03, NFR-04 | — | **Post-MVP**: Integración WebRTC signaling, SFU, bitrate adaptativo |
| RF-EDGE-01 | Init hardware: verifica cámara, carga modelo, emite AS-08/AS-09 | HU-DEVICE-002 | Device Edge | RN-EDGE-01 | NFR-01, NFR-03 | — | Hardware: arranque <60s, cámara OK/ERROR |
| RF-EDGE-02 | Verificación campo visual cámara (obstrucción/mala posición) | HU-DEVICE-002 | Device Edge | RN-EDGE-02 | NFR-03 | — | Hardware: detecta obstrucción → AS-09, pausa detección |
| RF-EDGE-03 | Captura continua video rostro, landmarks → habilita análisis | HU-DEVICE-002 | Device Edge | RN-EDGE-03 | NFR-03, NFR-04 | — | Unitarias: frame reader FPS. Integración: landmarks detectados |
| RF-EDGE-04 | Detección somnolencia (PERCLOS, parpadeo, cierre ojos, bostezo, cabeceo) → AS-01..04 | HU-DEVICE-001 | Device Edge | RN-EDGE-04 | NFR-04 | — | Unitarias: detector con dataset etiquetado. Integración: <2s/frame |
| RF-EDGE-05 | Detección distracciones: teléfono (AS-05), mirada fuera (AS-06), movimientos (AS-05) | HU-DEVICE-001 | Device Edge | RN-EDGE-05 | NFR-04 | — | Unitarias: clasificador distracciones. Integración: precisión >90% |
| RF-EDGE-06 | Detección cinturón: presencia + posición → AS-07 intermitente si no | HU-DEVICE-001 | Device Edge | RN-EDGE-06 | NFR-04 | — | Unitarias: detector cinturón. Integración: AS-07 si no detectado |
| RF-EDGE-07 | Alerta sonora diferenciada por event_type + escalamiento persistencia | HU-DEVICE-004 | Device Edge | RN-EDGE-07 | NFR-04 | — | Integración: sound_pattern por AS-XX, escalamiento >10s, cola alertas |
| RF-EDGE-08 | Registro local eventos + alert_log + evidencia en buffer SQLite offline | HU-DEVICE-003 | Device Edge | RN-EDGE-08 | NFR-02, NFR-06 | — | Integración: buffer SQLite, pending_events, evidence_path, retries |
| RF-EDGE-09 | Gestión estado monitoreo: pausa sin rostro >30s (Espera), reanuda al detectar | HU-DEVICE-002 | Device Edge | RN-EDGE-09 | NFR-03 | — | Integración: Activo↔Espera por detección rostro |
| RF-EDGE-10 | Detección conectividad (ping/HEAD), sync auto online, reintentos exponenciales | HU-DEVICE-003 | Device Edge | RN-EDGE-10 | NFR-03, NFR-07 | — | Integración: ping 30s → online sync → backoff 1m/2m/4m/1h |
| RF-EDGE-11 | Aplicación device_config recibida (umbrales, sound_pattern, volumen, sync) | HU-DEVICE-002, HU-DEVICE-004 | Device Edge | RN-EDGE-11 | NFR-03 | — | Integración: config pull tras sync, merge defaults, aplica en caliente |
| RF-EDGE-12 | Liberación almacenamiento: retención 7d + auto-limpieza tras ACK | HU-DEVICE-003 | Device Edge | RN-EDGE-12 | NFR-02, NFR-06 | — | Integración: storage <90% → AS-09, retención evidencia 7d |
| RF-EDGE-13 | Video streaming WebRTC a demanda (Post-MVP) | HU-DEVICE-005, HU-API-012 | Device Edge | RN-EDGE-13 | NFR-03, NFR-04 | — | **Post-MVP**: WebRTC H.264, bitrate adaptativo, auto-stop 30s |

---

## Matriz Épicas → HUs → Sprints (MVP)

| Épica | HUs (MVP) | Sprints | SP Total | Prioridad |
|-------|-----------|---------|----------|-----------|
| Setup y Foundation | HU-API-SETUP, HU-DB-001a, HU-DB-001b | 0 | 21 | Must |
| Seguridad y cuentas | HU-API-001, 002, 003, HU-PORTAL-001, HU-APP-001 | 1 | 34 | Must |
| Gestión de dispositivos | HU-API-005, 006 | 1-2 | 13 | Must |
| Parametrización | HU-API-004 | 1 | 8 | Must |
| Telemetría y sincronización | HU-API-007, 008, HU-DEVICE-001, 002, 003, 004 | 1-3 | 68 | Must |
| Monitoreo y notificaciones | HU-API-009, HU-APP-002 | 3 | 16 | Must |
| Analítica y reportes | HU-API-010, 011, HU-DB-002, HU-PORTAL-003, 004, HU-APP-003 | 3-5 | 45 | Must/Should |
| Device Edge | HU-DEVICE-001, 002, 003, 004 | 1-3 | 47 | Must |

---

## Referencias Cruzadas

| Origen | Destino | Referencia |
|--------|---------|------------|
| HU (nueva) | RF / RN / NFR / ADR / TC | Plantilla [`./_template-hu.md`](./_template-hu.md) |
| Épica | Funcionalidades / HUs | [`../03-product-definition/product-backlog.md`](../03-product-definition/product-backlog.md) |
| NFR | Componentes / Pruebas | [`./non-functional.md`](./non-functional.md) |
| RF | RN / F-SRS / Épica / HU | [`./functional.md`](./functional.md) |
| RN | Entidades / Módulos | [`../02-domain/entities-and-rules.md`](../02-domain/entities-and-rules.md) |
| ADR | Decisiones arquitectura | [`../05-architecture/decisions/records/`](../05-architecture/decisions/records/) |
| Modelo datos | Entidades / Atributos / Relaciones | [`../06-data-architecture/02-modules-entities.md`](../06-data-architecture/02-modules-entities.md) |
| API Design | Endpoints / Contratos | [`../07-api-design/api-design.md`](../07-api-design/api-design.md) |
| Estrategia pruebas | Unitarias / Integración / E2E / Hardware | [`../11-quality-assurance/test-strategy.md`](../11-quality-assurance/test-strategy.md) |
| Convenciones ágiles | Branching / Commits / Issues | [`../00-documentation-governance/agile-conventions.md`](../00-documentation-governance/agile-conventions.md) |

---

## Próximos Pasos

1. **Validar matriz** con PO + Arquitecto + Tech Leads (revisión 1h).
2. **Crear issues en GitHub Projects** desde `user-stories.md` (IDs, títulos, ACs, SP, labels `repo:api|db|device|portal|app`, `moscow:must|should|could`, `sprint:N`).
3. **Mantener sincronizada** esta matriz al crear/modificar HUs, RFs, ADRs o NFRs.
4. **Definir `cross-cutting.md`** (reglas transversales: auth, audit, obs, idempotencia, tz, errores) y vincular aquí.