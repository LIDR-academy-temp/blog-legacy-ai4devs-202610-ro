# Prompts

Aquí van **todos los prompts que lanzaste** para hacer el ejercicio, en el orden en que los
lanzaste, con el modelo y la herramienta de cada uno.

Esto no es papeleo. Lo que se revisa es **cómo pediste las cosas**, no solo lo que salió: un
resultado flojo con un prompt bueno y un resultado flojo con un prompt vago necesitan feedback
distinto, y sin este archivo no se distinguen.

## Cómo rellenarlo

- Un apartado `## Prompt N` por cada prompt.
- **Pega el prompt tal cual lo lanzaste**, dentro del bloque de código, aunque ocupe diez líneas
  y aunque tenga faltas. No lo reescribas para que quede bien: el que arreglaste mentalmente
  después no es el que lanzaste.
- Incluye también los que **no funcionaron**. Suelen ser los más útiles de leer.
- `Modelo` y `Herramienta` en todos. Si cambiaste de una a otra a mitad, se nota aquí.

Borra el ejemplo de abajo cuando escribas el primero.

---

## Prompt 1

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
No cambies ningún archivo ni audites nada aún. Quiero que hagas un mapeo únicamente de la feature de comentarios, de punta a punta: los endpoints que la conponen incluyendo el archivo y líneas, qué tabla y columnas usa (en migración y semilla) y qué componentes de la web la muestran. Incluye además endpoints de otras features que devuelvan o utilicen comentarios. Si no tienes claro si algo corresponde a la feature, no lo asignes, y dímelo en su lugar.
```

**Qué salió:** Mapeo de las tres capas (endpoints en `routes.ts` con líneas, tabla y columnas en migración y semilla, componentes de la web), sin tocar archivos ni evaluar nada. Funcionó a la primera.

## Prompt 2

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
Guarda este mapeo en un artefacto en docs/auditoria/mapeo.md para poder consultarlo
```

**Qué salió:** Guardó el mapeo tal cual en `docs/auditoria/mapeo.md`, con enlaces relativos a las líneas.

## Prompt 3

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
Quiero que audites sólo la feature de comentarios que has mapeado. Contrástale con las reglas de negocio del README.md. No tienes que cambiar nada de código. Escribe los resultados en docs/auditoria/comentarios.md como una tabla que incluya las siguientes columnas: Qué es | Dónde está | Qué se rompe y quién lo nota | Cuántos sitios hay que tocar |
¿Se deshace con un revert? | ¿Bloquea o desbloquea a otro?
Sigue las siguientes reglas:
+ Cada hallazgo en una sola frase, que indique el archivo y la linea y el fragmento citado literal. Si no puedes indicar la linea no escribas el hallazgo. Si puedes añadir el enlace en el fichero a la línea para poder comprobarlo yo más fácilmente, hazlo.
+ Si algún defecto se repite en varios sitios, una sola fila con la lista de los distintos sitios.
+ Sólo debes rellenar Qué es, Donde está, Cuantos sitios (contados y listados) y Revert. Deja en blanco Qué se rompe y Bloquea
+ No añadas severidad. Un máximo de 8 hallazgos
+ Si se trata de una propuesta que no se haya medido, ponlo fuera de la tabla en un a sección aparte que indique cómo habría que medirlo. Anota al final cuántos hallazgos has encontrado en total, aunque no los escribas todos.
```

**Qué salió:** Tabla con 6 hallazgos contrastados con el README, cita literal y enlace en cada fila, y las columnas de daño y bloqueo en blanco como pedí. Por su cuenta lanzó GET contra la API local para medir efectos. Dejó fuera 2 propuestas sin medir y 9 hallazgos sin regla del README: 17 en total.

## Prompt 4

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
El segundo punto dice que en vez de sólo los aprobados cuenta todo lo que no es pendiente. Es cierto que hay Aprobado, aprobado y ok, pero entiendo que los 3 casos son aprobados, si bien hay una falta de normalización del término que no debería producirse, pero sí que muestra sólo aprobados.
```

**Qué salió:** Le discutí la fila 2 y tenía yo razón: con los datos actuales el recuento no incumple la regla 5. Lo comprobó en la base y reescribió la fila: el problema son dos criterios distintos de «aprobado» y el detalle oculta 3 aprobados. Añadió una tercera propuesta sin medir: 18 en total.

## Prompt 5

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
De los puntos 1 a 4, que son  los que me ha dado tiempo a hacer comprobaciones, comprueba si algún caso se ve afectado por otro de los que hay. Sólo dame esa relación, pero no modifiques la tabla aún que quiero comporbarlo.
```

**Qué salió:** Relaciones entre las filas 1-4 sin tocar el archivo: la 1 bloquea a la 2 y a la 3 (arreglarlas antes expone tres correos más), la 3 es la causa de la 2 y la 4 es independiente. Orden que deduce: 1 → 3 → 2.

## Prompt 6

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
Ok, dado que la regla 3 es la causa y la 2 es el síntoma, vamos a cambiar el orden de la 2 y la 3. Cambia en el archivo el orden de las filas, pero no los identificadores de fila, para que sepamos cuál era el orden original
```

**Qué salió:** Intercambió las filas 2 y 3 (orden 1, 3, 2, 4, 5, 6) conservando los identificadores originales.

## Prompt 7

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
Agrega en la columna de si bloquea o desbloquea a otro las que sí y las que no, indicando cual a cual
```

**Qué salió:** Rellenó la casilla de bloqueo de las filas 1-4 con lo del análisis anterior. Avisó de que la cabecera del archivo sigue diciendo que esa columna va en blanco a propósito.
