# ADR-006: Almacenamiento multimedia compatible con S3

**Estado:** Aceptada — decisión actualizada el 2026-11-03
**Fecha original:** 2026-08-22
**Autores:** Equipo SomnGuard
**Equipos involucrados:** Arquitectura, Backend, Device, DevOps

---

## Contexto

SomnGuard conserva evidencia multimedia asociada a eventos. PostgreSQL debe guardar sus metadatos y referencias, mientras los bytes viven en un servicio de objetos. La integración del API ya usa una interfaz S3 mediante el SDK de MinIO; los proveedores reales varían según el entorno.

## Decisión

Se adopta **almacenamiento de objetos compatible con la API S3** como contrato común. MinIO es el nombre histórico de la ADR y del cliente Java utilizado por el adaptador; no implica que todos los ambientes ejecuten un servidor MinIO.

| Ambiente | Implementación documentada | Endpoint S3 | Evidencia |
|---|---|---|---|
| Desarrollo local | Adobe S3Mock 4.12.4. En Compose el servicio conserva el alias `minio` por compatibilidad con la configuración del API. | Dentro de Compose: `http://minio:9090`; desde el host: `http://localhost:9000` | `somnguard-api/docker-compose.yml`, `.env.example` y `docs/10-devops/local-setup.md` |
| VPS documentado | SeaweedFS 4.48 | `http://somnguard-storage:8333` | `docs/10-devops/manual-despliegue-somnguard.md`, sección 4.3 y evidencia del despliegue del 2026-10-30 |
| Otros ambientes | Proveedor S3 compatible configurado por variables de entorno. No se declara un proveedor productivo adicional hasta que exista una configuración versionada y verificada. | `S3_ENDPOINT` | Configuración del ambiente |

### Contrato y datos

- El bucket se configura con `S3_BUCKET`; el bucket usado actualmente es `somnguard-evidence`.
- Los objetos usan una clave con la forma `{device_id}/{YYYY}/{MM}/{DD}/{event_id}.jpg`.
- El API sube y recupera la evidencia mediante su adaptador de almacenamiento S3. Device y clientes no acceden directamente al proveedor.
- PostgreSQL conserva los metadatos y la clave del objeto; no almacena los bytes multimedia.
- `minio_key` se mantiene como nombre físico histórico de una columna de base de datos. No es el nombre del proveedor ni del contrato de configuración.
- Las credenciales y el endpoint se entregan por `S3_ACCESS_KEY`, `S3_SECRET_KEY`, `S3_ENDPOINT`, `S3_REGION` y `S3_BUCKET`. Los valores reales no se versionan.

### Capacidades fuera del estado verificado

Versionado de objetos, cifrado administrado por el servicio, URLs prefirmadas, reglas de ciclo de vida, retención de cinco años y alertas de capacidad no se consideran capacidades implementadas por esta decisión. Cada una requiere configuración verificable y pruebas antes de describirse como activa.

## Consecuencias

### Positivas

- Arquitectura, desarrollo y despliegue hablan de un único contrato: almacenamiento de objetos compatible con S3.
- Se pueden usar proveedores distintos por ambiente sin exponerlos a los módulos del dominio.
- El API y PostgreSQL mantienen sus responsabilidades separadas.

### Negativas / Trade-offs

- S3Mock, SeaweedFS y otros proveedores pueden diferir en persistencia y capacidades; las pruebas deben cubrir el comportamiento usado por SomnGuard.
- El API usa el SDK de MinIO, por lo que la compatibilidad con otro servicio se debe validar con operaciones reales de lectura y escritura.

### Riesgos y pendientes

- Mantener alineados `S3_ENDPOINT` y las credenciales entre API y proveedor en cada ambiente.
- Incorporar a un repositorio versionado la configuración de despliegue del proveedor del VPS; el manual actual documenta el estado observado, pero no reemplaza el Compose de producción.
- Definir e implementar retención, cifrado, versionado y estrategia de respaldo antes de depender de esas capacidades.

## Alternativas consideradas

| Alternativa | Resultado |
|---|---|
| MinIO como proveedor obligatorio en todos los ambientes | Retirada como decisión de proveedor único; desarrollo usa S3Mock y el VPS documentado usa SeaweedFS. |
| Almacenar bytes en PostgreSQL | Descartada; la base conserva metadatos y referencias, no los objetos. |
| Crear una abstracción propietaria no compatible con S3 | Descartada; el API ya integra el contrato S3. |

## Referencias

- [Arquitectura del sistema](../../architecture-document.md#66-almacenamiento-multimedia)
- [Manual de despliegue](../../../10-devops/manual-despliegue-somnguard.md#43-almacenamiento-s3-compatible)
- [Setup local](../../../10-devops/local-setup.md)
- Implementación del API: `somnguard-api/src/main/java/com/somnguard/telemetry_service/adapter/out/storage/`
- Configuración local del API: `somnguard-api/docker-compose.yml` y `somnguard-api/.env.example`
