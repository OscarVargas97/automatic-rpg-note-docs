---
entrada: "Esquema del vault de campaña"
categoria: "Esquema del vault de campaña"
estado: "Borrador"
prioridad: "Must"
complejidad: "Media"
---

# Esquema del vault de campaña

Este es el esquema que el sistema **producirá y mantendrá** en el vault de una campaña real
— no confundir con `VAULT_MAP.md`, que describe el vault de gestión de *este* repo. Todo
nombre, personaje o lugar usado como ejemplo aquí abajo es ficticio e ilustrativo.

## Principio de enrutamiento

Cada fragmento de una transcripción cae en una de dos categorías:

1. **Merece nota propia** — una entidad con nombre (Personaje, Lugar, Facción, Objeto) o un
   Hilo narrativo con entidad suficiente para seguir su rastro sesión a sesión.
2. **Solo queda en el resumen de la Partida** — menciones de paso, ambigüedad de nombre, o
   información insuficiente para justificar una nota nueva.

Ver `Pipeline de ingesta y enrutamiento.md` para la lógica paso a paso de esa decisión.

## Carpeta `raws/`

Junto a `campaña/`, en la raíz del vault: guarda la transcripción cruda (raw) de cada sesión,
tal como la produce `Pipeline de transcripción.md`, antes de cualquier resumen o
clasificación. Se escribe siempre, sin excepción — es el respaldo si el resto del pipeline se
equivoca. Un archivo por sesión (o por audio de origen, si la sesión no llegó a unificarse
todavía); `Partidas` enlaza al archivo correspondiente vía `transcripcion_fuente`.

## Convención común a todas las carpetas

Igual que en `VAULT_MAP.md`: el nombre de archivo es el título, los wikilinks resuelven las
relaciones, y cada nota lleva `partida_de_origen` (la Partida donde se creó) y
`ultima_mencion` (la Partida más reciente que la tocó) — así cualquier entidad es
trazable hasta la sesión que la originó o la actualizó por última vez.

## Personajes (`campaña/personajes/`)

| Campo | Valores |
|---|---|
| `nombre` | nombre del personaje |
| `tipo` | `PJ` · `NPC` · `Deidad / Entidad` · `Antagonista` |
| `estado` | `Vivo` · `Muerto` · `Desaparecido` · `Desconocido` |
| `facciones` | wikilinks a `campaña/facciones/` |
| `lugares_asociados` | wikilinks a `campaña/lugares/` |
| `relacion_con_grupo` | `Aliado` · `Neutral` · `Hostil` · `Desconocido` |
| `partida_de_origen`, `ultima_mencion` | wikilinks a `campaña/partidas/` |
| `etiquetas` | array libre |

Cuerpo: `## Descripción`, `## Historial en partidas` (una línea por sesión, con wikilink y
qué pasó), `## Notas del máster` (opcional, información que los jugadores no conocen).

## Lugares (`campaña/lugares/`)

| Campo | Valores |
|---|---|
| `nombre` | nombre del lugar |
| `tipo` | `Región` · `Ciudad` · `Edificio` · `Punto de interés` · `Plano / Reino` |
| `pertenece_a` | wikilink a un lugar padre, para jerarquía |
| `facciones_presentes` | wikilinks a `campaña/facciones/` |
| `partida_de_origen`, `ultima_mencion` | wikilinks a `campaña/partidas/` |

Cuerpo: `## Descripción`, `## Eventos ocurridos aquí` (lista con wikilinks a Partidas).

## Facciones (`campaña/facciones/`)

| Campo | Valores |
|---|---|
| `nombre` | nombre de la facción |
| `tipo` | `Gremio` · `Reino` · `Culto` · `Familia` · `Organización criminal` · `Otro` |
| `relacion_con_grupo` | `Aliado` · `Neutral` · `Hostil` · `Desconocido` |
| `lider` | wikilink a `campaña/personajes/`, opcional |
| `sede` | wikilink a `campaña/lugares/`, opcional |
| `partida_de_origen`, `ultima_mencion` | wikilinks a `campaña/partidas/` |

Cuerpo: `## Descripción`, `## Objetivos conocidos`, `## Historial en partidas`.

## Objetos (`campaña/objetos/`)

| Campo | Valores |
|---|---|
| `nombre` | nombre del objeto |
| `tipo` | `Arma` · `Artefacto mágico` · `Documento` · `Llave narrativa` · `Otro` |
| `poseedor_actual` | wikilink a `campaña/personajes/`, opcional |
| `partida_de_origen`, `ultima_mencion` | wikilinks a `campaña/partidas/` |

Cuerpo: `## Descripción`, `## Historial de posesión` (lista cronológica de quién lo tuvo y
desde qué partida).

## Hilos narrativos (`campaña/hilos-narrativos/`)

| Campo | Valores |
|---|---|
| `nombre` | nombre del hilo |
| `estado` | `Abierto` · `En progreso` · `Resuelto` · `Abandonado` |
| `personajes_involucrados` | wikilinks a `campaña/personajes/` |
| `partida_de_origen`, `ultima_mencion` | wikilinks a `campaña/partidas/` |

Cuerpo: `## Qué se sabe`, `## Preguntas abiertas`, `## Historial en partidas`.

`estado` es el único campo de esta lista que el sistema puede cambiar sin que sea una
decisión de diseño: cerrar un hilo (`Resuelto`/`Abandonado`) es un hecho observable de la
sesión, no una promoción a `Canon` — eso solo aplica a las notas de `diseno-del-sistema/` de
este repo, nunca a las de una campaña.

## Partidas (`campaña/partidas/`)

La pieza que cierra el ciclo: el resumen que el usuario pidió explícitamente para cada
sesión.

| Campo | Valores |
|---|---|
| `numero_de_sesion` | entero, calculado igual que el `id` de una tarea (máximo existente + 1) |
| `fecha` | fecha de la sesión |
| `campaña` | nombre libre — el vault puede servir a más de una mesa |
| `participantes` | array de texto |
| `duracion_minutos` | opcional |
| `transcripcion_fuente` | ruta al archivo en `raws/` con la transcripción cruda de esta sesión, opcional |

Cuerpo:

- `## Resumen` — uno o dos párrafos generados por Claude a partir de la transcripción. Lo
  que un jugador que faltó a la sesión leería para ponerse al día.
- `## Entidades nuevas` — lista de wikilinks a lo que esta sesión creó, agrupado por key
  (Personajes / Lugares / Facciones / Objetos / Hilos).
- `## Entidades actualizadas` — lista de wikilinks a lo que ya existía y cambió, con una
  línea de qué cambió cada una.
- `## Decisiones y cabos sueltos` — cosas dichas en mesa que hay que recordar y no encajan
  como campo de ninguna entidad (una promesa a un NPC, una regla acordada, algo pendiente
  para la próxima sesión).
- `## Transcripción cruda` — opcional, enlace al archivo correspondiente en `raws/`. Nunca se
  borra aunque ya esté resumida: es el respaldo si el resumen se equivocó.

## Qué no decide el sistema solo

- Nunca fusiona dos notas que podrían ser la misma entidad con nombre distinto — las deja
  como candidatas separadas en `## Decisiones y cabos sueltos` de la Partida.
- Nunca escribe en `## Notas del máster` de un Personaje a partir de la transcripción de
  jugadores — ese campo es manual, información que el grupo no dijo en voz alta.
- Nunca borra ni reescribe contenido que un humano escribió a mano en una nota existente —
  solo añade (nueva línea de historial, campo actualizado con el dato nuevo).
