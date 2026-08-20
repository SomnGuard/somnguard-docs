<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Modelo de amenazas de seguridad

**Estado:** En progreso
**Fecha:** 2026-08-19

</div>

</div>

Análisis STRIDE del sistema SomnGuard, alineado con el documento de arquitectura, la política de seguridad y el diseño de autenticación de la API.

## Superficie de ataque

| Componente | Tipo de exposición | Datos en riesgo |
|------------|-------------------|-----------------|
| API REST backend | Pública (Internet) | Eventos, evidencia, cuentas |
| Portal web (React JS) | Pública (Internet) | Credenciales, sesiones |
| App móvil (React Native) | Pública (Internet) | Credenciales, notificaciones |
| Dispositivo edge (Raspberry Pi) | Campo (física) | Buffer local, API key del dispositivo |
| Almacenamiento multimedia (MinIO/S3) | Interna | Evidencia multimedia (PII) |
| Base de datos PostgreSQL | Interna | PII, datos transaccionales |

## Amenazas identificadas (STRIDE)

| ID | Categoría STRIDE | Amenaza | Componente afectado | Probabilidad | Impacto | Mitigación |
|----|-----------------|---------|---------------------|--------------|---------|------------|
| T-001 | Spoofing | Suplantación de identidad por token robado | API REST | Media | Alto | JWT con expiración corta, renovación, revocación |
| T-002 | Spoofing | Envío de eventos con API key robada desde dispositivo clonado | API / telemetry-service | Media | Alto | API keys por dispositivo, rotación y revocación |
| T-003 | Tampering | Modificación de eventos en tránsito (sincronización offline) | API / edge | Baja | Alto | TLS 1.2+, firma de lotes |
| T-004 | Tampering | Manipulación del buffer local del dispositivo | Edge | Media | Medio | Integridad del buffer, API key no expuesta al usuario final |
| T-005 | Repudiation | Registro de eventos sin trazabilidad de origen | telemetry-service | Baja | Medio | `audit_login` + trazabilidad de eventos (NFR-06) |
| T-006 | Information Disclosure | Exposición de PII o evidencia en logs o respuestas | Backend | Media | Alto | Logs sin PII, cifrado en reposo, RBAC |
| T-007 | Information Disclosure | Acceso no autorizado a evidencia multimedia | MinIO/S3 | Media | Alto | URLs firmadas, RBAC por rol |
| T-008 | Denial of Service | Saturación de endpoints públicos | API REST | Media | Medio | Rate limiting, 429, límites por API key |
| T-009 | Elevation of Privilege | Escalada por asignación incorrecta de roles | security | Baja | Alto | Matriz rol × recurso, pruebas de autorización |

## Controles de seguridad por capa

| Capa | Controles |
|------|-----------|
| Edge | API key del dispositivo, buffer local con integridad, alerta local sin depender de red |
| Transporte | TLS 1.2+ en API y sincronización |
| API | JWT + RBAC, validación de idempotencia (RN-08), rate limiting |
| Almacenamiento | Evidencia en MinIO con acceso controlado; PostgreSQL solo datos estructurados |
| Clientes | Sesiones con expiración, tokens de refresco, cifrado en tránsito |

## Datos sensibles (PII)

| Dato | Clasificación | Almacenamiento | Acceso permitido |
|------|---------------|----------------|------------------|
| Email | PII contacto | Cifrado en reposo | Owner + Admin |
| Evidencia multimedia | PII (imagen/video del conductor) | MinIO con acceso controlado | Owner + Admin |
| Contraseña | Credencial | Hash bcrypt (nunca plano) | Nadie (solo verificación) |
| Ubicación / datos del vehículo | PII | Cifrado en reposo | Owner + Admin |

## Procedimiento ante incidente

Ver [política de seguridad](../00-documentation-governance/security-policy.md).

## Referencias

- [NFR](../04-requeriments/non-functional.md)
- [Documento de arquitectura](./architecture-document.md)
- [Autenticación API](../07-api-design/authentication.md)
- OWASP Top 10: https://owasp.org/www-project-top-ten/