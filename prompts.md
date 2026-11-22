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
No hagas commits, push, PRs ni cambies de rama hasta que yo te lo pida. No modifiques código ni uses /refactor.

Audita SOLO la feature de comentarios del blog (crear, editar, borrar y listar comentarios), en sus tres capas: rutas de la API, tabla en la base de datos y lo que la web muestra. No audites posts, autores, categorías ni etiquetas.

Dame una lista numerada de hallazgos (bugs, huecos de seguridad, inconsistencias entre capas). Para cada uno: qué es en una frase, y archivo:línea exacta que lo demuestra. No los ordenes por prioridad todavía y no propongas arreglos.
```

**Qué salió:** 33 hallazgos con archivo:línea, en 3 capas (16 API, 6 BD, 11 web). Respetó las restricciones: no tocó archivos ni git. Admitió que el conteo del hallazgo 11 lo sacó de la semilla sin arrancar la app. Prompt redactado con ayuda de Claude (chat).


## Prompt 2

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
Escribe en docs/auditoria/comentarios.md la tabla de auditoría SOLO con estos hallazgos, que he comprobado abriendo el código: #1, #2, #3, #5, #6. Añade también #4 indicando "mecanismo confirmado en routes.ts:330; efecto (dos comentarios simultáneos) sin verificar".

Usa la tabla que ya existe en el archivo, una fila por hallazgo, con estas seis casillas:
- Qué es (una frase)
- Dónde, con archivo:línea que lo demuestra
- Qué se rompe y quién lo nota ("nadie todavía" es válido)
- Cuántos sitios hay que tocar: contados en el código, no estimados
- ¿Se deshace con un revert? Sí/No, y si no, por qué
- ¿Bloquea o desbloquea a otro hallazgo? Nombrándolo ("a ninguno" es válido)

Ordénalos así: primero la precedencia (lo que abarata o desbloquea a otro va antes); dentro de cada nivel, daño dividido por radio. Si un hallazgo tiene el daño más alto y la precedencia más baja, pártelo en dos filas: una contención que corta el daño hoy y el arreglo de fondo después.
Las propuestas que nadie ha medido van fuera de la tabla, en "Propuestas sin medir", con lo que habría que medir.

No toques código, ni ramas, ni git. Explícame en 3 líneas por qué elegiste ese orden.
```

**Qué salió:** escribió la tabla con 7 filas (#5 partido en contención y arreglo de fondo) y explicó el orden. Se corrigió: la clave de moderación NO firma sesiones, eso solo lo decía un comentario. Al escribir, borró mi línea de hallazgos comprobados y tuve que volver a ponerla. Prompt redactado con ayuda de Claude (chat).