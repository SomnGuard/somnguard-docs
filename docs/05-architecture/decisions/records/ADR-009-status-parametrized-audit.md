# ADR-009: Estados Parametrizados + Auditoría Append-Only (status_category + status)

**Estado:** Aceptada
**Fecha:** 2026-08-22
**Autores:** Equipo SomnGuard
**Equipos involucrados:** Arquitectura, Backend, Device, DBA

---

## Contexto

SomnGuard tiene múltiples entidades con ciclo de vida y estados:
- **Device:** Registrado → Asignado → Activo ↔ Offline → Suspendido → Retirado
- **Event:** Detectado → Registrado → Sincronizado → Analizado → Archivado
- **User:** Pendiente → Activo → Suspendido → Eliminado (soft)
- **DeviceConfig:** Borrador → Publicado → Deprecado
- **Notification:** Pendiente → Enviado → Entregado → Leído / Fallido

Problemas del enfoque "un campo `status` VARCHAR por tabla":
- **Inconsistencia:** Cada módulo inventa sus strings (`active`, `ACTIVE`, `Active`, `1`)
- **Sin semántica:** No se sabe qué estados son "finales", "transitorios", "error"
- **Transiciones implícitas:** Cualquier valor → cualquier valor; sin validación
- **Difícil consulta:** "Todos los dispositivos activos" = `WHERE status IN ('Activo', 'ACTIVE', 'active', ...)`
- **Auditoría inexistente:** No hay histórico de cambios de estado

Requisitos (SRS, cross-cutting.md, guía ADR-004):
- **Estados parametrizados** (configurables sin deploy)
- **Transiciones validadas** (reglas de negocio)
- **Auditoría append-only** (histórico inmutable de cambios)
- **Soft delete** estándar (nunca DELETE físico)

---

## Decisión

### 1. Modelo Unificado de Dos Campos

Toda entidad con ciclo de vida usa **exactamente dos columnas**:

| Columna | Tipo | Descripción |
|---------|------|-------------|
| `status_category` | VARCHAR(30) NOT NULL | Grupo semántico: `ACTIVE`, `INACTIVE`, `PENDING`, `ERROR`, `ARCHIVED` |
| `status` | VARCHAR(50) NOT NULL | Valor específico dentro de la categoría |

> **Regla:** `status_category` deriva de `status` via catálogo (no se guarda redundante en app; se hace JOIN o vista).

### 2. Catálogo Centralizado (Parameterization Module)

Tablas en esquema `parameterization`:

```sql
-- Categoría de estado (alto nivel)
CREATE TABLE status_category (
    code        VARCHAR(30) PRIMARY KEY,  -- ACTIVE, INACTIVE, PENDING, ERROR, ARCHIVED
    name        VARCHAR(100) NOT NULL,
    description TEXT,
    sort_order  INTEGER NOT NULL,
    is_final    BOOLEAN NOT NULL DEFAULT FALSE,  -- true = estado terminal (ej. ARCHIVED)
    created_at  TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at  TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- Estado específico (bajo nivel)
CREATE TABLE status (
    code              VARCHAR(50) PRIMARY KEY,           -- ACTIVE, OFFLINE, SUSPENDED, etc.
    status_category   VARCHAR(30) NOT NULL REFERENCES status_category(code),
    name              VARCHAR(100) NOT NULL,
    description       TEXT,
    entity_type       VARCHAR(50) NOT NULL,              -- device, event, user, device_config, notification
    sort_order        INTEGER NOT NULL,
    is_initial        BOOLEAN NOT NULL DEFAULT FALSE,    -- estado inicial al crear entidad
    is_terminal       BOOLEAN NOT NULL DEFAULT FALSE,    -- no hay transiciones salientes
    created_at        TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at        TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- Transiciones permitidas (regla de negocio)
CREATE TABLE status_transition (
    from_status   VARCHAR(50) NOT NULL REFERENCES status(code),
    to_status     VARCHAR(50) NOT NULL REFERENCES status(code),
    allowed_roles VARCHAR(100)[],  -- roles que pueden ejecutar (NULL = cualquiera)
    description   TEXT,
    created_at    TIMESTAMPTZ NOT NULL DEFAULT now(),
    PRIMARY KEY (from_status, to_status)
);
```

