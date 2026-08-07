---
entrada: "Pipeline de ingesta y enrutamiento"
categoria: "Ingesta y enrutamiento (Claude)"
estado: "Borrador"
prioridad: "Must"
complejidad: "Alta"
---

# Pipeline de ingesta y enrutamiento

Cómo una grabación de mesa se convierte en notas actualizadas del vault de campaña. Depende
del esquema definido en `Esquema del vault de campaña.md` — léelo primero si algo aquí no
cuadra con los campos que se mencionan.

## Flujo, paso a paso

1. **Captura.** Se graba la sesión (herramienta de captura — fuera de alcance de esta nota,
   candidata a otra pieza de `diseno-del-sistema/` cuando se especifique).
2. **Transcripción.** Whisper local convierte el audio en texto plano. Sin diarización
   garantizada de entrada — si el modelo elegido no separa hablantes, el texto es un bloque
   continuo y el paso 3 no puede asumir que sabe quién dijo qué.
3. **Ingesta con Claude.** Claude lee la transcripción completa de la partida — no por
   fragmentos sueltos, porque una mención ambigua al principio puede resolverse con contexto
   del final.
4. **Detección de entidades.** Por cada mención de un nombre propio (personaje, lugar,
   facción, objeto) o de un hilo narrativo:
   - Busca por nombre exacto o alias conocido en la carpeta correspondiente de
     `campaña/` — el nombre de archivo es el título, así que es una búsqueda directa, no una
     consulta a una base de datos separada.
   - **Si existe**: actualiza la nota — añade una línea a su `## Historial en partidas`
     (o equivalente), actualiza `ultima_mencion`, ajusta campos que cambiaron de forma
     observable (`estado: Vivo` → `Muerto`, `relacion_con_grupo`, etc.).
   - **Si no existe y la mención tiene entidad propia** (no es solo de paso): crea el stub
     con el frontmatter mínimo del esquema y `partida_de_origen` apuntando a la partida
     actual.
   - **Si es ambigua o insuficiente** (mención de pasada, nombre no confirmado, posible
     duplicado de una entidad ya existente con nombre parecido): no crea ni actualiza nada —
     queda registrada solo en el resumen de la Partida, señalada para revisión manual.
5. **Escritura de la Partida.** Se crea (o actualiza, si la sesión se procesa en más de una
   pasada) la nota de `campaña/partidas/` con el resumen, las listas de entidades nuevas y
   actualizadas, y los cabos sueltos — este paso **nunca se salta**: es la razón por la que
   el sistema existe, igual que el paso 4 de segmentos en `obsidian-task-solve` para
   Jueguito.
6. **Nada se sobrescribe a mano.** Contenido escrito por un humano en una nota existente
   (una nota de máster, una descripción ampliada) nunca se reemplaza — el pipeline solo
   añade líneas de historial y actualiza campos de frontmatter que le pertenecen a él.

## Decisiones que este documento deja abiertas

- **Diarización**: si el transcriptor elegido no separa hablantes, ¿el resumen distingue
  "dijo el máster" de "dijo un jugador"? Afecta directamente qué puede ir en `## Notas del
  máster` sin arriesgar a filtrar información de jugador ahí. Sin resolver — candidata a
  tarea de `Investigación`.
- **Frecuencia de ingesta**: ¿se procesa la transcripción completa al final de la sesión, o
  en fragmentos durante la sesión? El paso 3 asume "completa, al final" por ahora; procesar
  en vivo cambiaría el paso 4 (contexto incompleto al decidir si algo es ambiguo).
- **Cómo se dispara la ingesta** (comando manual, automático al terminar la grabación):
  fuera de alcance de esta nota, depende de la herramienta de captura del paso 1.

## Referencia cruzada

Cuando este pipeline tenga una implementación real (no solo esta especificación), su
funcionamiento se documenta en `ObsidianRPG_Obsidian/documentacion-tecnica/`, área "Ingesta
y enrutamiento (Claude)" — esta nota queda como el diseño, esa como el reflejo de lo que el
código realmente hace. Si algún día divergen, gana la documentación técnica y esta nota se
corrige.
