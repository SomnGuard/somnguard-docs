<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Ambientes

**Estado:** En progreso
**Fecha:** 2026-08-19

</div>

</div>

Los ambientes del sistema SomnGuard, cómo se configuran y qué reglas rigen en cada uno. Complementa a [git-conventions](../00-documentation-governance/git-conventions.md) y a [local-setup.md](./local-setup.md).

## Los tres ambientes

Cada ambiente es también una **rama protegida**: no se trabaja directamente sobre ella, recibe cambios por Pull Request y representa una etapa de madurez creciente.

| Ambiente | Rama | Propósito | Base de datos | Migraciones |
|----------|------|-----------|---------------|-------------|
| **develop** | `develop` | Integración del trabajo en curso | BD de desarrollo | Forward + rollback local permitido |
| **qa** | `qa` | Validación funcional y técnica (previa al release) | BD de QA | **Forward-only** |
| **main** | `main` | Producción / documentación estable | BD de producción | **Forward-only** |

Promoción: **`develop → qa → main`**. El detalle del flujo de ramas (`hu-<repo>-###-<ambiente>`) está en [git-conventions.md](../00-documentation-governance/git-conventions.md) y [ci-cd-strategy.md](./ci-cd-strategy.md).

> **Estado real:** hoy los ambientes materializan la **documentación** (somnguard-docs) y el **código fuente del backend** (somnguard-api). El despliegue de la aplicación por ambiente es parte del estado objetivo (ver `05-architecture/architecture-document.md` §13).

## Configuración por ambiente — `.env.<ambiente>`

La configuración de cada ambiente vive en un archivo de entorno propio:

| Archivo | Ambiente |
|---------|----------|
| `.env.develop` | develop |
| `.env.qa` | qa |
| `.env.main` | main |

Docker Compose lee `.env` por defecto, por lo que el archivo se pasa **explícitamente** en cada comando:

```bash
docker compose --env-file .env.develop up postgres -d
```

Estos archivos parametrizan, como mínimo, la conexión a Postgres (host, puerto, nombre de BD, usuario, clave) del ambiente correspondiente.

## Base de datos por ambiente

- Cada ambiente tiene su **propia base de datos**, con la misma estructura: **una BD `somnguard`, un schema por módulo** (`security`, `parameterization`, `device_management`, `telemetry_service`, `monitoring`, `analytics`). Ningún módulo escribe en `public`.
- **Datos por ambiente:**
  - `develop` admite **datos de prueba** (seeds con `context`/`labels` de Liquibase).
  - `qa`/`main` reciben **solo catálogos de negocio** (semillas reales, idempotentes con `ON CONFLICT DO NOTHING`).
- En `qa`/`main` las migraciones son **forward-only**; el rollback es plan de contingencia documentado, no operación rutinaria.

## Advertencia de secretos

> **Riesgo real y prioritario.** Los archivos `.env.*` no deben contener contraseñas reales versionadas en git. Si una credencial queda en el repositorio (o en su historial), cualquiera con acceso obtiene acceso a los datos del ambiente, incluida PII sujeta a la Ley 1581/2012.

Reglas:

- Versionar únicamente un **`.env.example`** con valores ficticios como plantilla; mantener los valores reales **fuera de git**.
- Ante una credencial que haya estado versionada: **rotar primero, avisar después** (ver [política de seguridad](../00-documentation-governance/security-policy.md)).
- Estado objetivo: mover los secretos a un **Secret Manager** cuando exista la capa de despliegue.

## Referencias

- [local-setup.md](./local-setup.md) — comandos por ambiente
- [ci-cd-strategy.md](./ci-cd-strategy.md) — validación y promoción automatizadas
- [git-conventions.md](../00-documentation-governance/git-conventions.md) — ramas protegidas y flujo de promoción
- [architecture-document.md](../05-architecture/architecture-document.md#13-despliegue) — topología de despliegue por ambiente