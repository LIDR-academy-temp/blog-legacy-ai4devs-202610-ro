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

**Modelo:** Opus 1M xHigh
**Herramienta:** Claude Code

```
Este es el ejemplo. Bórralo.

El prompt va aquí dentro, entero y con sus saltos de línea,
para que se sepa dónde empieza y dónde acaba.
```

**Qué salió:** (opcional, una línea) funcionó a la primera / tuve que insistir / me inventó una ruta que no existe.



## Prompt 1

**Modelo:** Opus xhigh
**Herramienta:** Claude Code

```
Actúa como auditor de código de un sistema heredado. Audita únicamente los endpoints de la feature de /comentarios (sus rutas en la API, su tabla en la base de datos y lo que la web enseña de ella)

1. Investiga qué comportamiento y consumidores podrían verse afectados. Distingue lo que el código demuestra de los riesgos que se derivan de ello y de lo que no puedes determinar. No supongas por qué se tomó una decisión.

Devuelve como máximo 6 hallazgos para verificar, agrupando los defectos repetidos en un solo hallazgo con todos los lugares donde aparecen. Para cada uno, usa una fila con estas seis columnas:


> 1. **Qué es:** una frase.
> 2. **Dónde está:** archivo, línea y fragmento que lo demuestra. No incluyas un hallazgo sin evidencia.
> 3. **Qué se rompe y quién lo nota:** diferencia los efectos comprobados de los posibles riesgos; si no se sabe, indícalo.
> 4. **Cuántos sitios habría que tocar:** enuméralos y cuéntalos; si no puedes verificar el número, dilo.
> 5. **¿Se deshace con un revert?** Sí, no o no se puede determinar; explica por qué.
> 6. **¿Bloquea o desbloquea otro hallazgo?** Identifícalo, o indica que no se encontró ninguno.


No asignes una severidad automática ni presentes opiniones como hechos. Si propones una mejora sin evidencia suficiente, preséntala aparte como hipótesis y sugiere cómo comprobarla. **No modifiques el código.**

Por ejemplo, podrías elegir las rutas de comentarios y preguntar qué se rompería al añadir un segundo autor.

y déjala escrita en docs/auditoria/comentarios.md

```



## Prompt 2

**Modelo:** Opus xhigh
**Herramienta:** Claude Code

```
 Dejalo en una sola tabla , filas hallazgos y columnas las demas , limita el texto dentro de las columnas a 
  texto solo justo y necesario y deja el link al codigo mas bien.

```



## Prompt 3

**Modelo:** Opus xhigh
**Herramienta:** Claude Code

```
Clasifica cada hallazgo en base a Hecho, Riesgo, propuesta. 

Yo te voy a ir dando lineamientos. 

```



## Prompt 4

**Modelo:** Opus xhigh
**Herramienta:** Claude Code

```
❯ Mira que los links si estaen bien no me re-direccionan  
```



## Prompt 5

**Modelo:** Opus xhigh
**Herramienta:** Claude Code

```
 Hagamos un mermeid topologico con dependnecias basado en los hallazgos que encontramos y en base a eso definimos 
  la prioridad. 

```

