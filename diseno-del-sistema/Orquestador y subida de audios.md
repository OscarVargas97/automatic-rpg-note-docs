---
entrada: "Orquestador y subida de audios"
categoria: "Interfaz / Flujo de uso"
estado: "Borrador"
---

# Orquestador y subida de audios

Punto de entrada único para transcribir una sesión: sube uno o varios audios, dispara el
módulo de transcripción de `Pipeline de transcripción.md`, y deja el raw guardado en `raws/`
del vault destino. Implementado por [[Tema #4 — Orquestador y subida de audios para
transcripción]] como la app Django `orquestador`, en la raíz del repo `automatic-rpg-note`.
Stack de interfaz resuelto en `Arquitectura del proyecto Django.md`: Django + htmx + Alpine.js
(sin componentes React — ver Log de decisiones 2026-08-07).

## Proyectos de campaña

Antes de subir audios, se gestionan **proyectos de campaña** (`ProyectoCampana`): nombre +
ruta de filesystem a un vault de Obsidian. El nombre evita a propósito la palabra "proyecto"
Django a secas — es la campaña/vault de destino, no el proyecto de código.

Al crear un proyecto, se genera en su `ruta_vault` la estructura mínima del vault de campaña
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

El raw se escribe siempre en `raws/` del vault del proyecto (`<vault>/raws/<fecha>_trabajo-
<id>.md`), texto plano con marca de tiempo por segmento — igual que especifica
`Pipeline de transcripción.md`.

## Background: Huey

Ver Log de decisiones 2026-08-07 en `meta/contexto-para-ia.md`. La transcripción corre en un
proceso aparte (`make huey`, además de `make run`) usando Huey con backend SQLite — no Celery,
porque el uso real es un solo usuario en su propia máquina, no un sistema multiusuario que
justifique un broker como Redis. Un `TrabajoTranscripcion` tiene estado
`pendiente → en_curso → listo` (o `error`, con el mensaje guardado). La vista de detalle de
proyecto hace polling con htmx (`hx-trigger="every 3s"`) sobre cada trabajo hasta que deja de
estar `pendiente`/`en_curso`.

## Punto de extensión: ingesta con Claude

El código deja un comentario explícito en `orquestador/tasks.py`, justo después de escribir
el raw y marcar el trabajo `listo`, señalando dónde engancharía la ingesta con Claude
(`Pipeline de ingesta y enrutamiento.md`) el día que exista esa tarea. Hoy el pipeline
implementado termina en el raw — no hay clasificación automática todavía.

## Fuera de alcance de esta pieza

- Conversión de formato de audio antes de transcribir: faster-whisper decodifica varios
  formatos de fábrica (vía su decodificador interno), así que no se agregó una dependencia de
  conversión aparte. No verificado contra todos los formatos posibles de grabación.
- Subida robusta (reintentos, subida por partes) ante un corte de red a mitad de una subida de
  300+ MB: se acepta el riesgo para el prototipo, como la propia tarea dejó abierto.
- Diarización — sigue fuera de alcance, igual que en `Pipeline de transcripción.md`.
