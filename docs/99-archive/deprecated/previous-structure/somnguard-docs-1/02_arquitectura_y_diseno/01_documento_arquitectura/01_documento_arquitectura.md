# Documento de Arquitectura

**Proyecto:** SomnGuard  
**Autor:** Equipo SomnGuard  
**Fecha de creación:** 2026-05-14  
**Estado:** Aprobado

---

## 1. Propósito

Este documento describe la arquitectura general de SomnGuard, cubriendo la plataforma web, la app móvil, el backend, el dispositivo Raspberry Pi con cámara, la base de datos y la sincronización entre componentes.

Su objetivo es dejar trazada una vista técnica clara del sistema para guiar el desarrollo, la integración y las decisiones posteriores.

## 2. Alcance

SomnGuard se concibe como un sistema integral de software y hardware para monitoreo de somnolencia y fatiga al volante. La arquitectura incluye:

- Plataforma web para descarga de la app, acceso de administradores y dashboard de consulta.
- App móvil para usuario final.
- Backend en C#/.NET como API principal.
- Dispositivo Raspberry Pi con cámara y software local de captura y visión.
- Base de datos PostgreSQL administrada con Liquibase.
- Servicio externo o independiente para almacenamiento de evidencia multimedia.

Quedan fuera de este documento las implementaciones detalladas de cada módulo, que se documentan en archivos técnicos o funcionales complementarios.

## 3. Contexto del sistema

SomnGuard busca detectar patrones de fatiga, somnolencia y microsueños para generar alertas preventivas y registrar evidencia asociada a eventos críticos. El sistema combina captura en edge, procesamiento local, sincronización hacia servidor y consulta desde plataforma web y móvil.

### 3.1 Actores principales

- Usuario final o conductor.
- Administrador de plataforma.
- Dispositivo SomnGuard instalado en campo.
- Backend central.
- Servicio de almacenamiento multimedia.

### 3.2 Flujo general

1. El dispositivo captura datos y evidencia desde la cámara.
2. El software local analiza o prepara la información para envío.
3. El backend recibe eventos, metadatos y referencias a evidencia.
4. La base de datos guarda la información estructurada.
5. La web y la app móvil consultan información y muestran alertas o dashboard.

## 4. Vista arquitectónica general

SomnGuard se organiza como una arquitectura híbrida edge-cloud con un backend modular.

- En el edge, el dispositivo Raspberry Pi ejecuta captura y lógica local básica.
- En el servidor, una API en C#/.NET centraliza reglas, autenticación, eventos, alertas y consultas.
- La persistencia estructurada vive en PostgreSQL.
- La evidencia multimedia se almacena fuera de la base de datos y se referencia mediante URL o identificador.

La forma de trabajo recomendada para el backend es un monolito modular con principios de clean architecture, de modo que cada dominio quede separado por responsabilidad sin fragmentar demasiado el despliegue.


```
[Device: Raspberry Pi] --- (sync/API) ---> [API .NET] ---> [PostgreSQL]
	|                                 
	+--- (multimedia) --------------> [Multimedia Storage]

[Web Client]
[Mobile Client] --> (API) --> [API .NET]
```

## 5. Estilo de arquitectura

### 5.1 Backend

El backend se define como un monolito modular en C#/.NET, organizado por dominios funcionales y con separación clara entre capa de presentación, aplicación, dominio e infraestructura.

### 5.2 Frontend

Se contemplan dos clientes:

- Web en React JS.
- Móvil en React Native.

### 5.3 Edge

El dispositivo Raspberry Pi funciona como nodo de captura y preprocesamiento. Su software local debe ser capaz de operar aunque exista conectividad limitada o nula.

## 6. Componentes principales

### 6.1 Plataforma web

La plataforma web cumple dos funciones principales:

- Página de acceso para descarga de la app móvil.
- Login y dashboard para administradores.

### 6.2 App móvil

