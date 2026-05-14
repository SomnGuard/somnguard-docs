# Metodología Adoptada

**Autor:** Equipo SomnGuard  
**Fecha de creación:** 2026-04-12  
**Última actualización:** 2026-05-03  
**Estado:** Aprobado  

---

## 1. Resumen ejecutivo

El proyecto SomnGuard adopta **Scrum** como metodología ágil de trabajo. El enfoque se centra en entregas iterativas cada sprint (1 a 1.5 semanas), validación frecuente con el equipo, y mejora continua a través de ceremonias esenciales: Sprint Planning y Sprint Review.

## 2. Principios

- Entregas iterativas e incrementales cada sprint.
- Colaboración activa del equipo completo.
- Transparencia sobre el estado de tareas y blockers.
- Mejora continua: feedback del equipo en cada sprint.
- Flexibilidad: ajustes en la duración del sprint si es necesario.

## 3. Ciclo de vida y cadencia

### Duración del sprint

- **Estándar:** 1 a 1.5 semanas (flexible según necesidad).
- **Inicio:** Martes de cada semana.
- **Fin:** Lunes de la semana de conclusión.

### Eventos principales por sprint

1. **Sprint Planning (Inicio del sprint)**
   - Duración: 1 a 2 horas
   - Participantes: todo el equipo
   - Objetivo: seleccionar historias de usuario (HUs) a trabajar, asignar responsables, definir subtareas si aplica
   - Output: Sprint Backlog comprometido


2. **Sprint Review (Fin del sprint)**
   - Duración: 1 hora
  - Participantes: equipo
   - Objetivo: demostrar las HUs completadas, recoger feedback y definir acciones de mejora
   - Output: feedback y acciones de mejora para el siguiente sprint

## 4. Roles y responsabilidades

| Rol | Persona(s) | Responsabilidades |
|-----|-----------|------------------|
| **Líder Técnico** | Cristian Javier Palma Sotto | Coordinación, decisiones técnicas, revisión de PRs, eliminación de blockers |
| **Desarrollador Full-Stack** | Cristian Javier Palma Sotto, Juan Carlos Jurado Castañeda, Johan Steven Rodriguez Charry, Brayan Alberto Perdomo | Implementación de HUs, pruebas locales, estimaciones, feedback técnico |

**Nota:** No hay Product Owner formal aún ni stakeholders formales definidos. El Líder Técnico actúa como coordinador central, el equipo participa en decisiones sobre priorización y Cristian Javier Palma Sotto también participa como desarrollador full-stack.

## 5. Artefactos

- **Product Backlog:** lista de todas las HUs y requisitos pendientes, priorizada por el Líder Técnico.
- **Sprint Backlog:** HUs seleccionadas para el sprint actual, con subtareas asignadas.
- **Incremento:** código funcional entregable al final del sprint (en rama `develop` o `qa`).
- **Actas de Sprint:** resumen de acuerdos, compromisos y decisiones de cada sprint planning y review.

## 6. Historias de usuario y criterios de aceptación

### Formato de HU

```
Código: HU-###
Título: [Descripción breve]

Como <persona/rol>, quiero <acción>, para <beneficio>.

Criterios de aceptación:
- [ ] Criterio 1
- [ ] Criterio 2
- [ ] Criterio 3

Notas:
- Consideraciones técnicas o dependencias
```

### Criterios de aceptación

- **Claros:** descriptos en lenguaje comprensible
- **Verificables:** se pueden probar y evaluar objetivamente
- **Automatizables:** preferentemente validables mediante tests o checklist

### Subtareas

Cada HU puede desglosarse en subtareas para facilitar el seguimiento:

- Ejemplo: HU-001 Backend Modular
  - [ ] Estructura de proyecto C# inicializada
  - [ ] Configuración de dependencias principales
  - [ ] Primera clase/módulo de ejemplo
  - [ ] Tests unitarios básicos

## 7. Definition of Done (DoD)

Una historia se considera **completada** cuando cumple todos los siguientes criterios:

- ✅ Subtareas/criterios de aceptación cerrados
- ✅ Pull Request creada y revisada por Líder Técnico
- ✅ Cambios integrados a rama `develop` o rama destino
- ✅ Documentación mínima actualizada
- ✅ Verificación básica en entorno local

**Nota:** Las pruebas automatizadas se agregarán a DoD en sprints posteriores, según capacidad del equipo.

## 8. Calidad y pruebas

**Estado:** Por definir en sprints posteriores.

- Se iniciarán con pruebas unitarias básicas en módulos críticos (análisis visual, base de datos).
- Cobertura objetivo: a definir en sprint de calidad.
- La integración de CI/CD con GitHub Actions facilitará validaciones automáticas.

## 9. Integración y despliegue continuo

### Pipeline CI/CD

- **Herramienta:** GitHub Actions
- **Triggers:**
  - Build automático en cada push a ramas 
  - Tests (cuando estén disponibles)
  - Linters y análisis estático de código

### Flujo de despliegue

```
hu-###-dev → Develop (PR + review)
     ↓
   Develop (integración)
     ↓
   hu-###-qa → qa (pruebas manuales/automáticas)
     ↓
   hu-###-main → main (producción controlada)
```

- Merge a `Develop`: aprobación de 1 revisor
- Merge a `qa`: validación de tester/QA
- Merge a `main`: aprobación explícita del Líder Técnico

### Releases

- Versionado: `v{MAJOR}.{MINOR}.{PATCH}` (ej: v1.0.0)
- Changelog automático generado desde commits
- Artefactos empaquetados en `/releases/`

## 10. Gestión de riesgos

### Riesgos monitoreados

| Riesgo | Impacto | Mitigación |
|--------|---------|-----------|
| Privacidad y datos sensibles | Alto | Revisar normativa, tests de seguridad, auditoría |
| Dependencias de hardware no disponibles | Medio | Solicitud temprana, compra anticipada |
| Cambios de requisitos | Medio | Sesiones de refinement, feedback frecuente |

### Proceso de mitigación

- Identificar riesgos en Sprint Planning
- Asignar responsable de seguimiento
- Registrar en acta y revisar en Sprint Review

## 11. Documentación del proceso

Mantener actualizados:

- `05_metodologia_adoptada.md` — este documento
- `/00_Gestion_Proyecto/03_actas/` — actas de cada sprint
- `/02_Arquitectura_y_Diseño/` — decisiones técnicas y arquitectura

## 12. Herramientas

| Función | Herramienta |
|---------|------------|
| Gestión de backlog y tracking | ClickUp |
| Repositorio y control de versiones | GitHub |
| CI/CD | GitHub Actions |
| Comunicación | Discord o ClickUp |
| Diseño y prototipos | Figma |
| Documentación | Markdown y otros formatos de texto compatibles |

## 13. Métricas de seguimiento

Por ahora se registrarán de forma manual:

- **Velocidad del sprint:** HUs completadas / HUs comprometidas (%)
- **Cumplimiento:** Historias que cumplieron DoD en el sprint
- **Blockers:** número de impedimentos reportados y tiempo de resolución
- **Calidad:** defectos encontrados post-sprint (a reportar en futuro)

Se formalizará seguimiento automático cuando CI/CD esté maduro.

---

**Vigencia:** A partir del 2026-04-12

_Última actualización: 2026-05-03_
