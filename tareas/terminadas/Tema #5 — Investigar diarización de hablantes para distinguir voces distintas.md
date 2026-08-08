---
id: "TSK-5"
titulo: "Tema #5 — Investigar diarización de hablantes para distinguir voces distintas"
estado: "Listo"
tipo: "Investigación"
disciplina: "Transcripción y audio"
prioridad: "P1 - Alta"
hito: "Prototipo"
segmentos_a_actualizar: ["Diseño del sistema", "Documentación técnica", "Contexto para IA"]
segmentos_actualizados: true
definicion_de_hecho: true
documentacion_a_actualizar: ["[[Transcripción con Whisper local]]"]
diseno_de_referencia: ["[[Pipeline de transcripción]]", "[[Pipeline de ingesta y enrutamiento]]"]
costo_asociado: []
rama: "feature/TSK-5-tema-5-investigar-diarizacion-de-hablant"
responsable: ""
estimacion_dias: ""
fechas: ""
bloqueada_por: ""
---

## Problema
`Pipeline de ingesta y enrutamiento.md` deja abierta la pregunta de si el resumen puede
distinguir "dijo el máster" de "dijo un jugador". Antes de resolver esa pregunta hace falta
saber si el pipeline puede siquiera separar voces distintas — hoy la transcripción es un
bloque continuo sin esa información, y Whisper no la resuelve de fábrica.

## Resultado esperado
`Pipeline de transcripción` y `Transcripción con Whisper local` documentan si se adopta
diarización de hablantes (candidata: `pyannote.audio`, evaluada en la máquina real de mesa —
NVIDIA RTX 3070) para identificar qué segmentos de la transcripción corresponden a cada voz
distinta, con etiquetas genéricas sin identidad (ej. `SPEAKER_00`, `SPEAKER_01`) — sin
determinar todavía cuál de esas voces es el máster. También documentan qué requiere
adoptarla (dependencias, token de HuggingFace, impacto en tiempo de procesamiento) y cómo
cambiaría el formato del raw. La decisión (adoptar o descartar) queda registrada en el Log
de decisiones de `contexto-para-ia.md`. Si se adopta, la integración al pipeline real y la
identificación de cuál hablante es el máster quedan para tareas aparte.

## Referencias
- docs/diseno-del-sistema/Pipeline de transcripción.md
- docs/diseno-del-sistema/Pipeline de ingesta y enrutamiento.md
- docs/documentacion-tecnica/Transcripción con Whisper local.md
- docs/meta/contexto-para-ia.md

Origen: conversación con Oscar, 2026-08-07.

## Registro de cierre

Ejecutado en la rama `feature/TSK-5-tema-5-investigar-diarizacion-de-hablant`, sin commit
todavía.

Decisiones resueltas:
- Candidata original (`pyannote.audio`) → **descartada**. Confirmado empíricamente en la
  máquina de referencia (RTX 3070) que `pyannote/speaker-diarization-3.1` es un repo gated
  en HuggingFace: sin token falla con `GatedRepoError 401`. Oscar pidió explícitamente una
  solución "gratuita y sin servicios externos... modelos locales", así que una dependencia
  que exige cuenta + aceptar términos + token no cumple, aunque la inferencia en sí corra
  local después de la descarga.
- Ruta alternativa → **Silero VAD + SpeechBrain ECAPA-TDNN + clustering propio**, verificada
  con instalación y carga real en GPU, sin ningún login. No es un paquete "todo en uno" como
  pyannote: agrupar embeddings por hablante (clustering) queda como integración propia. Se
  evaluó `simple-diarizer` (PyPI) como wrapper ya armado, pero sin releases desde diciembre
  de 2022 — se prefiere integración directa sobre Silero + SpeechBrain antes que depender de
  un paquete sin mantenimiento.
- Formato del raw si se adopta → cada línea pasa de `[start - end] texto` a
  `[start - end] SPEAKER_00: texto`, etiquetas genéricas sin identidad.

Excepciones duras encontradas: ninguna. No se cambió el transcriptor elegido (Whisper local
sigue igual — la diarización es un componente nuevo y aparte, no un reemplazo), no se
promovió ninguna pieza de diseño a Canon (`Pipeline de transcripción.md` sigue en
`Borrador`), no se escribió código de implementación real del sistema (las pruebas corrieron
en un venv aparte con `uv venv`/`uv pip install`, fuera del proyecto, y ya se borró).

Segmentos:
- Diseño del sistema → actualizado: `Pipeline de transcripción.md` documenta la candidata
  descartada, la ruta viable, su costo (PyTorch como dependencia nueva) y el cambio de
  formato del raw si se adopta.
- Documentación técnica → actualizado: `Transcripción con Whisper local.md` documenta lo
  mismo desde la perspectiva de implementación, con los comandos reales usados para
  verificar cada pieza.
- Contexto para IA → actualizado: fila nueva en el Log de decisiones de
  `contexto-para-ia.md` con la decisión y su razón.

Supuestos asumidos: la profundidad de la evaluación se decidió sin frenar a esperar
credenciales — en vez de pedirle a Oscar un token de HuggingFace y un audio de prueba, se
hizo la evaluación técnica completa posible sin esos dos insumos (instalación real,
compatibilidad de dependencias, confirmación empírica del bloqueo de pyannote). La calidad
real de separación de voces con audio de mesa queda sin verificar — no había un audio con
más de un hablante disponible en el entorno de la investigación.

Fuera de alcance: integración de la diarización al pipeline real de `orchestrator/` y la
identificación de cuál hablante es el máster — candidatas a tarea `Feature` aparte con
`obsidian-task` cuando haga falta. Verificar la calidad real de VAD+embeddings+clustering
con un audio de prueba (no hace falta que sea de una sesión real) — candidata a retomarse
antes de esa tarea `Feature`, no una tarea nueva en sí misma.

Origen: implementación con Claude, 2026-08-07.
