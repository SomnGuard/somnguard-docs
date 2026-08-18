<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Estrategia de pruebas

**Estado:** En progreso
**Fecha:** 2026-08-16

</div>

</div>

Estrategia de pruebas inicial del proyecto SomnGuard, alineada con la metodología adoptada (Scrum) y la arquitectura definida. Complementa la sección de calidad de `../00-documentation-governance/metodologia-adoptada.md`.

## Objetivo

Garantizar la calidad del sistema desde el inicio, con un enfoque incremental: comenzar con pruebas unitarias en módulos críticos y madurar hacia pruebas de integración, API y E2E conforme avance el desarrollo.

## Niveles de prueba

| Nivel | Alcance | Herramienta propuesta | Prioridad |
|-------|---------|----------------------|-----------|
| Unitarias | Lógica de dominio y casos de uso del backend | JUnit 5 + Mockito (Maven) | Alta |
| Integración | Persistencia con PostgreSQL, repositorios y migraciones Liquibase | Spring Boot Test + Testcontainers | Alta |
| API | Contratos REST de la API | Spring MockMvc / TestRestTemplate | Media |
| Frontend web | Componentes React JS | Jest + React Testing Library | Media |
| Móvil | Componentes React Native | Jest (React Native Testing Library) | Media |
| E2E | Flujos críticos completos | Playwright (web) | Baja (sprints posteriores) |
| Manual | Aceptación de historias de usuario | Checklist + entorno `qa` | Alta |

## Criterios de cobertura

- Los módulos críticos (análisis visual en edge, ingesta de eventos, alertas) tendrán prioridad de cobertura.
- Objetivo inicial: al menos un test por caso de uso crítico.
- La cobertura objetivo porcentual se define en un sprint de calidad (ver preguntas abiertas).

## Estrategia por módulo

- **Security**: pruebas de autenticación, roles y permisos; uso de datos ficticios (nunca datos reales).
- **DeviceManagement**: asignación/desasignación, historial y configuración JSONB.
- **Telemetry**: ingesta de eventos, deduplicación por sincronización offline e integridad de evidencia.
- **Monitoring**: generación de notificaciones y transición de estados (emitida, leída, cerrada).
- **Parameterization**: validación de catálogos y reglas de referencia (FK).

## Datos de prueba

- Usar siempre datos ficticios (normativa del proyecto).
- Entornos: `local`, `develop`, `qa`.
- El entorno `qa` es el punto de validación antes de `main`.

## Criterios de aceptación (DoD)

Una historia se considera probada cuando (aplica a la HU):

- [ ] Pruebas unitarias de los casos de uso implementados pasan.
- [ ] Pruebas de integración con base de datos pasan en entorno `develop`.
- [ ] Flujo principal verificado manualmente en entorno `qa`.
- [ ] No se introdujeron regresiones en módulos relacionados.

## Relación con CI/CD

Las pruebas se ejecutan en GitHub Actions (ver `../10-devops/ci-cd-strategy.md`): unitarias e integración en cada push, E2E en release. El pipeline debe bloquear el merge si las pruebas fallan.

## Pendientes

- Definir cobertura objetivo.
- Definir herramienta de reportes de cobertura y calidad (ej: JaCoCo).
- Incorporar pruebas automatizadas al DoD formal (según metodología).