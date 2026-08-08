---
documento: "Orquestador de transcripción"
area: "Infraestructura / Despliegue"
estado: "Vigente"
ruta_en_el_repo: "orchestrator/ (app Django, raíz de automatic-rpg-note)"
herramientas: ["Python", "Django", "Huey", "htmx", "Alpine.js"]
ultima_revision: "2026-08-07"
---

# Orquestador de transcripción

Implementación real de [[Orquestador y subida de audios]] — [[Tema #4 — Orquestador y
subida de audios para transcripción]]. App Django `orchestrator`, en la raíz del repo de
código (`automatic-rpg-note`, no en `docs/`). Código en inglés, texto de la UI en español —
ver `CLAUDE.md` sección 5, "Idioma del código".

## Modelos (`orchestrator/models.py`)

| Modelo | Campos | Nota |
|---|---|---|
| `CampaignProject` | `name`, `vault_path`, `created_at` | Al guardar, llama a `ensure_vault_structure()` — crea `raws/` y `campaña/{...}/` en `vault_path` si faltan, idempotente. |
| `TranscriptionJob` | `project` (FK), `status` (`pending`/`in_progress`/`done`/`error`, mostrado como Pendiente/En curso/Listo/Error), `progress` (0-100), `speaker_count` (opcional, `null` = sin diarización), `raw_path`, `error_message`, `created_at`, `updated_at` | Una fila por subida (uno o varios audios). |
| `UploadedAudio` | `job` (FK), `file` (FileField), `order` | `order` fija el orden de transcripción/unión de segmentos. |

## Vistas (`orchestrator/views.py`, `orchestrator/urls.py`)

| Ruta | Vista | Qué hace |
|---|---|---|
| `/` | `project_list` | Lista proyectos + form de creación. |
| `/projects/browse-folders/` | `browse_folders` | Explorador de carpetas del filesystem del servidor (parámetro `path`, por defecto `Path.home()`). Devuelve el fragmento `_folder_browser.html`: subcarpetas del path actual (oculta las que empiezan con `.`), link para subir al padre, y un botón "Usar esta carpeta" que escribe el path en el input `vault_path` vía JS inline (`hx-on:click`). No hay forma de leer una ruta absoluta real desde un `<input type="file">` del navegador por seguridad — como servidor y navegador son la misma máquina (uso local de un solo usuario), se resuelve navegando el filesystem del lado del servidor en vez de con un diálogo nativo del SO. |
| `/projects/create/` | `project_create` | Crea `CampaignProject` (POST). Dos modos, mismo endpoint (campo `mode`): `new` (cualquier ruta, crea la estructura si falta) o `existing` (la ruta debe tener ya `raws/` + `campaña/`, si no error). Rechaza `vault_path` duplicado en ambos modos. |
| `/projects/<id>/` | `project_detail` | Detalle: form de subida + lista de trabajos. |
| `/projects/<id>/jobs/create/` | `job_create` | Guarda los audios, crea `TranscriptionJob` + `UploadedAudio`, encola `transcribe_job` y devuelve el fragmento htmx de la fila del trabajo. Lee `speaker_count` del form (opcional): vacío → sin diarización; si viene, rechaza con 400 si no es un entero ≥ 2. |
| `/jobs/<id>/status/` | `job_status` | Fragmento htmx para polling (`_job_row.html`). |

## Transcripción en background (`orchestrator/tasks.py`)

`transcribe_job` es un `@db_task()` de Huey (`huey.contrib.djhuey`). Por cada
`UploadedAudio` del trabajo, en orden: transcribe con `core.transcription.load_model()`
(mismo helper que usa `check_transcription`, ver abajo), acumula los segmentos con offset de
tiempo, y al final escribe un único archivo en `<vault_path>/raws/`. Actualiza `status` y
`progress` (0-100, según cuánto del audio total ya se transcribió) en cada etapa; si algo
falla, `status = "error"` con el traceback corto en `error_message`.

Si `job.speaker_count` está seteado, antes de transcribir cada audio corre
`core.diarization.diarize()` (ver [[Transcripción con Whisper local]], sección
"Diarización") y cada línea del raw antepone la etiqueta de hablante que más se solapa con
ese segmento. El import de `core.diarization` es perezoso (dentro del `if
job.speaker_count`) — los jobs sin diarización no pagan el costo de cargar PyTorch.

