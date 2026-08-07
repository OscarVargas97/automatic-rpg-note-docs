---
id: "TSK-2"
titulo: "Tema #2 — Script de setup de transcripción"
estado: "Sin empezar"
tipo: "Feature"
disciplina: "Transcripción y audio"
prioridad: "P1 - Alta"
hito: "Prototipo"
segmentos_a_actualizar: ["Documentación técnica"]
segmentos_actualizados: false
definicion_de_hecho: false
documentacion_a_actualizar: ["[[Transcripción con Whisper local]]"]
diseno_de_referencia: ["[[Pipeline de transcripción]]", "[[Esquema del vault de campaña]]"]
costo_asociado: []
rama: ""
responsable: ""
estimacion_dias: ""
fechas: ""
bloqueada_por: ""
---

## Problema
Levantar todo lo necesario para transcribir una sesión (instalar Whisper local, bajar el
modelo, dejarlo listo para correr) es hoy un proceso manual sin documentar ni automatizar —
cada vez que alguien lo necesita, lo reconstruye desde cero.

## Resultado esperado
Existe un script en `src/` que, al correrlo, deja instalado y listo para usar todo lo
necesario para transcribir audio con Whisper local, según lo que haya definido [[Tema #1 —
Investigar implementación de transcripción con Whisper local]] — sin pasos manuales
adicionales más allá de ejecutarlo. El resultado es un módulo, no un script de un solo uso:
recibe la ruta de un vault de Obsidian como parámetro (no asume que el destino es
`ObsidianRPG_Obsidian/`), acepta un audio o varios (unificándolos en una sola transcripción
cuando son varios), y guarda siempre el raw en `raws/` del vault destino. Usa GPU (la
máquina de referencia tiene una RTX 3070 de 8GB, con CUDA) cuando está disponible, y cae a
CPU si no la hay. `Transcripción con Whisper local` refleja el resultado real: qué instala
el script, cómo se invoca, y su ruta en el repo.

## Referencias
- ObsidianRPG_Obsidian/diseno-del-sistema/Pipeline de transcripción.md
- src/ (carpeta nueva, todavía no existe)

Origen: conversación con Oscar, 2026-08-07.
