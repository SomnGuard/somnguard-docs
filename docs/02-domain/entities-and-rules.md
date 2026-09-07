<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Entidades y reglas de negocio

**Estado:** En progreso
**Fecha:** 2026-08-19

</div>

</div>

Reglas de negocio del sistema. Fuente de verdad de las reglas `RN-*`; consolidan y equivalen a las reglas `RB-*` del informe de diseño (no se renumeran las ya publicadas; los nuevos identificadores usan `RN-`).

## Reglas de negocio

| ID | Módulo | Regla |
|----|--------|-------|
| RN-01 | security | Una cuenta es única por correo electrónico; las contraseñas se almacenan solo como hash |
| RN-02 | security | El acceso a funcionalidades se controla por roles y permisos (`role_feature`); un usuario hereda los permisos de sus roles |
| RN-03 | device-management | Un dispositivo solo puede pertenecer a la cuenta a la que fue asignado (`device_assignment`); el envío de eventos exige una API key válida |
| RN-04 | parameterization | Todo evento debe clasificarse con una categoría, un tipo y una severidad vigentes (catálogos de parameterization) |
| RN-05 | telemetry-service | Las alertas emitidas por el dispositivo se registran en `alert_log` asociadas al evento que las originó y al patrón de sonido reproducido |
| RN-06 | telemetry-service | La evidencia multimedia se conserva por un período de retención definido; los metadatos en `evidence` incluyen el tipo de medio y su referencia |
| RN-07 | monitoring | Las notificaciones de eventos críticos se generan automáticamente y se dirigen a la cuenta propietaria del dispositivo |
| RN-08 | telemetry-service | La sincronización offline no debe duplicar eventos: el dispositivo envía un identificador único (idempotencia) y la API descarta envíos repetidos |
| RN-09 | parameterization | El catálogo de sonidos (`sound_pattern`) es gestionado exclusivamente por el administrador técnico |
| RN-10 | device-management | La configuración básica del dispositivo (`device_config`) se descarga del backend y se aplica localmente antes de iniciar el monitoreo |