### 3. Seed Data (Ejemplos)

```sql
-- status_category
INSERT INTO status_category (code, name, description, sort_order, is_final) VALUES
('ACTIVE',     'Activo',      'Entidad operativa y funcional',           10, FALSE),
('INACTIVE',   'Inactivo',    'Entidad existente pero no operativa',     20, FALSE),
('PENDING',    'Pendiente',   'Entidad en proceso de activación',        30, FALSE),
('ERROR',      'Error',       'Entidad en estado de fallo',              40, FALSE),
('ARCHIVED',   'Archivado',   'Entidad finalizada, solo lectura',        50, TRUE);

-- status (device)
INSERT INTO status (code, status_category, name, description, entity_type, sort_order, is_initial, is_terminal) VALUES
('REGISTERED', 'PENDING',    'Registrado',     'Alta en plataforma, sin asignar',       'device', 10, TRUE,  FALSE),
('ASSIGNED',   'PENDING',    'Asignado',       'Asociado a usuario, sin activar',       'device', 20, FALSE, FALSE),
('ACTIVE',     'ACTIVE',     'Activo',         'Online, enviando telemetría',           'device', 30, FALSE, FALSE),
('OFFLINE',    'INACTIVE',   'Offline',        'Sin heartbeat > 5 min',                   'device', 40, FALSE, FALSE),
('SUSPENDED',  'INACTIVE',   'Suspendido',     'Admin deshabilitó',                       'device', 50, FALSE, FALSE),
('RETIRED',    'ARCHIVED',   'Retirado',       'Fin de vida, no reactivable',             'device', 60, FALSE, TRUE);

-- status (event)
INSERT INTO status (code, status_category, name, description, entity_type, sort_order, is_initial, is_terminal) VALUES
('DETECTED',       'PENDING',  'Detectado',       'Generado en device, en buffer local',    'event', 10, TRUE,  FALSE),
('REGISTERED',     'PENDING',  'Registrado',      'Persistido en device, no sincronizado',  'event', 20, FALSE, FALSE),
('SYNCHRONIZED',   'ACTIVE',   'Sincronizado',    'Confirmado por API (ACK)',               'event', 30, FALSE, FALSE),
('ANALYZED',       'ACTIVE',   'Analizado',       'Incluido en métricas/reportes',          'event', 40, FALSE, FALSE),
('ARCHIVED',       'ARCHIVED', 'Archivado',       'Retención cumplida, solo lectura',       'event', 50, FALSE, TRUE);

-- status (user)
INSERT INTO status (code, status_category, name, description, entity_type, sort_order, is_initial, is_terminal) VALUES
('PENDING_VERIFICATION', 'PENDING', 'Pendiente verificación', 'Correo no verificado',     'user', 10, TRUE,  FALSE),
('ACTIVE',               'ACTIVE',  'Activo',                'Cuenta operativa',           'user', 20, FALSE, FALSE),
('SUSPENDED',            'INACTIVE','Suspendido',            'Admin deshabilitó',          'user', 30, FALSE, FALSE),
('SOFT_DELETED',         'ARCHIVED','Eliminado (soft)',      'Ventana 30d recuperación',   'user', 40, FALSE, TRUE);

-- status_transition (device)
INSERT INTO status_transition (from_status, to_status, allowed_roles, description) VALUES
('REGISTERED', 'ASSIGNED',   ARRAY['user'],       'Usuario asocia device a su cuenta'),
('ASSIGNED',   'ACTIVE',     ARRAY['system'],     'Primer heartbeat recibido'),
('ACTIVE',     'OFFLINE',    ARRAY['system'],     'Sin heartbeat > 5 min'),
('OFFLINE',    'ACTIVE',     ARRAY['system'],     'Heartbeat recibido'),
('ACTIVE',     'SUSPENDED',  ARRAY['admin'],      'Admin suspende device'),
('SUSPENDED',  'ACTIVE',     ARRAY['admin'],      'Admin reactiva'),
('SUSPENDED',  'RETIRED',    ARRAY['admin'],      'Admin retira device'),
('REGISTERED', 'RETIRED',    ARRAY['admin'],      'Admin cancela alta');

-- status_transition (event) — solo system
INSERT INTO status_transition (from_status, to_status, allowed_roles, description) VALUES
('DETECTED',       'REGISTERED',    ARRAY['system'], 'Persistido en buffer local'),
('REGISTERED',     'SYNCHRONIZED',  ARRAY['system'], 'ACK recibido de API'),
('SYNCHRONIZED',   'ANALYZED',      ARRAY['system'], 'Procesado por analytics'),
('ANALYZED',       'ARCHIVED',      ARRAY['system'], 'Retención cumplida');
```

