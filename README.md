<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Repositorio de documentación

**Estado:** En progreso
**Fecha:** 2026-08-16

</div>

</div>

SomnGuard es un sistema integral (software + hardware) para el monitoreo de somnolencia y fatiga al volante: un dispositivo Raspberry Pi con cámara detecta patrones de riesgo, emite alertas preventivas y registra evidencia, mientras una plataforma web y una app móvil permiten consultar eventos, alertas y analítica.

Este repositorio centraliza toda la documentación del proyecto, organizada en secciones numeradas dentro de `docs/`.

## Estructura

```
docs/
├── 00-documentation-governance/   # Normativa, metodología y reglas de documentación
├── 01-project-context/            # Anteproyecto y cronograma
├── 02-domain/                     # Dominio del negocio: mapa de procesos
├── 03-product-definition/         # Definición del producto e investigación
├── 04-requeriments/               # SRS, funcionalidades, estructura por módulo
├── 05-architecture/               # Arquitectura y decisiones (ADRs)
├── 06-data-architecture/          # Modelo de datos y diccionarios
├── 07-api-design/                 # Diseño de API
├── 08-uml/                        # Diagramas UML (fuentes y exportaciones)
├── 09-modules/                    # Catálogo de módulos del backend
├── 10-devops/                     # CI/CD y despliegue
├── 11-quality-assurance/          # Pruebas y calidad
├── 12-user-experience/            # UX/UI
├── 13-operations/                 # Operaciones
├── 14-training-and-adoption/      # Capacitación y adopción
├── 15-project-control/            # Actas y control del proyecto
└── 99-archive/                    # Documentación archivada o deprecada
```

## Cómo usar este repositorio

1. **Empieza por la gobernanza**: lee `docs/00-documentation-governance/README.md` para conocer las reglas de documentación, normativa y metodología.
2. **Requisitos**: revisa el SRS y las funcionalidades en `docs/04-requeriments/`.
3. **Arquitectura**: consulta `docs/05-architecture/documento-arquitectura.md` para el detalle técnico.
4. **Datos**: el modelo vigente está en `docs/06-data-architecture/` (MER y módulos/entidades).
5. **Diagramas**: los casos de uso y sus exportaciones están en `docs/08-uml/`.
6. **Módulos del backend**: consulta el catálogo en `docs/09-modules/module-catalog.md`.
7. **Control del proyecto**: actas y seguimiento en `docs/15-project-control/`.

## Archivos raíz

| Archivo | Descripción |
|---------|-------------|
| [CHANGELOG.md](./CHANGELOG.md) | Historial de cambios de este repositorio |
| [CONTRIBUTING.md](./CONTRIBUTING.md) | Guía para contribuir |
| [CODE_OF_CONDUCT.md](./CODE_OF_CONDUCT.md) | Código de conducta |
| [LICENSE](./LICENSE) | Licencia del proyecto |
| [templates/](./templates/) | Plantillas reutilizables de documentos |
| [assets/](./assets/) | Recursos estáticos (logo, imágenes, exportaciones) |

## Formato de documentos

- **Markdown (`.md`)**: documentación textual y especificaciones.
- **Mermaid (`.mmd`)**: diagramas entidad-relación y casos de uso (fuentes editables).
- **PDF/DOCX/XLSX**: versiones de consulta o entrega de documentos oficiales.

## Historial

- **2026-08-16**: unificación de las estructuras anteriores en una sola, organizada por secciones y sin orientación a microservicios.