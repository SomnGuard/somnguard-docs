# ADR-006: Object Storage — MinIO (S3-Compatible) para Evidencia Multimedia

**Estado:** Aceptada
**Fecha:** 2026-08-22
**Autores:** Equipo SomnGuard
**Equipos involucrados:** Arquitectura, Backend, Device, DevOps

---

## Contexto

El sistema genera **evidencia multimedia** asociada a eventos críticos:
- **Frames JPEG** del momento de detección (~50-200 KB c/u)
- **Video clips** opcionales (futuro, ~1-5 MB c/u)
- Volumen estimado MVP: 10k eventos/día × 1 frame = ~2 GB/día = ~60 GB/mes

Requisitos:
- **Almacenamiento barato y escalable** (objetos, no filesystem)
- **Acceso vía HTTP** desde API (subida) y Portal/App (descarga/visualización)
- **Privacidad:** Solo usuarios dueños del dispositivo + admins
- **Retención:** 90 días por defecto, configurable por evento crítico (5 años)
- **Compatibilidad:** Device sube directo (pre-signed URL) o via API; Portal/App descarga

---

## Decisión

### 1. MinIO (S3-Compatible) como Object Storage
| Aspecto | Especificación |
|---------|----------------|
| **Software** | MinIO (open source, S3 API compatible) |
| **Despliegue** | Contenedor Docker (local/dev); MinIO Operator en K8s (prod) |
| **Bucket único** | `somnguard-evidence` |
| **Estructura de keys** | `{device_id}/{YYYY}/{MM}/{DD}/{event_id}.{ext}` |
| **Región** | `us-east-1` (dummy, MinIO single-node) |
| **Versiónamiento** | **Habilitado** (protege contra borrado accidental) |
| **Cifrado** | SSE-S3 (AES-256) en reposo; TLS 1.3 en tránsito |

### 2. Flujo de Subida (Ingesta)
| Origen | Método | Detalle |
|--------|--------|---------|
| **Device → API** | Multipart en `POST /telemetry/events` | API valida → sube a MinIO → guarda `evidence_id` + key en BD |
| **Device → MinIO (futuro)** | Pre-signed PUT URL | API genera URL firmada (TTL 15 min) → device sube directo → notifica API |
| **Portal/App** | Pre-signed GET URL | API genera URL firmada (TTL 1 hora) → cliente descarga/visualiza |

### 3. Metadatos en Base de Datos (no en MinIO)
| Tabla | Columnas Clave |
|-------|----------------|
| `telemetry.evidence` | `id` (PK UUID), `event_id` (FK), `minio_key` (VARCHAR), `media_type_id` (FK), `size_bytes`, `checksum_sha256`, `created_at`, `created_by` (device_id) |

> **Regla:** MinIO **solo almacena bytes**. Metadatos, permisos, relaciones → PostgreSQL.

### 4. Políticas de Acceso (IAM Style)
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DeviceWrite",
      "Effect": "Allow",
      "Principal": {"AWS": ["arn:aws:iam::device:*"]},
      "Action": ["s3:PutObject", "s3:PutObjectTagging"],
      "Resource": "arn:aws:s3:::somnguard-evidence/*",
      "Condition": {"StringEquals": {"s3:x-amz-meta-device-id": "${device_id}"}}
    },
    {
      "Sid": "UserReadOwn",
      "Effect": "Allow",
      "Principal": {"AWS": ["arn:aws:iam::user:*"]},
      "Action": ["s3:GetObject"],
      "Resource": "arn:aws:s3:::somnguard-evidence/*",
      "Condition": {"StringLike": {"s3:prefix": "${user_device_ids}/*"}}
    },
    {
      "Sid": "AdminFull",
      "Effect": "Allow",
      "Principal": {"AWS": ["arn:aws:iam::admin:*"]},
      "Action": ["s3:*"],
      "Resource": "arn:aws:s3:::somnguard-evidence/*"
    }
  ]
}
```
> **Implementación real:** En API (middleware), no en MinIO policies. MinIO usa credenciales admin únicas; API valida ownership antes de firmar URLs.

### 5. Retención y Lifecycle
| Regla | Acción |
|-------|--------|
| **Eventos normales** | Eliminar objeto a los 90 días (MinIO ILM rule) |
| **Eventos críticos** (severity = `critical`) | Retener 5 años (tag `retention=long` en PUT) |
| **Versiones antiguas** | Mantener últimas 3 versiones; purgar anteriores |
| **Bucket size alert** | Prometheus alert si `bucket_size > 500 GB` |

### 6. Device — Subida de Evidencia
- **Opción actual (MVP):** Device envía base64 en `event_json` + `evidence_base64` aparte → API decodifica + sube a MinIO
- **Opción futura (optimización):** API devuelve pre-signed PUT URL en `GET /devices/{id}/config` → device sube directo a MinIO → reduce bandwidth API

### 7. Portal/App — Visualización
- **Imagen:** `<img src="{pre-signed-get-url}">` (TTL 1h, cache browser)
- **Video:** `<video src="{pre-signed-get-url}">` (range requests soportados por MinIO)
- **Descarga:** Botón "Descargar" → misma URL pre-firmada con `Content-Disposition: attachment`

### 8. Configuración por Ambiente
| Variable | Local | Dev | Prod |
|----------|-------|-----|------|
| `MINIO_ENDPOINT` | `http://minio:9000` | `https://minio.dev.somnguard.com` | `https://minio.somnguard.com` |
| `MINIO_ACCESS_KEY` | `minioadmin` | Vault secret | Vault secret |
| `MINIO_SECRET_KEY` | `minioadmin` | Vault secret | Vault secret |
| `MINIO_BUCKET` | `somnguard-evidence` | `somnguard-evidence-dev` | `somnguard-evidence-prod` |
| `MINIO_REGION` | `us-east-1` | `us-east-1` | `us-east-1` |

