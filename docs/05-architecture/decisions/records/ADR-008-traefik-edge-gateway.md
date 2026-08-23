# ADR-008: Edge Gateway — Traefik v3 (Puerto Único 80/443, Routing API/Portal)

**Estado:** Aceptada
**Fecha:** 2026-08-22
**Autores:** Equipo SomnGuard
**Equipos involucrados:** Arquitectura, Backend, Frontend, DevOps, Device

---

## Contexto

SomnGuard expone múltiples servicios HTTP que deben ser accesibles desde internet:
- **API Backend** (Java Spring Boot) — `/api/v1/*` — puerto interno 8080
- **Portal Web** (React + Nginx) — `/` (SPA) — puerto interno 3000
- **App Móvil** — mismo endpoint API que portal
- **Device Edge** — mismo endpoint API (`/telemetry/*`, `/devices/{id}/config`)
- **Observabilidad** — Grafana (`:3000`), Traefik Dashboard (`:8080`), Prometheus (`:9090`)

Requisitos:
- **Puerto único público** (80 HTTP, 443 HTTPS) — firewall simple, sin puertos extra
- **Routing por path/host** — `/api/*` → API, `/` → Portal
- **TLS termination** — Certificados Let's Encrypt (prod) / self-signed (local)
- **Rate limiting** — Diferenciado por cliente (user vs device)
- **Observabilidad** — Métricas Prometheus, access logs JSON
- **Cero config en servicios** — Servicios no saben de TLS, dominio, puertos públicos

---

## Decisión

### Traefik v3 como Edge Gateway (Reverse Proxy + Load Balancer)

| Aspecto | Especificación |
|---------|----------------|
| **Versión** | Traefik v3.1 (latest stable) |
| **Modo** | Docker provider (auto-discovery via labels) |
| **EntryPoints** | `web` (80 HTTP → redirect a HTTPS), `websecure` (443 HTTPS) |
| **Certificados** | **Local:** self-signed generados en entrypoint; **Prod:** Let's Encrypt (ACME HTTP-01 / TLS-ALPN-01) |
| **Middleware** | `rate-limit`, `compress`, `headers` (CSP, HSTS), `redirect-regex` (www → non-www) |
| **Dashboard** | Habilitado en `:8080` (solo red local, auth básico) |
| **Métricas** | Prometheus endpoint `/metrics` (scrapeado por Prometheus) |

---

### 1. Routing Rules (Labels en docker-compose.yml)

#### API Backend (`somnguard-api`)
```yaml
labels:
  - "traefik.enable=true"
  # Router: /api/* → API
  - "traefik.http.routers.api.rule=PathPrefix(`/api`)"
  - "traefik.http.routers.api.entrypoints=websecure"
  - "traefik.http.routers.api.tls=true"
  # Middleware: rate limit usuarios (100 req/min)
  - "traefik.http.routers.api.middlewares=api-ratelimit@docker,api-headers@docker"
  - "traefik.http.middlewares.api-ratelimit.ratelimit.average=100"
  - "traefik.http.middlewares.api-ratelimit.ratelimit.burst=200"
  # Headers de seguridad
  - "traefik.http.middlewares.api-headers.headers.customrequestheaders.X-Forwarded-Proto=https"
  - "traefik.http.middlewares.api-headers.headers.sslredirect=true"
  - "traefik.http.middlewares.api-headers.headers.stsseconds=31536000"
  - "traefik.http.middlewares.api-headers.headers.stsincludesubdomains=true"
  - "traefik.http.middlewares.api-headers.headers.contenttypenosniff=true"
  - "traefik.http.middlewares.api-headers.headers.browserxssfilter=true"
  # Service
  - "traefik.http.services.api.loadbalancer.server.port=8080"
  - "traefik.http.services.api.loadbalancer.server.scheme=http"
```

#### Portal Web (`somnguard-portal`)
```yaml
labels:
  - "traefik.enable=true"
  # Router: / → Portal (SPA)
  - "traefik.http.routers.portal.rule=PathPrefix(`/`)"
  - "traefik.http.routers.portal.entrypoints=websecure"
  - "traefik.http.routers.portal.tls=true"
  # Middleware: rate limit más laxo (500 req/min), spa fallback
  - "traefik.http.routers.portal.middlewares=portal-ratelimit@docker,portal-headers@docker,spa-fallback@docker"
  - "traefik.http.middlewares.portal-ratelimit.ratelimit.average=500"
  - "traefik.http.middlewares.portal-ratelimit.ratelimit.burst=1000"
  # SPA fallback: / → /index.html para client-side routing
  - "traefik.http.middlewares.spa-fallback.replaceregex.regex=^(.*)$$"
  - "traefik.http.middlewares.spa-fallback.replaceregex.replacement=/$1"
  # Headers
  - "traefik.http.middlewares.portal-headers.headers.customrequestheaders.X-Forwarded-Proto=https"
  - "traefik.http.middlewares.portal-headers.headers.stsseconds=31536000"
  - "traefik.http.middlewares.portal-headers.headers.frameoptions=DENY"
  # Service
  - "traefik.http.services.portal.loadbalancer.server.port=3000"
```

