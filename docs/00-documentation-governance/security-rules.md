<div style="display:flex; align-items:center; justify-content:space-between;">

<div>
<img src="../../assets/icons/logo-somnguard.png" width="140"/>
</div>

<div align="right">

# SOMNGUARD

## Reglas de seguridad

**Estado:** En progreso
**Fecha:** 2026-08-19

</div>

</div>

Este documento establece las reglas de seguridad y privacidad que aplican a la documentación del repositorio, complementando la sección de seguridad de la normativa del proyecto.

## Contenido prohibido en el repositorio

No incluir en ningún documento, commit o recurso del repositorio:

- Credenciales: contraseñas, tokens, API keys, secretos o claves privadas.
- Información sensible o confidencial del negocio.
- Datos personales reales de usuarios, conductores o terceros.

## Alternativas permitidas

- Usar variables de entorno o archivos `.env` locales (no versionados).
- Referenciar la bóveda de secretos del proyecto en lugar de valores literales.
- Usar datos ficticios en ejemplos, diagramas y plantillas.

## Revisión de dependencias y herramientas

- Revisar dependencias críticas contra vulnerabilidades conocidas antes de integrarlas.
- Mantener los listados de dependencias actualizados.
- Reportar CVEs y planificar actualizaciones.

## Checklist antes de publicar un documento

- [ ] No contiene credenciales, tokens ni datos personales.
- [ ] Los diagramas no exponen secretos, URLs internas ni credenciales.
- [ ] Las capturas de pantalla no muestran información real de usuarios.
- [ ] Fue revisado antes de abrir el Pull Request.