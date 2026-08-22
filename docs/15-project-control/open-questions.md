<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Preguntas abiertas

**Estado:** En progreso
**Fecha:** 2026-08-20

</div>

</div>

Preguntas y decisiones pendientes del proyecto, recopiladas de actas, arquitectura y metodología. Una pregunta pasa a estado RESUELTA cuando se registra la decisión (preferentemente mediante ADR en `../05-architecture/decisions/`).

## Abiertas

| # | Pregunta | Origen | Responsable sugerido |
|---|----------|--------|---------------------|
| Q-001 | ¿Qué proveedor de nube se usa (AWS/Azure) o se despliega en infraestructura local? | Acta kick-off 2026-02-10 | Líder Técnico |
| Q-003 | ¿Se designa un Product Owner formal? | Metodología adoptada | Líder Técnico |
| Q-006 | ¿Cuál es la política de retención de datos al eliminar cuenta? | Funcionalidades del sistema | Líder Técnico |
| Q-007 | ¿Qué formato tiene el payload de sincronización offline (JSON + multimedia)? | Arquitectura, sección 11 | Arquitectura |
| Q-008 | ¿Qué canal usa la notificación push (FCM/APNs) y cómo se maneja el estado de lectura? | Arquitectura, sección 12 | Arquitectura |
| Q-009 | ¿Cuál es la cobertura objetivo de pruebas y la herramienta de reportes (JaCoCo)? | Estrategia de pruebas | Líder Técnico |
| Q-010 | ¿El despliegue productivo usa contenedores (Docker) y orquestación? (Docker Compose local ya está definido en `10-devops/local-setup.md`) | Estrategia CI/CD | Líder Técnico |
| Q-012 | ¿Cuándo se incorporan las pruebas automatizadas al DoD formal? | Metodología adoptada | Equipo |
| Q-013 | ¿Los estados de negocio (dispositivo, evento) usan catálogo parametrizable genérico o `VARCHAR + CHECK`? | Convenciones de modelado | Arquitectura de Datos |
| Q-015 | ¿Qué modelo externo alimenta el resumen IA de `analytics` (costo/latencia)? | Módulo analítico | Arquitectura |
| Q-016 | ¿Qué stack de observabilidad se adopta (logs, métricas, trazas, alertas)? | Operaciones | Líder Técnico |
| Q-017 | ¿Valores definitivos de RTO/RPO y configuración de PITR para producción? | Respaldo y recuperación | Líder Técnico |

## Resueltas

| # | Pregunta | Decisión | Referencia | Fecha |
|---|----------|----------|------------|-------|
| QR-001 | ¿Se orienta el backend a microservicios? | No; monolito modular | ADR de arquitectura, `../05-architecture/architecture-document.md` | 2026-05-14 |
| QR-002 | ¿Qué lenguaje usa el backend? | Java 21 (Spring Boot 4.1.1, Maven) | [ADR-001](../05-architecture/decisions/records/ADR-001-backend-java-spring-boot.md) | 2026-08-16 |
| QR-003 | ¿Qué mecanismo de autenticación usa la API? | JWT RS256 + RBAC, con renovación de tokens | [authentication.md](../07-api-design/authentication.md) | 2026-08-19 |
| QR-004 | ¿Cómo se autentica el dispositivo contra la API? | API key por dispositivo, con hash y rotación | [authentication.md](../07-api-design/authentication.md) | 2026-08-19 |
| QR-005 | ¿Qué proveedor de almacenamiento multimedia se usa? | MinIO (S3-compatible) | [dependencies.md](./dependencies.md), [backup-and-recovery.md](../13-operations/backup-and-recovery.md) | 2026-08-19 |
| QR-006 | ¿Están aprobados los módulos base del monolito (security, parameterization, device-management, telemetry-service, monitoring)? | Sí; aprobados como parte de la arquitectura vigente del monolito modular, junto con analytics (ADR-003) | [ADR-001](../05-architecture/decisions/records/ADR-001-backend-java-spring-boot.md), [ADR-002](../05-architecture/decisions/records/ADR-002-hexagonal-architecture.md), [catálogo de módulos](../09-modules/module-catalog.md) | 2026-08-20 |
| QR-007 | ¿Cuál es el canal oficial de coordinación (Slack/Teams/otro)? | Discord | [adopted-methodology.md](../00-documentation-governance/adopted-methodology.md) §12 | 2026-08-20 |
| QR-011 | ¿Cómo se versiona y documenta la API en vivo (OpenAPI/Swagger)? | SpringDoc/OpenAPI (`/swagger-ui.html`) | [api-design.md](../07-api-design/api-design.md) | 2026-08-19 |