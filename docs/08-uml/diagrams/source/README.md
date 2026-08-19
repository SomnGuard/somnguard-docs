# Fuentes de diagramas

Diagramas UML editables en formato Mermaid (`.mmd`). Todo diagrama del proyecto debe tener su fuente aquí y su exportación en `../exports/`.

Tipos de diagramas:

- `cu-*.mmd` — casos de uso del sistema.
- `sd-*.mmd` — diagramas de secuencia (flujos del sistema).
- `ac-*.mmd` — diagramas de actividades (procesos).
- `es-*.mmd` — diagramas de estados (ciclos de vida).
- `cd-*.mmd` — diagramas de clases (dominio).
- Otros tipos (`flowchart`, `erDiagram`, etc.) cuando apliquen, con prefijo descriptivo (`fc-*`, `er-*`).

Diagramas vigentes:

- `cu-account-device.mmd` — cuenta y dispositivo.
- `cu-device.mmd` — ciclo de vida del dispositivo.
- `cu-general.mmd` — visión general.
- `cu-local-management.mmd` — operación local del dispositivo.
- `cu-plataforma.mmd` — funcionalidades de la plataforma.
- `cu-analytics-visualization.mmd` — módulo analítico.
- `sd-detection-alert.mmd`, `sd-offline-sync.mmd`, `sd-authentication.mmd`, `sd-password-reset.mmd`, `sd-device-registration.mmd`, `sd-events-query.mmd`, `sd-critical-notification.mmd`, `sd-report-generation.mmd` — secuencias.
- `ac-event-detection.mmd`, `ac-offline-sync.mmd`, `ac-report-generation.mmd`, `ac-account-registration.mmd` — actividades.
- `es-device.mmd`, `es-event.mmd`, `es-account.mmd` — estados.
- `cd-domain.mmd` — clases del dominio.

Convenciones:

- Nombres en kebab-case en minúsculas, en inglés, sin acentos.
- El contenido es el código Mermaid puro (sin delimitadores ` ```mermaid `).
- Al crear o modificar un diagrama: exportar su `.png` a `../exports/` y actualizar el índice en `../../diagram-index.md`.

Exportación (requiere Node.js):

```powershell
npx -y @mermaid-js/mermaid-cli -i source/<diagrama>.mmd -o ../exports/<diagrama>.png -b white
```