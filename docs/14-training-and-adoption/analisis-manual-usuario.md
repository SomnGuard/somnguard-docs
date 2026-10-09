# Análisis para el manual de usuario de SomnGuard

**Fecha de revisión:** 9 de octubre de 2026  
**Versión app revisada:** `somnguard-app` 
**Fuente guía:** `manual-usuario.pdf` (21 páginas)  
**Alcance:** repositorios `somnguard-portal`, `somnguard-app`, `somnguard-api`, `somnguard-db`, `somnguard-device` y `somnguard-docs`.

## 1. Informe de análisis

### 1.1 Guía de referencia

El PDF es un ejemplo de manual de usuario terminado para Simple Stock Flow, no una norma técnica general. Sirve como guía editorial y de estructura. Sus elementos aplicables son: portada con producto, versión, fecha y público; índice; explicación de propósito y perfiles; acceso y salida; descripción de la interfaz; capítulos por tarea; pasos numerados con nombres visibles de botones y campos; tablas de campos, permisos y mensajes; figuras numeradas con pie explicativo; preguntas frecuentes; y aclaraciones cuando la interfaz no ofrece una operación.

El estilo es directo, dirigido a quien usa el sistema, con frases breves y vocabulario de la interfaz. Las instrucciones explican resultado esperado, restricciones por perfil, estados vacíos y mensajes de error. El ejemplo distingue lo que puede hacer cada tipo de cuenta y deja fuera capacidades que solo existen internamente. Para SomnGuard se conserva este enfoque, se cubren tanto el portal web como la app móvil y se añaden advertencias donde el código revela diferencias entre una pantalla y el control real del dispositivo. No se copia el contenido ni la organización específica de inventario/ventas del ejemplo.

### 1.2 Mapa real del repositorio

| Área | Contenido encontrado | Papel en la experiencia de usuario |
|---|---|---|
| `somnguard-portal/` | SPA React + TypeScript + Vite, rutas protegidas, vistas, formularios y cliente API | Portal web para usuarios y administradores |
| `somnguard-app/SomnGuard-app/` | App React Native + Expo Router, autenticación, pestañas, perfil y monitoreo LiveKit/relay | App móvil para consultar estado/eventos, cuenta y video en vivo |
| `somnguard-api/` | Backend Java/Spring modular; seguridad, dispositivos, telemetría, notificaciones, catálogos y streaming | Autenticación, permisos y operaciones que sirven las pantallas |
| `somnguard-db/` | Migraciones Liquibase; esquemas de seguridad, parametrización, dispositivos, telemetría, notificaciones y analítica | Entidades y relaciones persistentes |
| `somnguard-device/` | Servicio de borde en Python para cámara, análisis, alertas sonoras, buffer y sincronización | Detecta y comunica eventos; no es una pantalla de usuario final |
| `somnguard-docs/` | Gobernanza, requisitos, arquitectura, datos, contratos API, módulos, operación y capacitación | Evidencia y contexto complementario; existe onboarding técnico, no manual de usuario vigente |

Los nombres genéricos mencionados en la solicitud no coinciden literalmente con todas las carpetas. La estructura real tiene `somnguard-portal` además de `somnguard-app`; el código fuente de la app está anidado en `SomnGuard-app/`. `somnguard-docs` ya contiene documentación extensa. Se eligió `docs/14-training-and-adoption/` porque su README declara esa sección como destino de manuales, guías de capacitación y adopción.

### 1.3 Funcionalidades verificadas

**Portal web, usuario (`/user/*`):** inicio con accesos a monitoreo e historial; vista Monitoreo con estado, conexión automática a un dispositivo asociado, video en vivo cuando los servicios de streaming están disponibles, control de cámara y pausa/reanudación de detección; historial personal con filtro por severidad/fechas, paginación, detalle y evidencia cuando exista; notificaciones personales con acción de marcar como leída; preferencias de notificación.

**Portal web, administrador (`/admin/*`):** resumen general; inventario con filtro y paginación; registrar, editar, asignar y desasignar dispositivos; vincular mediante código; crear token de aprovisionamiento; consultar detalle y rotar API Key; editar y publicar configuración por dispositivo; consultar eventos con filtros y abrir evidencia; consultar y marcar avisos, además de reintentar notificaciones pendientes; mantener roles, funcionalidades/permisos y catálogos de eventos/severidades/sonidos/medios.

