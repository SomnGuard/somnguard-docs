<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Convenciones de git

**Estado:** En progreso
**Fecha:** 2026-08-19

</div>

</div>

## Ramas protegidas

`develop` y `main` representan ambientes padre y no se trabajan directamente.

| Rama | Propósito | Regla |
|------|-----------|-------|
| `develop` | Integración de trabajo en desarrollo | Recibe PRs desde ramas hijas (`hu-<repo>-###-dev`) |
| `qa` | Validación de pruebas antes de producción | Recibe PRs desde `hu-<repo>-###-qa` |
| `main` | Producción / documentación estable | Recibe solo PRs desde `hu-<repo>-###-main` |

## Ramas documentales

| Tipo de rama | Cuándo usarla | Ejemplo | Tipo de commit |
|--------------|---------------|---------|----------------|
| `feat` | Documento nuevo | `feat/doc-api-guidelines` | `docs` |
| `fix` | Corrección de contenido | `fix/doc-scope` | `fix` |
| `chore` | Reorganización o renombrado | `chore/doc-move-adr-003` | `chore` |
| `docs` | Actualización de documento existente | `docs/doc-module-catalog` | `docs` |

El tipo de rama describe intención. El tipo del commit sigue Conventional Commits: aunque la rama se llame `feat/*`, los commits dentro del repositorio documental usan `docs` (no `feat`), según los tipos permitidos más abajo.

## Ramas por historia de usuario

Cada historia usa una rama por ambiente. Los cambios se propagan entre ambientes copiándolos o con `cherry-pick` (no por merge directo entre `develop`, `qa` y `main`).

| Caso | Rama base | Formato | Ejemplo |
|------|-----------|---------|---------|
| Desarrollo de HU | `develop` | `hu-<repo>-###-dev` | `hu-api-012-dev` |
| Validación en QA | `qa` | `hu-<repo>-###-qa` | `hu-api-012-qa` |
| Publicación a producción | `main` | `hu-<repo>-###-main` | `hu-api-012-main` |

El prefijo `<repo>` corresponde a la HU (DEVICE, API, DB, APP, PORTAL — ver `agile-conventions.md`). Las ramas `hu-*` son un caso especial para trazabilidad por historia. No siguen el formato `<tipo>/doc-*`.


## Flujo hacia develop

```bash
git switch develop
git pull origin develop
git switch -c hu-api-012-dev

git add <archivos>
git commit -m "docs(04-requeriments): add user story for event ingestion"
git push origin hu-api-012-dev
```

Abrir PR de `hu-api-012-dev` hacia `develop`.


## Propagación hacia qa y main

Para mover una historia al siguiente ambiente, copiar los cambios o usar `cherry-pick` desde el ambiente anterior:

```bash
git switch develop
git switch -c hu-api-012-qa
git cherry-pick <commit-hu-api-012-dev>
git push origin hu-api-012-qa
```

Abrir PR de `hu-api-012-qa` hacia `qa`. Repetir el mismo procedimiento para producción:

```bash
git switch qa
git switch -c hu-api-012-main
git cherry-pick <commit-hu-api-012-qa>
git push origin hu-api-012-main
```

Abrir PR de `hu-api-012-main` hacia `main`.


## Conventional Commits

Formato obligatorio:

```text
<type>(NN-section): short description in English
```

Tipos permitidos:

| Tipo | Uso |
|------|-----|
| `docs` | Crear o actualizar documentación |
| `fix` | Corregir contenido incorrecto |
| `chore` | Mover, renombrar, reordenar o actualizar metadatos |
| `refactor` | Reestructurar documentación sin cambiar significado |

No usar `feat`, `style`, `test`, `perf`, `build` ni `ci` para commits de este repositorio documental.

Ejemplos:

```bash
docs(04-requeriments): add traceability matrix
docs(09-modules): register auth module
fix(01-project-context): clarify project scope
chore(08-uml): export sequence diagrams to PNG
refactor(00-documentation-governance): split contribution rules by topic
```

## Reglas de commits

- La descripción del commit va en inglés.
- El contenido de los documentos puede estar en español.
- Los commits deben ser pequeños y trazables.
- Si se documentan varios módulos, usar un commit por módulo cuando sea posible.
- No mezclar cambios funcionales de varias secciones sin razón clara.

Cuando se detecta un error crítico en `main` que no puede esperar el flujo normal de release:

| Caso | Rama base | Formato | Ejemplo |
|------|-----------|---------|---------|
| Corrección urgente en documentación estable | `main` | `fix/doc-<descripcion>` | `fix/doc-broken-api-contract` |