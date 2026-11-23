# Prompts

Aquí van **todos los prompts que lanzaste** para hacer el ejercicio, en el orden en que los
lanzaste, con el modelo y la herramienta de cada uno.

Esto no es papeleo. Lo que se revisa es **cómo pediste las cosas**, no solo lo que salió: un
resultado flojo con un prompt bueno y un resultado flojo con un prompt vago necesitan feedback
distinto, y sin este archivo no se distinguen.

## Prompt 1

**Modelo:** Opus 5.5
**Herramienta:** Claude Code (app de escritorio), en otra sesión

```
Ayudame a hacer el ejercicio sin compilar de momento. Explicame paso a paso.
```

**Qué salió:** me explicó los pasos y me dio una base de prompt para la auditoría, que usé casi tal cual en el Prompt 2.

## Prompt 2

**Modelo:** Opus 5.5
**Herramienta:** Claude Code (extensión de VS Code)

```
Audita SOLO la feature de comentarios de este blog (crear, editar, borrar y listar),
en sus tres capas: rutas de la API en backend/start/routes.ts, la tabla de comentarios
en backend/database/migrations/ y lo que muestra/envía el frontend en frontend/src/.
No audites posts, autores, categorías ni etiquetas.

Escribe el resultado en docs/auditoria/comentarios.md como UNA tabla, una fila por
hallazgo, con estas 6 columnas:
1. Qué es (una frase)
2. Dónde está: archivo:línea y la línea de código que lo demuestra (si no hay línea, no hay fila)
3. Qué se rompe y quién lo nota ("nadie todavía" vale)
4. Cuántos sitios hay que tocar (contados, no estimados)
5. ¿Se deshace con un revert? Sí/No y por qué no
6. ¿Bloquea o desbloquea a otro hallazgo? Nombrándolo, o "a ninguno"

Orden: primero lo que desbloquea o abarata a otros; dentro de cada nivel, daño/radio.
Si un hallazgo es el más grave pero tiene poca precedencia, pártelo en dos filas:
contención hoy + arreglo de fondo después.
Las propuestas que nadie ha medido van FUERA de la tabla, con qué habría que medir.
No modifiques código, solo escribe ese archivo.
```

**Qué salió:**  funcionó a la primera: escribió docs/auditoria/comentarios.md con 14 hallazgos y una sección fuera de la tabla.
