---
id: "TSK-1"
titulo: "Tema #1 — Investigar implementación de transcripción con Whisper local"
estado: "Listo"
tipo: "Investigación"
disciplina: "Transcripción y audio"
prioridad: "P1 - Alta"
hito: "Prototipo"
segmentos_a_actualizar: ["Diseño del sistema", "Documentación técnica"]
segmentos_actualizados: true
definicion_de_hecho: true
documentacion_a_actualizar: ["[[Transcripción con Whisper local]]"]
diseno_de_referencia: ["[[Pipeline de transcripción]]", "[[Pipeline de ingesta y enrutamiento]]"]
costo_asociado: []
rama: "feature/TSK-1-investigar-implementacion-de-transcripci"
responsable: ""
estimacion_dias: ""
fechas: ""
bloqueada_por: ""
---

## Problema
`Pipeline de ingesta y enrutamiento.md` da por hecho que Whisper local convierte el audio en
texto, pero no dice qué variante (whisper.cpp vs. faster-whisper), qué modelo, ni qué se
necesita instalado para que eso funcione en la máquina de quien corre la sesión.

## Resultado esperado
`Pipeline de transcripción` y `Transcripción con Whisper local` describen: variante de
Whisper elegida, modelo, requisitos de hardware/software, formato de audio de entrada y de
texto de salida, y qué debe existir en la máquina para correr una transcripción —
suficiente para que [[Tema #3 — Instalación de dependencias de transcripción en Django]] sepa
exactamente qué automatizar.

## Decisión pendiente
Diarización (si el modelo separa hablantes) queda fuera de alcance — ya está señalada como
abierta en `Pipeline de ingesta y enrutamiento.md`; se resuelve en una tarea aparte si hace
falta.

## Referencias
- ObsidianRPG_Obsidian/diseno-del-sistema/Pipeline de ingesta y enrutamiento.md
- ObsidianRPG_Obsidian/meta/contexto-para-ia.md

Origen: conversación con Oscar, 2026-08-07.

## Registro de cierre

Ejecutado en la rama `feature/TSK-1-investigar-implementacion-de-transcripci`, sin commit
todavía.

Decisiones resueltas:
- Variante de Whisper (whisper.cpp vs. faster-whisper) → **faster-whisper**, porque corre en
  CPU sin compilar nada (`pip install`), y la ventaja de whisper.cpp (Apple Neural
  Engine/Metal) solo aplica en Mac M-series, sin confirmación de que sea el hardware real de
  mesa.
- Modelo → `small` por defecto, `base` como repliegue en hardware limitado.
- Formato de entrada/salida → WAV 16kHz mono de entrada; segmentos con timestamp + texto de
  salida.

Excepciones duras encontradas: ninguna. No se cambió el transcriptor elegido (sigue siendo
Whisper local), no se promovió nada a Canon, no se escribió código de implementación (eso es
[[Tema #3 — Instalación de dependencias de transcripción en Django]], antes numerada Tema
#2 — ver el reordenamiento del 2026-08-07 en [[Tema #2 — Base del proyecto Django]]).

Segmentos:
- Diseño del sistema → actualizado: `Pipeline de transcripción.md` pasó de `Idea` a
  `Borrador` con la variante elegida, modelo, formatos y requisitos.
- Documentación técnica → actualizado: `Transcripción con Whisper local.md` pasó de
  `Borrador` a `Vigente` con instalación, modelo, formatos y lo que queda fuera de alcance
  (diarización). `ruta_en_el_repo` sigue vacía a propósito — el script todavía no existe.

Supuestos asumidos: no se confirmó si la máquina real de mesa es Mac con chip M-series: si lo
fuera, whisper.cpp vuelve a ser candidato (queda anotado como alternativa en la pieza de
diseño, no descartado).

Fuera de alcance: diarización (separar quién habló) — candidata a tarea aparte con
`obsidian-task` si el piloto en mesa la necesita.

Origen: implementación con Claude, 2026-08-07.

## Corrección posterior

Oscar confirmó el hardware real: NVIDIA RTX 3070 de 8GB, no "sin confirmar" como asumió el
cierre original. `Pipeline de transcripción.md` y `Transcripción con Whisper local.md` se
actualizaron para reflejarlo — CUDA por defecto en esa máquina, CPU como fallback (el módulo
tiene que seguir funcionando en otra máquina/vault). De la misma corrección salieron tres
reglas de diseño nuevas que no estaban en el cierre original: el pipeline se diseña como
módulo reusable (ruta de vault como parámetro), soporta uno o varios audios por
transcripción, y el raw se guarda siempre en `raws/` del vault destino
(`Esquema del vault de campaña.md` también se actualizó con esa carpeta). No cambia el
resultado ya cerrado de esta tarea (variante, modelo, formatos); lo amplía.

Origen: conversación con Oscar, 2026-08-07.
