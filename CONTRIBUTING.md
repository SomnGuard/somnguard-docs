# Contributing

Gracias por tu interés en contribuir a la documentación de SomnGuard.

Este documento aplica al repositorio `somnguard-docs`. Las reglas de contribución de otros repositorios del proyecto pueden definirse por separado.

Antes de comenzar, por favor revisa el README y el [Código de Conducta](CODE_OF_CONDUCT.md).

## Ramas principales

Este repositorio usa dos ramas principales:

* `develop`: rama base para preparar y revisar cambios de documentación.
* `main`: rama estable con la documentación aprobada.

No se deben realizar cambios directamente sobre `main`. Los cambios deben prepararse desde `develop` y fusionarse mediante Pull Request.

## Flujo de trabajo

El flujo general es:

1. Actualizar la rama `develop`.
2. Crear una rama de trabajo desde `develop`.
3. Realizar los cambios necesarios en la documentación.
4. Crear commits descriptivos.
5. Enviar un Pull Request hacia `develop` para revisión.
6. Fusionar `develop` hacia `main` cuando la documentación esté aprobada y lista para publicarse.

## Convención de ramas

Utiliza nombres cortos y descriptivos para las ramas.

Ejemplos:

* `docs/actualizar-readme`
* `docs/agregar-manual-instalacion`
* `fix/corregir-enlaces`

## Convención de commits

Se recomienda el uso de Conventional Commits.

Ejemplos:

* `docs(readme): actualizar guía de instalación`
* `docs(manual): agregar pasos de configuración`
* `fix(links): corregir enlaces rotos`

## Pull Requests

Cada Pull Request debe incluir:

* Descripción clara de los cambios realizados.
* Relación con la historia de usuario, tarea o incidencia correspondiente, cuando aplique.
* Evidencias cuando aplique, como capturas de pantalla, vistas previas o registros.

Los Pull Requests estarán sujetos a revisión antes de ser aprobados y fusionados.

## Estándares de calidad

Antes de enviar una contribución:

* Verifica que los enlaces funcionen correctamente.
* Revisa ortografía, formato y consistencia de la documentación.
* Mantén los archivos organizados según la estructura existente.
* Evita incluir cambios no relacionados en un mismo Pull Request.

## Reporte de problemas

Si encuentras un error o deseas proponer una mejora, crea un Issue describiendo claramente el problema, la ubicación del documento afectado y cualquier información relevante.

Gracias por contribuir a la documentación de SomnGuard.
