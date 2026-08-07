# CLAUDE.md

Contexto operativo del proyecto. Léelo entero antes de tocar nada.

---

## 1. Qué es esto

**Este repo es la documentación y especificación de un sistema — no es el sistema en sí.**

El sistema que se está especificando aquí: una mesa de rol (o de creación de lore de un
juego) se graba, un transcriptor de voz la convierte en texto, y Claude procesa esa
transcripción para clasificar automáticamente la información en las notas correspondientes
de un vault de Obsidian — Personajes, Lugares, Facciones, Objetos, Hilos narrativos — y deja
un resumen de la sesión en una nota de Partida. El objetivo es que un máster o un grupo de
juego termine una sesión y tenga el lore actualizado sin transcribir ni ordenar nada a mano.

- **Pitch:** transcripción de voz + Claude → lore de campaña clasificado automáticamente en
  su key correspondiente, con un resumen por partida.
- **Transcriptor elegido:** Whisper local (whisper.cpp / faster-whisper) — ver la fila del
  7 de agosto de 2026 en el Log de decisiones de `ObsidianRPG_Obsidian/meta/contexto-para-ia.md`.
  Sin costo por uso, sin depender de conexión durante la sesión de mesa.
- **Hito actual:** Especificación — todavía no existe código de implementación.

Tratar esto como desarrollo de software significa: las tareas de este proyecto no son
"escribir un párrafo de documentación", son unidades de trabajo con alcance, segmentos que
tocan, y una definición de hecho — igual que si el producto fuera código, aunque hoy el
producto sea un documento de arquitectura o un esquema de vault.

---

## 2. Territorio: vault vs. código

`ObsidianRPG_Obsidian/` es el vault de Obsidian versionado en este repo — la especificación,
las tareas, el lore ilustrativo. `src/` es el código de implementación real del sistema
(scripts de transcripción, pipeline de ingesta, etc.) — decisión tomada el 2026-08-07, ver
el Log de decisiones en `ObsidianRPG_Obsidian/meta/contexto-para-ia.md`. Ambos viven en este
mismo repositorio, sin split de repos por ahora.

Si encuentras código de implementación real fuera de `src/` (no snippets ilustrativos dentro
de un documento de especificación), algo se saltó el proceso — repórtalo antes de seguir.

---

## 3. La regla central: segmentos

Cada tarea de `ObsidianRPG_Obsidian/tareas/` declara **qué segmentos toca antes de
ejecutarse**:

`Diseño del sistema` · `Documentación técnica` · `Costos` · `Contexto para IA` ·
`Muro de Ideas` · `Ninguno`

Una tarea **no está terminada** hasta que:

1. El resultado declarado en `## Resultado esperado` es observable (un documento escrito,
   un esquema validado, una decisión registrada — según lo que la tarea prometió).
2. Todos los segmentos declarados están actualizados.
3. La casilla **`segmentos_actualizados`** está marcada en el frontmatter de la tarea.

Si durante la ejecución aparece un segmento afectado que nadie previó, **añádelo en ese
momento** y actualízalo. No lo dejes para después.

No hay segmento `Lore` aquí: este proyecto no tiene narrativa propia que proteger — el
"lore" es el dominio del sistema que se está especificando, no de este repo. Tampoco hay
contrato vault↔GitHub como en Jueguito: no hay repo de implementación todavía, así que las
tareas no generan borradores de issue.

---

## 4. Documentación técnica

`ObsidianRPG_Obsidian/documentacion-tecnica/` describe cómo funciona (o funcionará) el
sistema por dentro. Nace vacía — se va llenando tarea por tarea, a medida que se especifican
áreas reales. Áreas esperadas, sin compromiso de que existan todas desde el día uno:

| Área | Cubre |
| --- | --- |
| Transcripción (Whisper) | Cómo se captura audio, configuración del modelo, formato de salida |
| Ingesta y enrutamiento (Claude) | Cómo se lee la transcripción y se decide a qué key va cada fragmento |
| Esquema de datos / Frontmatter | Contrato de campos por tipo de nota — vive primero en `diseno-del-sistema/`, se documenta aquí solo cuando hay implementación real que seguirlo |
| Vault y escritura de archivos | Cómo se crean/actualizan notas sin pisar contenido escrito a mano |
| Infraestructura / Despliegue | Dónde corre el pipeline, requisitos, instalación |