### 4. Validación en Código (Domain Service)

```java
// parameterization/domain/service/StatusTransitionService.java
public interface StatusTransitionService {
    /**
     * Valida y ejecuta transición de estado.
     * @throws IllegalStateTransitionException si no permitida
     */
    void transition(EntityWithStatus entity, String toStatus, UserContext context);
}

// Implementación
@Service
public class StatusTransitionServiceImpl implements StatusTransitionService {
    private final StatusTransitionRepository repo;
    
    public void transition(EntityWithStatus entity, String toStatus, UserContext context) {
        String fromStatus = entity.getStatus();
        
        // 1. Existe transición definida?
        StatusTransition rule = repo.findByFromAndTo(fromStatus, toStatus)
            .orElseThrow(() -> new IllegalStateTransitionException(
                "No transition defined from " + fromStatus + " to " + toStatus));
        
        // 2. Rol permitido?
        if (rule.getAllowedRoles() != null && !rule.getAllowedRoles().isEmpty()) {
            boolean hasRole = context.getRoles().stream()
                .anyMatch(r -> rule.getAllowedRoles().contains(r));
            if (!hasRole) throw new AccessDeniedException("Role not allowed for this transition");
        }
        
        // 3. Ejecutar
        entity.setStatus(toStatus);
        entity.setStatusCategory(repo.getCategory(toStatus));
        entity.setUpdatedAt(Instant.now());
        entity.setUpdatedBy(context.getUserId());
        
        // 4. Auditar (append-only)
        auditLogRepository.save(new StatusAuditLog(
            entity.getClass().getSimpleName(),
            entity.getId(),
            fromStatus,
            toStatus,
            context.getUserId(),
            Instant.now()
        ));
    }
}
```

### 5. Auditoría Append-Only (Tablas `_audit`)

Para **cada entidad con estado**, tabla de auditoría automática (trigger PG):

```sql
-- Ejemplo: device_status_audit
CREATE TABLE device_status_audit (
    id              BIGSERIAL PRIMARY KEY,
    device_id       UUID NOT NULL REFERENCES device_management.device(id),
    from_status     VARCHAR(50),
    to_status       VARCHAR(50) NOT NULL,
    from_category   VARCHAR(30),
    to_category     VARCHAR(30) NOT NULL,
    changed_by      UUID,  -- user_id o device_id (NULL = system)
    changed_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    context_json    JSONB  -- metadata extra (razón, IP, etc.)
);

-- Trigger (genérico, una función por tabla)
CREATE OR REPLACE FUNCTION audit_status_change()
RETURNS TRIGGER AS $$
BEGIN
    IF OLD.status IS DISTINCT FROM NEW.status THEN
        INSERT INTO device_status_audit (device_id, from_status, to_status, from_category, to_category, changed_by, context_json)
        VALUES (NEW.id, OLD.status, NEW.status, OLD.status_category, NEW.status_category, 
                COALESCE(current_setting('app.current_user_id', TRUE), NULL)::UUID,
                jsonb_build_object('trigger', TG_OP));
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

CREATE TRIGGER trg_device_status_audit
AFTER UPDATE OF status, status_category ON device_management.device
FOR EACH ROW EXECUTE FUNCTION audit_status_change();
```

