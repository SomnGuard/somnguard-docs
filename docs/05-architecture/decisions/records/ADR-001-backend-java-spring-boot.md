# ADR-001: Backend en Java Spring Boot

**Estado:** Aceptada
**Fecha:** 2026-08-16
**Autores:** Equipo SomnGuard
**Equipos involucrados:** Arquitectura, Desarrollo

---

## Contexto

El stack inicial del proyecto se definió en el kick-off (2026-02-10) con **C#/.NET** como lenguaje del backend. Durante el desarrollo se decidió reemplazar el backend por **Java con Spring Boot**, manteniendo el resto de la arquitectura (monolito modular, PostgreSQL, Liquibase, edge Raspberry Pi, React JS y React Native) sin cambios.

El backend es el componente central de la plataforma: centraliza autenticación, gestión de dispositivos, recepción de telemetría, alertas y parametrización, y es la única vía de integración con los clientes web y móvil.

## Decisión

Se decide implementar el backend de SomnGuard en **Java 21 (LTS) con Spring Boot 3.x y Maven**, organizado como monolito modular con principios de clean architecture.

Detalles de la decisión:

- **Lenguaje:** Java 21 (LTS), versión soportada de largo plazo.
- **Framework:** Spring Boot 3.x (Spring Web, Spring Security, Spring Data JPA, Spring Validation).
- **Build tool:** Maven (`pom.xml`).
- **Base de datos:** PostgreSQL, sin cambios.
- **Migraciones:** Liquibase, sin cambios.
- **Estructura de paquetes por módulo:** `interfaces`, `application`, `domain`, `infrastructure` (ver `../../architecture-document.md`, sección 8).
- **Autenticación:** a definir en una ADR posterior (ver `../../../15-project-control/open-questions.md`).

## Consecuencias

### Positivas

- Java LTS garantiza soporte prolongado y estabilidad.
- Spring Boot ofrece ecosistema maduro: seguridad, persistencia, validación, testing y observabilidad integrados.
- Gran disponibilidad de talento y documentación para Java/Spring.
- Maven es el estándar del ecosistema Spring y simplifica la configuración del build.
- Liquibase se mantiene, por lo que el versionado del esquema no cambia respecto al plan original.

### Negativas / Trade-offs

- La base de código en C#/.NET existente debe reescribirse o descartarse.
- Se pierde la integración con herramientas del ecosistema .NET ya configuradas.
- Documentación y plantillas previas que mencionaban .NET deben actualizarse.

### Riesgos

- Retraso por la migración de la base inicial del backend.
- Riesgo de mezclar convenciones del stack anterior en el código nuevo.
- Que la decisión se replantee a futuro si no se alinea con las necesidades del edge (Python) — mitigado por la separación de responsabilidades entre edge y servidor.

## Alternativas consideradas

| Alternativa | Por qué se descartó |
|-------------|---------------------|
| Mantener C#/.NET | Se decidió el cambio de stack por preferencia y disponibilidad del equipo con el ecosistema Java |
| Node.js (TypeScript) | No aporta beneficios claros sobre Spring Boot para el dominio (persistencia relacional, seguridad, madurez) |
| Python (FastAPI/Django) | Python se mantiene solo para el análisis visual en el edge; no se quería un lenguaje adicional en el servidor |

## Referencias

- Documento de arquitectura: `../../architecture-document.md`
- Acta kick-off: `../../../15-project-control/01-meeting-minutes/01_kickoff.md`
- Reglas de documentación de módulos: `../../../00-documentation-governance/structure-rules.md`
- Preguntas abiertas: `../../../15-project-control/open-questions.md`