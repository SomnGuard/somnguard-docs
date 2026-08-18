# Actas del Proceso

## Propósito

Este documento centraliza las actas de reuniones del paroyecto SomnGuard, registrando decisiones, acuerdos, compromisos y seguimiento. Sirve como referencia histórica y de auditoría.

---

## Acta #1 – Kick-off del Proyecto SomnGuard

**Fecha:** 2026-02-10  
**Hora:** 08:00 - 11:00  
**Ubicación:** Presencial  
**Moderador:** Cristian Javier Palma Sotto

### Participantes

- Cristian Javier Palma Sotto (Líder Técnico / PM)
- Juan Carlos Jurado Castañeda (Full Stack Developer)
- Johan Steven Rodriguez Charry (Full Stack Developer)
- Brayan Alberto Perdomo (Full Stack Developer)

### Orden del día

1. Presentación de SomnGuard y objetivos del proyecto
2. Alcance funcional de la versión 1.0
3. Stack tecnológico y arquitectura base
4. Hardware: Raspberry Pi + Cámara
5. Cronograma y fecha de entrega
6. Roles del equipo y metodología
7. Herramientas y configuración inicial

### Resumen de discusiones

Se presentó la visión del proyecto **SomnGuard**: un sistema integral (software + hardware) diseñado para mejorar la seguridad vial mediante el monitoreo en tiempo real del estado de alerta del conductor.

**Objetivo principal:** Detectar patrones de fatiga, somnolencia y microsueños para generar alertas preventivas y registrar evidencia en eventos críticos, reduciendo accidentes causados por somnolencia al volante.

**Alcance funcional confirmado:**
- Análisis visual de señales capturadas por cámara
- Detección de estado de riesgo en conductor
- Sistema de alertas preventivas en tiempo real
- Registro de información pre/durante/post-evento
- Plataforma de consulta y análisis de evidencia
- Dirigido a conductores de transporte (particular, público, carga)

**Aspectos fuera de alcance (v1.0):**
- Control automático del vehículo
- Responsabilidades legales derivadas de accidentes

Se confirmó la disponibilidad de recursos (equipo 4 full-stack) y la fecha límite: **Agosto 2026** para entrega final.

### Acuerdos

- ✅ Stack tecnológico: **C#, Python, PostgreSQL, React JS, React Native**
- ✅ Hardware: **Raspberry Pi + Cámara** como dispositivo de captura
- ✅ Metodología: **Scrum con sprints de 1 semana**
- ✅ Equipo: 4 desarrolladores full-stack bajo liderazgo de Cristian
- ✅ Cloud: A definir posteriormente (pendiente evaluación AWS/Azure)
- ✅ Reuniones semanales: Lunes 09:00 para seguimiento de sprint
- ✅ Canal de coordinación: A confirmar (Slack/Teams)
- ✅ Deadline: **Agosto 2026**

### Compromisos

| Responsable | Compromiso | Fecha límite |
|-------------|-----------|------------|
| Cristian Javier Palma Sotto | Definir arquitectura base (servidor + device) | 2026-02-17 |
| Juan Carlos Jurado Castañeda | Investigar stack Python/C# para análisis visual | 2026-02-17 |
| Johan Steven Rodriguez Charry | Configurar Raspberry Pi con cámara e integración | 2026-02-17 |
| Brayan Alberto Perdomo | Diseño DB PostgreSQL inicial | 2026-02-17 |
| Cristian Javier Palma Sotto | Crear backlog inicial y primeras user stories | 2026-02-24 |

### Próximas acciones

1. Crear repositorio principal en GitHub.
2. Configurar acceso inicial: GitHub, repositorio de código.
3. Investigar y documentar opciones de cloud (AWS/Azure).
4. Preparar Sprint #1 Planning para **2026-04-27**.
5. Adquirir/verificar disponibilidad de Raspberry Pi y cámara.

### Estado: Completado

---

_Última actualización: 2026-05-02_