Requiere un segundo proceso corriendo: `make huey` (además de `make run`). Sin eso, los
trabajos quedan en `pending` indefinidamente — no hay nada que los procese.

Configuración en `config/settings.py`, sección `HUEY`: `SqliteHuey`, `immediate: False` (a
propósito — `True` ejecutaría los jobs en el mismo proceso de la request, anulando el punto
de tenerlos en background).

## `core/transcription.py` — corrección sobre lo que dejó Tema #3

Se extrajo `load_model()` de `check_transcription.py` a este módulo para no duplicar la
lógica GPU→CPU entre el comando y el task de Huey. Al reutilizarla se encontró un bug real:
`WhisperModel(device="cuda")` puede construirse sin error aunque falten librerías de cómputo
(p. ej. `libcublas.so.12`) — CTranslate2 solo las carga en el primer uso real, no al
construir el objeto. `check_transcription` (Tema #3) nunca lo detectó porque solo construye
el modelo, nunca transcribe. `load_model()` ahora fuerza una transcripción mínima (1s de
silencio, en memoria, sin archivo) dentro del mismo `try`, así que un fallo de GPU en el
primer uso real cae a CPU igual que un fallo de construcción.

## Formato del raw

Texto plano en `<vault>/raws/<fecha_UTC>_job-<id>.md`. Un encabezado `## <nombre de
archivo>` por audio de origen, seguido de sus segmentos `[inicio - fin] texto` (o `[inicio -
fin] SPEAKER_00: texto` si el job pidió diarización) con offset acumulado — ver [[Orquestador
y subida de audios]] para el razonamiento de por qué se unen segmentos en vez de concatenar
audio crudo.

## Frontend

`templates/orchestrator/`: `base.html` (Tailwind vía CDN con paleta oscura inspirada en
shadcn/ui, htmx y Alpine.js vía CDN, sin build de Node), `projects.html`,
`project_detail.html` (incluye el campo opcional "Número de hablantes" —
`input[name=speaker_count]`, `type=number`, `min=2` — en el form de subida),
`_job_row.html` (fragmento reusado en la creación del trabajo y en el polling — se
autoexcluye del polling agregando o quitando `hx-trigger` según el `status`),
`_folder_browser.html` (explorador de carpetas, ver tabla de vistas arriba). Los
identificadores del template (nombres de variable, `id`, atributos `name` de
los campos del form) están en inglés; el texto que el usuario lee (labels, botones,
mensajes) está en español.

## Verificado

Flujo completo probado de punta a punta (modelos, `call_local()` del task, y las vistas HTTP
reales vía `Client`), repetido después del rename a inglés: crear proyecto → estructura de
vault generada → subir 2 audios → job encolado (`pending`) → procesado → raw escrito con
segmentos de ambos audios y offsets correctos → `status = "done"`. También probado:
`mode="existing"` contra una ruta sin estructura (rechaza con el mensaje correcto),
`mode="new"` crea y registra, y el rechazo de `vault_path` duplicado. También probado
`browse_folders`: listado de subcarpetas, exclusión de ocultas, link "subir" al padre desde
una subcarpeta, y fallback a `Path.home()` si el `path` pedido no existe. No probado: un
audio real de 2-3 horas (se usó un WAV de prueba corto), ni la subida por HTTP de un archivo
de cientos de MB.

**Diarización (Tema #6)**: probado con `call_local()`, un job sin `speaker_count` y otro con
`speaker_count=2` contra el mismo audio (ruido sintético, sin voz real) — ambos terminan
`status = "done"` sin excepciones, confirmando que el camino con diarización activada no
rompe el flujo existente ni cuando Silero VAD no detecta voz. También probada la vista
`job_create` real vía `Client` HTTP: sin `speaker_count` (200, sin diarización), con
`speaker_count=3` (200, job creado con diarización pedida), y rechazo correcto (400) de
`speaker_count=1` y de un valor no numérico. No probado: calidad real de separación de voces
— sigue sin haber un audio de prueba con más de un hablante (ver [[Transcripción con Whisper
local]]).
