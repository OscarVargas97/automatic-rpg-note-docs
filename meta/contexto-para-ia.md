---
titulo: "Contexto para IA"
---

# Contexto para IA

Página canónica para el contrato de agentes. Si esta nota y `CLAUDE.md` se contradicen, gana
`CLAUDE.md` — es la que se lee primero en toda sesión, y esta nota no la duplica.

## Log de decisiones

Fila nueva cada vez que se cambia el transcriptor elegido, se promueve una pieza de
`diseno-del-sistema/` a `Canon`, o se ajusta una convención de proceso. Obligatorio antes de
tocar cualquiera de esas cosas, no después.

| Fecha | Decisión | Por qué | Quién |
|---|---|---|---|
| 2026-08-07 | Transcriptor elegido: Whisper local (whisper.cpp / faster-whisper), no una API en la nube. | Sin costo por uso, sin depender de conexión durante una sesión de mesa. Revisable si la calidad no alcanza en el piloto. | Oscar |
| 2026-08-07 | Taxonomía inicial del vault de campaña: Personajes, Lugares, Facciones, Objetos, Hilos narrativos, más Partidas para el resumen de sesión. | Cubre el worldbuilding clásico de una mesa de rol sin sobre-especificar de entrada; "NPCs con relación a jugadores" y "Reglas de mesa" quedaron fuera por ahora — ver Muro de Ideas. | Oscar |
| 2026-08-07 | Se adopta el patrón de gestión de Jueguito (`CLAUDE.md` + `VAULT_MAP.md` + skills `obsidian-task`/`obsidian-task-solve`) para este proyecto, sin el contrato vault↔GitHub. | El contrato vault↔GitHub de Jueguito depende de un repo de implementación que aquí todavía no existe — se añadirá cuando arranque la implementación real. | Oscar |
| 2026-08-07 | El código de implementación real del sistema vive en `src/`, dentro de este mismo repo — no en un repo aparte. | Arranca más simple: un solo repositorio que gestionar mientras el sistema es chico. Revisable si crece lo suficiente para justificar el split. | Oscar |
| 2026-08-07 | Se revierte la fila anterior: el código pasa a `automatic-rpg-note-src`, un repositorio aparte con su propio historial (extraído de `src/` con `git subtree split` para no perder los commits ya hechos). Este repo (`automatic-rpg-note`) se queda solo con el vault, `CLAUDE.md`, `VAULT_MAP.md` y los skills. Localmente ambos viven como carpetas hermanas bajo un directorio contenedor (`automatic-rpg-note/`) que no es en sí un repositorio git. | Con un solo repo, cambiar de rama o rebasear el código (Tema #3) arrastraba también las notas del vault que Obsidian tenía abiertas — el vault no debería depender de en qué rama de código se está parado. | Oscar |
| 2026-08-07 | Se revierte otra vez la fila anterior: `automatic-rpg-note-src` pasa a ser la raíz del proyecto superior `automatic-rpg-note` (mismo historial, mismo remoto, ahora en la carpeta contenedora) — código, `Makefile`, `.gitignore`, `.claude/`, `CLAUDE.md` y `VAULT_MAP.md` en su raíz. El vault (este repo) se renombra a `docs/`, sigue siendo un repositorio git aparte con su propio remoto, pero ahora vive **dentro** de esa raíz y queda explícitamente en el `.gitignore` del proyecto superior. | Ni "un solo repo" ni "submodule" resolvían el problema real: cambiar de rama de código no debía tocar el vault. Un submodule seguiría siendo un puntero versionado que puede divergir entre ramas; una carpeta simplemente gitignored no puede — git del proyecto superior nunca la toca, sin importar en qué rama se esté. | Oscar |
| 2026-08-07 | Stack de frontend para el orquestador (Tema #4): Django + htmx + Alpine.js, Tailwind con estética inspirada en shadcn/ui — no shadcn/ui real (es React + Radix, incompatible con templates Django sin convertir Django en una API pura). Procesamiento en background asumido: Huey con backend SQLite, no Celery. | Uso real es un solo usuario corriendo todo en su propia máquina (`Arquitectura del proyecto Django.md`), no un sistema multiusuario — no se justifica la infraestructura de una API separada + build de Node + broker de mensajería. Revisable si el uso real deja de ser local/personal. | Oscar |
| 2026-08-07 | Convención de idioma del código: todo el código (apps, modelos, campos, vistas, urls, templates, identificadores, comentarios, docstrings — Python y HTML/JS) en inglés; el texto que el usuario lee corriendo la app (labels, botones, mensajes de error, help de comandos) en español. Aplica al repo raíz `automatic-rpg-note`, no a `docs/`. La app `orquestador` (Tema #4) se renombra a `orchestrator` con todos sus identificadores; ver detalle en `Orquestador de transcripción.md`. | Decisión explícita de Oscar tras ver el primer código de Tema #4 en español — consistencia con la convención habitual de nombrar código en inglés, independiente de que el proyecto y su documentación sean en español. | Oscar |
