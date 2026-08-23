<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Requisitos Funcionales — RF-*

**Estado:** Borrador
**Fecha:** 2026-08-22

</div>

</div>

> **Fuentes:** [SRS](../04-requeriments/01-srs/) (RF-1..RF-10, RNF-1..RNF-9, Apéndices),
> [entities-and-rules.md](../02-domain/entities-and-rules.md) (RN-*),
> [software-analysis.md](./software-analysis.md) (F-01..F-10),
> [product-backlog.md](../03-product-definition/product-backlog.md) (épicas MoSCoW),
> [module-catalog.md](../09-modules/module-catalog.md) (7 módulos).

---

## Convención de Identificadores

| Prefijo | Módulo | Descripción | Ejemplo |
|---------|--------|-------------|---------|
| `RF-SEC-##` | Security | Autenticación, autorización, cuentas, auditoría | `RF-SEC-01` Registro de cuenta |
| `RF-PAR-##` | Parameterization | Catálogos: event_type, severity, sound_pattern, media_type | `RF-PAR-01` CRUD catálogos |
| `RF-DEV-##` | Device Management | Alta, asignación, configuración, estado de dispositivos | `RF-DEV-01` Asociación device-usuario |
| `RF-TEL-##` | Telemetry Service | Ingesta eventos, evidencia, alert_log, sincronización offline | `RF-TEL-01` Registro evento idempotente |
| `RF-MON-##` | Monitoring | Notificaciones push, alertas críticas, delivery tracking | `RF-MON-01` Notificación evento crítico |
| `RF-ANA-##` | Analytics | Línea de tiempo, métricas, resumen IA, reportes | `RF-ANA-01` Timeline eventos |
| `RF-EDGE-##` | Device Edge | Detección visión, alertas locales, buffer offline, sync | `RF-EDGE-01` Detección somnolencia |

> **Regla:** El prefijo usa la abreviatura del módulo en mayúsculas (3-4 letras). Numeración secuencial por módulo.

---

## Requisitos Funcionales por Módulo

### Security (RF-SEC-*)

| ID | Descripción | RN Origen | Funcionalidad SRS | Prioridad MoSCoW | Épica |
|----|-------------|-----------|-------------------|------------------|-------|
| RF-SEC-01 | Registro de cuenta con validación de correo y teléfono únicos, hash de contraseña | RN-SEC-01, RN-SEC-02 | RF-9.1 | Must | Seguridad y cuentas |
| RF-SEC-02 | Autenticación JWT RS256 + refresh token; login con correo/contraseña | RN-SEC-03, RN-SEC-04 | RF-9.2 | Must | Seguridad y cuentas |
| RF-SEC-03 | Cierre de sesión con invalidación de token/sesión | RN-SEC-05 | RF-9.3 | Must | Seguridad y cuentas |
| RF-SEC-04 | Recuperación de contraseña via token temporal enviado por correo | RN-SEC-06 | RF-9.4 | Must | Seguridad y cuentas |
| RF-SEC-05 | Actualización de datos personales (validación unicidad correo/teléfono) | RN-SEC-07 | RF-9.5 | Should | Seguridad y cuentas |
| RF-DEV-06 | Asociación dispositivo-usuario (1 device ↔ 1 user) | RN-DEV-01 | RF-9.6 | Must | Gestión de dispositivos |
| RF-DEV-07 | Desasociación dispositivo (libera para otro usuario) | RN-DEV-02 | RF-9.7 | Should | Gestión de dispositivos |
| RF-SEC-08 | Eliminación de cuenta con soft-delete (ventana 30 días recuperación) | RN-SEC-08 | RF-9.8 | Could | Seguridad y cuentas |
| RF-SEC-09 | Auditoría de login (audit_login): IP, user-agent, éxito/fallo, timestamp | RN-SEC-09 | RNF-4.3 | Must | Seguridad y cuentas |
| RF-SEC-10 | Control de acceso por roles (RBAC): admin, user; features por role_feature | RN-SEC-10 | RNF-4.4 | Must | Seguridad y cuentas |

