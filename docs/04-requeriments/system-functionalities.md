<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Posibles funcionalidades del sistema

**Estado:** En progreso
**Fecha:** 2026-05-03

</div>

</div>

> **Nota:** los nombres de módulos se alinean con el catálogo vigente en [09-modules/module-catalog.md](../09-modules/module-catalog.md).

## 1. Modulo: Seguridad, Cuenta y Autorizacion (security)

### Funcionalidad: Registro y validacion de cuenta
Permite crear cuenta de usuario plataforma, validar identidad basica y dejar habilitado el acceso inicial al sistema.

### Funcionalidad: Inicio y cierre de sesion
Permite autenticar al usuario, abrir sesion segura y finalizar sesion cuando corresponda.

### Funcionalidad: Gestion de perfil de usuario
Permite consultar y actualizar datos de la cuenta asociados a la persona.

### Funcionalidad: Eliminacion de cuenta con retencion
Permite solicitar baja de cuenta manteniendo retencion de datos segun politica definida.

### Funcionalidad: Gestion de rol
Permite definir perfiles de acceso por tipo de usuario (usuario plataforma, admin tecnico u otros).

### Funcionalidad: Gestion de permiso
Permite configurar acciones autorizadas sobre funcionalidades especificas.

### Funcionalidad: Gestion de modulo y formulario
Permite organizar funcionalidades por modulo y pantalla para control de acceso y navegacion.

### Funcionalidad: Asignacion de roles a usuario
Permite asociar uno o varios roles a una cuenta para habilitar permisos efectivos.

## 2. Modulo: Gestion de Dispositivos Asociados (device-management)

### Funcionalidad: Asociar dispositivo
Permite vincular un dispositivo SomnGuard a una cuenta de usuario.

### Funcionalidad: Desasociar dispositivo
Permite retirar la relacion entre usuario y dispositivo manteniendo historial de asignacion.


### Funcionalidad: Gestion de configuracion de dispositivo
Permite parametrizar y actualizar configuracion enviada desde plataforma al dispositivo.

## 3. Modulo: Ingesta y Sincronizacion de Datos (telemetry-service)

### Funcionalidad: Registrar eventos y evidencia
Permite almacenar eventos, metadatos y evidencia multimedia generada por el dispositivo.

### Funcionalidad: Almacenar eventos offline
Permite conservar eventos localmente cuando no hay conectividad.

### Funcionalidad: Sincronizar datos con servidor
Permite transmitir JSON y archivos multimedia al backend cuando hay conectividad.

### Funcionalidad: Recepcion de telemetria y evidencia
Permite recibir, validar y persistir los datos enviados por dispositivos asociados.


## 4. Modulo: Monitoreo de Eventos y Alertamiento (monitoring)

### Funcionalidad: Gestion de alertas
Permite generar y controlar alertas vinculadas a eventos detectados.

### Funcionalidad: Notificacion critica in-app
Permite enviar notificaciones al usuario plataforma ante eventos criticos.

### Funcionalidad: Seguimiento de estado de notificaciones
Permite controlar el ciclo de notificacion (emitida, leida, cerrada u otro estado).

## 5. Modulo: Parametrizacion y Catalogos (parameterization)

### Funcionalidad: Gestion de catalogo de estados
Permite administrar estados logicos comunes para entidades del sistema.

### Funcionalidad: Gestion de severidad
Permite parametrizar niveles de criticidad para eventos y alertas.

### Funcionalidad: Gestion de tipo de medio
Permite administrar tipos de evidencia multimedia soportados.

### Funcionalidad: Gestion de patron de sonido
Permite definir patrones de audio para alertas locales en dispositivo.

### Funcionalidad: Gestion de categoria de evento
Permite mantener clasificaciones de eventos para analitica y reglas de negocio.
