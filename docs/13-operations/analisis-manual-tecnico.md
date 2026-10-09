# Análisis del manual técnico de SomnGuard

**Fecha de revisión:** 9 de octubre de 2026  
**Alcance:** repositorios `somnguard-api`, `somnguard-db`, `somnguard-device`, `somnguard-portal`, `somnguard-app` y `somnguard-docs`.  
**Método:** lectura estática del código, configuración y documentación. No se levantaron servicios ni se ejecutaron pruebas o comandos de despliegue.

## 1. Análisis de la guía de referencia

El PDF de referencia tiene 33 páginas y está dirigido a personal que despliega, opera, mantiene o amplía un sistema. Su índice organiza el contenido en once capítulos: alcance y convenciones; arquitectura y directorios; requisitos; instalación y primer arranque; configuración; operación; API; mantenimiento del código; diagnóstico; comandos de consulta rápida; y documentación relacionada. Incluye una portada, índice, figuras de arquitectura y flujo, tablas comparativas, ejemplos de terminal, avisos y una fecha explícita de comprobación.

Su recomendación editorial principal es que el manual sirva para actuar sobre el sistema real. Separa lo que está desplegado de lo opcional, explica los requisitos y el efecto de los comandos, distingue operaciones reversibles de destructivas, usa la documentación ejecutable de la API como fuente del contrato y señala lo no verificado en vez de presentarlo como probado. La organización y ese criterio se adaptan aquí; no se trasladan los módulos de inventario ni los nombres, puertos o comandos de Simple Stock Flow.

## 2. Mapa del proyecto revisado

Los repositorios son directorios hermanos. La carpeta que los contiene no es un repositorio Git raíz ni se encontró un `docker-compose.yml` que arranque todos los componentes. Cada unidad conserva su configuración y ciclo de construcción.

| Repositorio | Implementación revisada | Función técnica |
|---|---|---|
| `somnguard-api/` | Java 21, Spring Boot 4.1.1, Maven; monolito modular con adaptadores web, persistencia, almacenamiento y streaming | Autenticación, permisos, gestión de dispositivos, eventos, notificaciones, analítica y contratos REST/WebSocket |
| `somnguard-db/` | PostgreSQL 16 y Liquibase; Compose propio para PostgreSQL y el servicio de migraciones | Seis esquemas de dominio y changelogs versionados |
| `somnguard-device/` | Python 3.11+, OpenCV/MediaPipe, almacenamiento local SQLite, cámara y sonido | Procesamiento en el borde, buffer offline, envío de eventos, heartbeat y publicación de video |
| `somnguard-portal/` | React 19, TypeScript, Vite 8 y `oxlint` | Portal web para usuarios y administración |
| `somnguard-app/SomnGuard-app/` | React Native, Expo SDK 57, Expo Router, LiveKit/WebRTC | App Android/iOS; los módulos nativos de video necesitan una compilación de desarrollo o APK |
| `somnguard-docs/` | Arquitectura, contratos, migraciones, DevOps, operación y onboarding | Fuente complementaria; varios documentos conservan descripciones que no coinciden con el código actual |

### 2.1 Baseline observado

Las ramas y commits observados durante la revisión fueron: API `chore/production-deployment` (`ddcf413`); DB `develop` (`90561d2`); dispositivo `develop` (`9540992`); portal `feat/endpoint-consumption-admin-panel` (`e180d72`); app `develop` (`963d6e9`); documentación `main` (`bd8726f`). El `README.md` de `somnguard-db` tenía cambios locales antes de esta revisión; se leyó y se dejó intacto. No hay un commit único que identifique una versión integrada de toda la solución.

## 3. Funcionalidades y procesos técnicos identificados

- **Base y migraciones:** PostgreSQL se levanta desde `somnguard-db/docker-compose.yml`; Liquibase toma `changelog/changelog-master.yaml` y los archivos SQL agrupados por DDL, DML, DCL, TCL y rollbacks. La configuración de desarrollo usa el puerto 5432 por omisión.
- **API:** el Compose del repositorio de API inicia el backend y S3Mock; el servidor de base de datos no está incluido en ese Compose. LiveKit y Coturn son servicios opcionales bajo el perfil `livekit`. La API publica por defecto el puerto 8080, Swagger UI en `/swagger-ui.html` y Actuator en `/actuator/health`.
- **Autorización:** los usuarios usan JWT RS256. El dispositivo se autentica con `X-Device-ID` y `X-API-Key`; el registro inicial usa un token de aprovisionamiento temporal. La API tiene grupos de rutas para autenticación, dispositivos, streaming, telemetría/eventos, catálogos, notificaciones y administración. Swagger/OpenAPI define los cuerpos exactos.
- **Evidencia:** los metadatos y referencias viven en PostgreSQL; los objetos multimedia usan un almacenamiento S3 compatible. La composición local usa S3Mock con volumen. El endpoint y las credenciales de producción dependen del ambiente.
- **Dispositivo:** el proceso admite Windows, Linux y Raspberry Pi OS según su README, requiere Python 3.11+ y cámara; el sonido local requiere parlante. El código conserva eventos pendientes en SQLite y continúa el procesamiento local durante una caída de red. La sincronización se reintenta al restablecerse la conectividad.
- **Video:** API, portal, app y dispositivo incluyen LiveKit/relay. LiveKit no se inicia en el Compose normal; se requiere habilitarlo y anunciar una dirección accesible desde cada cliente. Expo Go no incorpora el módulo nativo LiveKit. El APK de Android se configura como distribución interna EAS.
- **Web y móvil:** el portal tiene comandos de desarrollo, lint y build. La app móvil tiene lint y perfiles EAS para builds internos; su perfil `production` no declara variables de entorno en `eas.json`.
- **CI:** el workflow del API configura Java 21, compila, ejecuta Maven tests, Checkstyle y un Docker build. No se encontró un pipeline equivalente de raíz que publique y coordine todos los repositorios.