#### Device API (Rate Limit Específico)
```yaml
# Middleware adicional para device endpoints
- "traefik.http.middlewares.device-ratelimit.ratelimit.average=1000"
- "traefik.http.middlewares.device-ratelimit.ratelimit.burst=2000"
# Router device: mismo /api/telemetry/* pero middleware device-ratelimit
# Se logra con Priority en router o middleware chain condicional
```

---

### 2. TLS / Certificados

| Ambiente | Método | Detalle |
|----------|--------|---------|
| **Local (docker-compose)** | Self-signed | `traefik generate` o `mkcert` en `keys/`; `traefik.http.routers.*.tls=true` |
| **Dev / QA** | Let's Encrypt Staging | `certificatesResolvers.letsencrypt.acme.caserver=https://acme-staging-v02.api.letsencrypt.org/directory` |
| **Staging / Prod** | Let's Encrypt Prod | `certificatesResolvers.letsencrypt.acme.email=ops@somnguard.com`; `storage=/certs/acme.json` (volume persistente) |
| **Wildcard** | DNS-01 challenge | Para `*.somnguard.com` (requiere DNS provider API: Cloudflare, Route53, etc.) |

---

### 3. Rate Limiting Diferenciado

| Cliente | Límite | Burst | Implementación |
|---------|--------|-------|----------------|
| **Usuario autenticado (JWT)** | 100 req/min | 200 | Middleware `api-ratelimit` (basado en `X-Forwarded-For` + `Authorization` header) |
| **Device (API Key)** | 1000 req/min | 2000 | Middleware `device-ratelimit` (basado en `X-Device-ID`) |
| **Anónimo (login, register)** | 10 req/min | 20 | Middleware `auth-ratelimit` en `/auth/*` |
| **Portal (assets, SPA)** | 500 req/min | 1000 | Middleware `portal-ratelimit` |

> **Nota:** Rate limit en Traefik es **por IP origen**. Para rate limit por **usuario/device real** (no IP), implementar en API (Redis + Lua script) — Traefik solo protege contra DDoS básico.

---

### 4. Seguridad Headers (OWASP Recomendados)

```yaml
# Middleware global headers (aplicado a todos los routers)
- "traefik.http.middlewares.security-headers.headers.sslredirect=true"
- "traefik.http.middlewares.security-headers.headers.stsseconds=31536000"
- "traefik.http.middlewares.security-headers.headers.stsincludesubdomains=true"
- "traefik.http.middlewares.security-headers.headers.stspreload=true"
- "traefik.http.middlewares.security-headers.headers.forceSTSHeader=true"
- "traefik.http.middlewares.security-headers.headers.contenttypenosniff=true"
- "traefik.http.middlewares.security-headers.headers.browserxssfilter=true"
- "traefik.http.middlewares.security-headers.headers.frameoptions=DENY"
- "traefik.http.middlewares.security-headers.headers.referrerpolicy=strict-origin-when-cross-origin"
- "traefik.http.middlewares.security-headers.headers.customresponseheaders.Server="
- "traefik.http.middlewares.security-headers.headers.customresponseheaders.X-Powered-By="
```

---

### 5. CORS (Centralizado en Traefik)

```yaml
- "traefik.http.middlewares.cors.headers.accesscontrolalloworiginlist=https://portal.somnguard.com,https://app.somnguard.com,capacitor://localhost,http://localhost:3000"
- "traefik.http.middlewares.cors.headers.accesscontrolallowmethods=GET,POST,PUT,PATCH,DELETE,OPTIONS"
- "traefik.http.middlewares.cors.headers.accesscontrolallowheaders=Authorization,Content-Type,X-Device-ID,X-API-Key,X-Request-ID"
- "traefik.http.middlewares.cors.headers.accesscontrolallowcredentials=true"
- "traefik.http.middlewares.cors.headers.accesscontrolmaxage=3600"
```

---

### 6. Observabilidad

| Métrica Traefik | Uso |
|-----------------|-----|
| `traefik_entrypoint_requests_total` | Requests por entrypoint (web/websecure) |
| `traefik_entrypoint_request_duration_seconds` | Latencia por entrypoint |
| `traefik_service_requests_total` | Requests por service (api, portal) |
| `traefik_service_request_duration_seconds` | Latencia por service |
| `traefik_router_requests_total` | Requests por router (path) |
| `traefik_tls_cert_not_after_seconds` | Expiración certificados (alerta 30d antes) |

**Access Logs JSON:**
```json
{
  "time": "2026-08-22T14:30:00Z",
  "level": "info",
  "message": "access log",
  "ClientAddr": "192.168.1.100",
  "ClientHost": "192.168.1.100",
  "ClientUsername": "-",
  "RequestMethod": "POST",
  "RequestPath": "/api/v1/telemetry/events",
  "RequestProtocol": "HTTP/2.0",
  "StatusCode": 201,
  "ResponseSize": 156,
  "RequestDuration": 45.2,
  "TraceID": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "SpanID": "1234567890abcdef",
  "ServiceName": "api",
  "Middleware": ["api-ratelimit", "security-headers"]
}
```

