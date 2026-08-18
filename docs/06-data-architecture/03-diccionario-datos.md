<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Diccionario de datos

**Estado:** En progreso
**Fecha:** 2026-08-16

</div>

</div>

Diccionario de datos del modelo vigente (v2), alineado con [02-modulos-entidades.md](./02-modulos-entidades.md) y [01-modelo-entidad-relacion.mmd](./01-modelo-entidad-relacion.mmd). La versión anterior del modelo quedó archivada en `../99-archive/deprecated/data-model-v1/`.

## Convenciones generales

### Identificador

Todas las entidades utilizan:

| Campo | Tipo | Descripción |
| ----- | ---- | ----------- |
| id | UUID | Identificador único. |

### Auditoría

Todas las entidades transaccionales incluyen:

| Campo | Tipo | Descripción |
| ----- | ---- | ----------- |
| created_at | TIMESTAMPTZ | Fecha de creación del registro. |
| updated_at | TIMESTAMPTZ | Fecha de última actualización. |
| deleted_at | TIMESTAMPTZ | Borrado lógico (NULL = vigente). |
| created_by | UUID | Usuario que creó el registro. |
| updated_by | UUID | Usuario que actualizó el registro. |
| deleted_by | UUID | Usuario que eliminó el registro. |
| is_active | BOOLEAN | Estado activo/inactivo del registro. |

### Observaciones generales

- Contraseñas y API keys se almacenan únicamente como **hash**.
- La configuración de dispositivos se guarda en `JSONB` para permitir evolución sin cambios de esquema.
- La evidencia multimedia no se persiste en la base de datos; se guarda una referencia (URL o identificador).

---

## Módulo security

### user

Representa a los usuarios del sistema.

| Campo | Tipo | Descripción |
| ----- | ---- | ----------- |
| id | UUID PK | Identificador único. |
| email | VARCHAR(255) | Correo electrónico de la cuenta. |
| password_hash | TEXT | Hash de la contraseña. |
| first_name | VARCHAR(100) | Nombre de pila. |
| last_name | VARCHAR(100) | Apellido. |
| phone | VARCHAR(30) | Teléfono de contacto. |

### role

Define los roles del sistema.

| Campo | Tipo | Descripción |
| ----- | ---- | ----------- |
| id | UUID PK | Identificador único. |
| name | VARCHAR(100) | Nombre del rol. |
| description | TEXT | Descripción del rol. |

### module

Agrupa funcionalidades del sistema.

| Campo | Tipo | Descripción |
| ----- | ---- | ----------- |
| id | UUID PK | Identificador único. |
| code | VARCHAR(50) | Código del módulo (ej: `security`). |
| name | VARCHAR(100) | Nombre del módulo. |
| description | TEXT | Descripción del módulo. |

### feature

Representa una funcionalidad protegida del sistema.

| Campo | Tipo | Descripción |
| ----- | ---- | ----------- |
| id | UUID PK | Identificador único. |
| module_id | UUID FK | Módulo al que pertenece (→ module.id). |
| code | VARCHAR(50) | Código de la funcionalidad. |
| name | VARCHAR(100) | Nombre de la funcionalidad. |
| description | TEXT | Descripción de la funcionalidad. |

### role_feature

Relaciona roles con funcionalidades.

| Campo | Tipo | Descripción |
| ----- | ---- | ----------- |
| id | UUID PK | Identificador único. |
| role_id | UUID FK | Rol (→ role.id). |
| feature_id | UUID FK | Funcionalidad (→ feature.id). |

### user_role

Asigna roles a los usuarios.

| Campo | Tipo | Descripción |
| ----- | ---- | ----------- |
| id | UUID PK | Identificador único. |
| user_id | UUID FK | Usuario (→ user.id). |
| role_id | UUID FK | Rol (→ role.id). |
| assigned_at | TIMESTAMPTZ | Fecha de asignación. |
| expires_at | TIMESTAMPTZ | Fecha de expiración (NULL = sin expiración). |

### password_reset_request

Solicitudes de recuperación de contraseña.

