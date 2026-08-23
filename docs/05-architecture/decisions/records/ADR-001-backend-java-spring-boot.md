# ADR-001: Backend en Java Spring Boot + Decisión de Autenticación

**Estado:** Aceptada
**Fecha:** 2026-08-16 (actualizada 2026-08-22)
**Autores:** Equipo SomnGuard
**Equipos involucrados:** Arquitectura, Desarrollo, Device

---

## Contexto

El stack inicial del proyecto se definió en el kick-off (2026-02-10) con **C#/.NET** como lenguaje del backend. Durante el desarrollo se decidió reemplazar el backend por **Java con Spring Boot**, manteniendo el resto de la arquitectura (monolito modular, PostgreSQL, Liquibase, edge Raspberry Pi, React JS y React Native) sin cambios.

El backend es el componente central de la plataforma: centraliza autenticación, gestión de dispositivos, recepción de telemetría, alertas y parametrización, y es la única vía de integración con los clientes web y móvil.

**Adicionalmente (2026-08-22):** Se requiere formalizar la decisión de autenticación para dos tipos de clientes distintos:
1. **Usuarios humanos** (Portal web, App móvil): login con correo/contraseña, sesiones, recuperación de contraseña
2. **Dispositivos edge** (Raspberry Pi): máquina-a-máquina, sin usuario interactivo, credenciales de larga duración

Requisitos: stateless, escalable, revocable, estándar de industria, rate limiting diferenciado.

---

## Decisión

### Stack Backend
Se decide implementar el backend de SomnGuard en **Java 21 (LTS) con Spring Boot 4.1.1 y Maven**, organizado como monolito modular con principios de clean architecture.

- **Lenguaje:** Java 21 (LTS)
- **Framework:** Spring Boot 4.1.1 (Spring Web, Spring Security, Spring Data JPA, Spring Validation)
- **Build tool:** Maven (`pom.xml`)
- **Base de datos:** PostgreSQL
- **Migraciones:** Liquibase
- **Estructura de paquetes por módulo:** hexagonal (puertos y adaptadores), ver [ADR-002](./ADR-002-hexagonal-architecture.md)

### Autenticación y Autorización
Se decide un **modelo dual** según tipo de cliente:

#### 1. Usuarios humanos (Portal, App, Admin) → JWT RS256
| Aspecto | Especificación |
|---------|----------------|
| **Algoritmo** | RS256 (RSA 2048 bits, firma asimétrica) |
| **Access Token** | JWT, 15 min, `Authorization: Bearer <jwt>` |
| **Refresh Token** | Opaco, 7 días, rotación en cada uso, hash en BD, blocklist en logout |
| **Claims obligatorios** | `sub` (user_id UUID), `roles` (array), `features` (array), `exp`, `iat`, `jti` |
| **JWKS Endpoint** | `GET /.well-known/jwks.json` para verificación distribuida sin compartir clave privada |
| **Rate Limit** | 5 req/min en `/auth/*` por IP; 100 req/min por usuario autenticado |

#### 2. Dispositivos Edge → API Key Opaca
| Aspecto | Especificación |
|---------|----------------|
| **Credencial** | API Key (32 bytes, base64url) + `device_id` (UUID) |
| **Headers** | `X-Device-ID: <uuid>` + `X-API-Key: <key>` |
| **Validación** | HMAC-SHA256 de `api_key` comparado con `device.api_key_hash` en BD |
| **Scope** | Solo `/telemetry/*` y `/devices/{id}/config` |
| **Rate Limit** | 1000 req/min por API Key (token bucket) |
| **Rotación** | Admin puede regenerar via PATCH `/devices/{id}/rotate-key` |

#### 3. RBAC (Role-Based Access Control)
- Modelo: `role` ↔ `feature` (N:M via `role_feature`) → `user_role` (N:M)
- Roles base: `admin` (todas las features), `user` (features propias)
- Enforcement: Middleware `@RequireFeature("feature.code")` en cada endpoint
- Denegación: 403 con `{code: "FORBIDDEN", message: "Feature X required"}`