La app móvil está orientada al usuario final y a la experiencia de consulta o recepción de información del sistema.

### 6.3 API backend

La API centraliza:

- Autenticación y autorización.
- Gestión de dispositivos asociados.
- Registro de eventos y evidencias.
- Seguimiento de alertas.
- Consulta de catálogos y parametrización.

### 6.4 Dispositivo Raspberry Pi

El dispositivo concentra:

- Captura desde cámara.
- Lógica local de visión o preparación de datos.
- Buffer local para operación offline.
- Sincronización posterior con el servidor.

### 6.5 Persistencia

La base de datos PostgreSQL almacena entidades estructuradas, catálogos, usuarios, dispositivos, eventos, alertas y referencias a evidencia multimedia.

### 6.6 Almacenamiento multimedia

La evidencia multimedia no se guarda directamente en PostgreSQL. Se almacena en un servicio aparte y la base de datos conserva solo la referencia, URL o identificador asociado.

## 7. Módulos funcionales del backend

Los módulos funcionales que deben existir a nivel de API son los siguientes:

### 7.1 Security

Responsable de cuentas, inicio de sesión, roles, permisos, módulos, formularios y asignación de accesos.

### 7.2 DeviceManagement

Responsable de asociar, desasociar y configurar dispositivos.

### 7.3 EventIngestion

Responsable de registrar eventos, metadatos, telemetría y evidencias desde el dispositivo.

### 7.4 Monitoring

Responsable de seguimiento de alertas, notificaciones y estados de seguimiento.

### 7.5 Parameterization

Responsable de catálogos como estados, severidad, tipo de medio, categoría de evento, patrón de sonido y tipo de evento.

## 8. Backend y clean architecture

La API se sugiere estructurar con una arquitectura limpia por módulos. Cada módulo debe organizarse al menos en:

- Presentación o API.
- Aplicación.
- Dominio.
- Infraestructura.

Con ese enfoque se mantiene un solo despliegue principal, pero con fronteras internas claras que facilitan crecimiento y mantenimiento.

### 8.1 Beneficios esperados

- Separación de responsabilidades.
- Menor acoplamiento entre dominios.
- Mejor mantenibilidad.
- Facilidad para escalar componentes si el proyecto lo requiere más adelante.

## 9. Datos y persistencia

### 9.1 Base de datos

La persistencia estructurada usa PostgreSQL.

### 9.2 Migraciones

Se emplea Liquibase para controlar cambios de esquema y versionado de base de datos.

La estructura prevista por carpetas para scripts de base de datos puede organizarse así:

```text
somnguar-db/
├── 01_ddl/
│   ├── 00_extensions/
│   ├── 01_schemas/
│   ├── 02_types/
│   ├── 03_tables/
│   ├── 04_views/
│   ├── 05_materialized_views/
│   ├── 06_functions/
│   ├── 07_procedures/
│   ├── 08_triggers/
│   └── 09_indexes/
├── 02_dml/
│   ├── 00_inserts/
│   ├── 01_updates/
│   ├── 02_deletes/
│   ├── 03_upserts/
│   └── 04_patches/
├── 03_dcl/
│   ├── 00_roles/
│   ├── 01_grants/
│   └── 02_policies/
├── 04_tcl/
│   ├── 00_transaction_blocks/
│   ├── 01_manual_recoveries/
│   └── 02_release_tags/
└── 05_rollbacks/
	├── 01_ddl/
	├── 02_dml/
	├── 03_dcl/
	└── 04_tcl/
```

Cada carpeta agrupa scripts relacionados por tipo de operación y facilita su mantenimiento, ejecución y reversión.

### 9.3 Entidades principales

La arquitectura de datos se alinea con los módulos ya definidos en el proyecto:

- person, user, role, permission, module, form.
- device, device_assignment, device_config.
- event, evidence.
- alert, notification.
- status_catalog, media_type_catalog, severity_catalog, event_category_catalog, sound_pattern, event_type.

