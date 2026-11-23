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

**Modelo:** Sonnet 5.5
**Herramienta:** Claude Code

```
produce una auditoría priorizada de la feature de comentarios y déjala escrita en docs/auditoria/comentarios.md. Una feature es una funcionalidad del blog de punta a punta: sus rutas en la API, su tabla en la base de datos y lo que la web enseña de ella.
Quiero que te centres en la funcionalidad y dominio de comentarios

Y el orden, : primero la precedencia (lo que abarata o desbloquea a otro hallazgo va antes), y dentro de cada nivel, el daño dividido por el radio. Si un hallazgo tiene el daño más alto y la precedencia más baja a la vez, se parte en dos filas, una contención que corta el daño hoy y el arreglo de fondo que espera su turno.

El formato no es negociable: una fila por hallazgo, con estas seis casillas.

Qué es. Una frase. Si necesitas tres, es que hay dos hallazgos.

Dónde está, con la línea que lo demuestra. Sin esta casilla, la fila no se escribe.

Qué se rompe, y quién lo nota. El efecto concreto y a quién le llega. "Nadie todavía" es una respuesta válida y buena.

Cuántos sitios hay que tocar. Contados, no estimados.

¿Se deshace con un revert? Sí o no, y si es que no, por qué no.

¿Bloquea o desbloquea a otro hallazgo? Nombrando al otro. "A ninguno" es la respuesta más frecuente.
```

**Qué salió:** (opcional, una línea) funcionó a la primera / tuve que insistir / me inventó una ruta que no existe.

## Prompt 2

**Modelo:** Sonnet 5.5
**Herramienta:** Claude Code

```
Estas seguto que quitar el email de la query, no impacta en otras funcionalidades?
Y que sugieres con H1 en detalle?
```

## Prompt 3

**Modelo:** Sonnet 5.5
**Herramienta:** Claude Code

```
H2 como puede borrar si no tiene la clave de moderacion?
```