---

## Consecuencias

### Positivas

- Java LTS garantiza soporte prolongado y estabilidad.
- Spring Boot ofrece ecosistema maduro: seguridad, persistencia, validación, testing y observabilidad integrados.
- Gran disponibilidad de talento y documentación para Java/Spring.
- Maven es el estándar del ecosistema Spring y simplifica la configuración del build.
- Liquibase se mantiene, por lo que el versionado del esquema no cambia respecto al plan original.
- **Auth dual**: Separación clara user vs device; JWT RS256 permite verificación distribuida via JWKS; API Key simple para device sin complejidad de certificados.
- **Revocation**: Refresh token blocklist + rotación = logout real; API Key regenerable por admin.
- **Estándares**: RFC 7519 (JWT), RFC 7807 (errors), librerías maduras en Java/Python/JS.

### Negativas / Trade-offs

- La base de código en C#/.NET existente debe reescribirse o descartarse.
- Se pierde la integración con herramientas del ecosistema .NET ya configuradas.
- Documentación y plantillas previas que mencionaban .NET deben actualizarse.
- **Gestión de claves RS256**: Rotación anual, backup seguro, distribución JWKS.
- **Payload JWT mayor** que HS256 (clave pública en header).
- **Refresh token rotation** añade complejidad en logout concurrente (mitigado: blocklist en Redis con TTL = refresh expiry).

### Riesgos

- Retraso por la migración de la base inicial del backend.
- Riesgo de mezclar convenciones del stack anterior en el código nuevo.
- Que la decisión se replantee a futuro si no se alinea con las necesidades del edge (Python) — mitigado por la separación de responsabilidades entre edge y servidor.
- **Fuga de clave privada RS256** → Mitigación: Vault en prod, `keys/` solo dev, rotación documentada en runbook.
- **API Key en logs/device** → Mitigación: Nunca loguear; device almacena en archivo seguro (chmod 600).

---

## Alternativas consideradas

| Alternativa | Por qué se descartó |
|-------------|---------------------|
| Mantener C#/.NET | Se decidió el cambio de stack por preferencia y disponibilidad del equipo con el ecosistema Java |
| Node.js (TypeScript) | No aporta beneficios claros sobre Spring Boot para el dominio (persistencia relacional, seguridad, madurez) |
| Python (FastAPI/Django) | Python se mantiene solo para el análisis visual en el edge; no se quería un lenguaje adicional en el servidor |
| JWT HS256 (simétrico) | Clave compartida = riesgo si se filtra; no permite verificación distribuida sin compartir secreto |
| OAuth2 / OIDC completo (Keycloak, etc.) | Overkill para MVP; añade complejidad (Auth server, flows) sin valor inmediato |
| mTLS para devices | Complejidad operacional (certificados en cada Pi, renovación, CA); API Key suficiente para MVP |
| Sesiones server-side (Redis) | No stateless; requiere sticky sessions o shared store; JWT RS256 es estándar |

---

## Referencias

- Documento de arquitectura: `../../architecture-document.md`
- Acta kick-off: `../../../15-project-control/01-meeting-minutes/01_kickoff.md`
- Reglas de documentación de módulos: `../../../00-documentation-governance/structure-rules.md`
- Preguntas abiertas: `../../../15-project-control/open-questions.md`
- **Autenticación detallada:** [`../../../07-api-design/authentication.md`](../../../07-api-design/authentication.md)
- **Cross-cutting concerns:** [`../cross-cutting.md`](../cross-cutting.md#1-autenticación-y-autorización)
- **ADR-002 (Hexagonal):** `./ADR-002-hexagonal-architecture.md`
- **RFC 7519:** https://tools.ietf.org/html/rfc7519
- **Spring Security JWT:** https://docs.spring.io/spring-security/reference/servlet/oauth2/resource-server/jwt.html