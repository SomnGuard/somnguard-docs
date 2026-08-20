<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Guía de contribución

**Estado:** En progreso
**Fecha:** 2026-08-19

</div>

</div>

Reglas para contribuir a la documentación del proyecto SomnGuard.

## Regla de oro

Todo cambio entra por **Pull Request**. No se commitea directo en `develop`, `qa` ni `main`.

## Estructura del repositorio

```
somnguard-docs/
├── docs/
│   ├── NN-seccion/          # Secciones numeradas (00-15)
│   │   ├── README.md        # Índice obligatorio de la sección
│   │   └── documento.md
│   └── 99-archive/          # Historia documental (nunca se elimina)
├── assets/
│   └── icons/               # Logos
├── .github/                 # Templates de PR
└── CHANGELOG.md
```

## Reglas por tema

| Tema | Regla |
|------|-------|
| **Docs nuevos** | Registrar en el `README.md` de la sección y en `CHANGELOG.md` si cambia estructura o contratos |
| **ADRs** | Crear desde `docs/05-architecture/decisions/_template-adr.md`; registrar en `docs/05-architecture/decisions/README.md` |
| **Módulos** | No crear carpetas de módulo sin ADR aprobada (ver `docs/00-documentation-governance/structure-rules.md`) |
| **Diagramas** | Fuente editable en `docs/08-uml/diagrams/source/` + exportación en `exports/`; registrar en `diagram-index.md` |
| **Enlaces** | Siempre relativos; verificar antes del PR |
| **Nombres** | `kebab-case` para archivos y carpetas; contenido en español, commits en inglés |
| **Secretos** | Nunca commitear credenciales, tokens ni datos personales (ver `security-policy.md`) |

## Flujo rápido

### Agregar un documento

1. Crear rama `feat/doc-<descripcion>` desde `develop`.
2. Crear el archivo con la estructura mínima (logo, título, estado, fecha).
3. Enlazarlo desde el `README.md` de la sección.
4. Commit con Conventional Commits (`docs(NN-seccion): ...`).
5. PR hacia `develop` con la plantilla de PR completada.

### Agregar una carpeta de sección

1. Nombrarla `NN-nombre-en-ingles`.
2. Crear `README.md` índice.
3. Registrar en `CHANGELOG.md`.

### Registrar una ADR

1. Copiar `_template-adr.md` a `records/ADR-NNN-titulo-corto.md`.
2. Completar contexto, decisión, consecuencias y alternativas.
3. Registrar en `records/README.md`.

### Documentar un módulo del backend

1. Verificar que el módulo esté en `docs/09-modules/module-catalog.md`.
2. Copiar la plantilla de módulo `docs/09-modules/modules/_template/module/` a `modules/<nombre-del-modulo>/`.
3. Completar README, data-model, events, decisions y runbook.

## Checklist pre-PR

- [ ] Encabezado estándar (logo, título, estado, fecha)
- [ ] Enlazado desde el `README.md` de su sección
- [ ] Enlaces relativos verificados
- [ ] Diagramas con fuente editable si aplica
- [ ] Sin credenciales ni datos personales
- [ ] CHANGELOG actualizado si modifica gobernanza, estructura o contratos compartidos

## Ver también

- [Reglas de documentación](docs/00-documentation-governance/documentation-rules.md)
- [Convenciones de git](docs/00-documentation-governance/git-conventions.md)
- [Política de seguridad](docs/00-documentation-governance/security-policy.md)
- [Plantilla de PR](.github/pull_request_template.md)