### Parameterization (RF-PAR-*)

| ID | Descripción | RN Origen | Funcionalidad SRS | Prioridad MoSCoW | Épica |
|----|-------------|-----------|-------------------|------------------|-------|
| RF-PAR-01 | CRUD catálogos base: event_category, severity, media_type, sound_pattern, event_type | RN-PAR-01 | Apéndice 1, 2 | Must | Parametrización |
| RF-PAR-02 | Gestión de sound_pattern: frecuencia Hz, duración, repeticiones, patrón (continuo/intermitente) | RN-PAR-02 | Apéndice 1 (AS-01..AS-09) | Must | Parametrización |
| RF-PAR-03 | Gestión de event_type con umbrales configurables (parpadeo, cierre ojos, bostezo, cabeceo, teléfono, mirada, cinturón) | RN-PAR-03 | Apéndice 2 (EV-SOM-*, EV-DIS-*, EV-CIN-*) | Must | Parametrización |
| RF-PAR-04 | Configuración de umbrales por defecto y overridables por device_config (JSONB) | RN-PAR-04 | RF-1.2, Apéndice 2 nota | Should | Parametrización |
| RF-PAR-05 | Versionado de catálogos (histórico de cambios, auditoría) | RN-PAR-05 | — | Could | Parametrización |

### Device Management (RF-DEV-*)

| ID | Descripción | RN Origen | Funcionalidad SRS | Prioridad MoSCoW | Épica |
|----|-------------|-----------|-------------------|------------------|-------|
| RF-DEV-01 | Alta de dispositivo: serial_number, firmware_version, api_key_hash, estado (Registrado/Asignado/Activo/Offline/Suspendido/Retirado) | RN-DEV-03 | RF-1.1, ES-device | Must | Gestión de dispositivos |
| RF-DEV-02 | Asociación/desasociación device ↔ user (ver RF-DEV-06, RF-DEV-07) | RN-DEV-01, RN-DEV-02 | RF-9.6, RF-9.7 | Must | Gestión de dispositivos |
| RF-DEV-03 | Configuración remota device_config (JSONB): umbrales, sound_pattern, volumen, intervalo sync | RN-DEV-04 | RF-1.2, RF-8.3 | Must | Gestión de dispositivos |
| RF-DEV-04 | Heartbeat dispositivo: last_seen, versión firmware, estado conectividad | RN-DEV-05 | RF-8.2 | Must | Gestión de dispositivos |
| RF-DEV-05 | Gestión de estados del dispositivo (state machine: Registrado → Asignado → Activo ↔ Offline → Suspendido → Retirado) | RN-DEV-06 | Apéndice 2 (EV-SYS-*), ES-device | Must | Gestión de dispositivos |
| RF-DEV-06 | Consulta de dispositivos por usuario (filtros: estado, fecha asignación) | RN-DEV-07 | RF-10.1 | Should | Gestión de dispositivos |

### Telemetry Service (RF-TEL-*)

