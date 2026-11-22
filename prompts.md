# Prompts

## Prompt 1

**Modelo:** Claude Opus 5.5 (`claude-opus-5-5`)
**Herramienta:** Claude Code

```
Definicion de Feature: Funcionalidad del blog de punta a punta o de inicio a fin. Esto se define en 3 capas:
- Sus rutas en la API.
- Sus tablas en la base de datos.
- Lo que se enseña en la web.

Una vez definido el concepto de feature. En este programa hay que destacar que existen 5.
1) Posts
2) Comentarios
3) Autores
4) Categorias
5) Etiquetas

Produce una auditoria priorizada de la feature de "Comentarios" y centrate solo en ella. Dejala escrita en docs/auditoria/comentarios.md, y no en el chat.
La feature de comentarios que se reduce en (crear, editar, borrar y listar en estas 3 capas).

El orden de priorizacion tiene que ser el siguiente: Precedencia (lo que abarata o desbloquea a otro hallazgo va antes) y dentro de cada nivel, el "daño dividido por el radio". Si un hallazgo tiene el daño mas alto y la precedencia mas baja a la vez, se parte en 2 filas. La propuestas que nadie ha medido todavia van fuera de la tabla, especificando lo que habria que medir.

El formato de salida tiene que ser el siguiente: Una fila por hallazgo y con estas 6 columnas.
Columna 1: "Que es". Una frase, si necesitas 3 es que hay dos hallazgos.
Columna 2: "Donde esta". Con la linea que lo demuestra. Si no se puede poner esta informacion no tienes que mostrar esta fila.
Columna 3: "Que se rompe y quien lo nota". El efecto concreto y a quien le llega. Puede pasar que no se informe a nadie.
columna 4: "Cuantos sitios hay que tocar". Sin estimaciones. Un total real.
Columna 5: "¿Se deshace con un revert?". Solo es posible poner estas opciones o "Si" o "No" y en caso que "No", explica el motivo.
Columna 6: "¿Bloquea o desbloquea a otro hallazgo?". En caso que si, nombra el otro hallazgo, en caso que no, indicalo con "A ninguno".

Una vez escrita esta auditoria. No hagas ningun cambio en el codigo.
Antes de hacer cualquier accion. Si tienes dudas sobre esta prompt, haz las preguntas que creas necesarias.
```

**Respuesta (resumen):** Sin preguntas previas, leyó `routes.ts`, la migración y la semilla de comentarios, el test existente y los componentes de la web. Escribió `docs/auditoria/comentarios.md` con 21 filas en 6 niveles de precedencia (C1–C20, con C2 partido en C2a contención / C2b arreglo de fondo) y 4 propuestas sin medir fuera de la tabla. No tocó código.

**Qué salió:** funcionó a la primera; las preguntas siguientes revelaron imprecisiones en algunas filas (C6, C10).

---

## Prompt 2

**Modelo:** Claude Opus 5.5 (`claude-opus-5-5`)
**Herramienta:** Claude Code

```
Sobre el hallazgo "C3", he comprobado el codigo en "route.ts" y no encuentro el problema que comentas que alguien que conozca el email puede borrar un comentario.
Tendria que conocer tambien el valor de "SECRETO-SESION" y ponerlo en el header. ¿es correcto?
```

**Respuesta:** No. La condición de `backend/start/routes.ts:355` es `if (!esModerador && !esAutor)`: el 403 solo salta si fallan las dos, así que basta con una (es un «o»). Con solo el correo en el cuerpo y sin cabecera, `esAutor` es `true` y se borra (C3); con solo la cabecera, se borra cualquier comentario (C4, otro hallazgo). El correo no se verifica y además es público por C2a (`routes.ts:153`, `select *`). Deducido leyendo el código, sin ejecutarlo.

**Qué salió:** aclaró la lectura del código; no hubo que cambiar la fila.

---

## Prompt 3

**Modelo:** Claude Opus 5.5 (`claude-opus-5-5`)
**Herramienta:** Claude Code

```
En el Hallazgo de nivel 2 - c6, indicas: 'El detalle solo muestra estado = "Aprobado"',que detalle hablas y donde esta?
```

**Respuesta:** «El detalle» es el endpoint `GET /posts/:slug` (`routes.ts:109`), cuya consulta de comentarios en `routes.ts:152-153` filtra `estado = 'aprobado'` en minúscula; SQLite distingue mayúsculas con `=`, así que las filas `'Aprobado'` (`blog_seeder.ts:510`, `:575`) y `'ok'` (`:522`) no salen. En la web lo consumen `DetallePost.jsx:34` y `Comentarios.jsx:19`. Reconoció un fallo en la fila: «Dónde está» solo cita la semilla y no `routes.ts:153`, y «los lectores no lo notan» es discutible. Ofreció corregir la fila sin hacerlo.

**Qué salió:** destapó un error de la auditoría (fila C6 incompleta).

---

## Prompt 4

**Modelo:** Claude Opus 5.5 (`claude-opus-5-5`)
**Herramienta:** Claude Code

```
No acabo de entender C10.
```

**Respuesta:** C10 no es un fallo visible para el usuario sino de dónde está escrito el código: crear/editar/borrar viven enteros en `routes.ts:295-393`, mezclando validación (`:310-321`, `:384`), regla de negocio (`:351-355`, `:379`), SQL (`db.rawQuery` en 8 líneas) y forma de la respuesta (`:333`, `:392`). Al no haber un sitio común, las reglas divergen: C12, C13, C11 y C2b son consecuencia. Va en nivel 3 porque arreglar esos hallazgos sin capas es parchear copias; está bloqueado por C1. Total de 8 archivos: `routes.ts`, 6 nuevos (controlador, validador, servicio, repositorio, modelo, transformer) y `package.json` (alias `#repositories/*`). Ofreció reescribir la columna 3 de C10.

**Qué salió:** explicación aceptada; la fila no se modificó.

---

## Prompt 5

**Modelo:** Claude Opus 5.5 (`claude-opus-5-5`)
**Herramienta:** Claude Code

```
Donde esta esta regla en Claude de separar loas cuatro trabajos?
```

**Respuesta:** En `CLAUDE.md`, sección «Arquitectura objetivo (para cada endpoint que toques)» (cadena ruta → controlador → validador → servicio → repositorio → modelo → transformer) y «Reglas que no se negocian». Precisó que «los cuatro trabajos» era su propio resumen, no una expresión del `CLAUDE.md`.

**Qué salió:** respondió con la cita literal y corrigió su propia formulación.

---

## Prompt 6

**Modelo:** Claude Opus 5.5 (`claude-opus-5-5`)
**Herramienta:** Claude Code

```
Revisa las instrucciones del fichero "propmpts.md" y vuelca toda la conversacion de esta sesion en este fichero. Borra el contenido anterior de este fichero
```

**Respuesta:** Leyó `prompts.md`, borró su contenido (instrucciones y ejemplo) y escribió este archivo siguiendo su formato: un apartado por prompt, pegado tal cual, con modelo, herramienta y qué salió, más un resumen de cada respuesta.