**App móvil:** iniciar sesión, registro, verificación de correo y recuperación de contraseña; pestañas Inicio, Monitoreo, Historial y Ajustes/Perfil; cuenta, dispositivo vinculado/desvinculado, historial filtrable con detalle/evidencia, preferencias, notificaciones, seguridad, privacidad, soporte y cierre de sesión. La sesión puede restaurarse al volver a abrir la app mediante los tokens almacenados en SecureStore y renovarse con la API; si el almacenamiento seguro no está disponible, la sesión solo permanece en memoria. En Monitoreo, la app inicia/detiene una sesión de video y controla por separado la pausa/reanudación de detección vía API. Video nativo LiveKit y conexión de respaldo relay están implementados; su disponibilidad depende de la configuración del entorno y la compilación nativa. Privacidad/soporte aún muestran mensajes locales, sin descarga/contacto/política externa.

**Dispositivo:** Raspberry Pi/Windows con cámara, análisis local, alertas sonoras y sincronización diferida. Puede trabajar sin conexión y guardar eventos pendientes. Estos procesos son automáticos y no se presentan como operaciones que el usuario final ejecuta en una pantalla. La distribución del dispositivo, instalación, conexión física y calibración requieren guía operativa aparte.

### 1.4 Perfiles, permisos y límites de interfaz

El portal distingue roles `ADMIN` y `USER`; las semillas de base de datos los nombran Administrador y Usuario. Las rutas se protegen con autenticación, rol y permisos. El administrador dispone de gestión y parametrización; el usuario estándar ve recursos asociados a su cuenta. La app móvil no muestra un selector de rol.

La ruta web de seguridad importa `UsersPage`, pero la página dice explícitamente que el listado está pendiente y muestra texto de ejemplo. Por eso no se documenta un módulo web de administración de usuarios. La creación de cuentas sí aparece en el flujo público de registro. Las pantallas móviles de privacidad/soporte incluyen botones que abren mensajes informativos locales; no se describe como si descargaran datos o abrieran un canal de contacto. En la versión actual, Privacidad ya no muestra el control anterior de consentimiento; conserva la información de tratamiento y el botón de descarga informativa.

En la app, `Iniciar monitoreo` abre la pantalla de Monitoreo. Allí **Reanudar cámara/Pausar cámara** solicita y cierra una sesión de streaming, y **Pausar detección/Reanudar detección** llama por separado a la API para cambiar la detección. La vista implementa LiveKit nativo, relay de respaldo, estados de conexión y eventos del día. La pantalla detiene la sesión de cámara al perder foco. En el portal web, `AutoLiveBox` también gestiona streaming y controles de detección. La presencia del código no garantiza que el entorno desplegado tenga LiveKit/relay configurados ni que una compilación Expo Go incluya el módulo nativo.

El video en vivo móvil y web depende del dispositivo y de la disponibilidad/configuración del backend y LiveKit/relay. Cuando no hay video, la app muestra estados o errores de conexión; la ausencia de video no prueba por sí sola que la detección se haya detenido. Los controles de cámara y detección son independientes.

### 1.5 Evidencia principal inspeccionada

- `somnguard-portal/src/app/router/AppRouter.tsx`, `paths.ts`, `layouts/AdminLayout.tsx`, `layouts/UserLayout.tsx`: rutas, perfiles, módulos y guards.
- `somnguard-portal/src/features/auth/ui/AuthModals.tsx` y páginas de verificación/restablecimiento: campos y flujo web de acceso.
- `somnguard-portal/src/features/device-management/ui/DeviceListPage.tsx`, `DeviceConfigPage.tsx`, `src/features/user-device/ui/MyDevicePage.tsx`, `src/features/streaming/ui/AutoLiveBox.tsx`: gestión de equipos, configuración y monitoreo.
- `somnguard-portal/src/features/user-events/ui/`, `user-alerts/ui/`, `notifications/ui/`, `security/ui/`: eventos, avisos, preferencias, seguridad y catálogos.
- `somnguard-app/SomnGuard-app/src/app/` y `src/features/{auth,dashboard,monitoring,history,profile}/`: rutas y pantallas móviles.
- `somnguard-app/SomnGuard-app/src/features/monitoring/{screens,components,hooks,services}/`: cámara LiveKit/relay, sesión de streaming y control de detección.
- `somnguard-app/SomnGuard-app/src/shared/api/{session,sessionStore,client}.ts` y `src/features/splash/screens/SplashScreen.tsx`: restauración de sesión y persistencia segura de tokens.
- `somnguard-api/src/main/java/com/somnguard/*/adapter/in/web/*Controller.java`: contratos de autenticación, cuenta, dispositivo, evento, evidencia, notificaciones, catálogos y streaming.
- `somnguard-db/01_ddl/03_tables/` y `02_dml/00_inserts/`: entidades persistidas y semillas de roles/permisos.
- `somnguard-device/README.md`, `app/device/`, `app/analysis/`, `app/sync/`, `config/device.default.json`: operación del nodo edge.
- `somnguard-docs/docs/07-api-design/`, `06-data-architecture/`, `09-modules/`, `14-training-and-adoption/technical-onboarding.md`: contratos y contexto; se usaron como complemento, priorizando implementación de la UI.

