<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Política de seguridad y gobierno de repositorios

**Estado:** En progreso
**Fecha:** 2026-08-19

</div>

</div>

Política para el gobierno de los repositorios del proyecto SomnGuard, protección de ramas, manejo de secretos y respuesta ante fugas. Complementa las reglas documentales de `./security-rules.md` (enfocada en el contenido de los documentos).

## Gobierno de repositorios

| Tema | Regla |
|------|-------|
| Repositorios | Uno por componente: `somnguard-docs`, `somnguard-api`, `somnguard-db`, `somnguard-app`, `somnguard-portal`, `somnguard-device` |
| Rama por defecto | `main` es estable y protegida |
| Protección de ramas | `main`, `qa` y `develop` requieren PR con revisión; prohibido push directo |
| Owners | Asignación manual por área en cada PR |
| Acceso | Mínimo privilegio: contribuidor a la rama propia, reviewer al flujo de PR, admin a `main` |
| Licencia | Definir licencia por repositorio antes del primer release |

## Manejo de secretos

- **Nunca** versionar: contraseñas, tokens, API keys, certificados, archivos `.env`, archivos `.pem`/`.key`.
- Usar secretos del proveedor (GitHub Secrets / variables de entorno) y referenciarlos con variables: `${{ secrets.X }}` o `${VAR}`.
- Los archivos `.env.*` reales se ignoran vía `.gitignore`; solo se versionan `.env.example`.
- Rotar credenciales inmediatamente ante cualquier sospecha de exposición.

## Protección de datos personales (Ley 1581/2012)

- No publicar datos personales reales en la documentación (nombres, correos, placas, ubicaciones).
- Usar datos ficticios en ejemplos, capturas y mockups.
- La evidencia multimedia del sistema no se expone públicamente (ver [`../05-architecture/architecture-document.md`](../05-architecture/architecture-document.md)).

## Respuesta ante una fuga

1. **Rotar primero**: revocar el secreto expuesto antes de cualquier otra acción.
2. **Eliminar el historial**: usar herramientas de limpieza de historial git (p. ej. `git filter-repo`) si el secreto quedó en commits.
3. **Revisar el alcance**: verificar qué se expuso (repo público/privado, forks, cachés).
4. **Registrar el incidente**: abrir incidencia de seguridad y documentar en [`../15-project-control/risks.md`](../15-project-control/risks.md).
5. **Prevenir**: añadir el patrón al secret scanning del pipeline y al `.gitignore`.

## Roles

| Rol | Responsabilidad |
|-----|-----------------|
| Mantenedor | Aprueba PRs, gestiona releases, aplica esta política |
| Contribuidor | Aporta documentación/código sin tocar ramas protegidas |
| Líder técnico | Aprueba PRs hacia `main` (ver [`../10-devops/ci-cd-strategy.md`](../10-devops/ci-cd-strategy.md)) |
| QA | Valida contenido en `qa` (ver `./definition-of-done.md`) |

## Ver también

- [Reglas de seguridad documental](security-rules.md)
- [Convenciones de git](git-conventions.md)
- [Estrategia de CI/CD](../10-devops/ci-cd-strategy.md)
- [Preguntas abiertas](../15-project-control/open-questions.md)
- [Riesgos](../15-project-control/risks.md)