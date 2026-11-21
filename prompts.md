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
En esta sesión vamos a hacer una auditoria **parcial** de un código heredado (legacy).
Los prompts del chat que yo vaya escribiendo los guardas en secuencia en el fichero prompts.md siguiendo las instrucciones que leerás en el propio fichero.
No leas el fichero README.md y no lo uses en tu contexto
No debes cambiar el código existente del programa, ni la estructura de la base de datos.
Para empezar necesito que analices y me expliques qué hace la funcionalidad (capability) de "comentarios" y solo eso por ahora lo guardas por escrito en el fichero "comentarios.md" en la carpeta /temp/. Luego en sucesivos prompts ya te pediré hacer una auditoria priorizada siguiendo unas reglas.
```

**Qué salió:** funcionó a la primera; `/temp/` se interpretó como `temp/` en la raíz del repo.


## Prompt 2

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
Ahora vas a realizar la auditoria de **solo** esta funcionalidad (feature) de "comentarios" con un agente.
Considera la funcionalidad de los comentarios del blog de punta a punta: sus rutas en la API, su tabla en la base de datos y lo que la web enseña de ella. (crear, editar, borrar y listar comentarios, en esas tres capas).
Tu objetivo es buscar hallazgos contrastados con el código. un hallazgo sin la linea que lo demuestre no es hallazgo.

El formato que debes usar es este: una fila por hallazgo, con estas seis casillas.
1. Qué es. Una frase. Si necesitas tres, es que hay dos hallazgos.
2. Dónde está, con la línea que lo demuestra. Sin esta casilla, la fila no se escribe.
3. Qué se rompe, y quién lo nota. El efecto concreto y a quién le llega. "Nadie todavía" es una respuesta válida y buena.
4. Cuántos sitios hay que tocar. Contados, no estimados.
5. ¿Se deshace con un revert? Sí o no, y si es que no, por qué no.
6. ¿Bloquea o desbloquea a otro hallazgo? Nombrando al otro. "A ninguno" es la respuesta más frecuente.

Deja el resultado escrito en docs/auditoria/comentarios.md, no en el chat.
Recuerda no modificar código, esto es una auditoria.
```

**Qué salió:** un agente de solo lectura sacó 33 hallazgos con su línea; contrastó las citas a mano y quitó una afirmación que no se había comprobado.

## Prompt 3

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
en el documento de comentarios dentro de auditoria, añade una sección que se titule Parte B: las tres líneas. y pon estos 3 puntos para que yo los complete (no tu). 

Cuántos hallazgos devolvió el agente, y cuántos comprobaste tú abriendo el archivo. Los dos números, tal cual salieron. No los redondees ni los expliques.

El hallazgo del que no supiste decidir la prioridad, y en una frase, por qué. Lo que interesa es qué dos cosas se contradecían: lo grave contra lo que bloquea, lo urgente contra lo caro, lo que te pedían contra lo que veías.

Algo que el agente afirmó y no pudiste verificar. Ojo al matiz: no es lo que dijo mal. Es lo que no pudiste comprobar ni a favor ni en contra, y te tocó dejar en el aire
```

**Qué salió:** funcionó a la primera; sección añadida con los tres puntos vacíos para rellenar a mano.

## Prompt 4

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
segun la lista de hallazgos muestrame una lista ordenada segun dependencias, es decir si A corrige a B, B va primero.
```

**Qué salió:** orden por niveles mostrado en el chat, sacado de la columna «Bloquea / desbloquea».

## Prompt 5

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
guarda esta lista en temp/lista_hallazgos.md
```

**Qué salió:** funcionó a la primera.

