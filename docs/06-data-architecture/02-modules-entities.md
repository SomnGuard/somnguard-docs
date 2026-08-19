# Modulos entidades

> **Proyecto:** SomnGuard
> **Versión:** 1.0
> **Convención:** Todos los nombres de entidades y atributos se encuentran en inglés.

---

# Convenciones generales

## Identificador

Todas las entidades utilizan:

| Campo | Tipo | Descripción |
| ----- | ---- | ----------- |
| id | UUID | Identificador único. |

---

## Auditoría

Todas las entidades transaccionales incluyen:

| Campo | Tipo |
| ----- | ---- |
| created_at | TIMESTAMPTZ |
| updated_at | TIMESTAMPTZ |
| deleted_at | TIMESTAMPTZ |
| created_by | UUID |
| updated_by | UUID |
| deleted_by | UUID |
| is_active | BOOLEAN |

---

# Modulos

# security

## Responsabilidad

Gestiona la autenticación, autorización y auditoría de usuarios.

---

## user

Representa a los usuarios del sistema.

| Campo | Tipo |
| ----- | ---- |
| id | UUID |
| email | VARCHAR(255) |
| password_hash | TEXT |
| first_name | VARCHAR(100) |
| last_name | VARCHAR(100) |
| phone | VARCHAR(30) |

---

## role

Define los roles del sistema.

| Campo | Tipo |
| ----- | ---- |
| id | UUID |
| name | VARCHAR(100) |
| description | TEXT |

---

## module

Agrupa funcionalidades del sistema.

| Campo | Tipo |
| ----- | ---- |
| id | UUID |
| code | VARCHAR(50) |
| name | VARCHAR(100) |
| description | TEXT |

---

## feature

Representa una funcionalidad protegida del sistema.

| Campo | Tipo |
| ----- | ---- |
| id | UUID |
| module_id | UUID |
| code | VARCHAR(50) |
| name | VARCHAR(100) |
| description | TEXT |

---

## role_feature

Relaciona roles con funcionalidades.

| Campo | Tipo |
| ----- | ---- |
| id | UUID |
| role_id | UUID |
| feature_id | UUID |

---

## user_role

Asigna roles a los usuarios.

| Campo | Tipo |
| ----- | ---- |
| id | UUID |
| user_id | UUID |
| role_id | UUID |
| assigned_at | TIMESTAMPTZ |
| expires_at | TIMESTAMPTZ |

---

## password_reset_request

Solicitudes de recuperación de contraseña.

| Campo | Tipo |
| ----- | ---- |
| id | UUID |
| user_id | UUID |
| token_hash | TEXT |
| expires_at | TIMESTAMPTZ |
| is_used | BOOLEAN |

---

## audit_login

Registro de intentos de autenticación.

| Campo | Tipo |
| ----- | ---- |
| id | UUID |
| user_id | UUID |
| email_attempted | VARCHAR(255) |
| outcome | VARCHAR(50) |
| ip_address | VARCHAR(45) |
| user_agent | TEXT |
| attempted_at | TIMESTAMPTZ |

---

# parameterization

## Responsabilidad

Gestiona los catálogos configurables del sistema.

---

## event_category

Clasificación de eventos.

| Campo | Tipo |
| ----- | ---- |
| id | UUID |
| code | VARCHAR(30) |
| name | VARCHAR(100) |
| description | TEXT |

---

## severity

Define los niveles de severidad.

| Campo | Tipo |
| ----- | ---- |
| id | UUID |
| code | VARCHAR(20) |
| name | VARCHAR(50) |
| priority | SMALLINT |

---

## media_type

Tipos de evidencia multimedia.

| Campo | Tipo |
| ----- | ---- |
| id | UUID |
| code | VARCHAR(20) |
| name | VARCHAR(50) |

---

## sound_pattern

Patrones de sonido utilizados por el dispositivo.

| Campo | Tipo |
| ----- | ---- |
| id | UUID |
| code | VARCHAR(30) |
| description | TEXT |
| frequency_hz | INTEGER |
| duration_ms | INTEGER |

---

## event_type

Tipos de eventos detectados por el sistema.

