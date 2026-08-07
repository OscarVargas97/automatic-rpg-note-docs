---
id: "TSK-4"
titulo: "Tema #4 — Orquestador y subida de audios para transcripción"
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
bloqueada_por: "Tema #3 — Instalación de dependencias de transcripción en Django (necesita las dependencias de transcripción instaladas para poder disparar la transcripción real desde una vista)"
---

## Problema
Hoy no hay forma de subir audios ni de disparar la transcripción sin correr comandos a mano;
nada coordina "subir → transcribir → raw listo" como un flujo único.

## Resultado esperado
Dentro del proyecto Django de [[Tema #2 — Base del proyecto Django]], existen las vistas del
orquestador: un punto de entrada donde se sube uno o varios audios de una sesión, se dispara
la transcripción usando lo instalado por [[Tema #3 — Instalación de dependencias de
transcripción en Django]], y queda el raw guardado en `raws/` del vault destino — sin pasos
manuales entre subir el audio y tener el raw listo. El diseño deja explícito el punto de
extensión para encadenar la ingesta con Claude el día que exista esa tarea, sin implementarla
ahora. `Orquestador y subida de audios` y `Orquestador de transcripción` reflejan el diseño y
la implementación real de estas vistas.

## Referencias
- ObsidianRPG_Obsidian/diseno-del-sistema/Pipeline de transcripción.md
- src/ (el proyecto Django de Tema #2 vive ahí; esta tarea agrega las vistas encima)

Origen: conversación con Oscar, 2026-08-07.

## Registro de reestructuración

Esta tarea era originalmente "Tema #3 — Orquestador y subida de audios para transcripción".
Al replantear el orden de trabajo, Oscar decidió: primero se levanta la base de Django
([[Tema #2 — Base del proyecto Django]]), después se integra en Django la instalación de las
dependencias externas de transcripción ([[Tema #3 — Instalación de dependencias de
transcripción en Django]]), y esta tarea pasa a ser la #4 — empezar a trabajar en las vistas
del orquestador sobre esa base ya montada. La decisión de stack que esta tarea tenía pendiente
(HTML simple vs. Django) queda resuelta por Tema #2: Django.

Origen: conversación con Oscar, 2026-08-07.
