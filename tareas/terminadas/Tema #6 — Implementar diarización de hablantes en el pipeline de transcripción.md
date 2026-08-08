---
id: "TSK-6"
titulo: "Tema #6 — Implementar diarización de hablantes en el pipeline de transcripción"
estado: "Listo"
tipo: "Feature"
disciplina: "Transcripción y audio"
prioridad: "P2 - Media"
hito: "Prototipo"
segmentos_a_actualizar: ["Diseño del sistema", "Documentación técnica"]
segmentos_actualizados: true
definicion_de_hecho: true
documentacion_a_actualizar: ["[[Transcripción con Whisper local]]", "[[Orquestador de transcripción]]"]
diseno_de_referencia: ["[[Pipeline de transcripción]]", "[[Orquestador y subida de audios]]"]
costo_asociado: []
rama: "feature/TSK-6-tema-6-implementar-diarizacion-de-hablan"
responsable: ""
estimacion_dias: ""
fechas: ""
bloqueada_por: ""
---

## Problema
TSK-5 confirmó que existe una ruta viable, local y sin credenciales externas para
diarización (Silero VAD + SpeechBrain ECAPA-TDNN + clustering), pero quedó solo evaluada —
el pipeline real (`orchestrator/tasks.py`, `core/transcription.py`) sigue sin poder separar
voces distintas en el raw.

## Resultado esperado
Al subir audio(s) para transcribir, el usuario puede indicar opcionalmente cuántos hablantes
distintos hay; si lo hace, el raw resultante marca cada segmento con una etiqueta genérica de
hablante (`SPEAKER_00`, `SPEAKER_01`, …) además del texto y el timestamp, usando Silero VAD +
SpeechBrain ECAPA-TDNN + clustering — sin depender de ninguna cuenta ni token externo. Si el
usuario no indica número de hablantes, el job se transcribe igual que hoy, sin diarización.
La calidad real de separación de voces con audio real de mesa queda como riesgo conocido a
validar, no como bloqueante de esta tarea.

## Referencias
- docs/diseno-del-sistema/Pipeline de transcripción.md
- docs/diseno-del-sistema/Orquestador y subida de audios.md
- docs/documentacion-tecnica/Transcripción con Whisper local.md
- docs/documentacion-tecnica/Orquestador de transcripción.md
- docs/tareas/terminadas/Tema #5 — Investigar diarización de hablantes para distinguir voces distintas.md

Origen: conversación con Oscar, 2026-08-07.

## Registro de cierre

Ejecutado en la rama `feature/TSK-6-tema-6-implementar-diarizacion-de-hablan`, sin commit
todavía.

Decisiones resueltas (ya cerradas al crear la tarea, sin volver a preguntar):
- Alcance de ejecución → **opcional por job**, campo `speaker_count` vacío por defecto.
- Número de hablantes → **lo indica el usuario**, no detección automática.
- Audio de prueba → **se siguió sin él**, calidad real queda como riesgo conocido documentado,
  no bloqueante.

Decisión técnica nueva durante la implementación: `silero_vad.read_audio()` depende de
`torchaudio`, que en la versión instalada (2.9+) requiere el paquete separado `torchcodec`
para decodificar audio — sin él, revienta con `ImportError`. En vez de agregar `torchcodec`
como dependencia nueva (con el mismo tipo de fragilidad de librerías de sistema que ya se vio
con `pyannote` en Tema #5), `core/diarization.py` decodifica el audio con
`faster_whisper.audio.decode_audio` (PyAV) — dependencia que el proyecto ya tenía probada — y
se lo pasa a Silero VAD como tensor. Confirmado con pruebas reales que este camino funciona.

Excepciones duras encontradas: ninguna. No se cambió el transcriptor elegido (Whisper local
sigue intacto — la diarización es un paso aparte, opcional). No se promovió ninguna pieza de
diseño a Canon (`Pipeline de transcripción.md` sigue en `Borrador`). Sí se escribió código de
implementación real del sistema — la propia tarea lo declaraba explícitamente
(`core/diarization.py`, cambios en `orchestrator/`), y "dónde vive" ya estaba resuelto (este
mismo repo, `CLAUDE.md` sección 2).

Segmentos:
- Diseño del sistema → actualizado: `Pipeline de transcripción.md` (formato de línea con
  hablante formalizado, mecanismo VAD+embeddings+clustering documentado, "sin verificar"
  actualizado a lo que de verdad falta) y `Orquestador y subida de audios.md` (campo opcional
  en el flujo de subida).
- Documentación técnica → actualizado: `Transcripción con Whisper local.md` y `Orquestador de
  transcripción.md`, con la implementación real, los módulos/campos nuevos, y los resultados
  de las pruebas hechas.

Verificado (real, con comandos ejecutados — no solo lectura de código):
- `python manage.py check` y `makemigrations`/`migrate` sin errores
  (`orchestrator/migrations/0003_transcriptionjob_speaker_count.py`).
- `core.diarization.diarize()` aislado, contra silencio y contra ruido sintético: no
  encuentra voz en ninguno de los dos (correcto, ninguno es habla), sin excepciones.
- `orchestrator.tasks.transcribe_job.call_local()` de punta a punta, un job sin
  `speaker_count` y otro con `speaker_count=2`, mismo audio de ruido: ambos `status = "done"`
  sin errores.
- Vista `job_create` real vía `Client` HTTP: sin `speaker_count` (200), con `speaker_count=3`
  (200), `speaker_count=1` (400, rechazado), `speaker_count="abc"` (400, rechazado).
- Todos los datos de prueba (proyectos, jobs, archivos, carpetas temporales) se crearon y se
  borraron dentro de la misma prueba — no quedó basura en `db.sqlite3` ni en el filesystem.

Supuestos asumidos: ninguno nuevo — los de la creación de la tarea siguen en pie (calidad de
diarización sin verificar con audio real).

Fuera de alcance: identificar cuál hablante es el máster (candidata a tarea aparte cuando
exista el pipeline de ingesta con Claude); mostrar `speaker_count` en la fila del trabajo en
la UI (no lo pedía el resultado esperado, se puede sumar después si hace falta).

Origen: implementación con Claude, 2026-08-07.
