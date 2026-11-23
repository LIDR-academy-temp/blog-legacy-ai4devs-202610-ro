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
A partir de ahora en está sesión, todos los prompts y su salida añademelos al fichero prompts.md (Es una plantilla)
```

**Qué salió:** Sustituí el ejemplo de la plantilla por esta entrada; a partir de aquí cada prompt de la sesión se añade como `## Prompt N` con su resultado.

## Prompt 2

**Modelo:** Opus 5.5 (orquestador) + subagente general-purpose (Opus 5.5)
**Herramienta:** Claude Code

```
Con un agente, produce la auditoría priorizada de la feature de comentarios y déjala escrita en docs/auditoria/comentarios.md, no en el chat.

Solo esa feature. No el sistema entero, aunque quepa y aunque el agente se ofrezca.

El formato no es negociable: una fila por hallazgo, con estas seis casillas.

Qué es. Una frase. Si necesitas tres, es que hay dos hallazgos.
Dónde está, con la línea que lo demuestra. Sin esta casilla, la fila no se escribe.
Qué se rompe, y quién lo nota. El efecto concreto y a quién le llega. "Nadie todavía" es una respuesta válida y buena.
Cuántos sitios hay que tocar. Contados, no estimados.
¿Se deshace con un revert? Sí o no, y si es que no, por qué no.
¿Bloquea o desbloquea a otro hallazgo? Nombrando al otro. "A ninguno" es la respuesta más frecuente.
```

**Qué salió:** El subagente escribió `docs/auditoria/comentarios.md` con 18 hallazgos (C1–C18) en una tabla priorizada; arrancó la API en el 3401 para confirmar la mayoría con curl y después restauró la base. Las 21 referencias `archivo:línea` de la tabla se cotejaron con el código y todas coinciden. Sin verificar: la carrera de C8, lo que pinta la web (C4, C9, C10, C12; solo se leyó el JSX) y la columna de revert, que es razonamiento. La Parte B queda en `_pendiente_` para que la rellene el humano.
