# ADR-003: Módulo analítico (analytics) en el backend

**Estado:** Aceptada
**Fecha:** 2026-08-19
**Autores:** Equipo SomnGuard
**Equipos involucrados:** Arquitectura, Desarrollo, Producto

---

## Contexto

El alcance funcional del sistema (funcionalidades del sistema, software analysis) incluye consultas analíticas sobre los eventos registrados: línea de tiempo de eventos, métricas de comportamiento del conductor, resumen descriptivo del comportamiento y generación de reportes. La arquitectura vigente definía cinco módulos (security, parameterization, device-management, telemetry-service, monitoring) sin un responsable claro para estas capacidades.

## Decisión

Se decide incorporar el módulo **analytics** al catálogo de módulos del backend, con la responsabilidad de:

- Línea de tiempo de eventos por dispositivo y rango de fechas.
- Métricas de comportamiento (frecuencia de eventos, severidad, horas de operación).
- Resumen descriptivo del comportamiento con asistencia de IA.
- Generación de reportes para el portal web.

El módulo **no agrega entidades transaccionales**: consulta datos de otros módulos (principalmente telemetry-service y device-management) a través de sus puertos de entrada y expone consultas y reportes. Sigue la misma estructura hexagonal definida en el ADR-002.

## Consecuencias

### Positivas

- Capacidades analíticas con dueño claro dentro del monolito modular.
- No duplica datos: lee de los módulos fuente mediante sus puertos.
- Las consultas pesadas (reportes) pueden optimizarse con vistas o proyecciones sin afectar la ingesta.

### Negativas / Trade-offs

- Depende de los puertos de entrada de otros módulos, por lo que requiere que esos puertos expongan las consultas necesarias.
- La generación de reportes puede ser costosa si no se diseñan proyecciones o materializaciones.

### Riesgos

- Que se acceda directamente a repositorios de otros módulos en vez de sus puertos — mitigado con la regla de dependencia del ADR-002.
- Alcance difuso de "resumen IA" — mitigado definiendo el alcance en las historias de usuario correspondientes.

## Alternativas consideradas

| Alternativa | Por qué se descartó |
|-------------|---------------------|
| Fusionar analytics en telemetry-service | Mezcla ingesta (escritura intensiva) con consultas y reportes; dificulta evolución independiente |
| Sistema de reportes externo (BI) | Sobrecoste operativo para el alcance actual; puede evaluarse más adelante |
| No registrar el módulo | Las funcionalidades analíticas quedarían sin responsable dentro del monolito |

## Referencias

- Catálogo de módulos: `../../../09-modules/module-catalog.md`
- Documento de arquitectura (sección 7.6): `../../architecture-document.md`
- Informe de diseño de software (sección 12): `../../software-design-report.md`
- ADR-002 (estructura hexagonal): `./ADR-002-hexagonal-architecture.md`