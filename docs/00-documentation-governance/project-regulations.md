<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Normativa del Proyecto

**Autor:** Equipo SomnGuard
**Estado:** Estable
**Fecha:** 2026-08-19
**Última actualización:** 2026-08-19

</div>

</div>

---

## 1. Objetivo

Este documento establece las normas, políticas y estándares que regirán la elaboración, revisión, aprobación y publicación de la documentación, código y artefactos del proyecto SomnGuard.

## 2. Alcance

Aplica a todo el equipo del proyecto:
- Documentación técnica y funcional
- Código fuente y configuraciones
- Ramas de control de versiones (Git)
- Entregables y artefactos de despliegue

## 3. Convenciones generales

### Lenguaje y formato

- **Documentación:** Español neutro. El contenido documental se redacta en español, evitando jerga local y explicando tecnicismos cuando sea necesario.
- **Código y commits:** Inglés. Los nombres de ramas, mensajes de commit, identificadores técnicos y comentarios de código se escriben en inglés.
- **Documentación:** Preferentemente en Markdown (`.md`). Cuando se necesite una versión de consulta sin vista previa, también puede exportarse a PDF; si se requiere un formato editable adicional, se admiten archivos tipo documento como `.docx`.
- **Diagramas:** Fuentes editables en Mermaid (`.mmd`) con convenciones UML, y exportación a `.png` para visualización o entrega.
- **Codificación:** UTF-8 sin BOM, salvo que una herramienta o entorno específico indique otra necesidad.
- **Fechas:** Formato `YYYY-MM-DD`

## 4. Nomenclatura y estilos

### Archivos y carpetas

- **Nombres de archivos:** `kebab-case`, minúsculas y guiones para separar palabras  
  Ejemplo: `test-plan.md`, `initial-architecture.md`

### Código

- **Lenguaje principal:** Java 21 (Spring Boot 4.1.1)
- **Java:** `camelCase` para variables y métodos; `PascalCase` para clases, interfaces y tipos
- **Constantes:** MAYÚSCULAS_CON_GUIONES

### Documentación

- **Títulos:** H1 para documentos principales, H2/H3 para secciones
- **Listas:** numeradas para procesos, viñetas para atributos
- **Énfasis:** **negrita** para términos clave, `código` para referencias técnicas

### Diagramas

- Los diagramas usan convenciones estándar (UML) y se crean en Mermaid (`.mmd`) con exportación `.png` (ver `documentation-rules.md`).

## 5. Estructura documental

La documentación se organiza en secciones numeradas dentro de `docs/`:

```
docs/
├── 00-documentation-governance/  # Normativa, metodología, reglas de documentación
├── 01-project-context/           # Contexto del proyecto: perfil, alcance, propuesta
├── 02-domain/                    # Dominio del negocio: mapa de procesos
├── 03-product-definition/        # Definición del producto e investigación
├── 04-requeriments/              # SRS, análisis y requisitos
├── 05-architecture/              # Arquitectura y decisiones (ADRs)
├── 06-data-architecture/         # Modelo de datos y diccionarios
├── 07-api-design/                # Diseño de API
├── 08-uml/                       # Diagramas UML (fuentes y exportaciones)
├── 09-modules/                   # Catálogo de módulos del backend
├── 10-devops/                    # CI/CD y despliegue
├── 11-quality-assurance/         # Pruebas y calidad
├── 12-user-experience/           # UX/UI
├── 13-operations/                # Operaciones
├── 14-training-and-adoption/     # Capacitación y adopción
├── 15-project-control/           # Actas y control del proyecto
└── 99-archive/                   # Documentación archivada o deprecada
```

### Encabezado mínimo en documentos

Todo documento debe iniciar con el encabezado estándar (ver `documentation-rules.md`):

```markdown
<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## [Título descriptivo]

**Estado:** Pendiente
**Fecha:** YYYY-MM-DD

</div>

</div>
```

## 6. Control de versiones de documentos

### Política de versionado

- **Sin versionado formal** mientras el proyecto está en fase inicial.
- Una vez estable, adoptar `v{MAJOR}.{MINOR}.{PATCH}` (ej: `v1.0.0`).

### Historial de cambios

Incluir tabla en cada documento importante:

| Fecha | Estado | Autor | Cambios |
|-------|--------|-------|---------|
| 2026-04-12 | Estable | Cristian Javier Palma Sotto | Creación inicial |

## 7. Control de código y ramas

### Estructura de ramas

- **`main`** — rama de producción, siempre estable y desplegable
- **`develop`** — rama de integración, para cambios integrados
- **`qa`** — rama de QA/pruebas antes de pasar a main
- **`hu-<repo>-###-dev/qa/main`** — desarrollo de historias de usuario (REPO ∈ DEVICE, API, DB, APP, PORTAL; ver `agile-conventions.md`)
- **`fix/doc-<descripcion>`** — correcciones urgentes sobre documentación estable

### Reglas de merge

- Todo cambio debe realizarse mediante **Pull Request (PR)**
- **Mínimo requisitos para merge:**
  - 1 aprobación de revisión técnica (Líder Técnico)
  - Pruebas locales exitosas
  - Sin conflictos