| Campo | Tipo |
| ----- | ---- |
| id | UUID |
| code | VARCHAR(30) |
| name | VARCHAR(100) |
| event_category_id | UUID |
| default_severity_id | UUID |
| default_sound_pattern_id | UUID |

---

# device-management

## Responsabilidad

Gestiona dispositivos, asignaciones y configuración.

---

## device

Representa un dispositivo físico.

| Campo | Tipo |
| ----- | ---- |
| id | UUID |
| serial_number | VARCHAR(100) |
| api_key_hash | TEXT |
| firmware_version | VARCHAR(50) |

---

## device_assignment

Historial de asignación de dispositivos.

| Campo | Tipo |
| ----- | ---- |
| id | UUID |
| device_id | UUID |
| user_id | UUID |
| assigned_at | TIMESTAMPTZ |
| unassigned_at | TIMESTAMPTZ |

---

## device_config

Configuración del dispositivo.

| Campo | Tipo |
| ----- | ---- |
| id | UUID |
| device_id | UUID |
| configuration | JSONB |

---

# telemetry-service

## Responsabilidad

Recibe y almacena la información generada por los dispositivos.

---

## event

Representa una ocurrencia detectada por el dispositivo.

| Campo | Tipo |
| ----- | ---- |
| id | UUID |
| device_id | UUID |
| event_type_id | UUID |
| occurred_at | TIMESTAMPTZ |
| is_offline_sync | BOOLEAN |

---

## evidence

Archivos asociados a un evento.

| Campo | Tipo |
| ----- | ---- |
| id | UUID |
| event_id | UUID |
| media_type_id | UUID |
| file_url | TEXT |

---

## alert_log

Registro histórico de las alarmas emitidas por el dispositivo.

Un evento puede no generar alarmas, generar una única alarma o múltiples alarmas conforme evoluciona la condición detectada.

| Campo | Tipo |
| ----- | ---- |
| id | UUID |
| event_id | UUID |
| sound_pattern_id | UUID |
| severity_id | UUID |
| triggered_at | TIMESTAMPTZ |

---

# monitoring

## Responsabilidad

Gestiona el envío y seguimiento de notificaciones.

---

## notification

Notificaciones enviadas a los usuarios como consecuencia de una alarma.

| Campo | Tipo |
| ----- | ---- |
| id | UUID |
| user_id | UUID |
| alert_log_id | UUID |
| title | VARCHAR(200) |
| message | TEXT |
| channel | VARCHAR(30) |
| sent_at | TIMESTAMPTZ |
| read_at | TIMESTAMPTZ |

---

# Relaciones

## security

- Un **MODULE** contiene múltiples **FEATURE**.
- Un **ROLE** posee múltiples **FEATURE** mediante **ROLE_FEATURE**.
- Un **USER** puede tener múltiples **ROLE** mediante **USER_ROLE**.
- Un **USER** puede generar múltiples **PASSWORD_RESET_REQUEST**.
- Un **USER** puede registrar múltiples **AUDIT_LOGIN**.

---

## parameterization

- Un **EVENT_CATEGORY** clasifica múltiples **EVENT_TYPE**.
- Un **EVENT_TYPE** define una **SEVERITY** por defecto.
- Un **EVENT_TYPE** define un **SOUND_PATTERN** por defecto.

---

## device-management

- Un **DEVICE** puede asignarse a múltiples usuarios mediante **DEVICE_ASSIGNMENT**.
- Un **DEVICE** puede tener una configuración almacenada en **DEVICE_CONFIG**.

---

## telemetry

- Un **DEVICE** genera múltiples **EVENT**.
- Un **EVENT** pertenece a un **EVENT_TYPE**.
- Un **EVENT** puede tener múltiples **EVIDENCE**.
- Una **EVIDENCE** pertenece a un **MEDIA_TYPE**.
- Un **EVENT** puede generar cero, una o múltiples **ALERT_LOG**.
- Un **ALERT_LOG** utiliza un **SOUND_PATTERN**.
- Un **ALERT_LOG** registra una **SEVERITY**.

---

## monitoring

- Un **USER** puede recibir múltiples **NOTIFICATION**.
- Un **ALERT_LOG** puede originar múltiples **NOTIFICATION**.