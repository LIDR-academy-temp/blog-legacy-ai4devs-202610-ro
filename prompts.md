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

**Modelo:** Sonet 5.5 Medium
**Herramienta:** Claude Code

```
Check repo at path "/home/alfonso/Documents/AI-Train/courses/L1DR/repo/blog-legacy-ai4devs-202610-ro" . We will audit the "commentarios" feature with its 3 layers, "crear", "ditar", "borrar", and "listar". 

We will use the following format. One row per finding with the following 6  features

"Que es". A single sentence. If you need three, it means there are two separate findings.
"Dónde está", including the specific line that demonstrates it. Without this field, the entry cannot be written.
"Qué se rompe, y quién lo nota". The specific impact and who is affected by it. "Nadie todavía" is a perfectly valid and acceptable answer.
"Cuántos sitios hay que tocar". An exact count, not an estimate.
"¿Se deshace con un revert?" Yes or no; if no, explain why.
"¿Bloquea o desbloquea a otro hallazgo?" Specify which one. "A ninguno" is the most common answer.

Write the findings in "docs/auditoria/"
```

**Qué salió:** funcionó a la primera .
