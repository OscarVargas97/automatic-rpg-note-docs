---
documento: "Transcripción con Whisper local"
area: "Transcripción (Whisper)"
estado: "Vigente"
ruta_en_el_repo: "core/management/commands/check_transcription.py"
herramientas: ["Whisper (local)", "Python"]
ultima_revision: "2026-08-07"
---

# Transcripción con Whisper local

Describe cómo transcribir el audio de una sesión de mesa con Whisper local. El diseño detrás
de estas decisiones vive en `diseno-del-sistema/Pipeline de transcripción.md` — esta nota es
el reflejo de implementación; si algún día divergen, gana esta y se corrige la de diseño.

El código real vive en el repo `automatic-rpg-note-src` (ver CLAUDE.md sección 2) — las
rutas de esta nota son relativas a ese repo, no a `automatic-rpg-note`.

## Instalación

`faster-whisper` es una dependencia declarada en `pyproject.toml` (fijada en `uv.lock`) del
proyecto Django de [[Tema #2 — Base del proyecto Django]] — se instala junto con el resto del
proyecto (`make install` / `uv sync`), sin paso manual aparte.

No hace falta compilar nada ni instalar FFmpeg aparte: faster-whisper decodifica audio con
PyAV, que trae sus propias librerías de FFmpeg.

## Cómo se dispara desde Django

[[Tema #3 — Instalación de dependencias de transcripción en Django]] agrega el comando de
gestión `check_transcription` a la app `core`:

```
uv run python manage.py check_transcription
```

El comando instancia el modelo `small` de faster-whisper — lo que dispara la descarga desde
Hugging Face Hub si todavía no está en caché local — intentando primero GPU
(`device="cuda", compute_type="float16"`) y cayendo a CPU (`device="cpu",
compute_type="int8"`) si CUDA no está disponible, sin cambiar de código. Al terminar, imprime
en qué dispositivo quedó listo el modelo. Esta tarea solo deja el modelo instalado y
verificado; no transcribe un audio real — eso es alcance de [[Tema #4 — Orquestador y subida
de audios para transcripción]].

**Verificado en esta implementación**: `check_transcription` se corrió en el sandbox (GPU
NVIDIA con CUDA disponible) y confirmó `Modelo 'small' listo para transcribir — GPU (cuda,
float16).`. No se verificó el camino de repliegue a CPU (la máquina de verificación sí tenía
GPU) — queda sin probar hasta que se corra en una máquina sin CUDA.

## GPU

Máquina de referencia: NVIDIA RTX 3070 de 8GB. Con CUDA 12 y cuDNN 9 instalados,
faster-whisper corre en GPU (float16) por defecto ahí — notablemente más rápido que CPU. No
es un requisito duro: como el módulo tiene que poder instalarse en cualquier máquina (ver
`Pipeline de transcripción.md`, sección de diseño como módulo reusable), si no hay GPU
disponible cae a CPU con cuantización int8 sin cambios de código.

## Modelo

Por defecto, `small` (244M parámetros). Si la máquina es limitada, `base` (74M) es la opción
de repliegue. `medium` o `large-v3` quedan disponibles si `small` no da la calidad suficiente
en el piloto en mesa.

## Formato de entrada

WAV, 16 kHz, mono. Si la herramienta de captura graba en otro formato, hace falta convertir
antes de pasarlo a faster-whisper.

## Formato de salida

Segmentos con marca de tiempo de inicio y fin, más el texto transcrito de cada uno —
formato nativo de la librería `faster-whisper` en Python
(`[%.2fs -> %.2fs] %s`). El paso de ingesta con Claude puede trabajar sobre el texto plano
concatenado sin necesitar los timestamps, pero conviene conservar el archivo con segmentos
por si hace falta anclar una mención a un momento puntual más adelante.

## Múltiples audios y guardado del raw

El módulo acepta un solo audio o varios (concatenados en orden, para sesiones grabadas en
más de un archivo). En ambos casos, el raw resultante se guarda siempre en `raws/`, en la
raíz del vault destino — nunca en un directorio temporal que se pueda perder.

## Fuera de alcance

Diarización (separar quién habló): no viene de fábrica en faster-whisper. Existe integración
opcional con `pyannote` — sin investigar todavía, candidata a tarea aparte si el piloto en
mesa la necesita.
