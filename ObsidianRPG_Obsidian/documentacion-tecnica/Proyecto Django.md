---
documento: "Proyecto Django"
area: "Infraestructura / Despliegue"
estado: "Vigente"
ruta_en_el_repo: "src/"
herramientas: ["Python"]
ultima_revision: "2026-08-07"
---

# Proyecto Django

Proyecto Django base del sistema, en `src/`. El diseño detrás vive en
`diseno-del-sistema/Arquitectura del proyecto Django.md` — esta nota es el reflejo de
implementación; si algún día divergen, gana esta y se corrige la de diseño.

## Estructura

```
src/
  manage.py
  requirements.txt
  config/          # settings, urls, wsgi, asgi
  core/            # app inicial, sin lógica de negocio todavía
```

## Instalación y arranque local

1. Python 3.9 o superior (según lo que fije [[Tema #1 — Investigar implementación de
   transcripción con Whisper local]]; no hay requisito adicional de versión desde este
   proyecto).
2. Crear un entorno virtual e instalar dependencias: `pip install -r requirements.txt`
   (instala `Django`).
3. Aplicar migraciones: `python manage.py migrate`.
4. Levantar el servidor de desarrollo: `python manage.py runserver`.

**Nota de esta implementación**: el sandbox donde se escribió este proyecto no tiene `pip` ni
`venv` disponibles (falta `python3-pip` / `python3.14-venv` a nivel de sistema, sin acceso a
`sudo`), así que los pasos de arriba no se pudieron ejecutar ni verificar aquí — el código
sigue la estructura estándar que genera `django-admin startproject` + `startapp`, pero
correrlo por primera vez en una máquina con `pip`/`venv` instalados queda pendiente de
confirmación manual.

## Apps

- `core` — app inicial, registrada en `INSTALLED_APPS`, sin modelos ni vistas propias
  todavía. Punto de partida para [[Tema #3 — Instalación de dependencias de transcripción en
  Django]] y [[Tema #4 — Orquestador y subida de audios para transcripción]].

## Base de datos

SQLite (`db.sqlite3`), no versionado (ver `.gitignore`). Suficiente para el prototipo.
