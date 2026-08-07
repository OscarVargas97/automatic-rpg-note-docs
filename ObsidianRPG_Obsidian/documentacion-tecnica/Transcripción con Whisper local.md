---
documento: "Transcripción con Whisper local"
area: "Transcripción (Whisper)"
estado: "Vigente"
ruta_en_el_repo: ""
herramientas: ["Whisper (local)", "Python"]
ultima_revision: "2026-08-07"
---

# Transcripción con Whisper local

Describe cómo transcribir el audio de una sesión de mesa con Whisper local. El diseño detrás
de estas decisiones vive en `diseno-del-sistema/Pipeline de transcripción.md` — esta nota es
el reflejo de implementación; si algún día divergen, gana esta y se corrige la de diseño.

`ruta_en_el_repo` queda vacía porque el script que automatiza esta instalación todavía no
existe — lo crea [[Tema #2 — Script de setup de transcripción]] en `src/`. Cuando exista, se
actualiza este campo.

## Instalación

1. Python 3.9 o superior instalado.
2. `pip install faster-whisper`.
3. Descargar (o dejar que se descargue en el primer uso) el modelo `small`.

No hace falta compilar nada ni instalar FFmpeg aparte: faster-whisper decodifica audio con
PyAV, que trae sus propias librerías de FFmpeg.

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
