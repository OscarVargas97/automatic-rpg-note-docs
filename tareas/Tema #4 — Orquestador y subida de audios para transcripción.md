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
nada coordina "subir → transcribir → raw listo" como un flujo único. Tampoco hay forma de
decirle al sistema a qué vault de Obsidian pertenece cada audio: el módulo de transcripción
ya recibe la ruta del vault destino como parámetro (`Pipeline de transcripción.md`, sección
`Diseño como módulo reusable`), pero nada en la interfaz captura ni persiste esa ruta todavía.

## Resultado esperado
Dentro del proyecto Django de [[Tema #2 — Base del proyecto Django]], existen las vistas del
orquestador: un punto de entrada donde se sube uno o varios audios de una sesión, se dispara
la transcripción usando lo instalado por [[Tema #3 — Instalación de dependencias de
transcripción en Django]], y queda el raw guardado en `raws/` del vault destino — sin pasos
manuales entre subir el audio y tener el raw listo. El diseño deja explícito el punto de
extensión para encadenar la ingesta con Claude el día que exista esa tarea, sin implementarla
ahora. `Orquestador y subida de audios` y `Orquestador de transcripción` reflejan el diseño y
la implementación real de estas vistas.

Antes de subir audios, el software permite gestionar **proyectos**: cada proyecto tiene un
nombre y una ruta de filesystem a un vault de Obsidian asociado (el parámetro que
`Pipeline de transcripción.md` ya espera). Al subir uno o varios audios, quien sube elige a
qué proyecto pertenecen, y el raw resultante se guarda en `raws/` del vault de ese proyecto —
nunca en una ruta hardcodeada. El diseño real (¿modelo Django con CRUD propio? ¿algo más
simple para el prototipo?) queda a resolver dentro de esta tarea, no está decidido de
antemano. Ojo con el nombre: "proyecto" aquí es la campaña/vault de destino, no el proyecto
Django (que además vive en su propio repo, `automatic-rpg-note-src`) — el diseño debe usar un
término que no choque con esa otra acepción (p. ej. "Proyecto de campaña" o similar).

Crear un proyecto también permite **agregarlo eligiendo un path**: al seleccionar la ruta
destino, el software genera ahí la estructura de carpetas del vault de campaña —
`raws/` y `campaña/{personajes,lugares,facciones,objetos,hilos-narrativos,partidas}/` — según
`Esquema del vault de campaña.md`. Si el path elegido ya tiene esa estructura (vault
existente), no la pisa; si está vacío o no existe, la crea. El diseño debe decidir qué pasa
si el path ya tiene una estructura parcial o distinta — no asumir que siempre parte de cero.

Origen del requisito de proyectos: conversación con Oscar, 2026-08-07.

Una sesión de mesa real puede durar varias horas — `Pipeline de transcripción.md` ya elige el
modelo `small` pensando en eso, pero el orquestador es donde eso se vuelve un problema
concreto de subida y de tiempo de proceso, no solo de calidad del modelo. El diseño de esta
tarea tiene que responder, para un audio del orden de 2-3 horas (WAV 16kHz mono, ~300-350
MB):

- **Tamaño de subida**: los límites por defecto de Django (`DATA_UPLOAD_MAX_MEMORY_SIZE` y
  afines) no alcanzan para un archivo de ese tamaño — hay que subirlos explícitamente y
  decidir hasta qué tamaño se acepta.
- **Síncrono vs. background**: si "subir → transcribir → raw listo" corre dentro de una sola
  request HTTP, una transcripción de varias horas puede exceder el timeout del servidor. Si
  no hay una razón fuerte para mantenerlo síncrono, el diseño debería contemplar un
  procesamiento en background (Celery, RQ, o algo más simple si el prototipo no lo justifica
  todavía) — decidir esto es parte de esta tarea, no un supuesto a asumir en silencio.
- **Robustez de la subida**: una subida de 300+ MB por HTTP, en la red de una mesa de rol, es
  candidata realista a cortarse a mitad de camino. El diseño debe decidir si hace falta algo
  más que una subida simple (reintentos, subida por partes) o si se acepta el riesgo para el
  prototipo y se revisa después.

Uso real: un solo usuario, corriendo las solicitudes HTTP en su propia máquina — no hay
requisito de multiusuario ni de despliegue remoto (`Arquitectura del proyecto Django.md`,
sección `## Estructura`). Eso resuelve el punto de background: **Huey con backend SQLite**
(no Celery — evita depender de Redis/RabbitMQ como broker aparte; alcanza con un segundo
proceso, `huey_consumer`, corriendo junto a `runserver`). Queda como supuesto asumido, no
como decisión cerrada — revisar si al ejecutar la tarea aparece una razón concreta para otra
cosa.

**Stack de frontend**: Django + [htmx](https://htmx.org/) + [Alpine.js](https://alpinejs.dev/),
con Tailwind CSS y una paleta/espaciado inspirados en shadcn/ui — sin los componentes React
de shadcn en sí (shadcn es React + Radix, no aplica dentro de templates Django). Elegido
sobre una alternativa con React + shadcn real porque esta última exige convertir Django en
una API pura (Django Ninja/DRF) más un frontend Vite/React aparte — dos procesos de dev, un
build de Node, una capa de API — infraestructura que no se justifica para un prototipo de un
solo usuario en su propia máquina. htmx permite marcar la barra de progreso del job de
transcripción (poll o SSE) sin salir del modelo de templates server-rendered de Django.

## Referencias
- `docs/diseno-del-sistema/Pipeline de transcripción.md`
- `docs/diseno-del-sistema/Arquitectura del proyecto Django.md`
- El proyecto Django de Tema #2 vive en la raíz de este mismo repo superior
  (`automatic-rpg-note`), esta tarea agrega las vistas encima — ver `CLAUDE.md` sección 2

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