## 4. Discrepancias y límites relevantes

1. `docs/14-training-and-adoption/technical-onboarding.md` todavía llama al backend un esqueleto y describe repositorios/carpetas que ya no corresponden al mapa actual. El código y los repos separados son la evidencia prioritaria.
2. `docs/10-devops/local-setup.md` indica `/health` y un perfil Maven `develop`. El API actual configura Actuator bajo `/actuator` y contiene `application-dev.yml`; la comprobación y el perfil descritos por ese onboarding están desfasados.
3. El perfil API `dev` configura `spring.jpa.hibernate.ddl-auto: update`, aunque la guía DevOps asigna los cambios de esquema a Liquibase en `somnguard-db`. No se debe usar el perfil de desarrollo contra datos compartidos hasta resolver esa responsabilidad.
4. `docs/10-devops/environments.md` declara que el despliegue por ambiente sigue siendo objetivo. La presencia de perfiles `qa`/`prod`, ramas o configuración no demuestra que exista un despliegue productivo operativo.
5. `docs/13-operations/backup-and-recovery.md` deja RTO/RPO, PITR, destino de respaldos y prueba de restauración como puntos abiertos. Los comandos ahí documentados no se ejecutaron en esta revisión; el manual no los presenta como un restore probado.
6. `docs/13-operations/observability.md` describe parte del stack de métricas/trazas como pendiente de despliegue. El healthcheck documentado en esa guía tampoco debe confundirse con las rutas Actuator del API.
7. En `somnguard-device/scripts/`, `provision.sh`, `install_service.sh`, `update.sh` y `factory_reset.sh` existen pero tienen cero bytes. No se describen los objetivos del `Makefile` que dependen de ellos como procesos automatizados listos para producción.
8. El dispositivo mantiene comentarios antiguos sobre pausar detección al quedar `OFFLINE`; la lógica actual y el README dicen que el monitoreo local continúa y guarda eventos para sincronización posterior.
9. En la app móvil, el video nativo requiere configuración LiveKit alcanzable y build con módulos nativos. Código de UI presente no confirma la disponibilidad del servicio en cada red o despliegue.
10. Los archivos `.env.example` incluyen valores de demostración y las claves RSA de desarrollo están bajo `somnguard-api/src/main/resources/keys/dev/`. No deben copiarse a ambientes compartidos o productivos ni publicarse secretos reales.

## 5. Matriz de trazabilidad

