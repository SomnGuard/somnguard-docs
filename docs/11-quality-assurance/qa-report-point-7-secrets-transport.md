# QA — Punto 7: endurecimiento de secretos y transporte

**Estado:** Implementación verificada parcialmente; cierre de QA pendiente  
**Fecha de preparación:** 2026-11-01  
**Rama:** `feature/point-7-secrets-transport-dev`  
**Alcance:** `somnguard-api`, `somnguard-app`, `somnguard-device`, `somnguard-portal`

## Objetivo

Validar que los secretos no tengan valores por defecto inseguros en Docker Compose, que Android no permita tráfico cleartext, que el dispositivo requiera TLS fuera de desarrollo y que el portal no persista tokens en `localStorage`.

## Resultado de QA

| Campo | Valor |
|---|---|
| Tester | Por asignar |
| Ambiente | Desarrollo / QA |
| Fecha de ejecución | Pendiente |
| Resultado | Pendiente — PASS / CONDITIONAL / FAIL |
| Defectos | Por registrar |
| Evidencia | Por adjuntar; no incluir secretos ni tokens |

## Casos de validación

| ID | Criterio | Pasos | Resultado esperado | Estado / evidencia |
|---|---|---|---|---|
| QA-07-01 | Compose exige variables sensibles | Use un archivo de entorno temporal de prueba sin `POSTGRES_PASSWORD` y ejecute `docker compose --env-file <archivo-temporal> config --quiet`. Repita sin `S3_SECRET_KEY` y sin `JWT_PRIVATE_KEY_PATH`. No edite ni comparta el `.env` local real. | Compose termina con error e identifica la variable faltante; no sustituye claves, contraseñas ni tokens por defaults. | **PASS** — rechaza `POSTGRES_PASSWORD`, `S3_SECRET_KEY` y `JWT_PRIVATE_KEY_PATH` faltantes en archivos temporales. |
| QA-07-02 | Compose acepta configuración explícita | Con un `.env` local completo (incluya explícitamente las variables opcionales aunque estén vacías para MailHog/FCM), ejecute `docker compose config --quiet`. | Termina con código 0. No adjunte salida expandida: puede contener configuración sensible. | **PASS** — `docker compose config --quiet` y `docker compose up -d` completan usando el `.env` local. |
| QA-07-03 | Sin cleartext en Android | Revise `SomnGuard-app/app.json` y la configuración Android generada con Expo. Confirme `usesCleartextTraffic` desactivado. | El manifiesto Android generado contiene `usesCleartextTraffic="false"`; la app no inicia llamadas HTTP sin TLS. | **PASS** — manifiesto generado por Expo muestra `usesCleartextTraffic="false"`; bundle Android exportado. |
| QA-07-04 | El device rechaza HTTP en producción | En PowerShell, establezca temporalmente `SOMNGUARD_ENV=production` y `SOMNGUARD_API_URL=http://localhost:8080`; ejecute `py -c "from app.common.config import load_env_config; load_env_config()"` desde `somnguard-device`. | La carga falla con error que indica que HTTPS es obligatorio fuera de `development`. | **PASS** — la carga rechaza `http://` en producción. |
| QA-07-05 | El device permite HTTP local solo en desarrollo | Establezca `SOMNGUARD_ENV=development` y `SOMNGUARD_API_URL=http://localhost:8080`; ejecute el mismo comando de QA-07-04. | La configuración carga correctamente. | **PASS** — acepta HTTP en desarrollo. |
| QA-07-06 | El device requiere WSS fuera de desarrollo | Con `SOMNGUARD_ENV=production`, pruebe `SOMNGUARD_WS_URL=ws://localhost:8080/ws/stream` y luego `wss://api.example.test/ws/stream`. | `ws://` se rechaza; `wss://` se acepta. | **PASS** — rechaza `ws://` y acepta `wss://` en producción. |
| QA-07-07 | El portal no almacena tokens en localStorage | Inicie sesión en el portal y revise DevTools → Application → Local Storage. Compruebe las claves `somnguard_token`, `somnguard_refresh_token` y el contenido de `somnguard_user`. | No hay claves de tokens; `somnguard_user` contiene solo datos de perfil. En la primera carga se limpian claves de token antiguas. | **PENDIENTE** — falta comprobación en navegador tras autenticación real; build y pruebas unitarias del portal pasan. |
| QA-07-08 | El refresh token usa cookie HttpOnly | Revise la respuesta de login en DevTools → Network → Cookies/Headers. | La respuesta incluye `Set-Cookie: somnguard_refresh` con `HttpOnly`, `SameSite=Lax` y ruta `/api/v1/auth`. El navegador envía la cookie al refrescar y la rota; el cuerpo de login del navegador no expone el refresh token. | **PASS (API)** — `SecurityControllersHttpTest` valida atributos, ausencia del token en el cuerpo y refresh usando solo la cookie. Falta flujo de navegador. |
| QA-07-09 | Cookie Secure por ambiente | Compruebe la cookie en desarrollo y en QA/producción. | En localhost HTTP de desarrollo puede no llevar `Secure`; en QA/producción sobre HTTPS sí lleva `Secure`. | **PENDIENTE** — falta verificar el atributo en ambos ambientes desplegados. |
| QA-07-10 | Logout revoca y elimina cookie | Cierre sesión desde el portal y revise la respuesta de `/auth/logout`. | El backend revoca el refresh token y envía cookie vacía expirada con los mismos atributos y ruta. El portal limpia el access token de memoria y el perfil local. | **PASS (API)** — `SecurityControllersHttpTest` valida logout con cookie y expiración `Max-Age=0`; falta interacción desde navegador. |
| QA-07-11 | Compatibilidad de la app móvil | Inicie sesión y cierre sesión desde la app nativa contra API HTTPS. | La app mantiene su flujo de tokens con `expo-secure-store`; el cambio de cookie web no rompe el cliente móvil. | **PARCIAL** — app 7/7 pruebas y bundle Android pasan; no hay Android SDK/emulador para probar login/logout en dispositivo. |

