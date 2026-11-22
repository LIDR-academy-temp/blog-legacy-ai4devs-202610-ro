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
 git checkout -b auditoria-fmj
```

**Qué salió:** creó la rama `auditoria-fmj` a partir de `s6/start`.

---

## Prompt 2

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
antes de auditar ni hacer nada. primero ve apuntando los prompst en el archivos promps.md siguiendo las instrucciones. no hagas nada sin que lo pide primero, puede sugerirmelo o dar pistas pero no hagas nada por inciativa propia
```

**Qué salió:** borró el ejemplo de la plantilla y apuntó los prompts 1 y 2.

---

## Prompt 3

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
empieza con la auditoria
```

**Qué salió:** leyó rutas, migración, semilla, test y front de comentarios; dejó 17 hallazgos con las seis casillas en `docs/auditoria/comentarios.md` y la Parte B con huecos para rellenar a mano.

---

## Prompt 4

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
borra lo auditado hasta ahora y vamos a centrarnos solo en aditar la parte de comentarios. no todo el sistema
```

**Qué salió:** borró la auditoría y preguntó el alcance (elegí «Estricto + recuentos»). La rehízo con 16 hallazgos: quitó la fila de arquitectura general de `routes.ts` y dejó la Parte B para rellenar a mano.

---

## Prompt 5

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
vamos a revisar, y ordenar la lista. cuentame el primer caso
```

**Qué salió:** explicó H1 con sus líneas y comprobó que la clave está en git desde el primer commit y que solo se usa en el DELETE. Planteó si H1 debía ir primero, cuando no incumple ninguna regla escrita.

---

## Prompt 6

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
primero incumplimiento de la norma y luego el fallo de seguridad
```

**Qué salió:** dejó el criterio anotado y un orden provisional (H2, H6, H3, H1, H4, el resto), sin tocar aún el archivo. Explicó H2 y lo comprobó lanzando la consulta contra la base local.

---

## Prompt 7

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
veo que indicas la linea pero no la clase, vamos a ser estrictos con el lugar donde se encuentra el problema
```

**Qué salió:** explicó que `routes.ts` no tiene clases y propuso un formato «archivo › contenedor › líneas». Lo rechacé en el prompt siguiente.

---

## Prompt 8

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
no, vamos a utilizar el formato siguiente 


El formato no es negociable: una fila por hallazgo, con estas seis casillas.

Qué es. Una frase. Si necesitas tres, es que hay dos hallazgos.

Dónde está, con la línea que lo demuestra. Sin esta casilla, la fila no se escribe.

Qué se rompe, y quién lo nota. El efecto concreto y a quién le llega. "Nadie todavía" es una respuesta válida y buena.

Cuántos sitios hay que tocar. Contados, no estimados.

¿Se deshace con un revert? Sí o no, y si es que no, por qué no.

¿Bloquea o desbloquea a otro hallazgo? Nombrando al otro. "A ninguno" es la respuesta más frecuente.
```

**Qué salió:** repasó la tabla contra el formato y encontró casillas que no lo cumplían (más de una frase, sitios estimados, revert condicional, «relacionado con»). Enseñó H2 ya corregido.

---

## Prompt 9

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
hazlo ya y asi tambien puedo ir mirando en el archivo
```

**Qué salió:** reescribió `docs/auditoria/comentarios.md` con el formato estricto y reordenado (primero las reglas incumplidas, después la seguridad y luego el resto), renumerado del H1 al H16.

---

## Prompt 10

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
ve contabilizando los revisados para rellenar luego la parte B del fichero. revisado el nuevo h1 pasa al siguiente
```

**Qué salió:** llevó la cuenta (1 de 16) y explicó H2 con los recuentos sacados de la base. Señaló que «bloqueado por H6» es discutible.

---

## Prompt 11

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
dime lo que tiene H6 para decidir si lo bloquea o no
```

**Qué salió:** enseñó H6 con los 3 comentarios afectados sacados de la base y los comentarios del código que dicen que son aprobados mal escritos. Comprobó en ambos órdenes que H6 no bloquea H2 y recomendó poner «A ninguno».

---

## Prompt 12

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
cambia a ninguno, marcalo como revisado, y de momento sube h6 al 3 puesto de la lista
```

**Qué salió:** puso «A ninguno» en H2 y en H6, subió H6 al puesto 3 (el antiguo H3 pasa a H4, H4 a H5 y H5 a H6) y llevó la cuenta a 2 de 16.

---

## Prompt 13

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
ya he revisado los archivos, marcalo como revisado y pasamos al siguiente
```

**Qué salió:** llevó la cuenta a 3 y explicó H4 releyendo el código. Avisó de que la concurrencia no la había verificado y de que, leída al pie de la letra, R6 no cubre este caso.

---

## Prompt 14

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
bajalo a seguridad y añade en la parte b este caso, indica que si he visto el codigo pero es mas dificil de verificar y estar seguros del problema
```

**Qué salió:** bajó el hallazgo del POST al final del bloque de seguridad (pasa a H6; la clave queda en H4 y el borrado con correo en H5) y lo escribió en la línea 3 de la Parte B. Cuenta: 4 de 16.

---

## Prompt 15

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
marcalo como revisado e indica este caso como ejemplo de no estar seguro de la prioridad el punto 2 de la parte B
```

**Qué salió:** escribió H4 en la línea 2 de la Parte B, con el motivo: es el más grave, pero no incumple ninguna regla escrita. Cuenta: 5 de 16.

---

## Prompt 16

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
mustrame lo que has puesto en la parte B, vamos a revisarlo primero y cuando este seguro hacer la entrega. ya he consumido el tiempo
```

**Qué salió:** enseñó la Parte B tal como estaba y señaló lo pendiente: el número de la línea 1, la nota de aviso y la redacción de las líneas 2 y 3.

---

## Prompt 17

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
si lo revisé, deja 16 / 6
```

**Qué salió:** puso `16 / 6` en la línea 1 de la Parte B.

---

## Prompt 18

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
me parece bien como esta, no modifiques nada. puedes realizar la entrega siguiendo los pasos que me has dicho
```

**Qué salió:** hizo el commit solo con `docs/auditoria` y `prompts.md`, empujó la rama `auditoria-fmj` a mi fork y abrió el pull request contra el repositorio del curso.
