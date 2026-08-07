---
id: "TSK-3"
titulo: "Tema #3 — Orquestador y subida de audios para transcripción"
estado: "Sin empezar"
tipo: "Feature"
disciplina: "Arquitectura del sistema"
prioridad: "P2 - Media"
hito: "Prototipo"
segmentos_a_actualizar: ["Diseño del sistema", "Documentación técnica"]
segmentos_actualizados: false
definicion_de_hecho: false
documentacion_a_actualizar: ["[[Orquestador de transcripción]]"]
diseno_de_referencia: ["[[Orquestador y subida de audios]]", "[[Pipeline de transcripción]]"]
costo_asociado: []
rama: ""
responsable: ""
estimacion_dias: ""
fechas: ""
bloqueada_por: "Tema #2 — Script de setup de transcripción (necesita el módulo de transcripción para poder orquestar sobre él)"
---

## Problema
Hoy no hay forma de subir audios ni de disparar la transcripción sin correr comandos a mano;
nada coordina "subir → transcribir → raw listo" como un flujo único.

## Resultado esperado
Existe un punto de entrada (interfaz/orquestador) donde se sube uno o varios audios de una
sesión, se dispara el módulo de transcripción de [[Tema #2 — Script de setup de
transcripción]], y queda el raw guardado en `raws/` del vault destino — sin pasos manuales
entre subir el audio y tener el raw listo. El diseño deja explícito el punto de extensión
para encadenar la ingesta con Claude el día que exista esa tarea, sin implementarla ahora.

## Decisión pendiente
Stack de la interfaz: HTML simple sin framework vs. Django. Se planteó la idea de un
orquestador que unifique subida, disparo de la transcripción y (a futuro) el resto del
pipeline, pero no se cerró el stack — se investiga y resuelve al ejecutar esta tarea.

## Referencias
- ObsidianRPG_Obsidian/diseno-del-sistema/Pipeline de transcripción.md
- src/ (el módulo de Tema #2 vive ahí; esta tarea agrega la orquestación/interfaz encima)

Origen: conversación con Oscar, 2026-08-07.
