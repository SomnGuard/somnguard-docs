# Changelog

## [Unreleased]

### Added
- Se añadió la nueva estructura de documentación base para SomnGuard.

- Se añadieron secciones de gobernanza, arquitectura, recursos y colaboración en el repositorio.

### Changed
- Se reorganizó la documentación anterior en la nueva estructura `docs/`.

- Se archivó la antigua estructura de documentación en `docs/99-archive/`.

- **Unificación de estructuras (2026-08-16):** se consolidaron las estructuras anteriores en una sola, organizada por secciones numeradas (`docs/NN-seccion`), manteniendo la gobernanza documental y sin orientación a microservicios:
  - La sección `09-microservices` se renombró a `09-modules` (monolito modular) con su catálogo de módulos.
  - Se corrigieron los nombres `06-data-arquitecture` → `06-data-architecture` y `07-api-desing` → `07-api-design`.
  - Se incorporaron los documentos actualizados del modelo de datos (`mer.mmd` y `modulos-entidades.md`) como fuente de verdad en `06-data-architecture/`.
  - La documentación anterior por fases (00-08) se conservó completa en `docs/99-archive/deprecated/previous-structure/`.
  - La primera versión del modelo de datos (ER conceptual, ER relacional y diccionario) se archivó en `docs/99-archive/deprecated/data-model-v1/`.
  - Se estandarizaron los encabezados de los documentos migrados y se actualizaron referencias de rutas en normativa, metodología, arquitectura y actas.

### Removed
- Se eliminaron las antiguas carpetas de documentación numeradas de nivel raíz tras archivarlas.

### Changed
- **Migración del backend a Java (2026-08-16):** se actualizó toda la documentación vigente de C#/.NET a **Java 21 + Spring Boot 3.x (Maven)**:
  - Documento de arquitectura, normativa (convenciones de lenguaje), metodología, reglas de estructura, catálogo de módulos y acta de kick-off.
  - Se registró la decisión como [ADR-001](docs/05-architecture/decisions/records/ADR-001-backend-java-spring-boot.md).
  - El resto del stack (Python, PostgreSQL, Liquibase, React JS, React Native, Raspberry Pi) se mantiene sin cambios.

### Added
- **Nuevos documentos (2026-08-16):**
  - `docs/05-architecture/decisions/records/ADR-001-backend-java-spring-boot.md` — decisión del stack del backend.
  - `docs/06-data-architecture/02-modelo-relacional.mmd` — esquema relacional implementable del modelo vigente.
  - `docs/06-data-architecture/03-diccionario-datos.md` — diccionario de datos del modelo vigente.
  - `docs/07-api-design/api-design.md` — propuesta inicial de diseño de la API REST.
  - `docs/10-devops/ci-cd-strategy.md` — estrategia de CI/CD con GitHub Actions.
  - `docs/11-quality-assurance/test-strategy.md` — estrategia de pruebas por nivel.
  - `docs/02-domain/glosario.md` — glosario del dominio, técnico y de proceso.
  - `docs/15-project-control/open-questions.md` — preguntas abiertas y decisiones resueltas.
