<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Reporte de QA — SOMNGUARD — Sprint NN / v[X.X.X]

**Estado:** Plantilla
**Fecha:** 2026-08-19

</div>

</div>

> **PLANTILLA** — Copiar como `qa-report-sprintNN.md` o `qa-report-vX.X.X.md` y completar. Eliminar esta línea antes de hacer commit.

> Última actualización: YYYY-MM-DD
> Autor: Por definir | Equipo: QA

## Resumen ejecutivo

| Campo | Valor |
|-------|-------|
| Sprint / Release | |
| Período de pruebas | YYYY-MM-DD a YYYY-MM-DD |
| Total HUs probadas | |
| Total casos de prueba | |
| Pasaron | |
| Fallaron | |
| Bloqueados | |
| Veredicto | PASS / CONDITIONAL / FAIL |

## Cobertura

| Tipo de prueba | Total | Pasaron | Fallaron | Cobertura |
|----------------|-------|---------|----------|-----------|
| Unitarias | | | | [X%] |
| Integración | | | | [X%] |
| Funcionales | | | | [X%] |
| Regresión | | | | [X%] |

## Defectos encontrados

| ID | HU ID | Severidad | Descripción | Estado | Responsable |
|----|-------|-----------|-------------|--------|-------------|
| BUG-001 | HU-<REPO>-NNN | P0/P1/P2/P3 | | Abierto / Resuelto | |

> Severidades: P0 bloquea release, P1 mismo sprint, P2 priorizable, P3 deuda menor.

## HUs verificadas

| HU ID | Título | Resultado | AC cubiertos | Observaciones |
|-------|--------|-----------|--------------|---------------|
| HU-<REPO>-NNN | | PASS / FAIL | AC-001, AC-002 | |

## Criterio de aceptación del gate

- [ ] Sin defectos P0 abiertos
- [ ] Sin defectos P1 sin plan de resolución en el sprint actual
- [ ] Cobertura de pruebas unitarias ≥ [80%]
- [ ] Smoke tests en ambiente `qa` o `main` según release: todos PASS
- [ ] Pruebas de regresión: sin nuevos fallos

## Evidencia

| Tipo | Herramienta | Enlace / Artefacto |
|------|-------------|-------------------|
| Reporte de cobertura | JaCoCo | |
| Resultados de integración | Postman / Newman | |
| Screenshots de fallos | | |

## Referencias

- [Evidencia de pruebas](./_template-test-evidence.md)
- [Matriz de trazabilidad](../04-requeriments/traceability-matrix.md)
- [Backlog](../03-product-definition/product-backlog.md)