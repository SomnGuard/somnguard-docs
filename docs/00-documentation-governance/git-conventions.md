<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Convenciones de git

**Estado:** En progreso
**Fecha:** 2026-06-19

</div>

</div>

## Ramas protegidas

`develop` y `main` representan ambientes padre y no se trabajan directamente.

| Rama | Propósito | Regla |
|------|-----------|-------|
| `develop` | Integración de trabajo en desarrollo | Recibe PRs desde ramas hijas (`hu-<numero>-dev`) |
| `qa` | Validación de pruebas antes de producción | Recibe PRs desde `hu-<numero>-qa` |
| `main` | Producción / documentación estable | Recibe solo PRs desde `hu-<numero>-main` |

## Ramas documentales

| Tipo de rama | Cuándo usarla | Ejemplo | Tipo de commit |
|--------------|---------------|---------|----------------|
| `feat` | Documento nuevo | `feat/doc-api-guidelines` | `docs` |
| `fix` | Corrección de contenido | `fix/doc-scope` | `fix` |
| `chore` | Reorganización o renombrado | `chore/doc-move-adr-003` | `chore` |
| `docs` | Actualización de documento existente | `docs/doc-module-catalog` | `docs` |

El tipo de rama describe intención. El tipo del commit sigue Conventional Commits.

## Ramas por historia de usuario

Cada historia usa una rama por ambiente. Los cambios se propagan entre ambientes copiándolos o con `cherry-pick` (no por merge directo entre `develop`, `qa` y `main`).

| Caso | Rama base | Formato | Ejemplo |
|------|-----------|---------|---------|
| Desarrollo de HU | `develop` | `hu-<numero>-dev` | `hu-01-dev` |
| Validación en QA | `qa` | `hu-<numero>-qa` | `hu-01-qa` |
| Publicación a producción | `main` | `hu-<numero>-main` | `hu-01-main` |

Las ramas `hu-*` son un caso especial para trazabilidad por historia. No siguen el formato `<tipo>/doc-*`.


## Flujo hacia develop

```bash
git switch develop
git pull origin develop
git switch -c hu-01-dev

git add <archivos>
git commit -m "docs(04-requirements): add scheduling availability user story"
git push origin hu-01-dev
```

Abrir PR de `hu-01-dev` hacia `develop`.


## Propagación hacia qa y main

Para mover una historia al siguiente ambiente, copiar los cambios o usar `cherry-pick` desde el ambiente anterior:

```bash
git switch develop
git switch -c hu-01-qa
git cherry-pick <commit-hu-01-dev>
git push origin hu-01-qa
```

Abrir PR de `hu-01-qa` hacia `qa`. Repetir el mismo procedimiento para producción:

```bash
git switch qa
git switch -c hu-01-main
git cherry-pick <commit-hu-01-qa>
git push origin hu-01-main
```

Abrir PR de `hu-01-main` hacia `main`.


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
docs(04-requirements): add scheduling user stories
docs(09-modules): register auth module
fix(01-context): clarify project scope
chore(08-uml): export sequence diagrams to SVG
refactor(00-governance): split contribution rules by topic
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