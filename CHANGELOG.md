# Changelog

## [Unreleased]

### Changed
- **Arquitectura hexagonal formalizada (2026-08-19):** la arquitectura limpia del backend se formaliza como hexagonal (puertos y adaptadores) en `architecture-document.md` (sección 8) y `software-design-report.md` (sección 3.2.1 y 8), con la estructura de paquetes `com.somnguard.<modulo>` (`application/port/{in,out}`, `application/usecase`, `domain/{model,service}`, `adapter/in/{web,amqp}`, `adapter/out/{persistence,storage}`, `platform`):
  - Nuevas ADRs: [ADR-002](docs/05-architecture/decisions/records/ADR-002-hexagonal-architecture.md) (arquitectura hexagonal) y [ADR-003](docs/05-architecture/decisions/records/ADR-003-analytics-module.md) (módulo analítico).
  - Módulos alineados en `architecture-document.md` (security, parameterization, device-management, telemetry-service, monitoring, analytics) y entidades actualizadas a las 20 del modelo vigente.
  - `docs/09-modules/module-catalog.md` actualizado: nueva estructura por módulo y registro de `analytics`.

### Added
- **Estructura del backend Java (2026-08-19):** `somnguard-api/backend-java/` con el árbol de carpetas completo del backend hexagonal (solo estructura, con `.gitkeep`), alineado a docs y ADRs: 6 módulos (security, parameterization, device-management, telemetry-service, monitoring, analytics) × `application/port/{in,out}`, `application/usecase`, `domain/{model,service}`, `adapter/in/{web,amqp}`, `adapter/out/{persistence,storage}`, más `platform/{error-handling,logging,observability}`. El backend .NET existente se conserva intacto.

## [Unreleased] (2026-08-19)

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
  - `docs/06-data-architecture/02-relational-model.mmd` — esquema relacional implementable del modelo vigente.
  - `docs/06-data-architecture/03-data-dictionary.md` — diccionario de datos del modelo vigente.
  - `docs/07-api-design/api-design.md` — propuesta inicial de diseño de la API REST.
  - `docs/10-devops/ci-cd-strategy.md` — estrategia de CI/CD con GitHub Actions.
  - `docs/11-quality-assurance/test-strategy.md` — estrategia de pruebas por nivel.
  - `docs/02-domain/glossary.md` — glosario del dominio, técnico y de proceso.
  - `docs/15-project-control/open-questions.md` — preguntas abiertas y decisiones resueltas.
  - `docs/05-architecture/software-design-report.md` — informe de diseño de software (modelo arquitectónico, componentes, modelo de datos, interfaces, patrones de diseño, reglas de negocio, seguridad y especificaciones técnicas).

### Added
- **Diagramas UML (2026-08-19):**
  - 8 diagramas de secuencia en `docs/08-uml/diagrams/source/` (detección y alerta, sincronización offline, autenticación, restablecimiento de contraseña, alta de dispositivo, consulta de eventos, notificación crítica y generación de reportes) con sus exportaciones en `exports/`.
  - 1 diagrama de clases de dominio (`cd-domain.mmd`) con su exportación.
  - Índice actualizado (`docs/08-uml/diagram-index.md`) con las secciones de secuencia y clases; informe de diseño enlazado a las fuentes editables.

### Added
- **Propuesta técnica (2026-08-19):** `docs/01-project-context/software-technical-proposal.md` — solución propuesta, arquitectura, stack tecnológico, diseño, metodología, plan de trabajo, recursos, costos referenciales, riesgos y entregables.

### Added
- **Análisis del software (2026-08-19):** `docs/04-requeriments/software-analysis.md` — modelo de dominio, casos de uso, vistas estáticas y vistas dinámicas (secuencia, actividades y estados) con trazabilidad funcionalidad → caso de uso → módulo → pruebas.
  - Nuevos diagramas en `docs/08-uml/diagrams/source/`: 4 de actividades (`ac-*.mmd`) y 3 de estados (`es-*.mmd`), con sus exportaciones en `exports/`.
  - Índice de diagramas actualizado con las secciones de actividades y estados.

### Changed
- **Nombres de archivos en inglés (2026-08-19):** todos los archivos y carpetas vigentes se renombraron a inglés (kebab-case) conservando el contenido en español: documentos (`documento-arquitectura` → `architecture-document`, `informe-diseno-software` → `software-design-report`, `propuesta-tecnica-software` → `software-technical-proposal`, `analisis-software` → `software-analysis`, `normativa-del-proyecto` → `project-regulations`, `metodologia-adoptada` → `adopted-methodology`, `glosario` → `glossary`, etc.), diagramas (`sd-*`, `ac-*`, `es-*`, `cd-*`, `cu-*`) y carpetas (`01-anteproyecto` → `01-project-proposal`, `02-cronograma` → `02-schedule`, `01-mapa-de-procesos` → `01-process-map`, `01-investigacion` → `01-research`, `01-actas` → `01-meeting-minutes`) junto con sus archivos binarios. Se actualizaron todas las referencias cruzadas.
