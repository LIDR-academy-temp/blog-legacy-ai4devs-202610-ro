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

**Modelo:** Sonnet 4.6
**Herramienta:** Claude Code

```
Hacé una auditoría priorizada de la feature de comentarios de este blog: crear, editar, borrar y listar comentarios, en sus tres capas (rutas de la API, tabla en la base de datos y lo que la web muestra). Solo esa feature: no audites posts, autores, categorías ni etiquetas, aunque lo veas.

El contraste son las reglas de negocio del README; en particular:
- Un comentario pendiente no se muestra en el detalle del post.
- El recuento de comentarios cuenta solo los aprobados.
- La respuesta pública de un post no expone el correo de quien comenta.
Si un hallazgo no rompe ninguna regla escrita, decilo en la fila.

Escribí el resultado en docs/auditoria/comentarios.md como una tabla con una fila por hallazgo, ordenada de mayor a menor prioridad, con estas seis columnas:
1. Qué es (una frase).
2. Dónde está: archivo y línea exacta que lo demuestra.
3. Qué se rompe y quién lo nota ("nadie todavía" es válido).
4. Cuántos sitios hay que tocar (contados, no estimados).
5. ¿Se deshace con un revert? Sí o no; si es no, por qué.
6. ¿Bloquea o desbloquea a otro hallazgo? Nombrándolo, o "a ninguno".

Reglas:
- No modifiques ningún archivo fuera de docs/auditoria/. No arregles nada.
- No uses el comando /refactor.
- Al final del chat decime cuántos hallazgos escribiste en total.
```

**Qué salió:** funcionó a la primera.

## Prompt 2

**Modelo:** Sonnet 4.6
**Herramienta:** Claude Code

```
No modifiques nada. Decime el slug de un post publicado que muestre comentarios en GET /posts/:slug, para poder comprobar el H1 en el navegador.
```

**Qué salió:** retorno la url del post.