| ID | Descripción | RN Origen | Funcionalidad SRS | Prioridad MoSCoW | Épica |
|----|-------------|-----------|-------------------|------------------|-------|
| RF-TEL-01 | Ingesta de eventos (POST /telemetry/events): validación device + API key, idempotencia (event_id único), persistencia | RN-TEL-01 | RF-7.1, RF-7.2, RF-8.3 | Must | Telemetría y sincronización |
| RF-TEL-02 | Ingesta de evidencia multimedia (imagen/video) → MinIO bucket `somnguard-evidence`; vinculación a event | RN-TEL-02 | RF-7.4, RF-8.3 | Must | Telemetría y sincronización |
| RF-TEL-03 | Registro de alert_log: código AS-XX, timestamp, event_id asociado, severidad | RN-TEL-03 | RF-6.1, RF-6.2, RF-7.3, Apéndice 1 | Must | Telemetría y sincronización |
| RF-TEL-04 | Sincronización offline-first: buffer local en device (SQLite), reintentos con backoff, deduplicación por event_id | RN-TEL-04 | RF-8.1, RF-8.2, RF-8.3 | Must | Telemetría y sincronización |
| RF-TEL-05 | Pull de device_config desde device (GET /devices/{id}/config) tras sync exitosa | RN-TEL-05 | RF-1.2, RF-8.3 | Must | Telemetría y sincronización |
| RF-TEL-06 | Consulta de eventos con filtros: device_id, event_type, severity, rango fechas, paginación | RN-TEL-06 | RF-10.1 | Must | Telemetría y sincronización |
| RF-TEL-07 | Limpieza automática de buffer local tras confirmación de sincronización (ACK) | RN-TEL-07 | RNF-2.4 | Must | Telemetría y sincronización |

### Monitoring (RF-MON-*)

| ID | Descripción | RN Origen | Funcionalidad SRS | Prioridad MoSCoW | Épica |
|----|-------------|-----------|-------------------|------------------|-------|
| RF-MON-01 | Notificación push/email/webhook al usuario ante evento crítico (EV-SOM-05, EV-DIS-02, EV-DIS-04, EV-CIN-01/02) | RN-MON-01 | RF-10.5 | Must | Monitoreo y notificaciones |
| RF-MON-02 | Plantillas de notificación por event_type + severity (canal: push, email, in-app) | RN-MON-02 | RF-10.5 | Should | Monitoreo y notificaciones |
| RF-MON-03 | Tracking de delivery: sent → delivered → read; reintentos con backoff | RN-MON-03 | — | Should | Monitoreo y notificaciones |
| RF-MON-04 | Preferencias de notificación por usuario (canales, horarios, severidad mínima) | RN-MON-04 | — | Could | Monitoreo y notificaciones |

### Analytics (RF-ANA-*)

| ID | Descripción | RN Origen | Funcionalidad SRS | Prioridad MoSCoW | Épica |
|----|-------------|-----------|-------------------|------------------|-------|
| RF-ANA-01 | Línea de tiempo cronológica de eventos (filtros: fecha, tipo, severidad, device) | RN-ANA-01 | RF-10.1 | Must | Analítica y reportes |
| RF-ANA-02 | Métricas agregadas: frecuencia por event_type, severidad media, tendencia temporal (día/semana/mes) | RN-ANA-02 | RF-10.2 | Must | Analítica y reportes |
| RF-ANA-03 | Resumen descriptivo generado por IA (análisis de patrones, tendencia, conclusiones interpretativas) | RN-ANA-03 | RF-10.3 | Should | Analítica y reportes |
| RF-ANA-04 | Reporte consolidado (PDF/HTML): timeline + métricas + resumen IA + evidencia; descarga y visualización | RN-ANA-04 | RF-10.4 | Should | Analítica y reportes |
| RF-ANA-05 | Video en tiempo real (WebRTC/WebSocket) a demanda desde app/portal (RF-10.6) | RN-ANA-05 | RF-10.6 | Could | Analítica y reportes |

### Device Edge (RF-EDGE-*) — *Ejecutados en el dispositivo (Python/Raspberry Pi)*

