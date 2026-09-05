# Changelog

## [Unreleased]

### Fixed (2026-09-05)
- **Modelo 06 a DDL real:** `device_config` un solo `version`, `status` en `user`, `is_active` en `role`/`evidence`/`audit_login`, `event` transaccional (no append-only), `evidence` 1:1 sin `checksum` (integridad v1 `size_bytes` + ETag), Resumen 31 tablas, orden real de migración, `.mmd` sin `file_url` y con `device_id`, `v_metrics_daily` vía `device_assignment`, `is_active`/FK matizados.
- **Auth/UML a código y api-design:** `token_hash` SHA-256, `outcome` de catálogo, expiración reset 1h, `created_by` NULL, features en minúscula, `POST` notificaciones-read y `POST /analytics/reports` propuesta, `PUT→PATCH` catálogos, `es-event` vía `ANALYZED`, `audit-login`/`users`/`role-features`/`/api/v1`/dueño config en `dependency-map`, envelope de errores y rate-limits con cifras, `Idempotency-Key` en api-design.
- **Eventos y features:** consumidores a módulos reales (sin `audit-service`/Kafka/`DriverCreated`, CloudEvents como transporte futuro), `features-analysis` 23 propuestas vs 17 implementadas, `event-catalog` con HUs/RFs reales.
- **Trazabilidad y gobierno:** `RF-DEV-06/07` a su sección, SP 65/44 sin doble conteo, ejemplos con IDs reales, `es-device`/`unassign` pendientes en seeds documentados, `notification_delivery` futura, Q-007→QR-017 (libera Q-008), TD-004/005/006 cerrados con fecha, observabilidad/onboarding/dependencies/SP/stream/scope/backlog actualizados, pdf duplicado exacto eliminado.
- **Q/TD:** cerradas Q-007 (QR-017 payload), Q-013 (QR-013 estados), Q-016 (QR-016 observabilidad); TD-004, TD-005 y TD-006 resueltos.
- **Contratos device-API:** `POST /devices/{id}/heartbeat` (saludo, `ASSIGNED->ACTIVE`) y `PATCH /devices/{id}/rotate-key` definidos en `api-design.md`, `authentication.md`, `HU-API-006` (RF-DEV-09); sync two-phase (`POST /telemetry/events` JSON + `POST /events/{id}/evidence` multipart, base64 descartado); idempotencia `201 {acked_ids, duplicate_ids}` (`409` solo evidencia); `event_type/severity` por código; UML (`es-device`, `sd-offline-sync`, `sd-device-registration`) alineado.
- **Endpoints security alineados a HU:** `forgot-password`/`reset-password`, `register`, `verify-email`, CRUD roles/features, `POST /role-features`, `PATCH /users/{id}` + `PATCH /users/me`; `GET /audit-login`; prefijo `/api/v1` en `dependency-map.md`.
- **IDs y trazabilidad:** duplicados `RF-DEV-06/07/08/09` y `HU-PORTAL-004` resueltos (video → `HU-PORTAL-005`); conteo real 26 MVP + 4 post-MVP (209 SP); tabla RF→HU de `functional.md` remapeada a HUs existentes; equivalencia `RN-01..10 ↔ RN-XXX` en `entities-and-rules.md`; estados a `En progreso`/`Pendiente` según gobernanza.
- **Links rotos:** `../cross-cutting.md` → `../../` en 7 ADRs; `event-catalog.md`, `module-catalog.md`, `data-dictionary.md`, logos UML, deprecado `03-data-dictionary.md` → `data-dictionary.md`; `decisions/README.md` con ADR-001..009.
- **Q/TD:** cerradas Q-007 (QR-017 payload), Q-013 (QR-013 estados), Q-016 (QR-016 observabilidad); TD-004 y TD-006 resueltos (TD-005 con semillas verificadas).

