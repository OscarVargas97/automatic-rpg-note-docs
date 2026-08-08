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
- **Salida**: texto plano por segmento, con marca de tiempo de inicio y fin de cada uno —
  `[inicio - fin] texto`. El paso 3 del pipeline de ingesta (lectura completa por Claude)
  puede trabajar sobre el texto plano sin necesitar los timestamps, pero conviene
  conservarlos en el archivo intermedio por si una tarea futura los necesita. Si el usuario
  indicó número de hablantes al subir el audio (ver `## Diarización de hablantes`), cada
  línea antepone una etiqueta genérica: `[inicio - fin] SPEAKER_00: texto`.

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

## Diarización de hablantes (evaluado en Tema #5, implementado en Tema #6)

Investigado en [[Tema #5 — Investigar diarización de hablantes para distinguir voces
distintas]] e implementado en [[Tema #6 — Implementar diarización de hablantes en el
pipeline de transcripción]]. La pregunta de origen (`Pipeline de ingesta y enrutamiento.md`)
es si el pipeline puede separar voces distintas — no todavía quién de ellas es el máster,
eso sigue fuera de alcance.

**Cómo se activa**: es opcional por trabajo de transcripción, no automático. Al subir el
audio, el usuario puede indicar cuántos hablantes distintos hay; si lo deja vacío, el job se
transcribe igual que antes de esta pieza, sin diarización — así los audios de un solo
hablante (o donde no importa distinguir voces) no pagan el costo extra de cómputo.

**Cómo funciona**: por cada audio, Silero VAD marca en qué tramos hay voz; esos tramos se
parten en ventanas de ~1.5s (lo mínimo para que SpeechBrain ECAPA-TDNN saque una huella de
voz estable); las huellas se agrupan con `AgglomerativeClustering` de scikit-learn en tantos
grupos como hablantes se pidieron; cada grupo se etiqueta `SPEAKER_00`, `SPEAKER_01`, etc. Un
segmento de Whisper hereda la etiqueta de la ventana de diarización con más solapamiento de
tiempo — si no hay ninguna (VAD no detectó voz en ese tramo), el segmento queda sin etiqueta.

**Candidata descartada**: `pyannote.audio`, la variante más citada para diarización con
Whisper. Su modelo (`pyannote/speaker-diarization-3.1`) es un repo "gated" en HuggingFace —
exige cuenta, aceptar términos y un token de autenticación. Confirmado empíricamente en la
máquina de referencia: sin token, falla con `GatedRepoError 401`. Se descarta porque el
proyecto necesita una solución sin servicios externos ni credenciales por máquina, y la
inferencia local no cambia eso — el bloqueo está en la descarga inicial del modelo.

**Candidata viable, sin autenticación**: un pipeline propio de VAD + embeddings de hablante +
clustering. Dos piezas se probaron en la máquina de referencia (RTX 3070) y cargaron y
corrieron en GPU sin ningún login ni token:

- **Silero VAD** (detección de actividad de voz) — pesos incluidos en el propio paquete pip,
  ~9 MB, no llega a pedirle nada a HuggingFace.
- **SpeechBrain ECAPA-TDNN** (`speechbrain/spkrec-ecapa-voxceleb`, embeddings de hablante) —
  se descarga de HuggingFace sin autenticación (solo una advertencia de límite de tasa por no
  usar token, no un bloqueo).

Estos dos resuelven "dónde hay voz" y la "huella" de cada segmento de voz; agrupar esas
huellas por hablante (clustering, ej. agglomerative clustering de scikit-learn) es
integración propia — a diferencia de pyannote, que trae eso resuelto en un solo `Pipeline`.
Se evaluó `simple-diarizer` (PyPI) como wrapper ya armado sobre esta misma base, pero su
último release es de diciembre de 2022 — sin mantenimiento activo, mejor escribir la
integración directo sobre Silero + SpeechBrain que depender de un wrapper abandonado.

**Costo de adoptar esta ruta**: introduce PyTorch como dependencia nueva. El stack actual de
transcripción usa `ctranslate2` (vía faster-whisper), no PyTorch — el `.venv` del proyecto
pesa ~426 MB sin él. Instalar SpeechBrain + Silero VAD suma varios GB, sobre todo el runtime
de CUDA que PyTorch trae consigo.

**Sin verificar**: la calidad real de separación de voces con audio de una sesión de mesa —
Tema #6 tampoco tuvo un audio con más de un hablante para probar. El código corre y produce
etiquetas, pero si esas etiquetas corresponden de verdad a hablantes distintos queda como
riesgo conocido hasta la primera vez que se use con audio real de mesa.

## Fuera de alcance de esta pieza

Identificar cuál hablante es el máster (mapear una etiqueta genérica a una identidad) — el
sistema solo distingue voces, no sabe cuál es cuál. Candidata a tarea aparte cuando el
pipeline de ingesta con Claude (`Pipeline de ingesta y enrutamiento.md`) exista y pueda
apoyar esa inferencia.
