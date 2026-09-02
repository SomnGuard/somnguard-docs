<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Decisiones internas — security

**Estado:** En progreso
**Fecha:** 2026-09-01

</div>

</div>

> Última actualización: 2026-09-01

## Registro de decisiones (ADR locales)

### SEC-001: Modelo de permisos simple (rol + feature, sin scope)

**Fecha:** 2026-09-01
**Estado:** Aceptada

**Contexto:** El sistema actual tiene solo 2 roles (`admin`, `user`) y 17 features. No hay necesidad de restricción de datos por flota/turno en esta fase.

**Decisión:** Modelo de permisos plano: `role_feature` relaciona rol con feature directamente, sin columna `scope_type`.

**Consecuencias:**
- + Simplicidad: una tabla, una relación
- + Admin tiene todas las features, user solo lectura básica
- + Fácil de auditar y mantener
- - No permite restricción por flota/turno (se puede añadir luego si se requiere)

**Alternativas descartadas:**
- RBAC con scope (Módulo → Feature → Scope): overengineering para 2 roles
- ABAC: demasiado complejo para fase actual

---

### SEC-002: Nomenclatura de features en minúscula con punto (`recurso.accion`)

**Fecha:** 2026-09-01
**Estado:** Aceptada

**Contexto:** Las features en BD usan formato `user.read`, `device.write`, `event.read`, etc. (minúscula, separadas por punto).

**Decisión:** Mantener formato actual: `{recurso}.{accion}` en minúscula.

**Ejemplos:**
- `user.read`, `user.write`
- `role.read`, `role.write`
- `device.read`, `device.write`, `device.config`
- `event.read`, `event.write`, `alert.read`
- `notification.read`, `notification.write`
- `analytics.read`, `analytics.report`
- `catalog.read`, `catalog.write`
- `audit.read`

**Consecuencias:**
- + Consistente con BD actual (seeds 011_insert_security_features.sql)
- + Simple y legible
- + No requiere migración de datos

---

### SEC-003: Solo 2 roles (`admin`, `user`)

**Fecha:** 2026-09-01
**Estado:** Aceptada

**Contexto:** Seeds en `somnguard-db/02_dml/00_inserts/009_insert_security_roles.sql` definen solo 2 roles.

**Decisión:** Mantener solo `admin` y `user` por ahora.

**Asignación actual (seeds 012_insert_security_role_feature.sql):**
- `admin`: **todas** las 17 features
- `user`: solo features de lectura básica:
  - `user.read` (security)
  - `device.read` (device_management)
  - `event.read` (telemetry)
  - `notification.read` (monitoring)
  - `analytics.read` (analytics)

**Consecuencias:**
- + Modelo simple y claro
- + Fácil de extender cuando se necesiten más roles
- - `user` tiene permisos limitados (solo lectura básica)

---

### SEC-004: 6 módulos fijos alineados con module-catalog.md

**Fecha:** 2026-09-01
**Estado:** Aceptada

**Contexto:** Catálogo de módulos en BD y docs coinciden en 6 módulos.

**Módulos (códigos en minúscula):**
1. `security` — Autenticación, autorización y auditoría
2. `device_management` — Registro, asignación y configuración de dispositivos
3. `telemetry` — Ingesta y procesamiento de eventos
4. `monitoring` — Notificaciones y alertas
5. `analytics` — Reportes, métricas y dashboards
6. `parameterization` — Catálogos configurables del sistema

**Consecuencias:**
- + Alineación total BD ↔ docs ↔ código
- + Features agrupadas por módulo real

---

### SEC-005: JWT con features resueltas (no roles)

**Fecha:** 2026-09-01
**Estado:** Aceptada

**Contexto:** Para autorización eficiente en servicios downstream.

**Decisión:** JWT lleva `features[]` resueltas como array de strings `["user.read", "device.write", ...]`, no roles.

**Ejemplo payload JWT:**
```json
{
  "sub": "uuid-usuario",
  "roles": ["admin"],
  "features": [
    "user.read", "user.write",
    "role.read", "role.write",
    "audit.read",
    "device.read", "device.write", "device.config",
    "event.read", "event.write", "alert.read",
    "notification.read", "notification.write",
    "analytics.read", "analytics.report",
    "catalog.read", "catalog.write"
  ],
  "exp": 1234567890
}
```

**Consecuencias:**
- + Servicios downstream no consultan BD de roles
- + Latencia de autorización mínima
- + Fácil validación: `if (features.includes('user.write'))`
- - JWT más grande (~300-500 bytes típicos)

---

### SEC-006: Role-feature sin scope_type (tabla simple)

**Fecha:** 2026-09-01
**Estado:** Aceptada

**Contexto:** Tabla `role_feature` en BD solo tiene `id, role_id, feature_id, created_at, created_by`.

**Decisión:** No añadir `scope_type` por ahora. La tabla permanece simple.

**Consecuencias:**
- + Esquema actual de BD soportado sin cambios
- + Migración futura posible añadiendo columna nullable
- - Sin restricción de datos por flota/turno (no requerido hoy)

---

### SEC-007: User-role con vigencia temporal (expires_at)

**Fecha:** 2026-09-01
**Estado:** Aceptada

**Contexto:** Necesidad de accesos temporales (coberturas, proyectos).

**Decisión:** `user_role.expires_at` TIMESTAMPTZ nullable (NULL = indefinido). Soft delete para revocación manual.

**Consecuencias:**
- + Expiración automática sin job de limpieza
- + Historial de asignaciones preservado
- - Query de roles activos requiere filtro compuesto

---

### SEC-008: Audit_login append-only sin soft delete

**Fecha:** 2026-09-01
**Estado:** Aceptada

**Contexto:** Logs de auditoría deben ser inmutables.

**Decisión:** Tabla `audit_login` sin `deleted_at`, `is_active`, `updated_at`. Solo `created_at`, `created_by`.

**Consecuencias:**
- + Inmutabilidad garantizada por diseño
- + Cumple requisitos de auditoría/compliance
- - Crecimiento indefinido (particionar por mes en futuro)

---

### SEC-009: Password reset con token hash (no token en claro)

**Fecha:** 2026-09-01
**Estado:** Aceptada

**Contexto:** Seguridad del flujo de reset de contraseña.

**Decisión:** Almacenar `token_hash` (BCrypt) en `password_reset_request`. Token en claro solo via email (expira 1h).

**Consecuencias:**
- + Si BD comprometida, tokens no servibles
- + Verificación: `BCrypt.check(token_plano, token_hash)`
- - Costo BCrypt en validación (aceptable: flujo raro)

---

### SEC-010: Bloqueo de cuenta por intentos fallidos (5 intentos, 15 min)

**Fecha:** 2026-09-01
**Estado:** Aceptada

**Contexto:** Protección contra fuerza bruta online.

**Decisión:**
- `failed_login_attempts` incrementa en cada fallo
- `locked_until = NOW() + 15 minutos` al alcanzar 5
- Reset contador en login exitoso o desbloqueo manual
- Log en `audit_login` con `outcome = FAILED_LOCKED`

**Consecuencias:**
- + Protección básica sin dependencias externas
- + Visibilidad en auditoría
- - No protege contra ataque distribuido (rate limit en API Gateway complementario)

---

## Referencias cruzadas

- [ADR-003: Analytics module](../../../../05-architecture/decisions/records/ADR-003-analytics-module.md)
- [ADR-009: Estados de negocio parametrizados](../../../../05-architecture/decisions/records/ADR-009-parametrized-status.md)
- [Modelo de datos](./data-model.md)
- [Catálogo de módulos](../../module-catalog.md)