> Equivalencias: RN-01..RN-10 ↔ RB-01..RB-10 del [informe de diseño](../05-architecture/software-design-report.md).
>
> Canónicos por módulo (usar estos en RF/HU nuevos; RN-01..10 quedan como legado):
> | Legado | Canónico | Tema |
> |--------|----------|------|
> | RN-01 | RN-SEC-01, RN-SEC-02 | Cuenta única + hash |
> | RN-02 | RN-SEC-10 | RBAC roles/features |
> | RN-03 | RN-DEV-01, RN-DEV-02, RN-DEV-03 | Assign + API key válida |
> | RN-04 | RN-PAR-01 | Clasificación por catálogos |
> | RN-05 | RN-TEL-03 | alert_log asociado |
> | RN-06 | RN-TEL-02 | Evidencia + retención |
> | RN-07 | RN-MON-01 | Notificaciones críticas |
> | RN-08 | RN-TEL-01, RN-TEL-04 | Idempotencia event_id |
> | RN-09 | RN-PAR-02 | sound_pattern solo admin |
> | RN-10 | RN-DEV-04, RN-TEL-05 | Descarga config antes de monitorear |
>
> ## Definición de reglas canónicas por módulo
>
> | ID | Regla |
> |----|-------|
> | RN-SEC-01 | Una cuenta es única por correo electrónico |
> | RN-SEC-02 | Las contraseñas se almacenan solo como hash (bcrypt, nunca plano) |
> | RN-SEC-03 | La autenticación usa JWT RS256 de corta duración + refresh token |
> | RN-SEC-04 | El login exige correo/contraseña válidos y registra el intento |
> | RN-SEC-05 | El logout invalida el refresh token (blocklist) |
> | RN-SEC-06 | La recuperación de contraseña usa token temporal de 1h, un solo uso |
> | RN-SEC-07 | La unicidad de correo/teléfono se valida al actualizar datos |
> | RN-SEC-08 | La eliminación de cuenta es soft-delete con ventana de recuperación |
> | RN-SEC-09 | Todo intento de login se audita (`audit_login`: IP, user-agent, éxito/fallo) |
> | RN-SEC-10 | El acceso se controla por roles y features (`role_feature`) |
> | RN-DEV-01 | 1 usuario ↔ 1 device vigente: ni el device puede estar asignado a otro usuario, ni el usuario tener otro device asignado (`device_assignment` con UNIQUE parcial por `device_id` y por `user_id`) |
> | RN-DEV-02 | Desasociar libera el dispositivo (`unassign` → `REGISTERED`) |
> | RN-DEV-03 | El alta genera `api_key_hash`; la key en claro se muestra una sola vez |
> | RN-DEV-04 | La configuración remota (`device_config` JSONB) es descargable por el device |
> | RN-DEV-05 | El heartbeat periódico define `ACTIVE`/`OFFLINE` (timeout 5 min) |
> | RN-DEV-06 | Los cambios de estado siguen `status_transition` según rol |
> | RN-DEV-07 | La consulta de dispositivos filtra por estado y fecha de asignación |
> | RN-DEV-08 | Provisioning de un uso + self-register idempotente (key expuesta una vez) |
> | RN-DEV-09 | Claim público reutilizable por device: solo sirve en REGISTERED, unassign lo libera |
> | RN-PAR-01 | Los catálogos base se administran por CRUD restringido |
> | RN-PAR-02 | `sound_pattern` solo lo gestiona el administrador |
> | RN-PAR-03 | `event_type` define umbrales configurables por tipo de evento |
> | RN-PAR-04 | Los defaults de catálogo admiten override por `device_config` |
> | RN-PAR-05 | Los cambios de catálogo quedan versionados con auditoría |
> | RN-TEL-01 | La ingesta es idempotente por `event_id` (duplicados se reportan, no son error) |
> | RN-TEL-02 | Cada evento lleva como máximo una evidencia en MinIO (`evidence.event_id` UNIQUE) |
> | RN-TEL-03 | Toda alerta del device se registra en `alert_log` con su evento y sonido |
> | RN-TEL-04 | El buffer offline reintenta con backoff y retención de 7 días |
> | RN-TEL-05 | El device descarga `device_config` tras cada sync exitosa |
> | RN-TEL-06 | La consulta de eventos filtra por device, tipo, severidad y fechas |
> | RN-TEL-07 | El buffer local se limpia tras el ACK del servidor |
> | RN-MON-01 | Los eventos críticos notifican automáticamente al propietario del device |
> | RN-MON-02 | Las plantillas de notificación dependen de tipo + severidad |
> | RN-MON-03 | El delivery se traza (`sent → delivered → read`) con reintentos |
> | RN-MON-04 | El usuario configura canales y severidad mínima |
> | RN-ANA-01 | La línea de tiempo ordena eventos por fecha con filtros |
> | RN-ANA-02 | Las métricas agregan por tipo, severidad y período |
> | RN-ANA-03 | El resumen IA es descriptivo y de menor prioridad (final del MVP) |
> | RN-ANA-04 | El reporte consolida timeline + métricas + resumen + evidencia |
> | RN-ANA-05 | El video en tiempo real es post-MVP (a demanda, WebRTC) |
> | RN-EDGE-01 | El device verifica cámara y modelo al iniciar (AS-08 ok / AS-09 error) |
> | RN-EDGE-02 | Con cámara obstruida se pausa la detección (AS-09) |
> | RN-EDGE-03 | La captura continua verifica rostro (landmarks) para habilitar el análisis |
> | RN-EDGE-04 | La somnolencia se clasifica por nivel con alertas AS-01..AS-04 |
> | RN-EDGE-05 | Las distracciones generan alertas AS-05/AS-06 |
> | RN-EDGE-06 | Sin cinturón se alerta AS-07 intermitente |
> | RN-EDGE-07 | Cada tipo de evento tiene alerta sonora diferenciada con escalamiento |
> | RN-EDGE-08 | Todo evento se registra localmente con su evidencia en SQLite |
> | RN-EDGE-09 | El monitoreo se pausa sin rostro y reanuda al detectarlo |
> | RN-EDGE-10 | El device detecta conectividad y sincroniza solo con red |
> | RN-EDGE-11 | La config recibida se aplica en runtime sin reinicio |
> | RN-EDGE-12 | El almacenamiento local se auto-limpia por retención tras ACK |
> | RN-EDGE-13 | El streaming de video es post-MVP y solo a demanda |

## Modelo de dominio por módulo

| Módulo | Entidades (agregados y tablas) |
|--------|--------------------------------|
| security | user, role, module, feature, role_feature, user_role, password_reset_request, audit_login |
| parameterization | event_category, severity, media_type, sound_pattern, event_type |
| device-management | device, device_assignment, device_config |
| telemetry-service | event, evidence, alert_log |
| monitoring | notification |
| analytics | vistas/reportes derivados (sin entidades transaccionales) |

Detalle de atributos en [`../06-data-architecture/02-modules-entities.md`](../06-data-architecture/02-modules-entities.md).

## Ver también

- [Mapa de dominio](domain-map.md)
- [Eventos de dominio](domain-events.md)
- [Reglas de negocio del diseño](../05-architecture/software-design-report.md#9-reglas-de-negocio)