Reglas:

- Si un cambio invalida un documento, **la tarea no está terminada hasta que el documento
  refleje la realidad**. Un documento desactualizado es peor que no tenerlo.
- Si un documento está marcado **Desactualizado**, no confíes en él: verifica y corrígelo.
- Si tu cambio contradice un documento vigente: o ajustas el cambio, o actualizas el
  documento **y** añades una fila al Log de decisiones.

---

## 5. Convenciones que aplican a todo commit

### Ramas y commits

- `main` siempre refleja el estado real de la especificación. Nadie commitea directo sin
  revisar.
- Ramas: `feature/TSK-12-slug`, `fix/TSK-12-slug` (mismo esquema que Jueguito).
- Commit: `[área] verbo en imperativo` — `[esquema] añade campo ultima_mencion a Personajes`.
- Este repo no es todavía un repositorio git — la primera tarea que lo necesite pide
  `git init` en vez de asumirlo.

### Documentos

- Prosa clara, español, sin relleno. Tablas solo para mapeos reales de valores (frontmatter,
  rutas, decisiones), nunca para narrar.
- Todo documento de `diseno-del-sistema/` o `documentacion-tecnica/` lleva frontmatter según
  `VAULT_MAP.md` — no inventes campos nuevos sin actualizar ese archivo primero.
- Ejemplos ilustrativos de código (snippets de un pipeline, de un prompt) están permitidos
  dentro de un documento de especificación — no cuentan como "código de implementación" para
  la sección 2 mientras no formen un sistema ejecutable real.

---

## 6. Definición de Hecho

- [ ] Cumple el `## Resultado esperado` de la tarea, de forma observable
- [ ] No contradice un documento de `diseno-del-sistema/` marcado `Canon` sin una fila nueva
      en el Log de decisiones
- [ ] Segmentos declarados actualizados
- [ ] Casilla `segmentos_actualizados` marcada en la tarea

---

## 7. Prohibiciones

- **No inventes lore de ejemplo sin marcarlo como ficticio.** Cualquier Personaje, Lugar,
  Facción u Objeto de ejemplo dentro de un documento de especificación necesita decir
  explícitamente que es ilustrativo — nunca se confunde con una campaña real.
- **No cambies el transcriptor elegido** (Whisper local) sin una fila en el Log de
  decisiones de `ObsidianRPG_Obsidian/meta/contexto-para-ia.md`.
- **No promuevas una entrada de `diseno-del-sistema/` a `Canon`** sin la misma fila.
- **No escribas código de implementación real del sistema** dentro de este repo sin que una
  tarea lo declare explícitamente y sin haber resuelto primero la decisión pendiente de la
  sección 2 (dónde vive el código).
- **No cierres una tarea** que tenga segmentos pendientes.

---

## 8. Comandos

_(No hay comandos verificados todavía — este repo no ejecuta nada, es especificación. Se
llena cuando exista una primera pieza de implementación real, por ejemplo un script de
prueba del pipeline de transcripción.)_

---

## 9. Cuando algo no cuadra

Si la tarea es ambigua, si el cambio choca con un documento `Canon`, o si no encuentras la
fuente de verdad de un dato: **pregunta antes de decidir**. Una decisión silenciosa se
convierte en deuda que nadie sabe que existe.

---

## 10. Vault de Obsidian

El mapa completo del vault — estructura de carpetas, esquema de frontmatter por tipo de
nota, mapeo de segmentos — está en [`VAULT_MAP.md`](./VAULT_MAP.md). Consúltalo antes de
crear o editar cualquier nota en `ObsidianRPG_Obsidian/`.

El esquema del vault que el sistema *producirá* para una campaña real (Personajes, Lugares,
Facciones, Objetos, Hilos narrativos, Partidas) es un documento de diseño, no la estructura
de este repo — vive en `ObsidianRPG_Obsidian/diseno-del-sistema/Esquema del vault de
campaña.md`. No lo confundas con `VAULT_MAP.md`, que describe el vault de *este* proyecto.
