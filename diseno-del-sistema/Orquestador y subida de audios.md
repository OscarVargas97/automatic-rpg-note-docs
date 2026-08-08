---
entrada: "Orquestador y subida de audios"
categoria: "Interfaz / Flujo de uso"
estado: "Borrador"
---

# Orquestador y subida de audios

Punto de entrada único para transcribir una sesión: sube uno o varios audios, dispara el
módulo de transcripción de `Pipeline de transcripción.md`, y deja el raw guardado en `raws/`
del vault destino. Implementado por [[Tema #4 — Orquestador y subida de audios para
transcripción]] como la app Django `orchestrator`, en la raíz del repo `automatic-rpg-note`.
Stack de interfaz resuelto en `Arquitectura del proyecto Django.md`: Django + htmx + Alpine.js
(sin componentes React — ver Log de decisiones 2026-08-07). Código en inglés, UI en español
— ver Log de decisiones y `CLAUDE.md` sección 5.

## Proyectos de campaña

Antes de subir audios, se gestionan **proyectos de campaña** (`CampaignProject`): nombre +
ruta de filesystem a un vault de Obsidian. El nombre evita a propósito la palabra "proyecto"
Django a secas — es la campaña/vault de destino, no el proyecto de código.

Se puede **crear un proyecto nuevo** o **abrir uno existente** (mismo form, con un toggle):
en modo "abrir existente" se valida que la ruta ya tenga `raws/`+`campaña/` antes de
registrarla, para no crear una estructura vacía por accidente si la base de datos se
reseteó pero el vault en disco sigue ahí.

La ruta se puede escribir a mano o elegir con un explorador de carpetas dentro de la propia
app (botón "Elegir carpeta…"). No existe forma de que un navegador entregue una ruta
absoluta real de un `<input type="file">` por seguridad — ni siquiera con la File System
Access API se obtiene un path usable por el servidor. Como servidor y navegador corren en la
misma máquina (uso local de un solo usuario), el explorador navega el filesystem del lado
del servidor en vez de abrir un diálogo nativo del sistema operativo — evita depender de que
haya un servidor gráfico disponible (relevante en WSL).

Al crear (o abrir) un proyecto, se asegura en su `vault_path` la estructura mínima del vault
de campaña
(`raws/` y `campaña/{personajes,lugares,facciones,objetos,hilos-narrativos,partidas}/`, según
`Esquema del vault de campaña.md`) usando `mkdir(parents=True, exist_ok=True)` carpeta por
carpeta: si la ruta ya tiene esa estructura, no se toca nada; si está vacía o parcial, se crea
solo lo que falta. Nunca borra ni sobreescribe.

## Subida y transcripción

Se sube uno o varios audios a un proyecto. La vista solo valida, guarda los archivos y encola
un job — no espera a que termine de transcribir (ver `## Background`).

**Varios audios por sesión**: no se concatena el audio crudo. Cada archivo se transcribe por
separado con faster-whisper (que decodifica el formato que sea) y los segmentos resultantes
se unen ajustando los tiempos por un offset acumulado, marcando de qué archivo de origen vino
cada segmento — mismo resultado observable que "concatenar y transcribir" (un raw unificado,
trazable a su origen), sin depender de que los archivos compartan formato/frecuencia ni de
una dependencia extra para unir bytes de audio.

El raw se escribe siempre en `raws/` del vault del proyecto (`<vault>/raws/<fecha>_job-
<id>.md`), texto plano con marca de tiempo por segmento — igual que especifica
`Pipeline de transcripción.md`.

## Background: Huey

Ver Log de decisiones 2026-08-07 en `meta/contexto-para-ia.md`. La transcripción corre en un
proceso aparte (`make huey`, además de `make run`) usando Huey con backend SQLite — no Celery,
porque el uso real es un solo usuario en su propia máquina, no un sistema multiusuario que
justifique un broker como Redis. Un `TranscriptionJob` tiene estado (mostrado en español)
`Pendiente → En curso → Listo` (o `Error`, con el mensaje guardado). La vista de detalle de
proyecto hace polling con htmx (`hx-trigger="every 3s"`) sobre cada trabajo hasta que deja de
estar pendiente/en curso.

## Punto de extensión: ingesta con Claude

El código deja un comentario explícito en `orchestrator/tasks.py`, justo después de escribir
el raw y marcar el trabajo listo, señalando dónde engancharía la ingesta con Claude
(`Pipeline de ingesta y enrutamiento.md`) el día que exista esa tarea. Hoy el pipeline
implementado termina en el raw — no hay clasificación automática todavía.

## Diarización de hablantes (Tema #6)

Al subir audio(s), el usuario puede indicar opcionalmente cuántos hablantes distintos hay
(campo "Número de hablantes", vacío por defecto). Si lo indica, el raw resultante marca cada
línea con una etiqueta genérica de voz (`SPEAKER_00`, `SPEAKER_01`, …) — ver el detalle del
mecanismo en `Pipeline de transcripción.md`, sección "Diarización de hablantes". Si lo deja
vacío, la subida y transcripción funcionan exactamente igual que antes de esta pieza.

## Fuera de alcance de esta pieza

- Conversión de formato de audio antes de transcribir: faster-whisper decodifica varios
  formatos de fábrica (vía su decodificador interno), así que no se agregó una dependencia de
  conversión aparte. No verificado contra todos los formatos posibles de grabación.
- Subida robusta (reintentos, subida por partes) ante un corte de red a mitad de una subida de
  300+ MB: se acepta el riesgo para el prototipo, como la propia tarea dejó abierto.
- Identificar cuál hablante es el máster — la diarización solo distingue voces, no sabe cuál
  es cuál (ver `Pipeline de transcripción.md`).
