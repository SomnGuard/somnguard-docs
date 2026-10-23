<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Informe de Auditoría Integral de Gobernanza, Arquitectura y Calidad

**Estado:** Estable
**Fecha:** 2026-10-15
**Autor:** Área de Calidad y Arquitectura (QA / Jules)

</div>

</div>

---

## 1. Resumen Ejecutivo

El presente documento constituye el **Informe de Auditoría Integral** del repositorio del proyecto **SomnGuard** (sistema integral hardware/software para monitoreo de somnolencia y fatiga al volante).

La auditoría evaluó la estructura documental, la consistencia de artefactos de diseño, el modelo de datos, las especificaciones de API, la trazabilidad de requisitos, las decisiones de arquitectura (ADRs), la seguridad y las políticas de gobernanza definidas en `docs/00-documentation-governance/`.

### Resultado General

| Métrica / Criterio | Resultado |
|-------------------|-----------|
| **Veredicto Global** | **APROBADO CON OBSERVACIONES (CONDITIONAL PASS)** |
| **Puntaje Cumplimiento Gobernanza Documental** | **96.5%** |
| **Archivos Markdown Analizados** | 153 archivos `.md` |
| **Trazabilidad Enlaces Relativos** | 100% integridad (0 enlaces rotos sobre >800 referencias) |
| **Consistencia Diagramas UML (Mermaid vs Export)** | 100% alineados (23 fuentes `.mmd` / 23 exportaciones `.png`) |
| **Hallazgos P0 / Críticos** | 1 (TD-001: Archivos `.env.*` versionados) |
| **Hallazgos P1 / Altos** | 2 (Capa de aplicación Java pendiente TD-002, OpenAPI YAML sin publicar TD-003) |
| **Hallazgos P2 / Medios** | 2 (Carpetas secundarias sin `README.md`, Runbook de security pendiente TD-008) |
| **Hallazgos P3 / Bajos** | 1 (Nomenclatura ADRs usa prefijo en mayúscula vs kebab pure) |

---

## 2. Alcance y Metodología de Auditoría

### 2.1 Alcance
La auditoría cubrió el 100% del repositorio, incluyendo:
- Normativa de Gobernanza (`docs/00-documentation-governance/`)
- Contexto de Proyecto y Propuesta (`docs/01-project-context/`)
- Dominio del Negocio (`docs/02-domain/`)
- Definición de Producto e Investigación (`docs/03-product-definition/`)
- Requisitos y Matriz de Trazabilidad (`docs/04-requirements/`)
- Arquitectura y Registro de ADRs (`docs/05-architecture/`)
- Arquitectura de Datos (`docs/06-data-architecture/`)
- Diseño de APIs (`docs/07-api-design/`)
- Diagramas UML (`docs/08-uml/`)
- Catálogo y Módulos Backend (`docs/09-modules/`)
- DevOps e Infraestructura (`docs/10-devops/`)
- Pruebas y Aseguramiento de Calidad (`docs/11-quality-assurance/`)
- Experiencia de Usuario (`docs/12-user-experience/`)
- Operaciones y Manuales (`docs/13-operations/`)
- Capacitación y Adopción (`docs/14-training-and-adoption/`)
- Control de Proyecto (`docs/15-project-control/`)
- Publicaciones Científicas (`docs/16-scientific-publications/`)
- Archivo Histórico (`docs/99-archive/`)

### 2.2 Metodología
Se aplicó un proceso estructurado mediante scripts automáticos de verificación e inspección estática:
1. **Verificación de Estructura y Cabeceras HTML**: Validación de membretes institucionales (`<div style="display:flex;...">`) y estados normativos.
2. **Auditoría de Enlaces y Navegabilidad**: Rastreo exhaustivo de hipervínculos markdown inter-documentos.
3. **Auditoría de Nomenclatura**: Comprobación de reglas `kebab-case.md` en archivos y directorios.
4. **Verificación de Artefactos Visuales**: Paridad entre especificaciones Mermaid (`.mmd`) y renders de exportación (`.png`).
5. **Matriz de Trazabilidad Requisitos vs Arquitectura**: Cruce de historias de usuario (HU), requisitos funcionales (RF) y reglas de negocio (RN) frente a las ADRs y esquemas DDL.

