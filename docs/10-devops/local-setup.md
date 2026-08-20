<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Setup local

**Estado:** En progreso
**Fecha:** 2026-08-19

</div>

</div>

Cómo levantar el entorno de desarrollo de SomnGuard en local: base de datos, migraciones y backend. Complementa a [ambientes](./environments.md).

## Prerrequisitos

- Docker + Docker Compose.
- Java 21 (Temurin) + Maven.
- Node.js (para portal web y app móvil, cuando se desarrolle).
- Repositorio `somnguard-api` clonado (backend Java).

## Arquitectura local

- **Una única base de datos PostgreSQL 16** en contenedor (BD `somnguard`, un schema por módulo).
- **Liquibase** integrado en el backend (Spring Boot): las migraciones se aplican al arrancar con el perfil activo.
- La configuración por ambiente vive en archivos de entorno: `.env.develop`, `.env.qa`, `.env.main`.

> **Nota de secretos:** los archivos `.env.*` no deben contener contraseñas reales versionadas. Usar `.env.example` como plantilla y mantener los valores reales fuera de git (ver [política de seguridad](../00-documentation-governance/security-policy.md)).

## Pasos

1. **Seleccionar ambiente y levantar Postgres:**

   ```bash
   docker compose --env-file .env.develop up postgres -d
   ```

2. **Levantar el backend** (aplica migraciones Liquibase al arrancar):

   ```bash
   mvn spring-boot:run -Dspring-boot.run.profiles=develop
   ```

3. **Verificar:** healthcheck en `http://localhost:8080/health`.

## Rollback local

```bash
mvn liquibase:rollback -Dliquibase.rollbackCount=1 -Dspring-boot.run.profiles=develop
```

Para volver a un release etiquetado: `rollback <tag>` (los tags se definen en `04_tcl/`; ver [convenciones de modelado](../06-data-architecture/modeling-conventions.md)).

## Apagar y limpiar

```bash
docker compose --env-file .env.develop down        # detiene los contenedores
docker compose --env-file .env.develop down -v     # además borra el volumen de datos
```

## Orden recomendado

Aplicar primero `parameterization` (catálogos) y `security` (identidad), y luego el resto de módulos que los referencian.

## Ver también

- [Ambientes](./environments.md)
- [Estrategia de CI/CD](./ci-cd-strategy.md)
- [Política de seguridad](../00-documentation-governance/security-policy.md)