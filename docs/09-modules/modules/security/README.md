<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Módulo: security — Autenticación, autorización y auditoría

**Estado:** En progreso
**Fecha:** 2026-09-01

</div>

</div>

> Última actualización: 2026-09-01
> Autor: Equipo backend

## Responsabilidad

Gestiona la autenticación, autorización y auditoría de usuarios del sistema SomnGuard. Define el modelo de permisos basado en roles (`admin`, `user`) y features por módulo.

## Entidades propias

| Entidad | Descripción |
|---------|-------------|
| `user` | Usuarios del sistema con credenciales y perfil |
| `role` | Roles del sistema (`admin`, `user`) |
| `module` | Módulos funcionales del sistema (6 módulos) |
| `feature` | Permisos atómicos por módulo (17 features) |
| `role_feature` | Relación rol-feature (sin scope) |
| `user_role` | Asignación de roles a usuarios con vigencia |
| `password_reset_request` | Solicitudes de recuperación de contraseña |
| `audit_login` | Registro de intentos de autenticación |

## Dependencias

| Módulo / recurso | Tipo | Motivo |
|------------------|------|--------|
| `platform` | transversal | Logging, observabilidad, errores |
| `parameterization` | port in | Catálogos de estados, severidades |
| `device_management` | port in | Validar device_id en asignaciones |

## Puertos (interfaces)

### Puertos de entrada (consumidos por adapters in)

| Puerto | Caso de uso típico |
|--------|--------------------|
| `AuthenticateUserPort` | Login de usuario, validación de credenciales |
| `AuthorizeRequestPort` | Verificar feature en requests |
| `ManageUserPort` | CRUD de usuarios, asignación de roles |
| `ManageRolePort` | CRUD de roles, asignación de features |
| `AuditLoginPort` | Registrar intentos de login (éxito/fallo) |

### Puertos de salida (implementados por adapters out)

| Puerto | Implementación |
|--------|----------------|
| `UserRepository` | `adapter/out/persistence` (PostgreSQL) |
| `RoleRepository` | `adapter/out/persistence` (PostgreSQL) |
| `ModuleRepository` | `adapter/out/persistence` (PostgreSQL) |
| `FeatureRepository` | `adapter/out/persistence` (PostgreSQL) |
| `PasswordResetRepository` | `adapter/out/persistence` (PostgreSQL) |
| `AuditLoginRepository` | `adapter/out/persistence` (PostgreSQL) |
| `TokenProvider` | `adapter/out/security` (JWT) |
| `PasswordHasher` | `adapter/out/security` (BCrypt/Argon2) |

## Base de datos

- Nombre lógico: `somnguard` (una sola BD; esquema por módulo)
- Motor: PostgreSQL 16 + Liquibase
- Esquema: `security`

## API expuesta

| Recurso | Nota |
|---------|------|
| `POST /api/v1/auth/login` | Autenticación, retorna JWT |
| `POST /api/v1/auth/refresh` | Renovación de access token |
| `POST /api/v1/auth/password-reset/request` | Solicitar reset de contraseña |
| `POST /api/v1/auth/password-reset/confirm` | Confirmar reset con token |
| `GET /api/v1/users` | Listar usuarios (`user.read`) |
| `GET /api/v1/users/{id}` | Detalle de usuario (`user.read`) |
| `POST /api/v1/users` | Crear usuario (`user.write`) |
| `PATCH /api/v1/users/{id}` | Actualizar usuario (`user.write`) |
| `DELETE /api/v1/users/{id}` | Desactivar usuario (`user.write`) |
| `GET /api/v1/roles` | Listar roles (`role.read`) |
| `POST /api/v1/roles` | Crear rol (`role.write`) |
| `POST /api/v1/roles/{id}/features` | Asignar feature a rol (`role.write`) |
| `POST /api/v1/users/{id}/roles` | Asignar rol a usuario (`role.write`) |
| `GET /api/v1/audit/login` | Logs de auditoría (`audit.read`) |

Ver contrato completo en [`../../../../07-api-design/contracts/`](../../../../07-api-design/contracts/)

## Links

- Data model: [data-model.md](./data-model.md)
- Eventos: [events.md](./events.md)
- Decisiones internas: [decisions.md](./decisions.md)
- Repo (código): `somnguard-api/backend-java` → `com.somnguard.security`