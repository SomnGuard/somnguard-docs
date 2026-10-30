# Evidencias de despliegue

Capturas que respaldan la ejecución del [manual de despliegue](../../../docs/10-devops/manual-despliegue-somnguard.md).

## Estructura

```text
despliegue/
└── YYYY-MM-DD/
    ├── 01-docker-ps.png
    ├── 02-traefik-tls.png
    └── ...
```

- Una carpeta por ejecución de despliegue, con la fecha en formato ISO (`YYYY-MM-DD`).
- Si hay más de un despliegue el mismo día, usar sufijo: `2026-10-30-b/`.
- No mezclar capturas de entornos distintos (producción, QA) en la misma carpeta.

## Convención de nombres

| Regla | Ejemplo correcto | Ejemplo incorrecto |
|-------|------------------|--------------------|
| `NN-descripcion-corta.png` | `04-api-health.png` | `Captura de pantalla 2026-10-30 123456.png` |
| Minúsculas, kebab-case, sin espacios | `03-liquibase-status.png` | `03_Liquibase Status.PNG` |
| Sin acentos ni `ñ` | `07-portal-spa.png` | `07-portal-añadir.png` |
| `NN` de dos dígitos, en el orden del manual | `02-traefik-tls.png` | `traefik.png` |
| PNG por defecto; JPG solo si la captura es fotográfica | `01-docker-ps.png` | `evidencia.bmp` |

Recomendaciones técnicas: ancho máximo 1920 px y peso menor a 1 MB por archivo. Evitar capturas de pantalla completa del escritorio; recortar solo la ventana o el bloque de terminal relevante.

## Capturas requeridas

| Archivo | Qué debe mostrar | Comando / origen | Sección del manual |
|---------|------------------|------------------|--------------------|
| `01-docker-ps.png` | Servicios arriba y `somnguard-postgres` en estado `healthy` | `docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Status}}"` | 5 |
| `02-traefik-tls.png` | Certificado válido y nombre correcto (candado + detalle del certificado del navegador) | Navegador en `https://somnguard.tech` → ver certificado | 4.1 / 5 |
| `03-liquibase-status.png` | Changesets aplicados, sin pendientes | `docker compose --env-file .env run --rm liquibase status --verbose` | 4.2 |
| `04-api-health.png` | `HTTP/1.1 200` y `{"status":"UP"}` | `curl -i https://api.somnguard.tech/actuator/health` | 4.4 / 5 |
| `05-postgres-ready.png` | `accepting connections` | `docker exec somnguard-postgres pg_isready -U somnguard -d somnguard` | 4.2 / 5 |
| `06-seaweedfs-storage.png` | Respuesta del servicio de almacenamiento | `curl http://localhost:8333/` | 4.3 / 5 |
| `07-portal-spa.png` | Portal cargado y consola del navegador sin errores | Navegador en `https://somnguard.tech` con DevTools → Console | 4.6 / 5 |
| `08-spa-refresh.png` | Ruta interna del portal responde 200 al refrescar (no 404) | `curl -I https://somnguard.tech/<ruta-interna>` | 5 |
| `09-download-host.png` | Descarga estática accesible | `curl -I https://download.somnguard.tech/<archivo>` | 4.6 / 5 |
| `10-livekit-test.png` | Señalización y medios conectados desde cliente autorizado | Prueba funcional LiveKit | 4.5 / 5 |
| `11-recursos.png` | Consumo de CPU/RAM y disco del VPS | `docker stats --no-stream`, `free -h`, `df -h` | 5.1 |
| `12-backup-restore.png` | Backup generado y restauración validada en instancia aislada | Sección 6.2 del manual | 6.2 |

Capturas opcionales según el alcance de la entrega:

| Archivo | Qué debe mostrar | Sección del manual |
|---------|------------------|--------------------|
| `13-traefik-logs.png` | Emisión del certificado en logs (sin contenido de `acme.json`) | 4.1 / 7 |
| `14-cors.png` | Cabeceras CORS correctas en DevTools → Network | 7 |
| `15-rollback.png` | `rollbackCount 1` y `verify-rollback.sh` exitoso | 4.2 |

## Antes de capturar (obligatorio)

Revisar el checklist de [reglas de seguridad](../../../docs/00-documentation-governance/security-rules.md#checklist-antes-de-publicar-un-documento). Ninguna captura puede contener:

- Contenido de `.env`, `storage.env`, `livekit.yaml`, `acme.json`, `jwt_private.pem` o `fcm-service-account.json`.
- Tokens, API keys, cookies o cabeceras `Authorization: Bearer ...`.
- Contraseñas visibles en salidas de terminal o en la barra de direcciones.
- Datos personales reales de usuarios, conductores o terceros.
- IPs públicas, correos o dominios internos que no estén ya publicados en el manual.

Si algo sensible aparece en pantalla, recortar, tapar o repetir la captura. No se corrige "después" en el repositorio: el historial de Git conserva la imagen original.

## Cómo referenciarlas en el manual

Desde `docs/10-devops/manual-despliegue-somnguard.md` hay que subir dos niveles:

```markdown
![Estado de contenedores](../../assets/images/despliegue/2026-10-30/01-docker-ps.png)
*Figura 1. `docker ps` — servicios arriba y PostgreSQL healthy.*
```

## Nota sobre el PDF

`docs/10-devops/Manual_Despliegue_SomnGuard.pdf` es un artefacto independiente: no se actualiza al editar el Markdown ni al añadir imágenes. Si el PDF se entrega como evidencia formal, hay que regenerarlo después de incorporar las capturas.
