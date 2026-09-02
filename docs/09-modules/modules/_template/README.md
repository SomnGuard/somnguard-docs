<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Plantilla de módulo — Guía de uso

**Estado:** Plantilla
**Fecha:** 2026-08-19

</div>

</div>

> **PLANTILLA** — Para crear documentación de un módulo del backend:

1. Copiar la carpeta `_template/module/` a `modules/<nombre-modulo>/` (p. ej. `modules/device_management/`).
2. Completar cada archivo según las instrucciones de su cabecera.
3. Registrar el módulo en [module-catalog.md](../../module-catalog.md).
4. Añadir el módulo al [README de 09-modules](../../README.md).

## Estructura de la plantilla

| Archivo | Propósito |
|---------|-----------|
| `module/README.md` | Identidad del módulo: responsabilidad, entidades, dependencias, puertos |
| `module/data-model.md` | Entidades propias del módulo (tablas y campos) |
| `module/events.md` | Eventos de dominio publicados y consumidos |
| `module/decisions.md` | Decisiones internas del módulo (las transversales van a ADR) |
| `module/runbook.md` | Operación: healthcheck, alertas, reinicio y logs |

## Estructura de código del módulo (hexagonal)

```
com.somnguard.<nombre-modulo>/
├── application/
│   ├── port/
│   │   ├── in/       # Puertos de entrada (usados por adapters in)
│   │   └── out/      # Puertos de salida (implementados por adapters out)
│   └── usecase/      # Casos de uso (orquestación)
├── domain/
│   ├── model/        # Entidades y value objects
│   └── service/      # Lógica de dominio pura
└── adapter/
    ├── in/
    │   ├── web/      # REST controllers
    │   └── amqp/     # Consumidores de mensajes (si aplica)
    └── out/
        ├── persistence/  # Repositorios (PostgreSQL)
        └── storage/      # Almacenamiento multimedia (MinIO/S3)
```

Reglas: ver [structure-rules.md](../../../00-documentation-governance/structure-rules.md) y ADR-002.

## Ver también

- [Catálogo de módulos](../../module-catalog.md)
- [Convenciones de modelado](../../../06-data-architecture/modeling-conventions.md)
- [Convenciones REST](../../../07-api-design/guidelines.md)