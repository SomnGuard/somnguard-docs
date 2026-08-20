<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Declaración de alcance

**Estado:** En progreso
**Fecha:** 2026-08-19

</div>

</div>

Alcance del proyecto SomnGuard para la entrega MVP: lo que se construye, lo que queda excluido, roles, supuestos y criterios de éxito.

## Objetivo

Detectar patrones de fatiga, somnolencia y microsueños en el conductor mediante un dispositivo con cámara en el vehículo, alertar preventivamente y registrar eventos con evidencia multimedia consultables en plataforma web y app móvil.

## En alcance (MVP)

- Dispositivo: captura, detección de fatiga/somnolencia y alerta sonora local
- Sincronización offline de eventos y evidencia
- Backend: autenticación, dispositivos, telemetría, notificaciones, parametrización
- Portal web: login, dashboard, administración
- App móvil: notificaciones y consulta
- Resumen generado por IA de eventos (de menor prioridad; se entrega al final del MVP)

## Fuera de alcance

- Transmisión de video en tiempo real al backend (la detección es local en el edge)
- Integración con sistemas externos de gestión de flotas (FMS) o terceros
- Facturación, planes de suscripción o pagos
- Desarrollo de hardware propio (se usa Raspberry Pi + cámara comercial)
- Funcionalidades de analítica avanzada fuera de la línea de tiempo y reportes definidos

## Roles involucrados

| Rol | Responsabilidad |
|-----|-----------------|
| Conductor / usuario final | Usa la app móvil y recibe alertas |
| Administrador de plataforma | Administra cuentas, dispositivos y catálogos |
| Equipo de desarrollo | Backend, frontends y edge |
| QA | Validación por ambiente |

## Supuestos

- La detección se ejecuta en el edge; el backend no procesa video en tiempo real.
- La evidencia multimedia se almacena fuera de PostgreSQL (MinIO/S3).
- Infraestructura final (local/nube) se define más adelante.
- El dispositivo funciona sin conectividad: alerta local y buffer offline con respaldo.

## Criterios de éxito

- Detección y alerta preventiva funcionando en el dispositivo.
- Eventos sincronizados sin duplicados.
- Notificación de eventos críticos recibida en la app.

## Ver también

- [Contexto del proyecto](./overview.md)
- [Perfil del proyecto](./project-profile.md)
- [Propuesta técnica](./software-technical-proposal.md)
- [Backlog de producto](../03-product-definition/product-backlog.md)