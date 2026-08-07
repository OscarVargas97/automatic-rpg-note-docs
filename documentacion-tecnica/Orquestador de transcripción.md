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
| `TranscriptionJob` | `project` (FK), `status` (`pending`/`in_progress`/`done`/`error`, mostrado como Pendiente/En curso/Listo/Error), `raw_path`, `error_message`, `created_at`, `updated_at` | Una fila por subida (uno o varios audios). |
| `UploadedAudio` | `job` (FK), `file` (FileField), `order` | `order` fija el orden de transcripción/unión de segmentos. |

## Vistas (`orchestrator/views.py`, `orchestrator/urls.py`)

| Ruta | Vista | Qué hace |
|---|---|---|
| `/` | `project_list` | Lista proyectos + form de creación. |
| `/projects/create/` | `project_create` | Crea `CampaignProject` (POST). Dos modos, mismo endpoint (campo `mode`): `new` (cualquier ruta, crea la estructura si falta) o `existing` (la ruta debe tener ya `raws/` + `campaña/`, si no error). Rechaza `vault_path` duplicado en ambos modos. |
| `/projects/<id>/` | `project_detail` | Detalle: form de subida + lista de trabajos. |
| `/projects/<id>/jobs/create/` | `job_create` | Guarda los audios, crea `TranscriptionJob` + `UploadedAudio`, encola `transcribe_job` y devuelve el fragmento htmx de la fila del trabajo. |
| `/jobs/<id>/status/` | `job_status` | Fragmento htmx para polling (`_job_row.html`). |

## Transcripción en background (`orchestrator/tasks.py`)

`transcribe_job` es un `@db_task()` de Huey (`huey.contrib.djhuey`). Por cada
`UploadedAudio` del trabajo, en orden: transcribe con `core.transcription.load_model()`
(mismo helper que usa `check_transcription`, ver abajo), acumula los segmentos con offset de
tiempo, y al final escribe un único archivo en `<vault_path>/raws/`. Actualiza `status` en
cada etapa; si algo falla, `status = "error"` con el traceback corto en `error_message`.

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
archivo>` por audio de origen, seguido de sus segmentos `[inicio - fin] texto` con offset
acumulado — ver [[Orquestador y subida de audios]] para el razonamiento de por qué se unen
segmentos en vez de concatenar audio crudo.

## Frontend

`templates/orchestrator/`: `base.html` (Tailwind vía CDN con paleta oscura inspirada en
shadcn/ui, htmx y Alpine.js vía CDN, sin build de Node), `projects.html`,
`project_detail.html`, `_job_row.html` (fragmento reusado en la creación del trabajo y
en el polling — se autoexcluye del polling agregando o quitando `hx-trigger` según el
`status`). Los identificadores del template (nombres de variable, `id`, atributos `name` de
los campos del form) están en inglés; el texto que el usuario lee (labels, botones,
mensajes) está en español.

## Verificado

Flujo completo probado de punta a punta (modelos, `call_local()` del task, y las vistas HTTP
reales vía `Client`), repetido después del rename a inglés: crear proyecto → estructura de
vault generada → subir 2 audios → job encolado (`pending`) → procesado → raw escrito con
segmentos de ambos audios y offsets correctos → `status = "done"`. También probado:
`mode="existing"` contra una ruta sin estructura (rechaza con el mensaje correcto),
`mode="new"` crea y registra, y el rechazo de `vault_path` duplicado. No probado: un audio
real de 2-3 horas (se usó un WAV de prueba corto), ni la subida por HTTP de un archivo de
cientos de MB.