### 1.6 Limitaciones del análisis visual

Se inició el portal Vite localmente, pero el entorno no contiene el ejecutable Chromium que requiere Playwright. No se generaron capturas. El manual incluye referencias numeradas y espacios explícitos para capturas reales de acceso, paneles, monitoreo e historial; deben sustituirse por imágenes tomadas en una instalación autorizada, sin mostrar credenciales, tokens, identificadores personales o evidencias sensibles.

## 2. Matriz de trazabilidad entre guía, manual y evidencia

| Sección recomendada por la guía | ¿Aplica? | Contenido de SomnGuard que se documenta | Evidencia del repositorio | Validación pendiente |
|---|---|---|---|---|
| Portada, versión y fecha | Sí | Nombre, edición de revisión, fecha y público del portal/app | README raíz de `somnguard-docs`; repositorios de UI | Confirmar versión comercial y responsable editorial |
| Índice | Sí | Navegación por capítulos de portal y app móvil | Estructura real de rutas y procedimientos | Actualizar páginas al cambiar el diseño final |
| Propósito y alcance | Sí | Monitoreo de somnolencia/fatiga, consulta y administración | `somnguard-device/README.md`; docs de contexto; pantallas | Precisar alcance contractual del producto desplegado |
| Público objetivo y permisos | Sí | Administrador y usuario estándar; límites en las rutas protegidas | `AppRouter.tsx`, guards, seeds `009_insert_security_roles.sql` y `012_insert_security_role_feature.sql` | Confirmar matriz de permisos del ambiente y nombres visibles de roles |
| Requisitos previos | Sí, con cautela | Cuenta activa, conexión, equipo asociado; streaming condicionado | `RequireAuth`, `profileService`, API device/stream, README API/device | Navegadores, versiones de Android/iOS, red, instalación y URL productiva |
| Acceso, registro y salida | Sí | Correo/contraseña, registro, verificación, recuperación, cierre | `AuthModals.tsx`, auth API; pantallas `features/auth` en app; `AuthController`, `AccountController` | Vigencia y entrega de códigos; disponibilidad de correo; activación inicial de administradores |
| Interfaz y navegación | Sí | Menús del portal por rol y pestañas móviles | `AdminLayout.tsx`, `UserLayout.tsx`, `ShellLayout.tsx`, `app/(tabs)/_layout.tsx` | Nombres finales, capturas y diseño de la versión publicada |
| Módulos y procesos | Sí | Monitoreo web/móvil, transmisión móvil LiveKit/relay, dispositivo, historial/eventos, avisos y preferencias | Vistas web `src/features/*/ui`; rutas Expo; `features/monitoring` móvil y servicios API | Validar funciones habilitadas en producción y permisos asignados |
| Campos, botones y acciones | Sí | Formularios de dispositivos, filtros de eventos, catálogo, registro, cuenta | Componentes UI y validaciones | Confirmar textos exactos tras probar la versión desplegada |
| Resultado y mensajes | Sí | Toasts, estados vacíos, restricciones de dispositivos/streaming y errores de API | `shared/api/errors.ts`; páginas y controladores | Confirmar mensajes presentados para errores reales y su resolución operativa |
| Gestión de información | Sí | Consultar eventos/evidencias; marcar notificaciones; editar catálogos/configuración con perfil admin | `EventQueryController`, `NotificationController`, páginas UI y esquema Liquibase | Retención, descarga/exportación y reglas de tratamiento de evidencia |
| Dispositivos | Sí | Alta, asignación, vínculo, token, estado, heartbeat, configuración y vista en vivo | `DeviceListPage.tsx`, `DeviceConfigPage.tsx`, `AutoLiveBox.tsx`; API/device | Procedimiento físico de instalación, recuperación, cobertura y red aprobada |
| Solución de problemas | Sí | Sin equipo, equipo desconectado, streaming sin imagen, filtros vacíos, credenciales/código inválido | Mensajes UI, `getUserMessage`, `AutoLiveBox`, servicios | Teléfono/canal de soporte, SLA y escalamiento |
| Recomendaciones y glosario | Sí | Uso seguro de monitor, evento, evidencia, severidad, heartbeat y sincronización | `docs/02-domain/glossary.md`; README dispositivo | Aprobación de terminología dirigida al usuario |
| Capturas | Sí, pendiente | Referencias de imagen en pasos representativos | No había capturas del entorno; ejecución Vite posible, Chromium ausente | Tomar capturas reales en QA con datos sintéticos y autorización |
| Anexos técnicos | Parcial | Solo equivalencias útiles de estados y pasos de ayuda | API/DB/device como evidencia | Se excluyen endpoints, SQL, credenciales y configuración interna del manual final |

