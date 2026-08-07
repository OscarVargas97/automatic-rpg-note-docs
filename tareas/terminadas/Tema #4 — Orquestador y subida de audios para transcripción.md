---
id: "TSK-4"
titulo: "Tema #4 — Orquestador y subida de audios para transcripción"
estado: "Listo"
tipo: "Feature"
disciplina: "Arquitectura del sistema"
prioridad: "P2 - Media"
hito: "Prototipo"
segmentos_a_actualizar: ["Diseño del sistema", "Documentación técnica"]
segmentos_actualizados: true
definicion_de_hecho: true
documentacion_a_actualizar: ["[[Orquestador de transcripción]]"]
diseno_de_referencia: ["[[Orquestador y subida de audios]]", "[[Pipeline de transcripción]]"]
costo_asociado: []
rama: "feature/TSK-4-tema-4-orquestador-y-subida-de-audios-pa"
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

## Registro de cierre

Ejecutado en la rama feature/TSK-4-tema-4-orquestador-y-subida-de-audios-pa, sin commit
todavía.

Decisiones resueltas:
- Varios audios por sesión → se transcribe cada uno por separado y se unen los segmentos con
  offset de tiempo, en vez de concatenar audio crudo, porque evita depender de que los
  archivos compartan formato/frecuencia y no agrega una dependencia de audio nueva
  (confirmado con el usuario antes de ejecutar).
- Background → Huey con backend SQLite, no Celery, porque el uso real es un solo usuario en
  su propia máquina (confirmado en conversación previa, registrado en el Log de decisiones).
- Frontend → Django + htmx + Alpine.js, Tailwind vía CDN con estética shadcn, sin build de
  Node (mismo origen).

Excepciones duras encontradas: ninguna — no se tocó el transcriptor elegido, no hay pieza
Canon involucrada (`Orquestador y subida de audios.md` y `Pipeline de transcripción.md`
siguen `Idea`/`Borrador`), y el código de implementación ya tenía resuelto dónde vive
(`CLAUDE.md` sección 2) y esta tarea lo declaraba explícitamente.

Segmentos:
- Diseño del sistema → actualizado: `Orquestador y subida de audios.md` pasa de stub vacío
  (`Idea`) a diseño real (`Borrador`) reflejando la implementación.
- Documentación técnica → actualizado: `Orquestador de transcripción.md` pasa de stub vacío
  a `Vigente`, con la implementación real documentada (modelos, vistas, task de Huey, y una
  corrección real encontrada en el módulo de Tema #3, ver abajo).

Supuestos asumidos:
- El endpoint de subida no ofrece elegir proyecto por dropdown — quien sube ya está en la
  página del proyecto (navegación implícita).
- Subida simple, sin reintentos ni subida por partes, para un audio de 300+ MB — el riesgo
  se acepta para el prototipo, como la propia tarea dejó abierto.
- No se agregó conversión de formato de audio: faster-whisper decodifica internamente varios
  formatos; no se probó contra todos los posibles.
- No se implementó el punto de extensión de ingesta con Claude — solo se dejó comentado en
  `orquestador/tasks.py`, como pedía el `## Resultado esperado`.

Hallazgo fuera del alcance original, corregido igual por ser un bug real encontrado al
reutilizar el código de Tema #3: `core/transcripcion.py` (antes lógica inline en
`check_transcription.py`) construía `WhisperModel(device="cuda")` sin detectar que la
inferencia real podía fallar por librerías de cómputo ausentes (`libcublas.so.12` en esta
máquina) — la construcción del objeto no falla, solo el primer uso real. Se corrigió
forzando una transcripción mínima (1s de silencio en memoria) dentro del mismo `try` que
decide GPU vs. CPU. Sin este fix, `check_transcription` podía reportar "GPU" y la primera
transcripción real del orquestador habría fallado en producción. Verificado con el flujo
completo end-to-end (ver `Orquestador de transcripción.md`, sección `## Verificado`).

Fuera de alcance: probar con un audio real de 2-3 horas y con la subida HTTP de un archivo
de cientos de MB — candidato a verificar en cuanto haya un audio de sesión real (mismo
patrón que el cabo suelto de CPU fallback que cerró Tema #3).

Origen: implementación con Claude, 2026-08-07.

## Adenda — 2026-08-07

Oscar señaló, ya revisando el resultado antes de commitear: el form de "Crear proyecto"
único no dejaba claro cómo volver a cargar un proyecto cuyo vault ya existe en disco (caso
real: la base de datos se resetea o se pierde, pero la carpeta del vault con sus datos
sigue ahí). La lógica de fondo (`asegurar_estructura_vault`, idempotente) ya soportaba ese
caso sin romper nada, pero no había ninguna señal en la UI de que fuera seguro reusarla así.

Se agregó un toggle "Crear proyecto nuevo" / "Abrir proyecto existente" (mismo endpoint,
campo `modo`): en modo `existente`, valida que la ruta ya tenga `raws/` + `campaña/` antes
de registrarla (si no, error explícito en vez de crear una estructura vacía por accidente).
Se agregó también rechazo de `ruta_vault` duplicada en ambos modos, caso que no estaba
cubierto antes. Verificado con los 4 casos (existente sin estructura → error; nuevo → crea;
duplicado → rechaza; existente con estructura real → abre). `Orquestador de
transcripción.md` actualizado con la tabla de vistas y la verificación.

Origen: implementación con Claude, 2026-08-07.

## Adenda 2 — 2026-08-07

Oscar pidió que todo el código del proyecto esté en inglés (identificadores, comentarios,
docstrings, nombres de app/modelos/vistas/urls/templates), dejando en español solo el texto
que el usuario lee corriendo la app. Se renombró la app `orquestador` → `orchestrator` y
todos sus identificadores (`ProyectoCampana`→`CampaignProject`,
`TrabajoTranscripcion`→`TranscriptionJob`, `AudioSubido`→`UploadedAudio`, campos, vistas,
urls, templates, `core/transcripcion.py`→`core/transcription.py`), regenerando la migración
inicial (sin datos reales que migrar todavía). El texto visible en la UI (labels, botones,
mensajes de error, help de `check_transcription`) se dejó igual, en español. Se agregó la
convención como regla en `CLAUDE.md` sección 5 para que no se pierda hacia adelante.

Repetida la batería completa de pruebas end-to-end (crear/abrir proyecto, subir y
transcribir, duplicados, validación de ruta existente) contra el código renombrado — mismos
resultados que antes del rename.

Origen: implementación con Claude, 2026-08-07.