| Campo | Tipo | Descripción |
| ----- | ---- | ----------- |
| id | UUID PK | Identificador único. |
| user_id | UUID FK | Usuario solicitante (→ user.id). |
| token_hash | TEXT | Hash del token de recuperación. |
| expires_at | TIMESTAMPTZ | Fecha de expiración del token. |
| is_used | BOOLEAN | Indica si el token ya fue utilizado. |

### audit_login

Registro de intentos de autenticación.

| Campo | Tipo | Descripción |
| ----- | ---- | ----------- |
| id | UUID PK | Identificador único. |
| user_id | UUID FK | Usuario autenticado (→ user.id, NULL si no existe). |
| email_attempted | VARCHAR(255) | Correo utilizado en el intento. |
| outcome | VARCHAR(50) | Resultado del intento (éxito/fallo). |
| ip_address | VARCHAR(45) | Dirección IP del intento (IPv4/IPv6). |
| user_agent | TEXT | User agent del cliente. |
| attempted_at | TIMESTAMPTZ | Fecha y hora del intento. |

---

## Módulo parameterization

### event_category

Clasificación de eventos.

| Campo | Tipo | Descripción |
| ----- | ---- | ----------- |
| id | UUID PK | Identificador único. |
| code | VARCHAR(30) | Código de la categoría. |
| name | VARCHAR(100) | Nombre de la categoría. |
| description | TEXT | Descripción de la categoría. |

### severity

Define los niveles de severidad.

| Campo | Tipo | Descripción |
| ----- | ---- | ----------- |
| id | UUID PK | Identificador único. |
| code | VARCHAR(20) | Código de la severidad. |
| name | VARCHAR(50) | Nombre de la severidad. |
| priority | SMALLINT | Prioridad numérica (mayor = más crítico). |

### media_type

Tipos de evidencia multimedia.

| Campo | Tipo | Descripción |
| ----- | ---- | ----------- |
| id | UUID PK | Identificador único. |
| code | VARCHAR(20) | Código del tipo de medio. |
| name | VARCHAR(50) | Nombre del tipo de medio (ej: imagen, video). |

### sound_pattern

Patrones de sonido utilizados por el dispositivo.

| Campo | Tipo | Descripción |
| ----- | ---- | ----------- |
| id | UUID PK | Identificador único. |
| code | VARCHAR(30) | Código del patrón. |
| description | TEXT | Descripción del patrón. |
| frequency_hz | INTEGER | Frecuencia en hercios. |
| duration_ms | INTEGER | Duración en milisegundos. |

### event_type

Tipos de eventos detectados por el sistema.

| Campo | Tipo | Descripción |
| ----- | ---- | ----------- |
| id | UUID PK | Identificador único. |
| code | VARCHAR(30) | Código del tipo de evento. |
| name | VARCHAR(100) | Nombre del tipo de evento. |
| event_category_id | UUID FK | Categoría (→ event_category.id). |
| default_severity_id | UUID FK | Severidad por defecto (→ severity.id). |
| default_sound_pattern_id | UUID FK | Patrón de sonido por defecto (→ sound_pattern.id). |

---

## Módulo device-management

### device

Representa un dispositivo físico.

| Campo | Tipo | Descripción |
| ----- | ---- | ----------- |
| id | UUID PK | Identificador único. |
| serial_number | VARCHAR(100) | Número de serie del dispositivo. |
| api_key_hash | TEXT | Hash de la API key del dispositivo. |
| firmware_version | VARCHAR(50) | Versión de firmware instalada. |

### device_assignment

Historial de asignación de dispositivos.

| Campo | Tipo | Descripción |
| ----- | ---- | ----------- |
| id | UUID PK | Identificador único. |
| device_id | UUID FK | Dispositivo (→ device.id). |
| user_id | UUID FK | Usuario (→ user.id). |
| assigned_at | TIMESTAMPTZ | Fecha de asignación. |
| unassigned_at | TIMESTAMPTZ | Fecha de desasignación (NULL = vigente). |

### device_config

Configuración del dispositivo.

| Campo | Tipo | Descripción |
| ----- | ---- | ----------- |
| id | UUID PK | Identificador único. |
| device_id | UUID FK | Dispositivo (→ device.id). |
| configuration | JSONB | Configuración parametrizable del dispositivo. |

---

## Módulo telemetry-service

### event

Representa una ocurrencia detectada por el dispositivo.