- **Merge a main** requiere aprobación explícita del Líder Técnico
- **Merge a develop** requiere aprobación de al menos 1 revisor

## 8. Mensajes de commit

### Formato: Conventional Commits

```
<tipo>(<ámbito>): <descripción breve>

<cuerpo opcional>

<referencias opcionales>
```

### Tipos de commit

Aplica a los repositorios de código (`somnguard-api` y demás):

- `feat` — nueva característica
- `fix` — corrección de bug
- `refactor` — refactorización de código
- `test` — adición/modificación de tests
- `docs` — cambios en documentación
- `chore` — tareas (dependencias, configuración)

> En el repositorio documental (`somnguard-docs`) solo se usan `docs`, `fix`, `chore` y `refactor` (ver `git-conventions.md`).

### Ejemplos

```
feat(auth): add JWT-based login

Implements JWT authentication for drivers and admins.
Refs: HU-API-001
```

```
fix(device): correct camera capture timeout

Adjusts image capture timeout on Raspberry Pi.
Refs: HU-DEVICE-007
```

## 9. Revisión y aprobación

### Revisión de código

- **Revisor designado:** Líder Técnico (Cristian Javier Palma Sotto)
- **Criterios:** funcionalidad, calidad, seguridad, pruebas

### Revisión de documentación

- **Documentos críticos** (Arquitectura, SRS): Líder Técnico + Product Owner (cuando aplique)
- **Documentación funcional:** Responsable de área + Líder Técnico

### Aprobación final

- **Documentos finales:** Líder Técnico Cristian Javier Palma Sotto
- **Estado:** Marcar como `Estable` en el encabezado y registrar en acta

## 10. Seguridad y privacidad

### Políticas de seguridad

- **No incluir en el repositorio:**
  - Credenciales (contraseñas, tokens, claves privadas)
  - Información sensible o confidencial
  - Datos personales reales

- **Alternativas:**
  - Usar `.env` local (no versionado)
  - Referenciar a bóveda de secretos
  - Usar datos ficticios en ejemplos

### Revisión de dependencias

- Revisar dependencias críticas contra vulnerabilidades antes de integrar
- Mantener listos de dependencias actualizados
- Reportar CVE y planificar updates

## 11. Gestión de licencias

### Cumplimiento de licencias

- Respetar licencias de software de terceros
- Validar compatibilidad con licencia del proyecto
- Evitar dependencias con licencias restrictivas (ej: GPL) sin evaluación

## 12. Entregables y artefactos

### Tipos de entregables

| Tipo | Descripción | Ubicación |
|------|-------------|-----------|
| Código fuente | Repositorio Git del código | Repositorios de código del proyecto |
| Documentación técnica | Manuales, guías, API docs | `docs/05-architecture`, `docs/07-api-design` |
| Documentación funcional | SRS, casos de uso | `docs/04-requeriments`, `docs/08-uml` |
| Datos | Modelos, esquemas | `docs/06-data-architecture` |
| Reportes de pruebas | Test reports, coverage | `docs/11-quality-assurance` |
| Binarios/Releases | Versiones empaquetadas | `/releases/v*.*.*.zip` |

### Versionado de releases

Formato: `somnguard-v{MAJOR}.{MINOR}.{PATCH}`

Ejemplo: `somnguard-v1.0.0.zip`

## 13. Gestión de cambios

### Para cambios menores

- Crear PR directamente
- Incluir descripción clara del cambio
- Solicitar revisión

### Para cambios mayores o críticos

- Documentar propuesta de cambio (RFC - Request For Change)
- Incluir: objetivo, alcance, impacto, plan de pruebas
- Revisar y aprobar antes de implementar
- Registrar en acta si procede

## 14. Auditoría y trazabilidad

### Mecanismos de registro

- **Actas de reuniones:** `docs/15-project-control/01-meeting-minutes/` — decisiones, acuerdos, compromisos
- **Commits:** Git log — cambios de código con referencias a tickets
- **Pull Requests:** GitHub — discusiones técnicas, revisiones

### Decisiones arquitectónicas

- Cuando se tome una decisión de arquitectura importante, documentar brevemente:
  - Opción elegida
  - Alternativas consideradas
  - Justificación
  - Fecha
  - Responsable

## 15. Contactos del proyecto

| Rol | Nombre |
|-----|--------|
| Líder Técnico | Cristian Javier Palma Sotto |
| Desarrollador | Juan Carlos Jurado Castañeda |
| Desarrollador | Johan Steven Rodriguez Charry |
| Desarrollador | Brayan Alberto Perdomo |

## 16. Anexos

Plantillas y referencias:

- Acta de reuniones: `docs/15-project-control/01-meeting-minutes/`
- SRS / Requisitos: `docs/04-requeriments/`
- Template de PR: `.github/pull_request_template.md` del repositorio

## 17. Glosario

| Término | Significado |
|---------|------------|
| PR | Pull Request — solicitud de cambio de código |
| RFC | Request For Change — solicitud de cambio mayor |
| HU | Historia de Usuario |
| BUG | Defecto/error reportado |
| SRS | Documento de Especificación de Requisitos |
| Git | Sistema de control de versiones distribuido |

---

_Última actualización: 2026-08-19_