## 3. Índice propuesto y adoptado

1. Antes de empezar: propósito, alcance, audiencia, plataformas y requisitos.
2. Cuentas y permisos: Administrador y Usuario, y diferencias de acceso.
3. Acceso al sistema: iniciar sesión, crear cuenta, verificar correo, recuperar contraseña y salir.
4. Portal web para usuarios: inicio, monitoreo, cámara/detección, historial, evidencia y notificaciones.
5. App móvil: Inicio, estado de monitoreo, historial, dispositivo y perfil.
6. Portal web para administradores: resumen, inventario y vinculación de dispositivos.
7. Configuración del dispositivo: consultar/publicar parámetros y solicitar sincronización.
8. Eventos, evidencias y notificaciones: consulta, filtros, detalle, lectura y reintentos.
9. Seguridad y catálogos: roles, funcionalidades y catálogos de parametrización.
10. Mensajes y solución de problemas frecuentes.
11. Recomendaciones de uso.
12. Glosario.
13. Capturas pendientes y pendientes de confirmación.

## 4. Pendientes de validación

1. Confirmar dirección oficial del portal, disponibilidad por ambiente y proceso real para obtener una cuenta administradora.
2. Confirmar versiones de navegador, Android/iOS, dispositivos móviles y procedimiento de distribución. El recurso público `SOMNGUARD_APK_EN_PROCESO.pdf` indica que no debe inferirse que la descarga de Android esté lista.
3. Confirmar política de registro: quién puede crear cuentas, cuándo se asigna rol y si el correo debe quedar verificado antes de iniciar sesión.
4. Validar entrega, expiración y reenvío de códigos de verificación y recuperación, así como el canal de correo operativo.
5. Confirmar en builds nativos de QA la recepción de video por LiveKit y relay, los permisos/red requeridos y la limpieza de la sesión al salir de Monitoreo.
6. Validar instalación, encendido, ubicación/ángulo de cámara, parlante, red y recuperación física del equipo con el equipo de operaciones.
7. Confirmar cuáles ambientes tienen LiveKit/relay habilitado, cuánto tarda en aparecer video y qué hacer cuando la detección está activa pero no hay imagen.
8. Aprobar privacidad, retención, acceso, exportación y eliminación de imágenes/videos de evidencia, además de la política de tratamiento de datos.
9. Confirmar el significado operativo de todos los estados que la API muestra (incluidos `DEVICE_PENDING`, `DEVICE_ACTIVE`, `DEVICE_OFFLINE`, suspendido y retirado) y quién puede cambiarlos.
10. Confirmar qué catálogos, severidades y parámetros pueden modificar los administradores sin afectar detección; documentar valores permitidos y ruta de reversión.
11. Verificar mediante ejecución los mensajes de error de correo, cuenta bloqueada, código vencido, dispositivo sin heartbeat, evento sin evidencia y fallo de streaming.
12. Sustituir las referencias de captura por imágenes reales en un ambiente autorizado, con información sintética y sin secretos.
13. Revisar si el módulo de usuarios web debe implementarse o retirarse de navegación: la vista actual es un marcador de posición.
14. Revisar las pantallas de soporte y privacidad móviles: los controles visibles abren mensajes informativos locales y no realizan descarga ni contacto externo; verificar el texto de tratamiento de datos aprobado.

## 5. Criterio de fidelidad

El manual describe operaciones que tienen una ruta y comportamiento visibles. La existencia de una operación únicamente en un controlador o tabla no basta para presentarla como función de usuario. Las tareas de instalación, emisión de tokens y rotación de credenciales quedan restringidas a los flujos administrativos visibles y se señalan como acciones sensibles. No se incluyen credenciales de ejemplo, endpoints, consultas SQL ni pasos de instalación para desarrolladores.