---

## Consecuencias

### Positivas
- **Costo near-zero local:** MinIO en Docker = gratis; prod: S3/MinIO managed ~$0.023/GB/mes
- **API S3 estándar:** SDKs en Java (Spring), Python (boto3), JS (AWS SDK) — mismo código
- **Escalabilidad horizontal:** MinIO distributed mode (Erasure coding) si crece
- **Versionamiento nativo:** Protección contra ransomware/borrado accidental
- **Pre-signed URLs:** Offload de bandwidth API; descarga directa browser→MinIO
- **Retención automática:** ILM rules en MinIO = sin jobs custom

### Negativas / Trade-offs
- **Componente extra** en infra (MinIO + Console en puerto 9001)
- **Credenciales admin** en API (single tenant) → Mitigación: Vault, rotación 90d
- **Consistencia eventual:** Objeto subido a MinIO vs registro en PG (mitigado: transacción API = PG commit + MinIO PUT; compensating transaction si falla MinIO)
- **Video grande** → multipart upload necesario (> 5 MB) → complejidad en device

### Riesgos
- **Bucket público por error** → Mitigación: Default private; tests de seguridad en CI (s3cmd ls falla sin credenciales)
- **Orphan objects** (en MinIO sin registro en PG) → Mitigación: Job semanal `LIST` vs `SELECT minio_key`; reporte discrepancias
- **Crecimiento descontrolado** → Mitigación: ILM + alertas Prometheus + dashboard Grafana

---

## Alternativas consideradas

| Alternativa | Por qué se descartó |
|-------------|---------------------|
| **PostgreSQL BYTEA / Large Object** | Bloat tabla, backup lento, no streaming, no CDN, no pre-signed URLs, mala performance > 100 MB |
| **Filesystem local (NFS / volume)** | No escalable, backup complejo, sin versionado, sin pre-signed URLs, single point of failure |
| **AWS S3 directo (prod) + MinIO (local)** | Vendor lock-in; MinIO es S3-compatible → mismo código en todos ambientes |
| **Azure Blob / GCS** | Equipo familiarizado con S3 API; MinIO corre en cualquier K8s/VM |
| **GridFS (MongoDB)** | Añade MongoDB solo para esto; overkill |
| **Base64 en BD (event.metadata)** | Límite 1 MB en PG `jsonb`; infla BD 33%; no streaming; mala idea |

---

## Referencias

- [cross-cutting.md](../cross-cutting.md) — (storage strategy implícita en observabilidad)
- [functional.md](../../../04-requeriments/functional.md) → RF-TEL-02, RF-EDGE-08, RF-ANA-05
- [architecture-document.md](../../architecture-document.md) §7 (Storage)
- [module-catalog.md](../../../09-modules/module-catalog.md) → `telemetry_service` adapter/out/storage
- MinIO docs: https://min.io/docs
- S3 API reference: https://docs.aws.amazon.com/AmazonS3/latest/API/API_Operations.html
- Spring Cloud AWS S3: https://spring.io/projects/spring-cloud-aws