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

**Modelo:** claude-opus-5-5
**Herramienta:** Claude Code

```
revisa el archivo prompts.md y sigue las instrucciones para loguear todos los prompts
```

**Qué salió:** funcionó a la primera: montó el hook `.claude/hooks/log-prompt.mjs` que registra cada prompt solo.

## Prompt 2

**Modelo:** claude-opus-5-5
**Herramienta:** Claude Code

```
produce una auditoria priorizada de la feature de comentarios y escribela en docs/auditoria/comentarios.md. El order de prioridad es el siguiente: primero la precedencia - lo que abarata o desbloquea algo va antes, y dentro de cada nivel, el daño dividio por el radio. Si un hallazgo tiene el daño mas alto y la precendecia mas bbaja a la vez, separalo en dos fila: una de contencion para corregir el daño de inmediato y el arreglo a fondo mas tarde. el formato debera ser una fila por hallazgo, con estas columnas:  que es (en una frase), donde esta (con la referencia al numero de linea - si no hay no se escribe la fila), que se rompe y quien lo nota (efecto concreto y donde se ve), cuantos lugares en el codigo hay que tocar (contados), se puede hacer revert? (si o no, y si no, por que no?), bloquea o desbloquea otro hallazgo? (nombrando el otro hallazgo si existe, o "ninguno" si es el caso)
```

**Qué salió:** 