### 6. Soft Delete Estándar (Cross-Cutting)

| Campo | Tipo | Regla |
|-------|------|-------|
| `deleted_at` | TIMESTAMPTZ | NULL = activo; timestamp = eliminado |
| `deleted_by` | UUID | Quien ejecutó el delete (user_id o device_id) |

**Nunca** `DELETE FROM tabla WHERE ...` en código de aplicación.
**Siempre** `UPDATE tabla SET deleted_at = now(), deleted_by = ? WHERE id = ? AND deleted_at IS NULL`.

**Índice parcial para performance:**
```sql
CREATE INDEX idx_device_active ON device_management.device (id) WHERE deleted_at IS NULL;
```

**Vistas "activas" por defecto:**
```sql
CREATE VIEW device_management.v_device_active AS
SELECT * FROM device_management.device WHERE deleted_at IS NULL;
```

---

## Consecuencias

### Positivas
- **Consistencia total:** Todos los módulos usan mismo patrón `status_category` + `status`
- **Parametrización real:** Cambiar/agregar estados = INSERT en catálogo (sin deploy)
- **Transiciones seguras:** Validadas en domain service; imposible estado inválido en BD
- **Auditoría completa:** Histórico inmutable de cada cambio (quién, cuándo, de qué a qué)
- **Consultas simples:** `WHERE status_category = 'ACTIVE'` funciona para device, event, user
- **Soft delete universal:** `deleted_at` en todas las tablas; vistas activas por defecto

### Negativas / Trade-offs
- **Overhead inicial:** Catálogo + triggers + domain service por entidad
- **Join extra** para leer `status_category` (mitigado: vista materializada o denormalizar en entidad)
- **Migración de datos legacy** si ya hay tablas con `status` VARCHAR suelto

### Riesgos
- **Transiciones faltantes** en catálogo → bloquean operación legítima → Mitigación: Test de cobertura de transiciones en CI
- **Rol `system` implícito** en transitions automáticas (heartbeat, sync) → Mitigación: `allowed_roles` NULL = system; documentar en catálogo
- **Performance triggers** en tablas de alta escritura (`event`) → Mitigación: Batch audit log (outbox pattern) o tabla `_audit` sin FKs estrictas

---

## Alternativas consideradas

| Alternativa | Por qué se descartó |
|-------------|---------------------|
| **Un solo campo `status` VARCHAR + CHECK constraint** | CHECK hardcodeado = deploy para cambiar; sin semántica de categoría; sin transiciones |
| **ENUM en PostgreSQL** | `ALTER TYPE ... ADD VALUE` requiere lock exclusivo; no parametrizable; sin auditoría nativa |
| **State machine library (Stateless, Squirrel) solo en código** | No protege BD; otro cliente (script, otro servicio) puede violar; sin auditoría centralizada |
| **Event Sourcing completo** | Overkill para MVP; complejidad de proyecciones, replay, snapshots |
| **Tabla `state_machine` genérica (entity_type, entity_id, state)** | Polimorfismo en BD = joins complejos, FKs imposibles, performance mala |

---

## Referencias

- [cross-cutting.md](../cross-cutting.md#5-estados-parametrizados-adr-004-guía-adaptado)
- [modeling-conventions.md](../../../06-data-architecture/modeling-conventions.md) — campos auditoría obligatorios
- [functional.md](../../../04-requeriments/functional.md) → RF-DEV-05, RF-TEL-*, RF-MON-*, RF-SEC-08
- [entities-and-rules.md](../../../02-domain/entities-and-rules.md) — RN-DEV-06, RN-TEL-*, RN-SEC-*
- Guía ADR-004 (SENA): `status_category` + `status` + append-only audit
- PostgreSQL Triggers: https://www.postgresql.org/docs/current/triggers.html