---
documento: "Orquestador de transcripción"
area: "Infraestructura / Despliegue"
estado: "Vigente"
ruta_en_el_repo: "orquestador/ (app Django, raíz de automatic-rpg-note)"
herramientas: ["Python", "Django", "Huey", "htmx", "Alpine.js"]
ultima_revision: "2026-08-07"
---

# Orquestador de transcripción

Implementación real de [[Orquestador y subida de audios]] — [[Tema #4 — Orquestador y
subida de audios para transcripción]]. App Django `orquestador`, en la raíz del repo de
código (`automatic-rpg-note`, no en `docs/`).

## Modelos (`orquestador/models.py`)

| Modelo | Campos | Nota |
|---|---|---|
| `ProyectoCampana` | `nombre`, `ruta_vault`, `creado_en` | Al guardar, llama a `asegurar_estructura_vault()` — crea `raws/` y `campaña/{...}/` en `ruta_vault` si faltan, idempotente. |
| `TrabajoTranscripcion` | `proyecto` (FK), `estado` (`pendiente`/`en_curso`/`listo`/`error`), `ruta_raw`, `error_mensaje`, `creado_en`, `actualizado_en` | Una fila por subida (uno o varios audios). |
| `AudioSubido` | `trabajo` (FK), `archivo` (FileField), `orden` | `orden` fija el orden de transcripción/unión de segmentos. |

## Vistas (`orquestador/views.py`, `orquestador/urls.py`)

| Ruta | Vista | Qué hace |
|---|---|---|
| `/` | `proyecto_list` | Lista proyectos + form de creación. |
| `/proyectos/crear/` | `proyecto_create` | Crea `ProyectoCampana` (POST). |
| `/proyectos/<id>/` | `proyecto_detail` | Detalle: form de subida + lista de trabajos. |
| `/proyectos/<id>/trabajos/crear/` | `trabajo_create` | Guarda los audios, crea `TrabajoTranscripcion` + `AudioSubido`, encola `transcribir_trabajo` y devuelve el fragmento htmx de la fila del trabajo. |
| `/trabajos/<id>/estado/` | `trabajo_status` | Fragmento htmx para polling (`_trabajo_row.html`). |

## Transcripción en background (`orquestador/tasks.py`)

`transcribir_trabajo` es un `@db_task()` de Huey (`huey.contrib.djhuey`). Por cada
`AudioSubido` del trabajo, en orden: transcribe con `core.transcripcion.cargar_modelo()`
(mismo helper que usa `check_transcription`, ver abajo), acumula los segmentos con offset de
tiempo, y al final escribe un único archivo en `<ruta_vault>/raws/`. Actualiza `estado` en
cada etapa; si algo falla, `estado = "error"` con el traceback corto en `error_mensaje`.

Requiere un segundo proceso corriendo: `make huey` (además de `make run`). Sin eso, los
trabajos quedan en `pendiente` indefinidamente — no hay nada que los procese.

Configuración en `config/settings.py`, sección `HUEY`: `SqliteHuey`, `immediate: False` (a
propósito — `True` ejecutaría los jobs en el mismo proceso de la request, anulando el punto
de tenerlos en background).

## `core/transcripcion.py` — corrección sobre lo que dejó Tema #3

Se extrajo `cargar_modelo()` de `check_transcription.py` a este módulo para no duplicar la
lógica GPU→CPU entre el comando y el task de Huey. Al reutilizarla se encontró un bug real:
`WhisperModel(device="cuda")` puede construirse sin error aunque falten librerías de cómputo
(p. ej. `libcublas.so.12`) — CTranslate2 solo las carga en el primer uso real, no al
construir el objeto. `check_transcription` (Tema #3) nunca lo detectó porque solo construye
el modelo, nunca transcribe. `cargar_modelo()` ahora fuerza una transcripción mínima (1s de
silencio, en memoria, sin archivo) dentro del mismo `try`, así que un fallo de GPU en el
primer uso real cae a CPU igual que un fallo de construcción.

## Formato del raw

Texto plano en `<vault>/raws/<fecha_UTC>_trabajo-<id>.md`. Un encabezado `## <nombre de
archivo>` por audio de origen, seguido de sus segmentos `[inicio - fin] texto` con offset
acumulado — ver [[Orquestador y subida de audios]] para el razonamiento de por qué se unen
segmentos en vez de concatenar audio crudo.

## Frontend

`templates/orquestador/`: `base.html` (Tailwind vía CDN con paleta oscura inspirada en
shadcn/ui, htmx y Alpine.js vía CDN, sin build de Node), `proyectos.html`,
`proyecto_detail.html`, `_trabajo_row.html` (fragmento reusado en la creación del trabajo y
en el polling — se autoexcluye del polling agregando o quitando `hx-trigger` según el
`estado`).

## Verificado

Flujo completo probado de punta a punta (modelos, `call_local()` del task, y las vistas HTTP
reales vía `Client`): crear proyecto → estructura de vault generada → subir 2 audios → job
encolado (`pendiente`) → procesado → raw escrito con segmentos de ambos audios y offsets
correctos → `estado = "listo"`. No probado: un audio real de 2-3 horas (se usó un WAV de
prueba corto), ni la subida por HTTP de un archivo de cientos de MB.