| ID | Descripción | RN Origen | Funcionalidad SRS | Prioridad MoSCoW | Épica |
|----|-------------|-----------|-------------------|------------------|-------|
| RF-EDGE-01 | Inicialización hardware: verificación cámara, carga modelo visión, emisión AS-08 (ok) / AS-09 (error) | RN-EDGE-01 | RF-1.1 | Must | Telemetría y sincronización |
| RF-EDGE-02 | Verificación campo visual cámara: obstrucción/mala posición → AS-09, pausa detección | RN-EDGE-02 | RF-2.1 | Must | Telemetría y sincronización |
| RF-EDGE-03 | Captura continua video rostro; detección rostro (landmarks) → habilita análisis | RN-EDGE-03 | RF-2.2 | Must | Telemetría y sincronización |
| RF-EDGE-04 | Detección somnolencia/fatiga: PERCLOS, parpadeo anómalo, cierre prolongado, bostezo, cabeceo → nivel (leve/moderada/severa/crítica) + AS-01..AS-04 | RN-EDGE-04 | RF-3.1, Apéndice 2 (EV-SOM-01..05) | Must | Telemetría y sincronización |
| RF-EDGE-05 | Detección distracciones: teléfono (AS-05), mirada fuera vía (AS-06), movimientos anómalos (AS-05) | RN-EDGE-05 | RF-4.1, RF-4.2, RF-4.3, Apéndice 2 (EV-DIS-01..05) | Must | Telemetría y sincronización |
| RF-EDGE-06 | Detección cinturón: presencia + posición correcta → AS-07 (intermitente) si no | RN-EDGE-06 | RF-5.1, RF-5.2, Apéndice 2 (EV-CIN-01,02) | Must | Telemetría y sincronización |
| RF-EDGE-07 | Generación alerta sonora diferenciada por event_type + escalamiento por persistencia (AS-01..AS-09) | RN-EDGE-07 | RF-6.1, RF-6.2, RF-6.3, Apéndice 1 | Must | Telemetría y sincronización |
| RF-EDGE-08 | Registro local de eventos + alert_log + evidencia (imagen frame) en buffer SQLite offline | RN-EDGE-08 | RF-7.1..7.4, RF-8.1 | Must | Telemetría y sincronización |
| RF-EDGE-09 | Gestión de estado monitoreo: pausa si no hay rostro >30s (modo espera), reanuda al detectar rostro | RN-EDGE-09 | RF-1.5, EV-SYS-03 | Must | Telemetría y sincronización |
| RF-EDGE-10 | Detección conectividad (ping/HTTP HEAD a API); sync automático cuando online; reintentos exponenciales | RN-EDGE-10 | RF-8.2, RF-8.3 | Must | Telemetría y sincronización |
| RF-EDGE-11 | Aplicación device_config recibida (umbrales, sound_pattern, volumen, intervalo sync) | RN-EDGE-11 | RF-1.2, RF-8.3 | Must | Gestión de dispositivos |
| RF-EDGE-12 | Liberación almacenamiento local: política retención (ej. 7 días) + auto-limpieza tras ACK sync | RN-EDGE-12 | RNF-2.4, RF-8.1 | Must | Telemetría y sincronización |
| RF-EDGE-13 | Video streaming tiempo real (WebRTC) a demanda desde app/portal | RN-EDGE-13 | RF-10.6 | Could | Analítica y reportes |

---

## Trazabilidad Consolidada (RF → RN → F → Épica → HU Futura)

