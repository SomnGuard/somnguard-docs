<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Respaldo y recuperación

**Estado:** En progreso
**Fecha:** 2026-08-19

</div>

</div>

Estrategia de respaldo y recuperación de la **base de datos** de SomnGuard. Alineada con la sección 16 del documento de arquitectura: una única instancia PostgreSQL 16, un schema por módulo, y evidencia multimedia fuera de la BD (MinIO/S3).

## Qué se respalda

| Activo | Contenido | Criticidad |
|--------|-----------|------------|
| BD PostgreSQL (todos los schemas) | Datos de negocio de los 6 módulos | Alta |
| MinIO/S3 (evidencia multimedia) | Imágenes/videos de eventos | Alta / PII |
| Changelogs Liquibase (en git) | Definición versionada del esquema | Media (recuperable desde git) |
| Archivos `.env.*` | Configuración por ambiente — **sin secretos versionados** | Fuera de git |

> El esquema no requiere backup de datos porque vive en git; lo que se respalda son los **datos**. La recuperación combina *restaurar datos* + *reaplicar/verificar migraciones al tag correcto*.

## Objetivos: RTO y RPO

| Objetivo | Definición | Valor (referencia, **por confirmar**) |
|----------|------------|----------------------------------------|
| **RPO** | Máxima pérdida de datos tolerable | Objetivo bajo para producción; requiere WAL archiving / PITR |
| **RTO** | Máximo tiempo para restaurar el servicio | A comprometer con el SLO |

> Los valores definitivos se fijarán al desplegar producción y se registrarán en [_template-sla-slo-sli.md](./_template-sla-slo-sli.md). `pg_dump` por sí solo acota el RPO al intervalo entre dumps; para un RPO fino se necesita **PITR**.

## Métodos de respaldo

### 1. Dump lógico (`pg_dump`)

```bash
docker compose --env-file .env.develop exec -T postgres \
  pg_dump -U <user> -d somnguard -F c -f /backups/somnguard_$(date +%Y%m%d_%H%M).dump

# Dump de un solo schema/módulo (respaldo granular)
docker compose --env-file .env.develop exec -T postgres \
  pg_dump -U <user> -d somnguard -n <schema_modulo> -F c -f /backups/<modulo>_$(date +%Y%m%d).dump
```

- Respaldo **por schema** permite recuperar un solo módulo sin tocar los demás.
- El RPO de este método es el **intervalo entre dumps**.

### 2. PITR (Point-In-Time Recovery) — para producción

Base física (`pg_basebackup`) + archivado continuo de WAL. Recomendado cuando exista producción real; su configuración concreta es un **punto abierto**.

## Restauración

```bash
docker compose --env-file .env.develop exec -T postgres \
  pg_restore -U <user> -d somnguard --clean --if-exists /backups/somnguard_YYYYMMDD_HHMM.dump
```

Orden recomendado de recuperación total:
1. Levantar PostgreSQL limpio del ambiente.
2. Restaurar datos (dump o PITR) y evidencia (MinIO).
3. Verificar con Liquibase que cada módulo está en el **tag de release** correcto (`04_tcl`); si falta esquema, aplicar `update`.
4. Verificar integridad: `pg_isready`, conteos de catálogos clave y unicidad de eventos por ID de idempotencia.

## Migraciones y rollback como recuperación

- Todo changeset forward tiene su **rollback espejo** en `05_rollbacks/`. En **local** se usa `rollback <tag>`.
- En `qa`/`main` las migraciones son **forward-only**: ante una migración defectuosa se prepara un **forward-fix**; si hubo daño de datos, se recurre al backup.
- Los **tags de release** (`04_tcl`) son los puntos de recuperación de esquema conocidos.

## Prueba de restauración (restore drill)

1. Tomar el último dump y restaurarlo sobre una BD **efímera** limpia.
2. Ejecutar `liquibase status` por módulo: no debe faltar ni sobrar esquema.
3. Validar conteos de catálogos y una muestra de integridad referencial.
4. Registrar resultado, duración (insumo para el RTO) y cualquier incidencia.

> **Punto abierto:** frecuencia formal del drill y su automatización en CI/CD.

## Consideraciones de seguridad

- Los dumps contienen PII (evidencia multimedia, correos): tratarlos como sensibles, cifrarlos en reposo y **no** versionarlos en git (ver [política de seguridad](../00-documentation-governance/security-policy.md)).
- La evidencia multimedia tiene retención configurable (RN-06): su respaldo debe preservar la política de retención.

## Puntos abiertos

- Valores definitivos de RTO/RPO.
- Configuración de PITR (archivado de WAL) para producción.
- Frecuencia y automatización del restore drill.
- Destino y cifrado de los backups en cada ambiente.

## Referencias

- [Convenciones de modelado](../06-data-architecture/modeling-conventions.md)
- [_template-sla-slo-sli.md](./_template-sla-slo-sli.md)
- [incident-management.md](./incident-management.md)
- [local-setup.md](../10-devops/local-setup.md)
- [Política de seguridad](../00-documentation-governance/security-policy.md)