| Recomendación de la guía | Aplica | Contenido adaptado a SomnGuard | Evidencia principal | Pendiente o límite |
|---|---|---|---|---|
| Alcance, público y convenciones | Sí | Operación, desarrollo e integración de los seis repositorios; comandos etiquetados por directorio y ambiente | `docs/13-operations/`, READMEs por repo | Confirmar lector responsable y política de publicación |
| Arquitectura y mapa de repositorios | Sí | API, DB/Liquibase, edge, portal, app y documentación; ausencia de orquestador común | Carpetas reales; `docker-compose.yml` API/DB; README de cada repo | Aprobar topología desplegada de cada ambiente |
| Requisitos y versiones | Sí | Java/Maven, Docker/Compose, Node, Python y hardware; dependencias distintas por componente | `pom.xml`, `package.json`, `pyproject.toml`, Compose y READMEs | Versiones mínimas de Node, Compose, Android/iOS y hardware validado |
| Instalación y arranque | Sí, parcial | Postgres → migraciones → API; portal, APK y edge se levantan por separado | `10-devops/local-setup.md`, Compose, scripts y READMEs | Ejecutar el flujo completo en una máquina de QA; crear guía de despliegue integrada |
| Configuración | Sí | `.env.example`, perfiles API, URL cliente, PostgreSQL, S3, correo, FCM, LiveKit y dispositivo | `.env.example` por repo; `application*.yml`; `eas.json` | Secret Manager, valores reales, configuración productiva de TLS/red |
| Salud, logs y operación | Sí | Actuator API, `docker compose ps/logs`, salud PostgreSQL y logs del edge | `SecurityConfig`, `application.yml`, Compose y `device/app` | Monitoreo/alertas instalados, retención y accesos por ambiente |
| API y seguridad | Sí | JWT de usuario; API key e ID del dispositivo; grupos de endpoints y Swagger | `*Controller.java`, `SecurityConfig.java`, `swagger-ui` | Exportar OpenAPI del build integrado y matriz vigente de permisos |
| Base de datos y migraciones | Sí | Seis esquemas Liquibase, orden del changelog, `status`; rollback local y forward-only QA/main | `changelog-master.yaml`, `10-devops`, README DB | Resolver `ddl-auto:update` en `dev`; validar restore real |
| Aplicación edge | Sí | Instalación Python, modelos, identidad, aprovisionamiento, modo offline y streaming | `somnguard-device/README.md`, `pyproject.toml`, `app/` | Automatización de instalación/reset vacía; homologación del hardware/modelos |
| Portal/app y release | Sí | `npm run dev/build/lint`; Expo/EAS; requerimiento de cliente nativo para LiveKit | `package.json`, `eas.json`, `app.json`, componentes de streaming | Canal de distribución, pipeline y variables productivas |
| Respaldo y recuperación | Sí, con reserva | DB + medios deben respaldarse juntos; no existe evidencia de restore drill | `13-operations/backup-and-recovery.md`, Compose DB/API | RTO/RPO, destino/cifrado, PITR y ensayo no destructivo |
| Mantenimiento y calidad | Sí | Maven/CI, build/lint del portal, lint/build app, tests/lint edge, migraciones | Workflows, scripts de `package.json`, Makefile y `pom.xml` | Estado real de cada pipeline; no se ejecutó ninguna suite en esta revisión |
| Diagnóstico | Sí | Fallas de conexión, schema, auth, streaming, edge offline, almacenamiento y compilación | Excepciones API, mensajes UI, configuración y logs | Ejecutar escenarios integrados y asignar canal/on-call |
| Figuras y comandos ejecutados | Sí, parcial | Diagrama lógico y comandos de referencia, con su fuente y condición | Repositorios actuales y documentación | Validar comandos en entorno QA; no se tomaron capturas del sistema ejecutándose |
| Documentos relacionados | Sí | Arquitectura, API, migraciones, setup, operación, seguridad y onboarding | `somnguard-docs/docs/*` | Corregir páginas de onboarding y local setup que describen versiones anteriores |

## 6. Índice propuesto

1. Sobre este manual: audiencia, alcance, convenciones y estado de verificación.
2. SomnGuard de un vistazo: topología, repositorios, dependencias, puertos y componentes opcionales.
3. Requisitos previos por rol técnico.
4. Preparar el entorno de desarrollo y levantar cada componente.
5. Configuración y manejo de secretos por repositorio.
6. Operación, salud, logs, base de datos y streaming.
7. API: autenticación, rutas por módulo, contratos y errores.
8. Edge: instalación local, modelos, identidad, provisión, buffer offline y límites actuales.
9. Mantenimiento: compilación, validaciones disponibles, capas y migraciones.
10. Respaldo, recuperación y actualización de versión.
11. Diagnóstico de problemas frecuentes.
12. Referencia rápida de comandos y documentación relacionada.

## 7. Pendientes de validación antes de usarlo como runbook de producción

1. Probar el arranque conjunto en una máquina QA y registrar versiones, nombres reales de servicios, URLs y resultados de salud.
2. Definir configuración de ambiente y despliegue integrado: proxy/TLS, DNS, puertos, secretos, URL pública y pipeline de publicación.
3. Acordar el perfil API para desarrollo: Liquibase frente a `ddl-auto:update`; confirmar que ningún proceso modifique un esquema compartido inadvertidamente.
4. Aprobar la configuración S3 productiva y el tratamiento/cifrado de evidencia.
5. Ejecutar backup y restauración en un entorno efímero; definir frecuencia, destino, cifrado, RTO/RPO y recuperación de evidencia junto con PostgreSQL.
6. Completar scripts de provisión, instalación, actualización y reset del edge; documentar procedimiento manual, responsable y permisos mientras tanto.
7. Validar LiveKit y TURN desde Android/iOS en LAN y red móvil; fijar puertos, URL WSS/TLS y compilación firmada.
8. Confirmar distribución móvil y completar variables del perfil EAS `production`.
9. Confirmar el estado disponible de detección de teléfono/cinturón y qué modelos están aprobados para los dispositivos de campo.
10. Confirmar retención de logs/eventos/evidencias, observabilidad desplegada, guardia operativa y contactos de incidente.
11. Actualizar `technical-onboarding.md`, `local-setup.md` y la política para la migración de esquemas para que coincidan con el estado real.

## 8. Criterio de fidelidad aplicado

El manual técnico explica los límites del código y de la infraestructura encontrados. Los comandos se presentan como instrucciones respaldadas por archivos de configuración o documentos del proyecto, no como una ejecución certificada. Se omiten valores secretos, se separa desarrollo de QA/producción y se identifican acciones no verificadas o scripts vacíos.
