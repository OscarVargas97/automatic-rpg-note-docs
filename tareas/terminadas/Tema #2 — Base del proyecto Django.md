---
id: "TSK-2"
titulo: "Tema #2 — Base del proyecto Django"
estado: "Listo"
tipo: "Feature"
disciplina: "Arquitectura del sistema"
prioridad: "P1 - Alta"
hito: "Prototipo"
segmentos_a_actualizar: ["Diseño del sistema", "Documentación técnica"]
segmentos_actualizados: true
definicion_de_hecho: true
documentacion_a_actualizar: ["[[Proyecto Django]]"]
diseno_de_referencia: ["[[Arquitectura del proyecto Django]]"]
costo_asociado: []
rama: "feature/TSK-2-tema-2-base-del-proyecto-django"
responsable: ""
estimacion_dias: ""
fechas: ""
bloqueada_por: ""
---

## Problema
Hoy no hay ninguna infraestructura de aplicación levantada: la instalación de dependencias
de transcripción y el futuro orquestador se pensaban como scripts sueltos en `src/`, sin un
framework que los una bajo una sola app que corra localmente.

## Resultado esperado
Existe un proyecto Django base en `src/`, con una app inicial ya creada dentro (nombre a
definir al ejecutar la tarea), que corre localmente con el servidor de desarrollo sin
errores — sin lógica de negocio todavía: ni instalación de dependencias de transcripción
([[Tema #3 — Instalación de dependencias de transcripción en Django]]), ni orquestación
([[Tema #4 — Orquestador y subida de audios para transcripción]]), ambas construyen encima
de esta base. Las dependencias del proyecto se gestionan con `uv` en vez de `pip`/`venv`
sueltos. En la raíz del repo existe un `Makefile` con los targets que resumen los comandos
que normalmente hacen falta para instalar, migrar y correr el proyecto (`make install`,
`make run`, etc.), en vez de tener que escribir el comando completo de `uv` cada vez.
`Proyecto Django` documenta cómo levantarlo y correrlo, con y sin `make`. `Arquitectura del
proyecto Django` deja registrada la elección de Django como stack y de `uv` como gestor de
dependencias — resuelve la decisión que tenía pendiente [[Tema #4 — Orquestador y subida de
audios para transcripción]] entre HTML simple sin framework y Django.

## Referencias
- src/ (proyecto Django)
- Makefile (raíz del repo)

Origen: conversación con Oscar, 2026-08-07.

## Registro de cierre

Ejecutado en la rama `feature/TSK-2-tema-2-base-del-proyecto-django`, sin commit todavía.

Decisiones resueltas:
- Nombre de la app inicial → `core`, porque no se apuesta un nombre de dominio (transcripción
  u orquestador) a una app que todavía no tiene lógica de negocio propia.
- Nombre del paquete de settings → `config`, convención habitual en proyectos Django que
  van a alojar varias apps, para no acoplar el paquete de configuración a un dominio.
- Stack de la interfaz (pendiente desde la vieja Tema #3) → **Django**, resuelto por esta
  tarea, porque unifica bajo un proyecto la instalación de dependencias y las vistas del
  orquestador sin coordinar piezas sueltas.

Excepciones duras encontradas: ninguna. No se cambió el transcriptor elegido, no se promovió
nada a Canon, no se inventó lore. Se escribió código de implementación real
(`src/manage.py`, `src/config/`, `src/core/`) — la tarea lo declara explícitamente y la
decisión de dónde vive el código ya estaba resuelta en `CLAUDE.md` sección 2 (`src/`).

Segmentos:
- Diseño del sistema → actualizado: `Arquitectura del proyecto Django.md` pasó de `Idea` a
  `Borrador`, con la estructura (`config/`, `core/`) y la resolución del stack.
- Documentación técnica → actualizado: `Proyecto Django.md` pasó de `Borrador` a `Vigente`,
  con `ruta_en_el_repo: "src/"`, estructura, apps e instalación.

Supuestos asumidos:
- Versión de Django fijada como rango (`>=5.0,<6.0`) en `requirements.txt`, sin fijar una
  versión exacta — se resuelve sola al instalar.
- El proyecto sigue exactamente la estructura estándar de `django-admin startproject` +
  `startapp`, escrita a mano.

**No verificado — bloqueante real, no un supuesto**: este sandbox no tiene `pip` ni `venv`
utilizables (falta `python3-pip` / `python3.14-venv` a nivel de sistema, y no hay `sudo`
disponible), así que no se pudo instalar Django ni confirmar que `python manage.py runserver`
arranca sin errores — el `## Resultado esperado` de la tarea lo exige explícitamente ("corre
localmente... sin errores"), por eso `definicion_de_hecho` queda en `false` y el `estado` en
`En curso`, no `Listo`, aunque los segmentos ya están al día. Falta que alguien con
`pip`/`venv` disponibles corra: `pip install -r src/requirements.txt`, `python
src/manage.py migrate`, `python src/manage.py runserver`, y confirme que no hay errores.

Fuera de alcance: cualquier app de negocio (instalación de dependencias, orquestador) — eso
es [[Tema #3 — Instalación de dependencias de transcripción en Django]] y [[Tema #4 —
Orquestador y subida de audios para transcripción]].

Origen: implementación con Claude, 2026-08-07.

## Ampliación de alcance y verificación

Oscar amplió el `## Resultado esperado` de esta tarea: el proyecto debía gestionar
dependencias con `uv` en vez de `pip`/`venv` sueltos, y la raíz del repo debía tener un
`Makefile` con atajos para los comandos de uso habitual.

Trabajo hecho:
- `src/requirements.txt` reemplazado por `src/pyproject.toml` (dependencias) + `src/uv.lock`
  (versionado, para instalaciones reproducibles).
- `Makefile` en la raíz con los targets `install`, `migrate`, `makemigrations`, `run`,
  `shell`, todos delegando en `uv run --directory src ...` (no `--project`: no cambia el
  directorio de trabajo del subproceso, y `manage.py` es una ruta relativa a `src/`).

Esto resolvió el bloqueante real que había dejado abierta esta tarea: `uv` se instala sin
`sudo` (script oficial, a `~/.local/bin`), así que en este mismo sandbox se pudo verificar de
punta a punta lo que antes no se pudo con `pip`/`venv`:
- `make install` → `uv sync --directory src` instaló Django 5.2.17 en `src/.venv`.
- `make migrate` → aplicó las migraciones iniciales sin errores.
- `make run` → el servidor de desarrollo respondió `200` en `http://127.0.0.1:8000/`.

`make` en sí no se pudo instalar de forma persistente en este sandbox (requiere `apt install
make`, y no hay `sudo`) — se extrajo su paquete `.deb` a una ruta temporal fuera del repo
solo para correr esta verificación puntual; en una máquina real se instala con el gestor de
paquetes del sistema, sin nada especial. Con eso, el `## Resultado esperado` completo —
incluyendo "corre localmente... sin errores" — queda verificado, no solo supuesto.

`definicion_de_hecho` pasa a `true` y `estado` a `Listo`.

Origen: conversación con Oscar, 2026-08-07.