| Campo | Tipo | Descripción |
| ----- | ---- | ----------- |
| id | UUID PK | Identificador único. |
| device_id | UUID FK | Dispositivo que generó el evento (→ device.id). |
| event_type_id | UUID FK | Tipo de evento (→ event_type.id). |
| occurred_at | TIMESTAMPTZ | Fecha y hora de la ocurrencia. |
| is_offline_sync | BOOLEAN | Indica si el evento fue sincronizado desde modo offline. |

### evidence

Archivos asociados a un evento.

| Campo | Tipo | Descripción |
| ----- | ---- | ----------- |
| id | UUID PK | Identificador único. |
| event_id | UUID FK | Evento asociado (→ event.id). |
| media_type_id | UUID FK | Tipo de medio (→ media_type.id). |
| file_url | TEXT | Referencia al archivo en el almacenamiento multimedia. |

### alert_log

Registro histórico de las alarmas emitidas por el dispositivo.

Un evento puede no generar alarmas, generar una única alarma o múltiples alarmas conforme evoluciona la condición detectada.

| Campo | Tipo | Descripción |
| ----- | ---- | ----------- |
| id | UUID PK | Identificador único. |
| event_id | UUID FK | Evento que originó la alarma (→ event.id). |
| sound_pattern_id | UUID FK | Patrón de sonido emitido (→ sound_pattern.id). |
| severity_id | UUID FK | Severidad de la alarma (→ severity.id). |
| triggered_at | TIMESTAMPTZ | Fecha y hora de la alarma. |

---

## Módulo monitoring

### notification

Notificaciones enviadas a los usuarios como consecuencia de una alarma.

| Campo | Tipo | Descripción |
| ----- | ---- | ----------- |
| id | UUID PK | Identificador único. |
| user_id | UUID FK | Usuario destinatario (→ user.id). |
| alert_log_id | UUID FK | Alarma que originó la notificación (→ alert_log.id). |
| title | VARCHAR(200) | Título de la notificación. |
| message | TEXT | Cuerpo de la notificación. |
| channel | VARCHAR(30) | Canal de envío (ej: in-app, push, email). |
| sent_at | TIMESTAMPTZ | Fecha de envío. |
| read_at | TIMESTAMPTZ | Fecha de lectura (NULL = no leída). |

---

## Relaciones principales

| Relación | Origen | Destino |
|----------|--------|---------|
| MODULE 1:N FEATURE | module.id | feature.module_id |
| ROLE N:M FEATURE | role_feature | role_feature.role_id / feature_id |
| USER N:M ROLE | user_role | user_role.user_id / role_id |
| USER 1:N PASSWORD_RESET_REQUEST | user.id | password_reset_request.user_id |
| USER 1:N AUDIT_LOGIN | user.id | audit_login.user_id |
| EVENT_CATEGORY 1:N EVENT_TYPE | event_category.id | event_type.event_category_id |
| SEVERITY 1:N EVENT_TYPE | severity.id | event_type.default_severity_id |
| SOUND_PATTERN 1:N EVENT_TYPE | sound_pattern.id | event_type.default_sound_pattern_id |
| DEVICE 1:N DEVICE_ASSIGNMENT | device.id | device_assignment.device_id |
| USER 1:N DEVICE_ASSIGNMENT | user.id | device_assignment.user_id |
| DEVICE 1:1 DEVICE_CONFIG | device.id | device_config.device_id |
| DEVICE 1:N EVENT | device.id | event.device_id |
| EVENT_TYPE 1:N EVENT | event_type.id | event.event_type_id |
| EVENT 1:N EVIDENCE | event.id | evidence.event_id |
| MEDIA_TYPE 1:N EVIDENCE | media_type.id | evidence.media_type_id |
| EVENT 1:N ALERT_LOG | event.id | alert_log.event_id |
| SOUND_PATTERN 1:N ALERT_LOG | sound_pattern.id | alert_log.sound_pattern_id |
| SEVERITY 1:N ALERT_LOG | severity.id | alert_log.severity_id |
| USER 1:N NOTIFICATION | user.id | notification.user_id |
| ALERT_LOG 1:N NOTIFICATION | alert_log.id | notification.alert_log_id |