## 10. Autenticación y autorización

El sistema debe contemplar:

- Registro y validación de cuenta.
- Inicio y cierre de sesión.
- Gestión de perfil.
- Asignación de roles a usuarios.
- Permisos por funcionalidad o formulario.
- Control de acceso por módulo.

La arquitectura debe permitir distinguir al menos entre usuario final y administrador de plataforma, aunque el modelo de roles pueda crecer después.

## 11. Sincronización entre edge y servidor

El dispositivo debe operar en modo offline cuando no exista conectividad.

### 11.1 Comportamiento esperado

- Captura local de eventos y evidencia.
- Almacenamiento temporal en el dispositivo.
- Reintento de sincronización cuando vuelva la conexión.
- Envío de datos estructurados en JSON y de archivos multimedia al backend o a su servicio de almacenamiento asociado.

### 11.2 Consideraciones técnicas

- Validación de integridad antes de enviar.
- Identificadores consistentes entre eventos y evidencia.
- Tolerancia a pérdida temporal de red.
- Evitar duplicados en la recepción.

## 12. Alertas y notificaciones

La arquitectura debe soportar alertas preventivas y notificaciones asociadas a eventos críticos.

### 12.1 Tipos de respuesta

- Alerta local en el dispositivo.
- Notificación push en la app móvil, con o sin la aplicación abierta.
- Seguimiento del estado de la notificación.


## 13. Despliegue

La solución se plantea como una arquitectura híbrida edge + servidor.

- El dispositivo Raspberry Pi queda en campo como nodo edge.
- El backend y la base de datos se despliegan en infraestructura centralizada, local o en nube según la decisión operativa posterior.
- La evidencia multimedia vive en un almacenamiento externo o especializado.

### 13.1 Vista de despliegue

- Cliente web: navegador.
- Cliente móvil: Android/iOS con React Native.
- Edge: Raspberry Pi + cámara.
- Servidor: API .NET.
- Datos: PostgreSQL.
- Multimedia: servicio de archivos u objeto externo.

## 14. Integración continua y despliegue continuo

Aunque el detalle de operación puede documentarse en otro archivo, esta arquitectura sí considera integración continua como parte del sistema.

### 14.1 Intención

- Validar cambios de código de forma automática.
- Facilitar despliegues controlados.
- Reducir errores de integración.

### 14.2 Herramienta esperada

- GitHub Actions, si se mantiene la línea ya definida en la documentación del proyecto.

## 15. Observabilidad y auditoría

La arquitectura debe prever trazabilidad básica de la operación del sistema.

### 15.1 Elementos mínimos

- Logs técnicos del backend.
- Registro de eventos recibidos.
- Registro de alertas generadas.
- Trazabilidad de sincronización entre dispositivo y servidor.

## 16. Respaldo y recuperación

La arquitectura debe contemplar respaldo básico de:

- Base de datos.
- Evidencias asociadas a eventos críticos.
- Configuración de dispositivos, cuando aplique.

También debe existir una estrategia de recuperación ante fallos de red, fallos de sincronización o pérdida temporal de conectividad.

## 17. Requisitos no funcionales

Los requisitos no funcionales principales que debe soportar la arquitectura son:

- Seguridad.
- Mantenibilidad.
- Tolerancia a fallos.
- Baja latencia en alertas críticas.
- Escalabilidad razonable.
- Trazabilidad de eventos.
- Privacidad de datos sensibles.

## 18. Restricciones y supuestos

La arquitectura parte de los siguientes supuestos:

- El proyecto usa C#, Python, PostgreSQL, React JS y React Native.
- El hardware principal es Raspberry Pi con cámara.
- La captura y parte del procesamiento ocurren en edge.
- La persistencia multimedia no se hará en la base de datos principal.
- La nube o infraestructura final puede ajustarse más adelante


---

**Vigencia:** A partir del 2026-05-14