| RF | RN Origen | F- SRS | Épica | Módulo | HU Candidata |
|----|-----------|--------|-------|--------|--------------|
| RF-SEC-01 | RN-SEC-01,02 | RF-9.1 | Seguridad y cuentas | Security | HU-API-001 |
| RF-SEC-02 | RN-SEC-03,04 | RF-9.2 | Seguridad y cuentas | Security | HU-API-002 |
| RF-SEC-03 | RN-SEC-05 | RF-9.3 | Seguridad y cuentas | Security | HU-API-003 |
| RF-SEC-04 | RN-SEC-06 | RF-9.4 | Seguridad y cuentas | Security | HU-API-004 |
| RF-SEC-05 | RN-SEC-07 | RF-9.5 | Seguridad y cuentas | Security | HU-API-005 |
| RF-DEV-06 | RN-DEV-01 | RF-9.6 | Gestión de dispositivos | Device Management | HU-API-006 |
| RF-DEV-07 | RN-DEV-02 | RF-9.7 | Gestión de dispositivos | Device Management | HU-API-007 |
| RF-SEC-08 | RN-SEC-08 | RF-9.8 | Seguridad y cuentas | Security | HU-API-008 |
| RF-SEC-09 | RN-SEC-09 | RNF-4.3 | Seguridad y cuentas | Security | HU-API-009 |
| RF-SEC-10 | RN-SEC-10 | RNF-4.4 | Seguridad y cuentas | Security | HU-API-010 |
| RF-PAR-01 | RN-PAR-01 | Apéndice 1,2 | Parametrización | Parameterization | HU-API-011 |
| RF-PAR-02 | RN-PAR-02 | Apéndice 1 | Parametrización | Parameterization | HU-API-012 |
| RF-PAR-03 | RN-PAR-03 | Apéndice 2 | Parametrización | Parameterization | HU-API-013 |
| RF-PAR-04 | RN-PAR-04 | RF-1.2 | Parametrización | Parameterization | HU-API-014 |
| RF-PAR-05 | RN-PAR-05 | — | Parametrización | Parameterization | HU-API-015 |
| RF-DEV-01 | RN-DEV-03 | RF-1.1 | Gestión de dispositivos | Device Management | HU-API-016 |
| RF-DEV-02 | RN-DEV-01,02 | RF-9.6,9.7 | Gestión de dispositivos | Device Management | HU-API-017 |
| RF-DEV-03 | RN-DEV-04 | RF-1.2,8.3 | Gestión de dispositivos | Device Management | HU-API-018 |
| RF-DEV-04 | RN-DEV-05 | RF-8.2 | Gestión de dispositivos | Device Management | HU-API-019 |
| RF-DEV-05 | RN-DEV-06 | EV-SYS-* | Gestión de dispositivos | Device Management | HU-API-020 |
| RF-DEV-06 | RN-DEV-07 | RF-10.1 | Gestión de dispositivos | Device Management | HU-PORTAL-001 |
| RF-TEL-01 | RN-TEL-01 | RF-7.1,7.2,8.3 | Telemetría y sincronización | Telemetry Service | HU-API-021 |
| RF-TEL-02 | RN-TEL-02 | RF-7.4,8.3 | Telemetría y sincronización | Telemetry Service | HU-API-022 |
| RF-TEL-03 | RN-TEL-03 | RF-6.1,6.2,7.3 | Telemetría y sincronización | Telemetry Service | HU-API-023 |
| RF-TEL-04 | RN-TEL-04 | RF-8.1,8.2,8.3 | Telemetría y sincronización | Telemetry Service | HU-DEVICE-001 |
| RF-TEL-05 | RN-TEL-05 | RF-1.2,8.3 | Telemetría y sincronización | Telemetry Service | HU-DEVICE-002 |
| RF-TEL-06 | RN-TEL-06 | RF-10.1 | Telemetría y sincronización | Telemetry Service | HU-PORTAL-002 |
| RF-TEL-07 | RN-TEL-07 | RNF-2.4 | Telemetría y sincronización | Telemetry Service | HU-DEVICE-003 |
| RF-MON-01 | RN-MON-01 | RF-10.5 | Monitoreo y notificaciones | Monitoring | HU-API-024 |
| RF-MON-02 | RN-MON-02 | RF-10.5 | Monitoreo y notificaciones | Monitoring | HU-API-025 |
| RF-MON-03 | RN-MON-03 | — | Monitoreo y notificaciones | Monitoring | HU-API-026 |
| RF-MON-04 | RN-MON-04 | — | Monitoreo y notificaciones | Monitoring | HU-APP-001 |
| RF-ANA-01 | RN-ANA-01 | RF-10.1 | Analítica y reportes | Analytics | HU-PORTAL-003 |
| RF-ANA-02 | RN-ANA-02 | RF-10.2 | Analítica y reportes | Analytics | HU-PORTAL-004 |
| RF-ANA-03 | RN-ANA-03 | RF-10.3 | Analítica y reportes | Analytics | HU-PORTAL-005 |
| RF-ANA-04 | RN-ANA-04 | RF-10.4 | Analítica y reportes | Analytics | HU-PORTAL-006 |
| RF-ANA-05 | RN-ANA-05 | RF-10.6 | Analítica y reportes | Analytics | HU-APP-002 |
| RF-EDGE-01 | RN-EDGE-01 | RF-1.1 | Telemetría y sincronización | Device Edge | HU-DEVICE-004 |
| RF-EDGE-02 | RN-EDGE-02 | RF-2.1 | Telemetría y sincronización | Device Edge | HU-DEVICE-005 |
| RF-EDGE-03 | RN-EDGE-03 | RF-2.2 | Telemetría y sincronización | Device Edge | HU-DEVICE-006 |
| RF-EDGE-04 | RN-EDGE-04 | RF-3.1 | Telemetría y sincronización | Device Edge | HU-DEVICE-007 |
| RF-EDGE-05 | RN-EDGE-05 | RF-4.1,4.2,4.3 | Telemetría y sincronización | Device Edge | HU-DEVICE-008 |
| RF-EDGE-06 | RN-EDGE-06 | RF-5.1,5.2 | Telemetría y sincronización | Device Edge | HU-DEVICE-009 |
| RF-EDGE-07 | RN-EDGE-07 | RF-6.1,6.2,6.3 | Telemetría y sincronización | Device Edge | HU-DEVICE-010 |
| RF-EDGE-08 | RN-EDGE-08 | RF-7.1..7.4,8.1 | Telemetría y sincronización | Device Edge | HU-DEVICE-011 |
| RF-EDGE-09 | RN-EDGE-09 | RF-1.5 | Telemetría y sincronización | Device Edge | HU-DEVICE-012 |
| RF-EDGE-10 | RN-EDGE-10 | RF-8.2,8.3 | Telemetría y sincronización | Device Edge | HU-DEVICE-013 |
| RF-EDGE-11 | RN-EDGE-11 | RF-1.2,8.3 | Gestión de dispositivos | Device Edge | HU-DEVICE-014 |
| RF-EDGE-12 | RN-EDGE-12 | RNF-2.4,8.1 | Telemetría y sincronización | Device Edge | HU-DEVICE-015 |
| RF-EDGE-13 | RN-EDGE-13 | RF-10.6 | Analítica y reportes | Device Edge | HU-DEVICE-016 |

