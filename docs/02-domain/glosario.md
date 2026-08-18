<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Glosario

**Estado:** En progreso
**Fecha:** 2026-08-16

</div>

</div>

Glosario de términos del dominio y técnicos utilizados en la documentación del proyecto. Referencia compartida para evitar ambigüedades.

## Términos del dominio

| Término | Significado |
|---------|-------------|
| SomnGuard | Sistema integral (software + hardware) de monitoreo de somnolencia y fatiga al volante. |
| Conductor | Usuario final que conduce un vehículo y es monitoreado por el dispositivo. |
| Fatiga | Estado de cansancio físico o mental que reduce el estado de alerta del conductor. |
| Somnolencia | Tendencia a dormirse; estado de alerta reducido detectado por el sistema. |
| Microsueño | Episodio breve (segundos) de sueño involuntario; evento crítico para el sistema. |
| Evento | Ocurrencia detectada por el dispositivo (ej: señal de fatiga). |
| Evidencia | Archivo multimedia (imagen, video) asociado a un evento. |
| Alerta | Alarma emitida por el dispositivo o el sistema ante un evento (local y/o notificada). |
| Notificación | Mensaje enviado al usuario como consecuencia de una alarma. |
| Severidad | Nivel de criticidad de un evento o alerta. |
| Patrón de sonido | Secuencia de sonido configurable emitida por el dispositivo en la alerta local. |
| Dispositivo | Hardware Raspberry Pi con cámara instalado en el vehículo. |
| Edge | Procesamiento local en el dispositivo (captura y preprocesamiento). |
| Sincronización offline | Reenvío de eventos y evidencia almacenados localmente cuando vuelve la conectividad. |
| Cuenta con retención | Baja de cuenta conservando datos según política definida. |

## Términos técnicos

| Término | Significado |
|---------|-------------|
| Backend | API central del sistema; monolito modular en Java 21 (Spring Boot 3.x, Maven). |
| Monolito modular | Aplicación con un solo despliegue, organizada internamente por dominios (módulos). |
| Clean architecture | Estilo de organización por capas (interfaces, application, domain, infrastructure) con dependencias hacia adentro. |
| Módulo | Dominio funcional del backend (security, parameterization, device-management, telemetry-service, monitoring). |
| Microservicio | Estilo de arquitectura por servicios independientes; **explícitamente descartado** en SomnGuard. |
| PostgreSQL | Sistema de gestión de base de datos relacional usado por el proyecto. |
| Liquibase | Herramienta de migraciones y versionado de esquema de base de datos. |
| JSONB | Tipo de datos JSON binario de PostgreSQL (usado en configuración de dispositivos). |
| UUID | Identificador único universal usado como llave primaria. |
| REST API | Estilo de API basado en recursos HTTP (prefijo `/api/v1`). |
| JWT | Token JSON Web Token (autenticación propuesta, pendiente de decisión). |
| Raspberry Pi | Mini computadora usada como dispositivo edge de captura. |
| React JS | Framework web del frontend de la plataforma. |
| React Native | Framework móvil (Android/iOS) de la app del usuario final. |
| GitHub Actions | Plataforma de CI/CD usada por el proyecto. |
| Maven | Herramienta de construcción y gestión de dependencias del backend. |

## Términos de proceso

| Término | Significado |
|---------|-------------|
| HU | Historia de Usuario; unidad de trabajo del backlog. |
| Sprint | Iteración de trabajo Scrum (1 a 1.5 semanas). |
| DoD | Definition of Done; criterios para considerar una historia completada. |
| DoR | Definition of Ready; criterios para iniciar un documento o historia. |
| SRS | Software Requirements Specification; especificación de requisitos del software. |
| ADR | Architecture Decision Record; registro de decisión de arquitectura. |
| PR | Pull Request; solicitud de integración de cambios con revisión. |
| RFC | Request For Change; propuesta de cambio mayor. |
| CI/CD | Integración y despliegue continuos. |
| CVE | Vulnerabilidad publicada y conocida en componentes de software. |
| Backlog | Lista priorizada de historias y requisitos del producto. |