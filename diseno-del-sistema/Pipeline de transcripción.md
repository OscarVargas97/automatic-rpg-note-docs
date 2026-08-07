---
entrada: "Pipeline de transcripción"
categoria: "Pipeline de transcripción"
estado: "Borrador"
prioridad: "Must"
complejidad: "Media"
---

# Pipeline de transcripción

Cómo el audio de una sesión de mesa se convierte en texto plano usando Whisper local — ver
el Log de decisiones en `meta/contexto-para-ia.md`, fila del 7 de agosto de 2026. Cubre solo
el paso 2 de `Pipeline de ingesta y enrutamiento.md` (transcripción), no lo que Claude hace
después con el texto.

## Variante elegida: faster-whisper

Entre whisper.cpp y faster-whisper (las dos variantes que dejó abiertas la decisión de
Whisper local), esta pieza elige **faster-whisper**:

- Corre en CPU sin necesitar GPU (con cuantización int8), lo cual importa porque no todas
  las máquinas de mesa van a tener una GPU dedicada.
- Se instala con `pip install faster-whisper` — no requiere compilar nada. whisper.cpp exige
  clonar el repo y compilarlo con CMake, un paso adicional que un script de setup tendría que
  automatizar aparte.
- La ventaja de whisper.cpp (aceleración vía Apple Neural Engine / Metal) solo aplica en Mac
  con chips M-series; sin ese contexto confirmado para quien va a correr el pipeline, no
  compensa la fricción extra de compilar.
- Da timestamps por segmento de fábrica, útil si en el futuro se quiere anclar una mención a
  un momento puntual de la sesión.

Si en el piloto en mesa la máquina real termina siendo un Mac con chip M-series, whisper.cpp
vuelve a ser candidato — queda anotado como alternativa, no descartado.

## Modelo

Empezar con el modelo `small` (244M parámetros): balance razonable entre calidad y velocidad
en CPU para una sesión de mesa de varias horas. Si el hardware es limitado, `base` (74M) es
la opción de repliegue — pierde precisión pero corre más rápido. `medium` o `large-v3` quedan
como opción si la calidad de `small` no alcanza en el piloto.

## Formato de entrada y salida

- **Entrada**: audio en WAV, 16 kHz, mono — el formato que el modelo espera de fábrica. Si la
  herramienta de captura (fuera de alcance de esta pieza, ver `Pipeline de ingesta y
  enrutamiento.md`) graba en otro formato o frecuencia, hace falta un paso de conversión
  antes de transcribir.
- **Salida**: texto plano por segmento, con marca de tiempo de inicio y fin de cada uno. El
  paso 3 del pipeline de ingesta (lectura completa por Claude) puede trabajar sobre el texto
  plano sin necesitar los timestamps, pero conviene conservarlos en el archivo intermedio por
  si una tarea futura los necesita.

## Requisitos para correr una transcripción

- Python 3.9 o superior.
- Paquete `faster-whisper` instalado (`pip install faster-whisper`).
- El archivo del modelo (`small` por defecto) descargado — se baja automáticamente la primera
  vez que se usa, o se puede pre-descargar.
- GPU: la máquina de referencia tiene una NVIDIA RTX 3070 de 8GB — con CUDA 12 / cuDNN 9,
  faster-whisper corre en GPU (float16) por defecto ahí. No es un requisito duro: si el
  módulo corre en una máquina sin GPU, cae a CPU con cuantización int8 sin cambiar de código,
  porque el módulo tiene que seguir funcionando en cualquier vault/máquina donde se instale
  (ver `## Diseño como módulo reusable`).

## Diseño como módulo reusable

Este pipeline no vive atado a `ObsidianRPG_Obsidian/`: se diseña como un módulo que recibe la
ruta del vault destino como parámetro (no hardcodeada), para poder apuntarlo a un vault de
Obsidian nuevo el día que exista una campaña real. Nada de lo que este módulo escribe asume
que el vault de destino es este repo de especificación.

## Múltiples audios por transcripción

El módulo soporta dos modos:

- **Un audio → una transcripción**: el caso simple, un archivo de entrada, un archivo de
  salida.
- **Varios audios → una transcripción unificada**: cuando una sesión quedó grabada en más de
  un archivo (corte de batería, pausa, grabación por partes), el módulo los concatena en
  orden y produce un único archivo de salida — marcando en el propio archivo qué segmento
  vino de qué audio de origen, para no perder la trazabilidad.

## Guardado del raw

La transcripción cruda (raw) se guarda **siempre** en `raws/`, en la raíz del vault destino
— ver la sección correspondiente en `Esquema del vault de campaña.md`. Este paso no se salta
nunca, sin importar si después hay o no un post-proceso: el raw es el respaldo si algo más
adelante en el pipeline se equivoca.

## Fuera de alcance de esta pieza

Diarización (separar quién habló) — ya señalada como abierta en `Pipeline de ingesta y
enrutamiento.md`. faster-whisper no la resuelve de fábrica; existe integración opcional con
`pyannote` que queda como candidata a investigación aparte si hace falta.
