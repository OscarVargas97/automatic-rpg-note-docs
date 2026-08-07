---
id: "TSK-2"
titulo: "Tema #2 — Base del proyecto Django"
estado: "En curso"
tipo: "Feature"
disciplina: "Arquitectura del sistema"
prioridad: "P1 - Alta"
hito: "Prototipo"
segmentos_a_actualizar: ["Diseño del sistema", "Documentación técnica"]
segmentos_actualizados: true
definicion_de_hecho: false
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
de esta base. `Proyecto Django` documenta cómo levantarlo y correrlo. `Arquitectura del
proyecto Django` deja registrada la elección de Django como stack — resuelve la decisión que
tenía pendiente [[Tema #4 — Orquestador y subida de audios para transcripción]] entre HTML
simple sin framework y Django.

## Referencias
- src/ (carpeta nueva, todavía no existe)

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
