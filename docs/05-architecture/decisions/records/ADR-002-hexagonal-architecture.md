# ADR-002: Arquitectura hexagonal (puertos y adaptadores) en el backend

**Estado:** Aceptada
**Fecha:** 2026-08-19
**Autores:** Equipo SomnGuard
**Equipos involucrados:** Arquitectura, Desarrollo

---

## Contexto

El ADR-001 define el backend como un monolito modular en Java 21 + Spring Boot 4.1.1 con principios de clean architecture, pero sin formalizar cómo se estructuran los módulos ni cómo se organizan los paquetes. El documento de arquitectura proponía una división genérica por capas (`interfaces`, `application`, `domain`, `infrastructure`), que en la práctica no garantiza que el dominio quede aislado de Spring, JPA o HTTP.

Además, el proyecto cuenta con una referencia de microservicio Go (`internal/`) que ya usa una estructura de puertos y adaptadores explícita (`application/port/in`, `application/port/out`, `application/usecase`, `domain/model`, `domain/service`, `adapter/in`, `adapter/out`, `platform`), lo que genera dos convenciones distintas para el mismo concepto.

## Decisión

Se decide formalizar la arquitectura limpia del backend como **arquitectura hexagonal (puertos y adaptadores)**, con la siguiente estructura de paquetes por módulo:

```text
com.somnguard.<modulo>/
├── application/
│   ├── port/
│   │   ├── in/            # Interfaces de casos de uso (IngestEventUseCase, GetEventQuery, ...)
│   │   └── out/           # Interfaces de repositorios y servicios (EventRepository, ...)
│   └── usecase/           # Implementaciones de casos de uso (IngestEventService, ...)
├── domain/
│   ├── model/             # Entidades de negocio (Event, Evidence, AlertLog, ...)
│   └── service/           # Servicios de dominio (evaluación de severidad, validación, ...)
└── adapter/
    ├── in/
    │   ├── web/           # Controladores REST
    │   └── amqp/          # Consumidores de mensajes (si aplica)
    └── out/
        ├── persistence/   # Adaptadores JPA/PostgreSQL
        └── storage/       # Adaptadores de almacenamiento multimedia (MinIO/S3)
```

Además, existe un paquete transversal **fuera de los módulos**, al nivel de `com.somnguard`, junto a ellos:

```text
com.somnguard/
├── security/
├── parameterization/
├── device_management/
├── telemetry_service/
├── monitoring/
├── analytics/
└── platform/              # Transversal, fuera de los módulos: errores, logging, observabilidad
```

`platform` no va dentro de cada módulo: es un paquete único que todos comparten; los módulos pueden depender de `platform` y `platform` no depende de ningún módulo.

> **Convención de nombres:** los módulos se nombran en kebab-case en el catálogo (`telemetry-service`, `device-management`); en los **paquetes Java** se usa snake_case (`com.somnguard.telemetry_service`, `com.somnguard.device_management`) porque los guiones no son válidos en identificadores de paquete.

Reglas de dependencia: el dominio no conoce Spring, JPA, HTTP ni la base de datos; los adaptadores dependen de los puertos; un módulo solo se comunica con otros módulos a través de sus puertos de entrada.

La estructura de paquetes del ADR-001 (`interfaces`, `application`, `domain`, `infrastructure`) queda **reemplazada** por esta estructura hexagonal para el código nuevo.

## Consecuencias

### Positivas

- El dominio queda aislado de la tecnología, lo que permite probar los casos de uso con mocks de puertos sin base de datos.
- Cambio tecnológico aislado: sustituir PostgreSQL, el almacenamiento multimedia o el transporte afecta solo a los adaptadores.
- Fronteras claras entre módulos del monolito: un módulo usa solo los puertos de entrada de otro.
- Una sola convención en el proyecto, alineada con la referencia del microservicio Go `internal/`.
- Camino evolutivo a microservicios: cada módulo ya está aislado por sus puertos.
- El paquete `platform` (fuera de los módulos, al nivel de `com.somnguard`) centraliza el código transversal (errores, logging, observabilidad) evitando duplicación en cada módulo.

### Negativas / Trade-offs

- Más carpetas e indirección por módulo que un enfoque por capas simple.
- Exige disciplina en las reglas de dependencia; sin revisión, las dependencias tienden a filtrarse hacia el dominio.
- Los módulos existentes en el repositorio .NET (referencia) no siguen esta estructura y quedan como legado.

### Riesgos

- Que los desarrolladores salten las reglas de dependencia por comodidad — mitigado con convenciones en el pipeline de CI (revisión de arquitectura).
- Sobrediseño si se aplica la estructura completa a módulos triviales (p. ej., parameterization) — mitigado permitiendo módulos sin adaptadores si no los requieren.

## Alternativas consideradas

| Alternativa | Por qué se descartó |
|-------------|---------------------|
| Capas genéricas `interfaces`/`application`/`domain`/`infrastructure` (propuesta inicial del ADR-001) | No formaliza los contratos entre capas ni aísla el dominio de la tecnología; permite acoplamiento accidental con Spring/JPA |
| Microservicios por dominio | Se descartó en el ADR-001 por complejidad operativa innecesaria para el tamaño del sistema |
| Arquitectura limpia sin puertos explícitos (solo dependencias dirigidas) | Depende de la disciplina sin contratos formales; los puertos hacen verificable el aislamiento |

## Referencias

- Documento de arquitectura (sección 8): `../../architecture-document.md`
- Informe de diseño de software (sección 3.2.1): `../../software-design-report.md`
- ADR-001 (stack y monolito modular): `./ADR-001-backend-java-spring-boot.md`
- Referencia de estructura hexagonal en Go: microservicio de notificaciones `internal/` (fuera de este repositorio)
- Reglas de documentación de módulos: `../../../00-documentation-governance/structure-rules.md`