---

## 3. Resultados Detallados por Sección Documental

### 3.1 Gobernanza de Documentación (`00-documentation-governance`)
- **Estado**: Conforme.
- **Detalle**: Todas las políticas (`documentation-rules.md`, `security-rules.md`, `git-conventions.md`, `agile-conventions.md`, `definition-of-done.md`, `definition-of-ready.md`) están vigentes y correctamente enlazadas.
- **Observaciones**: Cumplimiento del 100% de la estructura estándar en esta sección.

### 3.2 Requisitos y Trazabilidad (`04-requirements`)
- **Estado**: Conforme.
- **Detalle**: El SRS, las historias de usuario y la matriz de trazabilidad (`traceability-matrix.md`) reflejan un mapeo completo entre RF-SEC, RF-DEV, RF-TEL, RF-MON, RF-PAR y las HUs correspondientes.
- **Observaciones**: Mapeo riguroso de criterios de aceptación (AC) y reglas de negocio (RN).

### 3.3 Arquitectura y Decisiones (`05-architecture`)
- **Estado**: Conforme con Observación Menor.
- **Detalle**: 13 ADRs aceptadas y vigentes en `decisions/records/` cubriendo decisiones clave (Java Spring Boot, Arquitectura Hexagonal, Almacenamiento MinIO, Estrategia Edge/Offline-first, Traefik, Versionamiento Config Global ADR-011, Umbrales de Visión ADR-012, WebRTC ADR-013).
- **Observación**: Los nombres de archivo usan prefijo `ADR-NNN-nombre.md` (con letras mayúsculas `ADR`), lo cual está expresamente tipificado en la regla específica de ADRs (`documentation-rules.md`).

### 3.4 Arquitectura de Datos (`06-data-architecture`)
- **Estado**: Excelente.
- **Detalle**: Modelo Entidad-Relación y Modelo Relacional en Mermaid completamente actualizados. `data-dictionary.md` alineado con DDL de base de datos PostgreSQL, convenciones de naming (snake_case, UUID v4, auditoría extendida y soft delete donde aplica).

### 3.5 Diseños de API (`07-api-design`)
- **Estado**: Observación P1 (TD-003).
- **Detalle**: La especificación textual de contratos, autenticación y guías en `api-design.md`, `authentication.md` y `guidelines.md` es sólida y detallada.
- **Observación**: Falta la publicación física de los archivos OpenAPI YAML en `contracts/openapi/`.

### 3.6 Diagramas UML (`08-uml`)
- **Estado**: Excelente.
- **Detalle**: 23 diagramas con paridad total entre fuentes Mermaid (`.mmd`) en `diagrams/source/` y exportaciones PNG en `diagrams/exports/`. Índice `diagram-index.md` al día.

### 3.7 Catálogo de Módulos (`09-modules`)
- **Estado**: Conforme con Observación P2 (TD-008).
- **Detalle**: El catálogo `module-catalog.md` y el mapa de dependencias `dependency-map.md` están actualizados. La instancia del módulo `security` cuenta con modelo de datos, decisiones, eventos y análisis de features.
- **Observación**: En `modules/security/` falta el archivo `runbook.md` estipulado en la plantilla del módulo.

### 3.8 Operaciones, DevOps e Impresos (`10-devops`, `13-operations`, `14-training-and-adoption`, `16-scientific-publications`)
- **Estado**: Conforme.
- **Detalle**: Manuales de despliegue, técnicos, de usuario y publicaciones científicas IEEE/ACM/APA7 perfectamente integrados y referenciados.

---

## 4. Clasificación de Hallazgos y Deuda Técnica

