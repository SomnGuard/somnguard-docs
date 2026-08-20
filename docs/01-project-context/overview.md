<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Contexto del proyecto

**Estado:** En progreso
**Fecha:** 2026-08-19

</div>

</div>

Contexto general del proyecto SomnGuard: problema, objetivo, alcance y marco del sistema.

## Problema

La somnolencia y la fatiga al volante son causas frecuentes de accidentes. Los conductores no siempre perciben el inicio del microsueño, y no existe un mecanismo preventivo accesible que detecte el riesgo en tiempo real, alerte al conductor y deje evidencia de lo ocurrido.

## Objetivo

Detectar patrones de fatiga, somnolencia y microsueños en el conductor mediante un dispositivo con cámara (Raspberry Pi + visión por computadora), generar alertas preventivas inmediatas y registrar eventos con evidencia multimedia para su consulta posterior en plataforma web y app móvil.

## Solución en una frase

Un sistema integral edge-cloud: el dispositivo detecta en el vehículo (sin depender de conectividad), el backend centraliza datos, alertas y administración, y los clientes (web y móvil) permiten consulta y control.

## Componentes principales

| Componente | Tecnología | Rol |
|------------|------------|-----|
| Dispositivo (edge) | Raspberry Pi + cámara + Python | Captura, detección, alerta sonora, buffer offline |
| Backend | Java 21 + Spring Boot 3.x (Maven) | API central, monolito modular hexagonal |
| Base de datos | PostgreSQL + Liquibase | Persistencia estructurada |
| Multimedia | MinIO/S3 | Evidencia multimedia (fuera de la BD) |
| Portal web | React JS | Descarga de app, administración, dashboard |
| App móvil | React Native | Usuario final, notificaciones y consulta |

## Actores

- Conductor / usuario final.
- Administrador de plataforma.
- Dispositivo SomnGuard instalado en campo (actor técnico: genera y sincroniza eventos).

## Marco de referencia

- Arquitectura: `05-architecture/architecture-document.md`
- Propuesta técnica: `01-project-context/software-technical-proposal.md`
- Análisis del software: `04-requeriments/software-analysis.md`
- Alcance y plan: `01-project-context/01-project-proposal/` y `02-schedule/`

## Ver también

- [Propuesta técnica](software-technical-proposal.md)
- [Glosario](../02-domain/glossary.md)
- [Visión del producto](../03-product-definition/product-backlog.md)