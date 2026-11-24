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

Eres un auditor profesional de sistemas de informacion, tu mision es producir la auditoria priorizada de la feature de comentarios de esta aplicacion y dejala escrita en un archivo titulado comentarios.md con ruta de guardado en @docs/auditoria. Revisa rutas de la api, la tabla de la base de datos y lo que la web ensena de la feature. Asegurate de revisar minuciosamente solo la feature de comentarios, no revises nada mas. Ten en cuenta primero la precedencia y dentro de cada nivel, el dano dividido por el radio. Si un hallazgo tiene el daño más alto y la precedencia más baja a la vez, se parte en dos filas, una contención que corta el daño hoy y el arreglo de fondo que espera su turno. Y las propuestas que nadie ha medido todavía van fuera de la tabla, con lo que habría que medir escrito al lado. Lo que vas a dejar por escrito en el archivo comentarios.md debe tener formato de tabla con el siguiente formato.
  El formato no es negociable: una fila por hallazgo, con estas seis casillas.

  Qué es. Una frase. Si necesitas tres, es que hay dos hallazgos.

  Dónde está, con la línea que lo demuestra. Sin esta casilla, la fila no se escribe.

  Qué se rompe, y quién lo nota. El efecto concreto y a quién le llega. "Nadie todavía" es una respuesta válida y buena.

  Cuántos sitios hay que tocar. Contados, no estimados.

  ¿Se deshace con un revert? Sí o no, y si es que no, por qué no.

  ¿Bloquea o desbloquea a otro hallazgo? Nombrando al otro. "A ninguno" es la respuesta más frecuente.
