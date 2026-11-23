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

**Modelo:** Opus 5.5 · API Usage Billing
**Herramienta:** Claude Code v2.1.325

```
claude resume los ultimos 5 commits

```

**Qué salió:** please Run /login.

despues del codigo de mi cuenta meda 3 commits


## Prompt 2

**Modelo:** Opus 5.5 · API Usage Billing
**Herramienta:** Claude Code v2.1.325

```
busca en que archivos aparece la palabra comment

```

**Qué salió:** La palabra "comment" solo sale en un archivo de dependencias, backend/package-lock.json. Son 3 líneas, todas del paquete npm strip-json-comments (líneas 5945, 6848 y 6850). Ningún archivo del código usa esa palabra. tambien busco coment en español y ahi si encontro mas.



## Prompt 3

**Modelo:** Opus 5.5 · API Usage Billing
**Herramienta:** Claude Code v2.1.325

```
crea un temporizador visual interactivo de 45 minutos que empieze a correr ahora y avisame cuando cumpla cada 10
  minutos y cuando termine

```

**Qué salió:** creo el temporizador

## Prompt 4

**Modelo:** Opus 5.5 · API Usage Billing
**Herramienta:** Claude Code v2.1.325

```
crea un archivo comentarios.md en docs/auditoria

```

**Qué salió:** me creo el archivo con la tabla como dice el readme pero sin hallazgos aun.

 
## Prompt 5

**Modelo:** Opus 5.5 · API Usage Billing
**Herramienta:** Claude Code v2.1.325

```
Audita solo la feature de comentarios (crear,editar,borrar,listar) en backend, base de datos y frontend. Para cada
  hallazgo da: que es en una frase, archivo, linea que lo demuestra, que se rompe y quien lo nota, cuantos sitios hay
  que tocar (contados), si se deshace con un revert y si bloquea o desbloquea a otro hallazgo. No audites otras
  features. No modifiques codigo. esta informacion ponla en la tabla que creaste en el archivo comentarios.md. Sigue
  el siguiente orden Primero la precendecia: lo que desbloquea o abarata otro hallazgo va antes. Segundo: dentro de
  cada nivel, ordena por daño entre radio. Tercero: Si un hallazgo tiene el daño mas alto pero la precedencia mas
  baja, partelo en dos filas: una contencion inmediata y el arreglo de fondo. Cuarto: Las propuestas que nadie ha
  medido van fuera de la tabla, con lo que habria que medir al lado.

```

**Qué salió:** la tabla tiene 12 filas pero son 11 hallazgos, porque uno está partido en dos. Aparte hay 5 propuestas sin medir, fuera de la tabla.