---

## Referencias Cruzadas

- **Reglas de negocio (RN-*):** [entities-and-rules.md](../02-domain/entities-and-rules.md)
- **Funcionalidades de alto nivel (F-*):** [software-analysis.md](./software-analysis.md) §7
- **Épicas y priorización:** [product-backlog.md](../03-product-definition/product-backlog.md)
- **Catálogo de módulos:** [module-catalog.md](../09-modules/module-catalog.md)
- **Plantilla HU:** [_template-hu.md](./_template-hu.md)
- **Matriz de trazabilidad:** [traceability-matrix.md](./traceability-matrix.md) (actualizar tras crear HUs)
- **Requisitos no funcionales:** [non-functional.md](./non-functional.md) (RNF-* del SRS §3.3)

---

## Próximos Pasos

1. **Revisar y validar** esta lista con el equipo (PO + Arquitecto + Tech Leads).
2. **Crear `user-stories.md`** expandiendo cada HU candidata con: historia, criterios de aceptación (AC), story points, MoSCoW, sprint objetivo, dependencias.
3. **Actualizar `traceability-matrix.md`** con la trazabilidad completa RF ↔ HU ↔ Módulo ↔ Prueba ↔ ADR.
4. **Definir ADRs** para decisiones transversales (ver [cross-cutting.md](../05-architecture/cross-cutting.md) pendiente).