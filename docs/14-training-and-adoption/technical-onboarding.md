<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Onboarding técnico

**Estado:** En progreso
**Fecha:** 2026-08-19

</div>

</div>

Guía de entrada para nuevos integrantes técnicos del proyecto **SomnGuard**. Al terminar deberías poder clonar los repos, entender la arquitectura, levantar el entorno local y hacer tu primer cambio.

> **Estado real del proyecto:** existen la documentación (somnguard-docs) y el esqueleto del backend (somnguard-api, Java 21 + Spring Boot 3.x, estructura hexagonal). La capa de aplicación está en construcción. Complementa a [local-setup.md](../10-devops/local-setup.md).

## 1. Qué es el sistema (en una página)

Sistema de **prevención de accidentes por fatiga o somnolencia al volante**: un dispositivo edge (Raspberry Pi + cámara) detecta estados de riesgo con visión por computadora, emite alertas sonoras inmediatas y sincroniza eventos con evidencia multimedia a un backend central (monolito modular hexagonal, Java 21 + Spring Boot 3.x). Los clientes (portal web React JS y app móvil React Native) permiten consultar eventos, recibir notificaciones y administrar cuentas, dispositivos y catálogos.

Vocabulario mínimo: **dispositivo** (Raspberry Pi + cámara en campo), **evento** (detección registrada), **evidencia** (multimedia del evento), **alerta** (aviso local del dispositivo), **notificación** (mensaje hacia la cuenta). Diccionario completo en el [glosario](../02-domain/glossary.md).

## 2. Prerrequisitos

- **Docker** + **Docker Compose**.
- **Git** y acceso a los repositorios (`somnguard-docs`, `somnguard-api`).
- **Java 21 (Temurin)** + **Maven**.
- Un cliente PostgreSQL (psql, DBeaver, etc.) para inspeccionar la BD (opcional pero recomendado).

## 3. Estructura de repositorios

```
workspace/
├── somnguard-docs/             # Documentación (este repo) — rama develop
└── somnguard-api/              # Backend y frontends
    └── backend-java/
        └── src/main/java/com/somnguard/
            ├── security/            # Autenticación, RBAC, auditoría
            ├── parameterization/    # Catálogos configurables
            ├── device_management/   # Dispositivos, asignaciones, configuración
            ├── telemetry_service/   # Eventos, evidencia, alertas
            ├── monitoring/          # Notificaciones
            ├── analytics/           # Métricas, resumen IA, reportes
            └── platform/            # Transversal: errores, logging, observabilidad
```

Cada módulo sigue la estructura hexagonal (ver [structure-rules.md](../00-documentation-governance/structure-rules.md) y ADR-002):

```
application/port/{in,out}   application/usecase
domain/{model,service}
adapter/in/{web,amqp}       adapter/out/{persistence,storage}
```

## 4. Levantar el entorno local

```bash
# 1) Levantar Postgres para el ambiente de desarrollo
docker compose --env-file .env.develop up postgres -d

# 2) Levantar el backend (aplica migraciones Liquibase al arrancar)
mvn spring-boot:run -Dspring-boot.run.profiles=develop

# 3) Verificar
# Healthcheck: http://localhost:8080/health
```

**Orden recomendado de aplicación de migraciones:** primero `parameterization` (catálogos) y `security` (identidad); luego el resto de módulos que los referencian.

### Rollback y limpieza (solo local)

```bash
mvn liquibase:rollback -Dliquibase.rollbackCount=1 -Dspring-boot.run.profiles=develop
docker compose --env-file .env.develop down          # apagar
docker compose --env-file .env.develop down -v       # reinicio limpio
```

> En `qa`/`main` las migraciones son **forward-only**; los rollbacks son para desarrollo local y como plan de contingencia documentado.

## 5. Convenciones que debes conocer antes de tu primer cambio

- **Arquitectura hexagonal:** las dependencias apuntan al centro (`domain` ← `application` ← `adapter/*`); `platform` es transversal y no depende de módulos ([structure-rules.md](../00-documentation-governance/structure-rules.md)).
- **Una BD, un schema por módulo:** ningún módulo escribe en `public` ni en el schema de otro ([modeling-conventions.md](../06-data-architecture/modeling-conventions.md)).
- **Columnas de auditoría obligatorias** en tablas transaccionales (`created_at`, `updated_at`, `deleted_at`, `created_by`, `updated_by`, `deleted_by`, `is_active`).
- **Identificadores**: HUs `HU-<REPO>-NNN` (DEVICE/API/DB/APP/PORTAL, en GitHub Projects), criterios `AC-`, casos `TC-`, reglas `RN-`, NFR `NFR-`, ADRs en `05-architecture/decisions/records/` ([agile-conventions.md](../00-documentation-governance/agile-conventions.md)).
- **Cada changeset** Liquibase declara `id` + `author` únicos y su `rollback` espejo en `05_rollbacks/`.
- **Seeds idempotentes** (`INSERT ... ON CONFLICT DO NOTHING`); separar catálogos de datos de prueba con `context`/`labels`.
- **Secretos fuera de git:** los `.env.*` no versionan contraseñas reales; usar `.env.example` ([security-policy.md](../00-documentation-governance/security-policy.md)).
- **Git y ramas:** promoción `develop → qa → main`; commits con Conventional Commits ([git-conventions.md](../00-documentation-governance/git-conventions.md)).

## 6. Gobernanza y decisiones

- Toda decisión de arquitectura relevante requiere un **ADR** ([05-architecture/decisions/records/](../05-architecture/decisions/records/)).
- ADRs vigentes: ADR-001 (stack), ADR-002 (hexagonal), ADR-003 (analytics).
- Antes de contribuir, lee [CONTRIBUTING.md](../../CONTRIBUTING.md) y [00-documentation-governance/](../00-documentation-governance/).

## 7. Checklist de primer día

- [ ] Docker y Docker Compose funcionando.
- [ ] `somnguard-docs` y `somnguard-api` clonados.
- [ ] Postgres levantado con `.env.develop`.
- [ ] Backend compila y levanta (`mvn verify` + healthcheck OK).
- [ ] Leídos: [overview.md](../01-project-context/overview.md), [domain-map.md](../02-domain/domain-map.md), [architecture-document.md](../05-architecture/architecture-document.md), [modeling-conventions.md](../06-data-architecture/modeling-conventions.md).

## Referencias

- [local-setup.md](../10-devops/local-setup.md) · [modeling-conventions.md](../06-data-architecture/modeling-conventions.md)
- [overview.md](../01-project-context/overview.md) · [domain-map.md](../02-domain/domain-map.md) · [09-modules](../09-modules/README.md)
- [CONTRIBUTING.md](../../CONTRIBUTING.md) · [00-documentation-governance/](../00-documentation-governance/)