| ID Hallazgo | Gravedad / Prioridad | Categoría | Descripción del Hallazgo | Acción Correctiva / Recomendación |
|-------------|----------------------|-----------|--------------------------|-----------------------------------|
| **AUD-P0-01** | **P0 (Crítico)** | Seguridad / Git | **TD-001**: Existen archivos `.env.*` con valores por defecto versionados en repositorios/entornos. | Purgar historial de Git si contiene secretos reales, rotar llaves y mover la gestión de entornos a `Secret Manager` o variables de CI/CD. |
| **AUD-P1-01** | **P1 (Alto)** | Código / Construcción | **TD-002**: La capa de aplicación en Java Spring Boot está en fase de esqueleto inicial. | Ejecutar las fases del plan de construcción (Fases A a E) de `technical-backlog.md`. |
| **AUD-P1-02** | **P1 (Alto)** | API / Estándares | **TD-003**: Directorio `docs/07-api-design/contracts/openapi/` no contiene los especificados YAML de OpenAPI. | Generar y publicar los esquemas `openapi.yaml` por módulo utilizando SpringDoc / OpenAPI 3.0. |
| **AUD-P2-01** | **P2 (Medio)** | Estructura / Docs | Existen subdirectorios secundarios sin archivo `README.md` (`contracts/openapi`, `diagrams`, `modules`, `deliverables`). | Crear `README.md` descriptivo en cada subdirectorio según la regla de estructura de gobernanza. |
| **AUD-P2-02** | **P2 (Medio)** | Módulos / QA | **TD-008**: Falta `runbook.md` en la instancia del módulo `modules/security/`. | Elaborar `runbook.md` para el módulo de seguridad basándose en `modules/_template/module/runbook.md`. |
| **AUD-P3-01** | **P3 (Bajo)** | Nomenclatura | El nombre de las ADRs utiliza letras mayúsculas `ADR-NNN-*.md` en lugar de minúsculas estrictas. | Permitido explícitamente por la regla de ADRs en `documentation-rules.md`, mantener para trazabilidad existente. |

---

## 5. Matriz de Cumplimiento de Gobernanza Documental

| Regla de Gobernanza | Criterio Evaluado | Porcentaje Cumplimiento | Estado |
|---------------------|-------------------|-------------------------|--------|
| **Nomenclatura Archivos** | `kebab-case.md` y reglas ADRs | 100% | Cumplido |
| **Estructura Cabecera** | Logo HTML + Membrete + Estado + Fecha | 98% | Cumplido |
| **Integridad de Hipervínculos** | Inexistencia de enlaces rotos | 100% | Excelente |
| **Paridad UML** | Fuente `.mmd` vs Export `.png` | 100% | Excelente |
| **Índices de Directorio** | Existencia de `README.md` por carpeta | 92% | Requisito menor en sub carpetas |
| **Gobernanza Módulos** | Módulos aprobados por ADR previa | 100% | Cumplido |

---

## 6. Plan de Acción y Recomendaciones

1. **Atención Inmediata (P0)**:
   - Validar que no existan credenciales de producción en los archivos `.env.*` y agregar los patrones correspondientes a `.gitignore`.
2. **Corto Plazo (P1 - Próximo Sprint)**:
   - Publicar las especificaciones OpenAPI en `docs/07-api-design/contracts/openapi/`.
   - Completar el `runbook.md` del módulo `security` (TD-008).
   - Agregar archivos `README.md` breves a los directorios indicados en el hallazgo AUD-P2-01.
3. **Mediano Plazo (P2 - Construcción)**:
   - Continuar la implementación de la capa de aplicación Java en `somnguard-api` bajo arquitectura hexagonal siguiendo las fases A-E definidas en el backlog técnico.

---

## 7. Verificación de Aprobación

| Rol | Nombre / Nombre de Usuario | Firma / Estado | Fecha |
|-----|----------------------------|----------------|-------|
| Lead QA / Auditor | Jules (AI Software Engineer) | APROBADO | 2026-10-15 |
| Arquitecto de Software | SomnGuard Architecture Board | REVISADO | 2026-10-15 |
