---
id: "TSK-3"
titulo: "Tema #3 — Instalación de dependencias de transcripción en Django"
estado: "Sin empezar"
tipo: "Feature"
disciplina: "Transcripción y audio"
prioridad: "P1 - Alta"
hito: "Prototipo"
segmentos_a_actualizar: ["Documentación técnica"]
segmentos_actualizados: false
definicion_de_hecho: false
documentacion_a_actualizar: ["[[Transcripción con Whisper local]]"]
diseno_de_referencia: ["[[Pipeline de transcripción]]"]
costo_asociado: []
rama: ""
responsable: ""
estimacion_dias: ""
fechas: ""
bloqueada_por: "Tema #2 — Base del proyecto Django (necesita el proyecto Django levantado para integrar la instalación dentro de él)"
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
