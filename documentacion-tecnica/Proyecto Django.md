---
documento: "Proyecto Django"
area: "Infraestructura / Despliegue"
estado: "Vigente"
ruta_en_el_repo: "."
herramientas: ["Python", "Otro"]
ultima_revision: "2026-08-07"
---

# Proyecto Django

Proyecto Django base del sistema. Vive en un repositorio aparte, `automatic-rpg-note-src`
(https://github.com/OscarVargas97/automatic-rpg-note-src) — no en este repo de
especificación; `ruta_en_el_repo` es relativa a ese repo, no a `automatic-rpg-note`. Ver
CLAUDE.md sección 2 y el Log de decisiones en `meta/contexto-para-ia.md` (fila del
2026-08-07 sobre el split de repos). El diseño detrás vive en
`diseno-del-sistema/Arquitectura del proyecto Django.md` — esta nota es el reflejo de
implementación; si algún día divergen, gana esta y se corrige la de diseño.

`herramientas` incluye "Otro" porque el esquema de `VAULT_MAP.md` no tiene una entrada
propia para gestores de paquetes: se refiere a [uv](https://docs.astral.sh/uv/) (gestor de
dependencias y entorno virtual) y a `make` (atajos de comandos vía el `Makefile` de la raíz
de `automatic-rpg-note-src`).

## Estructura

```
automatic-rpg-note-src/
  Makefile         # atajos: install, migrate, makemigrations, run, shell
  manage.py
  pyproject.toml   # dependencias, gestionadas con uv
  uv.lock          # versionado, para instalaciones reproducibles
  config/          # settings, urls, wsgi, asgi
  core/            # app inicial + comando de gestión check_transcription
```

## Instalación y arranque local

Con `make` (recomendado, desde la raíz de `automatic-rpg-note-src`):

1. Instalar [uv](https://docs.astral.sh/uv/getting-started/installation/) si no está
   disponible (`curl -LsSf https://astral.sh/uv/install.sh | sh`, no requiere `sudo`) y
   `make` (`sudo apt install make` en Debian/Ubuntu, o el equivalente del sistema).
2. `make install` — corre `uv sync`, crea `.venv` e instala las dependencias según `uv.lock`
   (Django y faster-whisper).
3. `make migrate` — aplica migraciones.
4. `make run` — levanta el servidor de desarrollo.

Sin `make`, el equivalente directo con `uv`: `uv sync`, `uv run python manage.py migrate`,
`uv run python manage.py runserver`.

Python 3.10 o superior (lo fija `pyproject.toml`; `uv sync` descarga el intérprete si hace
falta).

**Verificado en esta implementación**: `make install`, `make migrate` y `make run` se
corrieron en el sandbox (Django 5.2.17 instalado, migraciones aplicadas, servidor
respondiendo `200` en `/`) — a diferencia del intento anterior con `pip`/`venv`, que no pudo
verificarse por falta de `sudo`. `uv` no necesita privilegios de administrador.

## Apps

- `core` — app inicial, registrada en `INSTALLED_APPS`, sin modelos ni vistas propias
  todavía. [[Tema #3 — Instalación de dependencias de transcripción en Django]] le agregó el
  comando de gestión `check_transcription` (`core/management/commands/`) — ver
  `documentacion-tecnica/Transcripción con Whisper local.md`. Sigue siendo el punto de
  partida para [[Tema #4 — Orquestador y subida de audios para transcripción]].

## Base de datos

SQLite (`db.sqlite3`), no versionado (ver `.gitignore` de `automatic-rpg-note-src`).
Suficiente para el prototipo.