---

### 7. Configuración por Ambiente (docker-compose)

```yaml
# docker-compose.yml (somnguard-docker-infra)
traefik:
  image: traefik:v3.1
  command:
    - "--providers.docker=true"
    - "--providers.docker.exposedbydefault=false"
    - "--entrypoints.web.address=:80"
    - "--entrypoints.web.http.redirections.entrypoint.to=websecure"
    - "--entrypoints.web.http.redirections.entrypoint.scheme=https"
    - "--entrypoints.websecure.address=:443"
    - "--entrypoints.websecure.http.tls=true"
    - "--certificatesresolvers.letsencrypt.acme.email=${ACME_EMAIL:-ops@somnguard.com}"
    - "--certificatesresolvers.letsencrypt.acme.storage=/certs/acme.json"
    - "--certificatesresolvers.letsencrypt.acme.httpchallenge.entrypoint=web"
    - "--api.dashboard=true"
    - "--api.insecure=false"
    - "--log.level=INFO"
    - "--log.format=json"
    - "--accesslog=true"
    - "--accesslog.format=json"
    - "--accesslog.fields.headers.defaultmode=keep"
    - "--metrics.prometheus=true"
  ports:
    - "${EDGE_HTTP_PORT:-80}:80"
    - "${EDGE_HTTPS_PORT:-443}:443"
    - "${EDGE_DASHBARD_PORT:-8080}:8080"
  volumes:
    - /var/run/docker.sock:/var/run/docker.sock:ro
    - traefik_certs:/certs
  labels:
    - "traefik.enable=true"
    - "traefik.http.routers.traefik-dashboard.rule=Host(`traefik.${DOMAIN:-local}`)"
    - "traefik.http.routers.traefik-dashboard.entrypoints=websecure"
    - "traefik.http.routers.traefik-dashboard.tls=true"
    - "traefik.http.routers.traefik-dashboard.middlewares=auth-basic@file"
    - "traefik.http.services.traefik-dashboard.loadbalancer.server.port=8080"
```

---

## Consecuencias

### Positivas
- **Puerto único 80/443:** Firewall simple, compatible con corporate proxies, CDN
- **Auto-discovery:** Servicios se registran solos via labels Docker — cero config manual
- **TLS centralizado:** Let's Encrypt automático; servicios internos HTTP plano
- **Rate limiting en edge:** Protege API antes de que llegue a la app
- **SPA fallback nativo:** `ReplacePathRegex` para React Router sin config en Nginx
- **Métricas + Logs unificados:** Prometheus + JSON logs correlacionados con `trace_id`
- **Cero downtime deploy:** Traefik detecta contenedores nuevos/old y drena conexiones

### Negativas / Trade-offs
- **Single point of failure** (mitigado: 2+ réplicas Traefik en prod + healthcheck)
- **Configuración en labels** — acopla docker-compose a Traefik (mitigado: labels documentados, plantillas)
- **Rate limit por IP** en Traefik vs por usuario en API — dual layer necesario
- **Certificados Let's Encrypt** — rate limits (50 certs/semana por dominio); staging para dev

### Riesgos
- **Label typo** → router no creado → 404 silencioso → Mitigación: Test de integración `curl -f https://local/api/health` en CI
- **ACME challenge falla** (DNS/HTTP) → cert no renovado → Mitigación: Alerta `traefik_tls_cert_not_after_seconds < 2592000` (30d)
- **Dashboard expuesto** → Mitigación: `auth-basic` middleware + solo red interna (`--api.insecure=false`)

---

## Alternativas consideradas

| Alternativa | Por qué se descartó |
|-------------|---------------------|
| **Nginx / OpenResty** | Config manual (nginx.conf), sin auto-discovery Docker, rate limit limitado, TLS manual |
| **Envoy Proxy** | Complejidad alta (xDS, config discovery), overkill para monolito + 2 frontends |
| **Kong / APISIX** | API Gateway completo (plugins, auth, transform) — complejidad y recursos innecesarios |
| **AWS ALB / Cloudflare** | Vendor lock-in; costo; no control local; Traefik corre en cualquier Docker/K8s |
| **Caddy** | Buena alternativa (auto HTTPS), pero menos maduro en Docker provider, rate limiting, métricas |
| **Servicios en puertos separados** (8080 API, 3000 Portal) | Firewall complejo, puertos expuestos, TLS duplicado, CORS manual |

---

## Referencias

- [cross-cutting.md](../cross-cutting.md) — (config por ambiente, seguridad headers)
- [local-setup.md](../../../10-devops/local-setup.md) — Docker Compose profiles
- [architecture-document.md](../../architecture-document.md) §Deployment
- Traefik v3 docs: https://doc.traefik.io/traefik/
- Traefik Docker provider: https://doc.traefik.io/traefik/providers/docker/
- Let's Encrypt ACME: https://letsencrypt.org/docs/
- OWASP Secure Headers: https://owasp.org/www-project-secure-headers/