### Changed (2026-09-04)
- **Rename:** `docs/04-requeriments/` → `docs/04-requirements/` (typo) con 25 referencias actualizadas.
- **Licencia:** `LICENSE` MIT 2026 SomnGuard (estaba pendiente).

### Fixed (2026-08-20)
- **Acta de kick-off fiel al stack original (C#/.NET)** con nota histórica de migración a Java (ADR-001); fecha de última actualización corregida.
- **Paquetes Java en snake_case** (`com.somnguard.telemetry_service`, `com.somnguard.device_management`) en arquitectura, diseño, ADR-002, análisis y onboarding, con nota de convención (módulos kebab-case en catálogo).
- **Reglas y gobernanza alineadas:** título de `structure-rules.md`; fuentes UML en Mermaid (`.mmd`, no `.wsd/.puml`); exportación estándar PNG; registro de ADR consolidado en `decisions/README.md` (sin tabla duplicada en `records/`); plantilla de PR con ramas `feat/*`; scopes de ejemplo con carpetas reales; formato HU unificado a `HU-<REPO>-NNN`; formato de eventos de dominio a `<entidad>.<accion>`.
- **Registro de decisiones:** QR-006 (módulos base aprobados) y QR-007 (canal de coordinación: Discord).
- **Actas de sprint con notas históricas:** nomenclatura HU anterior, repos "Cross/Back/BD" como nombres de trabajo, Sprint #1 de 18 días, Sprint #2 pendiente de registrar.
- **Contenido alineado:** estados de notificación (emitida, leída) según modelo de datos; épica "Seguridad y cuentas" en Pendiente (TD-002); resumen IA como funcionalidad de menor prioridad (se entrega al final del MVP); umbrales de alerta unificados (error rate 1%/5 min, p95 [200ms]/10 min) con notas de contexto para SLO y gates de deploy/rollback; referencia de autenticación corregida (ADR-001 y QR-003); diagrama de dependencias con aristas `SEC → DEV/MON/ANL`; contexto de `domain-map.md` completado.
- **Referencias y archivos:** enlaces de CHANGELOG a nombres reales; `08-uml/README.md` creado y enlazado; plantilla general movida a `99-archive/deprecated/templates/`; fecha del README de archivo alineada.
- **Correcciones de auditoría:** enlace OpenAPI corregido en `07-api-design/guidelines.md` y carpeta `contracts/openapi/` creada (placeholder de TD-003); umbral de error rate del runbook de módulo alineado a 1%/5 min; `cd-domain.mmd` alineado con el modelo relacional (cardinalidad de `password_reset_request` y relaciones vía `event_type`); ejemplos de nombres de archivo en inglés en `project-regulations.md`; formato de exportación UML unificado a PNG en `documentation-rules.md`; acta de kick-off refiere QR-007 (Discord); tabla duplicada eliminada de `records/README.md`; referencia de registro de ADR corregida en `CONTRIBUTING.md`.

### Added (2026-08-19)
- **Estructura documental ampliada con gobernanza y documentos de proceso:** se incorporaron documentos de proceso y gobernanza alineados con el monolito modular hexagonal y las convenciones del proyecto:
  - Raíz: `CONTRIBUTING.md`, `.github/pull_request_template.md`.
  - `00-documentation-governance`: `agile-conventions.md` (convenciones HU/AC/TC/RN/NFR/ADR con compatibilidad F-/RB-) y `security-policy.md`.
  - `01-project-context`: `overview.md`, `project-profile.md` y `scope-declaration.md`.
  - `02-domain`: `domain-map.md`, `entities-and-rules.md` (RN-01..RN-10 ↔ RB-01..RB-10) y `domain-events.md`.
  - `03-product-definition`: `product-backlog.md` (épicas desde F-01..F-10) y `_template-backlog.md`.
  - `04-requirements`: `non-functional.md` (NFR-01..NFR-08), `traceability-matrix.md` y `_template-hu.md`.
  - `05-architecture`: `security-threat-model.md` (STRIDE), `pattern-guide.md` y `_template-adr.md`.
  - `06-data-architecture`: `modeling-conventions.md` (auditoría, estados, orden DDL Liquibase).
  - `07-api-design`: `guidelines.md`, `authentication.md` (JWT/RBAC/API keys) y `contracts/` (destino de OpenAPI por módulo).
  - `09-modules`: plantilla de módulo completa en `modules/_template/` (README, data-model, events, decisions, runbook) adaptada a monolito hexagonal.
  - `10-devops`: `local-setup.md`, `environments.md` y 3 plantillas (despliegue, release, rollback); `ci-cd-strategy.md` enriquecido.
  - `11-quality-assurance`: `code-review.md` y plantillas de QA report y test evidence; `test-strategy.md` con identificación de casos `TC-`.
  - `12-user-experience`: 3 plantillas (UX flows, design system, UI spec).
  - `13-operations`: `observability.md`, `incident-management.md`, `backup-and-recovery.md` y 4 plantillas (observabilidad, SLA/SLO/SLI, runbook, postmortem).
  - `14-training-and-adoption`: `technical-onboarding.md`.
  - `15-project-control`: `risks.md`, `dependencies.md`, `technical-backlog.md`, `_template-sprint-plan.md` y preguntas abiertas ampliadas (Q-013, Q-015..Q-017).
  - READMEs de sección actualizados con el inventario de archivos.
- **Estructura del backend Java:** `somnguard-api/backend-java/` (repositorio aparte, aún en borrador) con el árbol de carpetas completo del backend hexagonal (solo estructura, con `.gitkeep`), alineado a docs y ADRs: 6 módulos (security, parameterization, device-management, telemetry-service, monitoring, analytics) × `application/port/{in,out}`, `application/usecase`, `domain/{model,service}`, `adapter/in/{web,amqp}`, `adapter/out/{persistence,storage}`, más `platform/{error-handling,logging,observability}`.
- **Diagramas UML:** 8 diagramas de secuencia en `docs/08-uml/diagrams/source/` (detección y alerta, sincronización offline, autenticación, restablecimiento de contraseña, alta de dispositivo, consulta de eventos, notificación crítica y generación de reportes) con sus exportaciones en `exports/`; 1 diagrama de clases de dominio (`cd-domain.mmd`).
- **Propuesta técnica:** `docs/01-project-context/software-technical-proposal.md` — solución propuesta, arquitectura, stack tecnológico, diseño, metodología, plan de trabajo, recursos, costos referenciales, riesgos y entregables.
- **Análisis del software:** `docs/04-requirements/software-analysis.md` — modelo de dominio, casos de uso, vistas estáticas y vistas dinámicas (secuencia, actividades y estados) con trazabilidad funcionalidad → caso de uso → módulo → pruebas.
  - Nuevos diagramas en `docs/08-uml/diagrams/source/`: 4 de actividades (`ac-*.mmd`) y 3 de estados (`es-*.mmd`), con sus exportaciones en `exports/`.

### Changed (2026-08-19)
- **Arquitectura hexagonal formalizada:** la arquitectura limpia del backend se formaliza como hexagonal (puertos y adaptadores) en `architecture-document.md` (sección 8) y `software-design-report.md` (sección 3.2.1 y 8), con la estructura de paquetes `com.somnguard.<modulo>` (`application/port/{in,out}`, `application/usecase`, `domain/{model,service}`, `adapter/in/{web,amqp}`, `adapter/out/{persistence,storage}`, `platform`):
  - Nuevas ADRs: [ADR-002](docs/05-architecture/decisions/records/ADR-002-hexagonal-architecture.md) (arquitectura hexagonal) y [ADR-003](docs/05-architecture/decisions/records/ADR-003-analytics-module.md) (módulo analítico).
  - Módulos alineados en `architecture-document.md` (security, parameterization, device-management, telemetry-service, monitoring, analytics) y entidades actualizadas a las 20 del modelo vigente.
  - `docs/09-modules/module-catalog.md` actualizado: nueva estructura por módulo y registro de `analytics`.
- **Nombres de archivos en inglés:** todos los archivos y carpetas vigentes se renombraron a inglés (kebab-case) conservando el contenido en español: documentos (`documento-arquitectura` → `architecture-document`, `informe-diseno-software` → `software-design-report`, `propuesta-tecnica-software` → `software-technical-proposal`, `analisis-software` → `software-analysis`, `normativa-del-proyecto` → `project-regulations`, `metodologia-adoptada` → `adopted-methodology`, `glosario` → `glossary`, etc.), diagramas (`sd-*`, `ac-*`, `es-*`, `cd-*`, `cu-*`) y carpetas (`01-anteproyecto` → `01-project-proposal`, `02-cronograma` → `02-schedule`, `01-mapa-de-procesos` → `01-process-map`, `01-investigacion` → `01-research`, `01-actas` → `01-meeting-minutes`) junto con sus archivos binarios. Se actualizaron todas las referencias cruzadas.

### Changed (2026-08-16)
- **Unificación de estructuras:** se consolidaron las estructuras anteriores en una sola, organizada por secciones numeradas (`docs/NN-seccion`), manteniendo la gobernanza documental y sin orientación a microservicios:
  - La sección `09-microservices` se renombró a `09-modules` (monolito modular) con su catálogo de módulos.
  - Se corrigieron los nombres `06-data-arquitecture` → `06-data-architecture` y `07-api-desing` → `07-api-design`.
  - Se incorporaron los documentos actualizados del modelo de datos (`01-entity-relationship-model.mmd` y `02-modules-entities.md`) como fuente de verdad en `06-data-architecture/`.
  - La documentación anterior por fases (00-08) se conservó completa en `docs/99-archive/deprecated/previous-structure/`.
  - La primera versión del modelo de datos (ER conceptual, ER relacional y diccionario) se archivó en `docs/99-archive/deprecated/data-model-v1/`.
  - Se estandarizaron los encabezados de los documentos migrados y se actualizaron referencias de rutas en normativa, metodología, arquitectura y actas.
- **Migración del backend a Java:** se actualizó toda la documentación vigente de C#/.NET a **Java 21 + Spring Boot 4.1.1 (Maven)**:
  - Documento de arquitectura, normativa (convenciones de lenguaje), metodología, reglas de estructura, catálogo de módulos y acta de kick-off.
  - Se registró la decisión como [ADR-001](docs/05-architecture/decisions/records/ADR-001-backend-java-spring-boot.md).
  - El resto del stack (Python, PostgreSQL, Liquibase, React JS, React Native, Raspberry Pi) se mantiene sin cambios.

### Removed (2026-08-16)
- Se eliminaron las antiguas carpetas de documentación numeradas de nivel raíz tras archivarlas.

### Added (2026-08-16)
- **Nuevos documentos:**
  - `docs/05-architecture/decisions/records/ADR-001-backend-java-spring-boot.md` — decisión del stack del backend.
  - `docs/06-data-architecture/02-relational-model.mmd` — esquema relacional implementable del modelo vigente.
  - `docs/06-data-architecture/03-data-dictionary.md` — diccionario de datos del modelo vigente.
  - `docs/07-api-design/api-design.md` — propuesta inicial de diseño de la API REST.
  - `docs/10-devops/ci-cd-strategy.md` — estrategia de CI/CD con GitHub Actions.
  - `docs/11-quality-assurance/test-strategy.md` — estrategia de pruebas por nivel.
  - `docs/02-domain/glossary.md` — glosario del dominio, técnico y de proceso.
  - `docs/15-project-control/open-questions.md` — preguntas abiertas y decisiones resueltas.
  - `docs/05-architecture/software-design-report.md` — informe de diseño de software (modelo arquitectónico, componentes, modelo de datos, interfaces, patrones de diseño, reglas de negocio, seguridad y especificaciones técnicas).

