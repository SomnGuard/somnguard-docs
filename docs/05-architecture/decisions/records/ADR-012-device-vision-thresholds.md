# ADR-012: Umbrales de visión edge HU-DEVICE-001 — desvíos justificados vs Apéndice 2

**Estado:** Aceptada (implementada en `somnguard-device`, validada en campo 2026-09-16)
**Fecha:** 2026-09-17
**Autores:** Equipo SomnGuard
**Equipos involucrados:** Device, Arquitectura
**HU afectada:** HU-DEVICE-001 (Sprint 2-3, Must) · **Artefactos base:** Apéndice 2 SRS, `data-dictionary.md` (seed Apéndice 2), RF-EDGE-04/05/06

---

## Contexto

Los umbrales iniciales del Apéndice 2 (bostezo 2/5 min, cabeceo >20° absoluto, mirada >3 s) resultaron inalcanzables o ciegos con el hardware real (cámara frontal + MediaPipe FaceLandmarker + solvePnP de 6 puntos). Medición de campo 2026-09-16 (logs + CSV de `scripts/debug_*.py`):

- Pitch frontal lee **+170° estable** (sesgo del solvePnP) → el gate de pose absoluta descartaba el 100% de los frames: EV-SOM-04/05 y la mirada estaban muertos.
- MAR externo saturaba en ~0.6 con la boca abierta de frente y solo subía al girar (perspectiva) → bostezo indetectable por umbral.
- Cada alerta corta a la anterior (`SoundPlayer.play()` hace `stop()` primero) → eventos simultáneos se "cancelaban".

Paralelamente se revisó literatura DMS (Euro NCAP Safe Driving v10.4, NHTSA distraction guidelines, Klauer naturalistic driving, revisiones PERCLOS, PLOS LDIE-FDNet, MDPI head-nodding). Varios umbrales del Apéndice 2 no tienen respaldo como "verdades universales" (en particular 2 bostezos/5 min y >20°).

## Decisión

Se adoptan los umbrales de la tabla siguiente como **defaults de ingeniería de `somnguard-device`** (todos configurables vía `detection_thresholds`, con calibración personal por encima vía `device_config.override.json`). La distinción normativa es explícita:

- **Umbral respaldado por literatura** → se cita la fuente.
- **Umbral de ingeniería inicial** → se marca como tal; requiere validación experimental propia.

| # | Evento | Apéndice 2 / HU original | Valor adoptado | Tipo | Justificación |
|---|--------|--------------------------|----------------|------|---------------|
| D-01 | EV-SOM-03 bostezo | 2 bostezos/5 min | **1 bostezo prolongado ≥3 s alerta** (conteo en ventana = evidencia) | Literatura + campo | PLOS: separación normal/fatiga ~3 s; en campo el 1º se contaba en silencio y parecía "no detectado" |
| D-02 | EV-SOM-04 cabeceo | tilt absoluto >20°/3 s | **tilt relativo ≥15°/3 s con neutro auto-cero** | Literatura + campo | MDPI: fatiga 12-20° vs normal 2-8°; el absoluto era inalcanzable por sesgo +170° medido |
| D-03 | EV-DIS-03 mirada | >3 s | **>2 s** | Literatura | NHTSA/Klauer: riesgo x3.8 desde 2 s, x8.9 a 5 s |
| D-04 | EV-SOM-01 parpadeo | rate anómalo | **+ cierre lento aislado 0.5-2 s** (`blink_slow_sec: 0.5`) | Literatura | Blink normal 0.1-0.4 s; había hueco ciego 0.6-2 s |
| D-05 | EV-SOM-05 microsueño | ojos>3 s + tilt | **sin cambio** | Campo | Pendiente de prueba real (pasos 5 del protocolo) |
| D-06 | EV-SOM-02 ojos | >2 s | **sin cambio (conservador)** | Ingeniería | Literatura sugiere 1-2 s, pero el costo FP de AS-02 es alto; se deja en 2.0 hasta validación |
| D-07 | EV-DIS-01/02 teléfono | >2 s />5 s | **sin cambio** | Literatura | Respaldados (NHTSA/Klauer); fail-safe sin modelo |
| D-08 | EV-CIN-01/02 cinturón | AS-07 intermitente | **`belt_enabled:false` (desactivado)** | HW | Cámara frontal a rostro no ve torso; activarlo = falsos garantizados. Código y tests listos tras el flag |
| D-09 | Alertas simultáneas | evento → alerta | **1 sonido por lote (mayor severidad) + ventana de prioridad 5 s** (`alert_priority_window_sec`) | Campo | `play()` corta al anterior; lo menor se loguea sin sonar |
| D-10 | MAR | borde externo [61,84,17,314,405,320] | **labio interno [61,13,82,291,87,14]** | Campo | El externo saturaba ~0.6 de frente; el interno supera 0.7 holgado |
| D-11 | Pose | umbrales absolutos + gate | **deltas relativos con wrap + auto-cero (60 muestras) + force-off 1 s en zona ciega** | Campo | Sin esto, tilt y mirada = 0 eventos con la cámara real |
| D-12 | Tilt durante bostezo | — | **suprimido mientras el bostezo está activo** | Campo | Bostezar echa la cabeza atrás 20-26° y disparaba AS-03 falso |

Parámetros por defecto resultantes (`config/device.default.json`): `yawn_min_duration_sec: 3.0`, `yawn_count_min: 1`, `head_tilt_deg_min: 15`, `gaze_duration_sec: 2.0`, `blink_slow_sec: 0.5`, `alert_priority_window_sec: 5.0`, `gaze_force_off_sec: 1.0`, `tilt_calib_samples: 60`. Implementación: `app/analysis/{somnolence,distraction,detector,ear_mar,landmarks}.py`, `app/device/manager.py`, `app/common/config.py`. Cobertura: 106 tests unitarios sin hardware.

## Alternativas consideradas

- **Mantener Apéndice 2 literal:** descartada; deja EV-SOM-03/04/05 y mirada inoperantes con el HW real (evidencia de campo).
- **Bajar ojos a 1.5 s y añadir EV-SOM-06 (sueño ≥3 s):** pospuesta a fase 2; exige nuevo tipo de evento (migración seed backend + API + docs), no se hace a escondidas en el device.
- **Score de fusión ponderado (PERCLOS+ojos+bostezo+cabeceo):** fase 2; PERCLOS ya existe como path a EV-SOM-01 (ventana 60 s, umbral 0.25).

## Consecuencias

- La HU, el Apéndice 2 y el seed deben actualizarse con D-01..D-04 o seguirán marcando desvío en auditoría (esta ADR es el puente).
- Calibración personal por equipo (`device_config.override.json`) sigue mandando sobre defaults.
- Riesgos conocidos: `tilt_calib_samples` exige ~5 s mirando al frente al arrancar; `yawn_count_min: 1` aumenta AS-02 por bostezo aislado (asumido); movimiento anómalo y PERCLOS-severidad sin validación de campo aún.

## Referencias

Euro NCAP Assessment Protocol SA Safe Driving v10.4 (drowsiness/microsleep/sleep 1-2 s/≥3 s) · NHTSA Driver Distraction Guidelines (>2 s) · Klauer et al. naturalistic driving (OR 3.8 @>2 s, 8.9 @>5 s) · PERCLOS review PMC10108649 · PLOS LDIE-FDNet (yawning >3 s) · MDPI Electronics 12(1):26 (head nod 12-20° vs normal 2-8°) · Evidencia de campo: logs/app 2026-09-16 + `data/mar_log.csv` + protocolo iterativo EV-SOM/DIS (10 eventos).
