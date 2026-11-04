# Informe de QA — Punto 10: alineación de infraestructura y ADR

**Proyecto:** SomnGuard
**Fecha:** 2026-11-03
**Repositorio:** `somnguard-docs`
**Rama:** `feature/point-10-infra-docs-alignment`
**Base:** `develop` (`4bfa83d`)
**Estado:** CONDITIONAL — documentación alineada con la evidencia disponible; falta versionar la configuración fuente del despliegue VPS para que sea reproducible.

## 1. Objetivo

Alinear las decisiones de arquitectura y la documentación de desarrollo, despliegue y operación con la infraestructura observada para almacenamiento multimedia, Traefik y observabilidad. La ADR-007 de observabilidad queda reemplazada al no existir el stack OTel/LGTM como despliegue versionado.

## 2. Alcance y repositorios revisados

| Repositorio | Uso en esta revisión |
|---|---|
| `somnguard-docs` | ADRs, arquitectura, setup local, manuales de despliegue/operación, requisitos y referencias relacionadas. |
| `somnguard-api` | Compose y `.env.example` para identificar el servicio local S3Mock, endpoints y variables `S3_*`; configuración de Actuator/Micrometer y cliente S3. |
| `somnguard-db` | Healthcheck de PostgreSQL y documentación de migraciones. |
| VPS documentado | Evidencia del manual de despliegue para Traefik v3.6.13 y SeaweedFS 4.48, fechada 2026-10-30. No se accedió al servidor durante esta tarea. |

## 3. Cambios implementados

- **ADR-006:** reemplaza la decisión de proveedor MinIO único por el contrato común de almacenamiento compatible con S3. Registra Adobe S3Mock para desarrollo local y SeaweedFS para el VPS documentado; diferencia el SDK Java de MinIO del servidor desplegado y marca como no verificadas las capacidades de versionado, cifrado, URLs prefirmadas, lifecycle y retención.
- **ADR-007 / ADR-014:** marca OTel + LGTM como decisión reemplazada y registra el baseline real de Actuator/Micrometer, logs del runtime y healthcheck de PostgreSQL, sin afirmar que existen colectores, dashboards o alertas centralizadas.
- **ADR-008:** conserva Traefik como componente registrado en el VPS, y aclara que los ejemplos detallados de la ADR no certifican configuración activa. La fuente Compose del VPS sigue pendiente de versionarse.
- **Documentación relacionada:** actualiza arquitectura, cross-cutting, setup local, manual de despliegue, observabilidad, respaldo, seguridad, requisitos, mapa de dependencias, glosario y registro de decisiones para usar el contrato S3 común y reflejar el estado verificado.

## 4. Criterios de aceptación

| Criterio | Resultado | Evidencia / observación |
|---|---|---|
| ADRs de almacenamiento, Traefik y observabilidad reflejan la decisión o su estado de reemplazo. | CONDITIONAL | ADR-006 alineada; ADR-007 reemplazada por ADR-014; ADR-008 registra Traefik en VPS. Falta incorporar al repositorio los Compose fuente del VPS para reproducción y revisión completa. |
| Arquitectura, desarrollo y despliegue usan una sola referencia conceptual para almacenamiento multimedia. | PASS documental | Contrato común S3; S3Mock y SeaweedFS descritos como proveedores por ambiente, no como contratos distintos. |
| La documentación de observabilidad distingue lo implementado de la propuesta LGTM. | PASS documental | Actuator/Micrometer y healthchecks se documentan como baseline; OTel/LGTM queda histórico y reemplazado. |
| Las afirmaciones de almacenamiento no atribuyen capacidades no verificadas. | PASS documental | Versionado, cifrado, URLs prefirmadas, retención/lifecycle y alertas se indican como no verificados o pendientes. |
| Despliegue del VPS reproducible desde configuración versionada. | PENDIENTE | El manual registra evidencia de ejecución, pero los archivos Compose de producción no están en los repositorios revisados. |

## 5. Validaciones ejecutadas

| Validación | Resultado | Detalle |
|---|---|---|
| Revisión de configuración de almacenamiento del API | PASS documental | Se revisaron `somnguard-api/docker-compose.yml` y `.env.example`: Adobe S3Mock 4.12.4, alias Compose `minio`, endpoint contenedor `http://minio:9090`, host `http://localhost:9000`, bucket y variables `S3_*`. |
| Revisión de persistencia local S3Mock | PASS documental | Compose monta `s3mock-data` en `/s3mockroot` y habilita `RETAIN_FILES_ON_EXIT=true`. |
| Revisión del baseline de observabilidad | PASS documental | Se cotejó la configuración del API y dependencias Actuator/Micrometer con el inventario de documentación. |
| Revisión de evidencia VPS | PASS documental limitada | Se consultó el manual y evidencia fechada 2026-10-30. No se ejecutaron comandos contra el VPS durante esta tarea. |
| `git diff --check` | PASS | Sin errores de whitespace. Git puede mostrar avisos de conversión LF/CRLF de Windows. |
| Pruebas de aplicación / despliegue | N/A | Los cambios son documentales; no se modificó código ejecutable ni se levantó el stack. |

## 6. Validación recomendada para QA

| ID | Caso | Resultado esperado | Estado |
|---|---|---|---|
| QA-10-01 | Levantar almacenamiento local con el Compose del API y subir/leer una evidencia de prueba. | El API usa S3Mock mediante el contrato S3 y mantiene los objetos tras reinicio del servicio. | Pendiente de ejecución en ambiente local. |
| QA-10-02 | Verificar el VPS contra la versión exacta de Traefik, SeaweedFS, endpoints y TLS documentados. | Coincidencia con el manual y sus evidencias, sin asumir que los ejemplos ADR sean configuración activa. | Pendiente de acceso al VPS. |
| QA-10-03 | Incorporar y revisar en un repositorio la configuración Compose fuente del VPS. | Despliegue reproducible, configuración saneada de secretos y endpoints comprobables. | Pendiente. |
| QA-10-04 | Revisar los endpoints Actuator en cada perfil/ambiente. | Solo endpoints aprobados y protegidos quedan accesibles según la política del ambiente. | Pendiente de validación de seguridad por QA/DevOps. |

## 7. Deuda y limitaciones

- Versionar la configuración fuente de Traefik y del proveedor de objetos del VPS en un repositorio definido por el equipo.
- Ejecutar una prueba de escritura/lectura S3 contra SeaweedFS; el manual señala que la respuesta HTTP de `curl` a `localhost:8333` no quedó capturada.
- Definir y probar backup/restauración de objetos, retención, versionado y cifrado antes de presentar esas capacidades como implementadas.
- Revisar autenticación y exposición de Actuator en los perfiles QA y producción.

## 8. Veredicto

**CONDITIONAL.** La documentación quedó alineada con la configuración del API y la evidencia disponible del VPS. La revisión no certifica un despliegue limpio ni permite reproducir completamente el entorno VPS mientras su Compose fuente no esté versionado. QA puede revisar y validar el cambio documental; para cerrar los pendientes operativos se requieren las pruebas y artefactos indicados arriba.
