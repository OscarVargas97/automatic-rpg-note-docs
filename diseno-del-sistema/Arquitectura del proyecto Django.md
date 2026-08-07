---
entrada: "Arquitectura del proyecto Django"
categoria: "Otros"
estado: "Borrador"
---

# Arquitectura del proyecto Django

Registra la elección de Django como base de aplicación para el sistema — resuelve la
decisión de stack (HTML simple sin framework vs. Django) que tenía pendiente
`Orquestador y subida de audios.md`: **Django**, porque unifica bajo un solo proyecto la
instalación de dependencias de transcripción ([[Tema #3 — Instalación de dependencias de
transcripción en Django]]) y las vistas del orquestador ([[Tema #4 — Orquestador y subida de
audios para transcripción]]) sin tener que coordinar dos piezas sueltas (script + interfaz
HTML aparte).

## Estructura

Un único proyecto Django, sin apps de negocio todavía. Vive en la raíz del repo
`automatic-rpg-note` (ver CLAUDE.md sección 2) — el mismo repo gestiona código y el resto del
proyecto; el vault de especificación (este archivo incluido) vive aparte, en `docs/`, un
repositorio propio gitignored por el repo de código:

- `config/` — paquete de settings del proyecto (`settings.py`, `urls.py`, `wsgi.py`,
  `asgi.py`). Se llama `config` y no por el nombre del sistema, para no acoplar el paquete de
  configuración a un dominio específico — las apps que vengan después (instalación de
  dependencias, orquestador) son las que llevan nombre de dominio.
- `core/` — app inicial vacía, registrada en `INSTALLED_APPS`. Punto de partida para que
  [[Tema #3 — Instalación de dependencias de transcripción en Django]] y [[Tema #4 —
  Orquestador y subida de audios para transcripción]] agreguen ahí su lógica, o creen apps
  propias si el alcance lo justifica al ejecutarlas.
- Base de datos: SQLite por defecto (`db.sqlite3`, no versionado) — suficiente para el
  prototipo; no hay requisito todavía que empuje a otro motor.
- Dependencias: [uv](https://docs.astral.sh/uv/) en vez de `pip`/`venv` sueltos —
  `pyproject.toml` declara las dependencias, `uv.lock` las fija para instalaciones
  reproducibles. Elegido porque no requiere privilegios de administrador para instalarse (a
  diferencia de `python3-pip`/`python3-venv` a nivel de sistema) y resuelve entorno virtual +
  instalación en un solo paso.
- Atajos de comandos: un `Makefile` en la raíz de `automatic-rpg-note` (`make install`,
  `make migrate`, `make run`, `make makemigrations`, `make shell`) delega en `uv run`, para no
  tener que escribir el comando completo cada vez.
- Uso: prototipo de un solo usuario, corrido localmente en la propia máquina — Django se
  eligió por comodidad de desarrollo (un único framework para vistas, ORM y el orquestador),
  no porque el sistema necesite servir a varios usuarios o desplegarse en un servidor
  compartido. Las decisiones de stack de aquí en adelante (frontend, procesamiento en
  background) parten de ese supuesto: optimizan por simplicidad de correr en local, no por
  escalar.

## Fuera de alcance de esta pieza

Cualquier app de negocio (instalación de dependencias, orquestador, ingesta con Claude) —
esas se diseñan en sus propias tareas, encima de esta base.
