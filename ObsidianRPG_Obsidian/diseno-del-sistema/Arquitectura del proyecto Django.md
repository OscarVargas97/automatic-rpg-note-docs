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

Un único proyecto Django en `src/`, sin apps de negocio todavía:

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

## Fuera de alcance de esta pieza

Cualquier app de negocio (instalación de dependencias, orquestador, ingesta con Claude) —
esas se diseñan en sus propias tareas, encima de esta base.
