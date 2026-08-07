---
id: "TSK-3"
titulo: "Tema #3 — Instalación de dependencias de transcripción en Django"
estado: "Listo"
tipo: "Feature"
disciplina: "Transcripción y audio"
prioridad: "P1 - Alta"
hito: "Prototipo"
segmentos_a_actualizar: ["Documentación técnica"]
segmentos_actualizados: true
definicion_de_hecho: true
documentacion_a_actualizar: ["[[Transcripción con Whisper local]]"]
diseno_de_referencia: ["[[Pipeline de transcripción]]"]
costo_asociado: []
rama: "feature/TSK-3-tema-3-instalacion-de-dependencias-de-tr"
responsable: ""
estimacion_dias: ""
fechas: ""
bloqueada_por: ""
---

## Problema
Levantar todo lo necesario para transcribir una sesión (instalar faster-whisper, bajar el
modelo, dejarlo listo para correr) es hoy un proceso manual sin documentar ni automatizar —
cada vez que alguien lo necesita, lo reconstruye desde cero. Esas dependencias son externas a
Django (no las trae el framework), así que instalarlas tiene que quedar integrado dentro del
proyecto que levantó [[Tema #2 — Base del proyecto Django]], no como un script aparte y
desconectado de la app.

## Resultado esperado
Dentro del proyecto Django de [[Tema #2 — Base del proyecto Django]], existe una forma de
dejar instalado y listo para usar todo lo necesario para transcribir audio con Whisper local
(según lo que definió [[Tema #1 — Investigar implementación de transcripción con Whisper
local]]) sin pasos manuales adicionales más allá de dispararla desde Django — por ejemplo, un
comando de gestión (`manage.py`) o un chequeo que corre al iniciar la app; la forma exacta se
decide al ejecutar esta tarea. Usa GPU (la máquina de referencia tiene una RTX 3070 de 8GB,
con CUDA) cuando está disponible, y cae a CPU si no la hay. `Transcripción con Whisper local`
refleja el resultado real: qué instala, cómo se dispara desde Django, y su ruta en el repo.
Esta tarea instala las dependencias; no implementa todavía la lógica de transcribir un audio
(eso es parte de [[Tema #4 — Orquestador y subida de audios para transcripción]]).

## Referencias
- ObsidianRPG_Obsidian/diseno-del-sistema/Pipeline de transcripción.md
- src/ (el proyecto Django de Tema #2 vive ahí)

Origen: conversación con Oscar, 2026-08-07.

## Registro de cierre

Ejecutado en la rama feature/TSK-3-tema-3-instalacion-de-dependencias-de-tr, sin commit
todavía.

Decisiones resueltas:
- Forma exacta de dejar instalado y listo Whisper local → comando de gestión
  `manage.py check_transcription` en la app `core` (no una app nueva: el alcance —una sola
  pieza de setup— no justifica crear `transcripcion/` aparte; queda de precedente para que
  Tema #4 decida si el orquestador sí lo justifica).
- Dependencia `faster-whisper` → declarada en `src/pyproject.toml` vía `uv add`, fijada en
  `src/uv.lock`, se instala junto con el resto del proyecto (`make install`).

Excepciones duras encontradas: ninguna — no se tocó el transcriptor elegido, no hay pieza
Canon involucrada, y el código de implementación ya tenía resuelto dónde vive (`src/`,
CLAUDE.md sección 2) y esta tarea lo declaraba explícitamente.

Segmentos:
- Documentación técnica → actualizado: `Transcripción con Whisper local.md` (ruta en el
  repo, cómo se dispara desde Django, resultado de la verificación real) y, como corrección
  necesaria fuera de la lista declarada pero dentro del mismo segmento, la línea de
  `Proyecto Django.md` que decía que `core` no tenía lógica propia todavía — dejó de ser
  cierto con este comando.

Supuestos asumidos:
- Modelo `small` como tamaño verificado (ya era el default elegido en
  `Pipeline de transcripción.md`, no se cambió).
- El comando vive en `core` en vez de una app `transcripcion` nueva.

Fuera de alcance: verificar el camino de repliegue a CPU (la máquina de verificación tenía
GPU con CUDA disponible) — no bloquea el cierre porque el código ya cae a CPU sin cambios si
`WhisperModel(..., device="cuda", ...)` lanza excepción, pero queda sin probar en una máquina
real sin GPU.

Origen: implementación con Claude, 2026-08-07.

## Verificación adicional — 2026-08-07

Se probó el repliegue a CPU ocultándole CUDA al proceso (`CUDA_VISIBLE_DEVICES=""` sobre
`manage.py check_transcription`) en vez de esperar una máquina real sin GPU. Cayó
correctamente a `CPU (int8)`, sin cambios de código. Cabo suelto cerrado — no se abre tarea
nueva para esto.

Origen: verificación con Claude, 2026-08-07.
