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

---

## Prompt 1

**Modelo:** claude-opus-5-5
**Herramienta:** Claude Code

```
1
```

## Prompt 2

**Modelo:** claude-opus-5-5
**Herramienta:** Claude Code

```
Todo en castellano, vamos a hacer una auditoria de la feature comentarios.
Vas a tener que buscar hallazgos. Para evitar falsos positivos, vas a tener que citar el fichero, línea y código donde ocurre. Cada hallazgo va a ir en una fila con seis casillas:
1. Qué es. Una frase. Si necesitas tres, es que hay dos hallazgos.
2. Dónde está, con fichero y la línea que lo demuestra. Sin esta casilla, la fila no se escribe.
3. Qué se rompe, y quién lo nota. El efecto concreto y a quién le llega. "Nadie todavía" es una respuesta válida y buena.
4. Cuántos sitios hay que tocar. Contados, no estimados. Te diría que contaras, y además listaras archivo y línea.
5. ¿Es reversible? Es decir, si te mando corregir el hallazgo y la liamos, ¿podemos volver atrás con un simple revert sin efectos secundarios? Si es que no, ¿por qué no?
6. ¿Bloquea o desbloquea a otro hallazgo? Nombrando al otro. "A ninguno" es la respuesta más frecuente.
Pregunta lo que vayas a necesitar antes de hacer esto.
Y obviamente, bajo ningún concepto toques nada del código.
```

## Prompt 3

**Modelo:** claude-opus-5-5
**Herramienta:** Claude Code

```
Ok, añade una lista de los hallazgos en orden de prioridad, baso pura y únicamente en las dependencias (bloqueos).
```

## Prompt 4

**Modelo:** claude-opus-5-5
**Herramienta:** Claude Code

```
Haz una segunda lista de los hallazgos según el coste, de más barato a más caro de arreglar.
```

## Prompt 5

**Modelo:** claude-opus-5-5
**Herramienta:** Claude Code

```
Vale, ahora añade un gráfico de dos ejes. Vertical para prioridad en base a coste (más barato es más prioritario), horizontal para prioridad en base a dependencias. Highlightea el cuadrante EZ PZ, donde son prioritarios en ambos sentidos.
```

## Prompt 6

**Modelo:** claude-opus-5-5
**Herramienta:** Claude Code

```
Por cuadrantes, obviamente empezaríamos por EZ PZ (no arregles nada, estamos teorizando). ¿Y después, harías barato pero no desbloqueante o desbloqueante pero caro? (no arregles nada todavía, solo responde).
```

## Prompt 7

**Modelo:** claude-opus-5-5
**Herramienta:** Claude Code

```
¿La parte barata de los que desbloquean? Implica partir el hallazgo en dos: contención rápida y arreglo para el largo plazo? ¿Eso lo aplicarías sobre el cuadrante "Desbloquea, pero caro"?
```

## Prompt 8

**Modelo:** claude-opus-5-5
**Herramienta:** Claude Code

```
Después del texto de EZ PZ, añade un resumen de este análisis. ¿Cómo y cuándo partirías hallazgos de este cuadrante? No arregles nada, y en su lugar, sí, anota en la Parte B, como razonamiento tuyo. Ya lo reescribiré yo manualmente si hace falta.
```

## Prompt 9

**Modelo:** claude-opus-5-5
**Herramienta:** Claude Code

```
Vamos con el PR.


<pasted_content id="c7e2">
Un pull request desde tu fork, con dos cosas dentro y ni una más:

Tu archivo de auditoría en docs/auditoria/, con la tabla y las tres líneas.

prompts.md, en la raíz del proyecto. Ya está ahí con la plantilla puesta.

git add docs/auditoria prompts.md
git commit -m "auditoria: comentarios + prompts"
git push -u origin auditoria-<tus-iniciales>
Con la rama empujada, GitHub te ofrece arriba el botón para abrir el pull request. Va contra el repositorio del curso, no contra tu fork.
</pasted_content id="c7e2">

Asegúrate que está todo y que se hace contra el repo correcto, el del curso.
```