## Evidencia de ejecución local

- API: imagen Docker construida; API y PostgreSQL en estado `healthy`, S3Mock y MailHog activos; health endpoint responde HTTP 200. CORS para `http://localhost:5173` responde con credenciales habilitadas.
- API: la clase `SecurityControllersHttpTest` pasa 6/6, incluyendo login, refresh y logout con cookie, usando claves RSA efímeras locales eliminadas después de la prueba.
- API: la suite completa ejecutó 160 pruebas; 7 tuvieron error de entorno (claves RSA de prueba ignoradas por Git ausentes y Testcontainers sin acceso al Docker socket desde el contenedor Maven).
- Portal: `npm run build` y 6 pruebas pasan.
- App: 7 pruebas pasan; `expo prebuild` y export Android pasan. No se pudo iniciar emulador porque Android SDK/ADB no está instalado.
- Device: 15 pruebas de `tests/unit/test_config.py` pasan. La suite completa requiere dependencias que no están instaladas (por ejemplo, NumPy) y permisos de temporales disponibles.
- Compose: el `.env` local no incluía `POSTGRES_DB`; se agregó `POSTGRES_DB=somnguard` (no secreto) y se confirmó `config --quiet` y `up -d` sin variables de proceso temporales. No se imprimieron ni cambiaron otros valores del `.env`.
- Los servidores del portal y Expo se iniciaron, pero esta sesión no pudo alcanzar sus puertos locales para verificar interacción visual/autenticación.
- La instalación con `npm ci` reportó vulnerabilidades de dependencias: 3 altas en portal y 5 altas (55 en total) en la app; deben triagearse por separado antes de una liberación.

### Preparación para el caso QA-07-04 en PowerShell

```powershell
$env:SOMNGUARD_ENV = 'production'
$env:SOMNGUARD_API_URL = 'http://localhost:8080'
py -c "from app.common.config import load_env_config; load_env_config()"
```

Restablezca las variables al terminar la prueba o cierre esa terminal. No copie valores del `.env` real al reporte.

## Criterio de cierre

- [ ] QA-07-01 a QA-07-11 ejecutados y con evidencia redactada.
- [ ] No hay secretos, refresh tokens ni datos personales en las evidencias.
- [ ] No hay defectos P0/P1 abiertos relacionados con los criterios del punto 7.
- [ ] QA firma el resultado y deja observaciones antes del merge.

## Rutas modificadas por la implementación

- API: `docker-compose.yml`, `.env.example`, `src/main/resources/application.yml`, `src/main/resources/application-dev.yml`, `src/main/java/com/somnguard/security/adapter/in/web/AuthController.java`, DTOs `RefreshRequest.java` y `LogoutRequest.java`, `src/test/resources/application-test.yml`, `src/test/java/com/somnguard/security/adapter/in/web/SecurityControllersHttpTest.java`.
- App: `SomnGuard-app/app.json`, `SomnGuard-app/.env.example`.
- Device: `app/common/config.py`, `app/device/manager.py`, `app/streaming/publisher.py`, `scripts/stream_publish.py`, `.env.example`, `README.md`, `tests/unit/test_config.py`.
- Portal: `src/shared/api/secureStorage.ts`, `src/shared/api/client.ts`, `src/features/auth/api/auth.api.ts`, `src/features/auth/model/AuthContext.tsx`, `README.md`.
- Docs: `docs/11-quality-assurance/qa-report-point-7-secrets-transport.md`.

## Limitaciones conocidas

- No se encontró un clon ni un remoto `somnguard-drive`; `somnguard-device` se clonó desde la rama `develop` del remoto SomnGuard para poder cubrir el criterio TLS.
- QA todavía debe ejecutar los casos en un ambiente con Android, API y portal configurados; este documento deja los pasos y resultados esperados, no declara pruebas aprobadas.
