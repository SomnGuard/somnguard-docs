<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Preguntas abiertas

**Estado:** En progreso
**Fecha:** 2026-08-16

</div>

</div>

Preguntas y decisiones pendientes del proyecto, recopiladas de actas, arquitectura y metodología. Una pregunta pasa a estado RESUELTA cuando se registra la decisión (preferentemente mediante ADR en `05-architecture/decisions/`).

## Abiertas

| # | Pregunta | Origen | Responsable sugerido |
|---|----------|--------|---------------------|
| Q-001 | ¿Qué proveedor de nube se usa (AWS/Azure) o se despliega en infraestructura local? | Acta kick-off 2026-02-10 | Líder Técnico |
| Q-002 | ¿Cuál es el canal oficial de coordinación (Slack/Teams/otro)? | Acta kick-off 2026-02-10 | Equipo |
| Q-003 | ¿Se designa un Product Owner formal? | Metodología adoptada | Líder Técnico |
| Q-004 | ¿Qué mecanismo de autenticación usa la API (JWT propuesto)? | Diseño de API | Arquitectura |
| Q-005 | ¿Cómo se autentica el dispositivo contra la API (API key con hash ya modelado)? | Modelo de datos | Arquitectura |
| Q-006 | ¿Cuál es la política de retención de datos al eliminar cuenta? | Funcionalidades del sistema | Líder Técnico |
| Q-007 | ¿Qué formato tiene el payload de sincronización offline (JSON + multimedia)? | Arquitectura, sección 11 | Arquitectura |
| Q-008 | ¿Qué canal usa la notificación push (FCM/APNs) y cómo se maneja el estado de lectura? | Arquitectura, sección 12 | Arquitectura |
| Q-009 | ¿Cuál es la cobertura objetivo de pruebas y la herramienta de reportes (JaCoCo)? | Estrategia de pruebas | Líder Técnico |
| Q-010 | ¿El despliegue usa contenedores (Docker) y orquestación? | Estrategia CI/CD | Líder Técnico |
| Q-011 | ¿Cómo se versiona y documenta la API en vivo (OpenAPI/Swagger)? | Diseño de API | Arquitectura |
| Q-012 | ¿Cuándo se incorporan las pruebas automatizadas al DoD formal? | Metodología adoptada | Equipo |

## Resueltas

| # | Pregunta | Decisión | Referencia | Fecha |
|---|----------|----------|------------|-------|
| R-001 | ¿Se orienta el backend a microservicios? | No; monolito modular | ADR de arquitectura, `documento-arquitectura.md` | 2026-05-14 |
| R-002 | ¿Qué lenguaje usa el backend? | Java 21 (Spring Boot 3.x, Maven) | [ADR-001](../05-architecture/decisions/records/ADR-001-backend-java-spring-boot.md) | 2026-08-16 |