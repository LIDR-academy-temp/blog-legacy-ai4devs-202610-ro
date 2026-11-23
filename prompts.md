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

# Prompts

Registrados automáticamente por el hook personal `~/.claude/hooks/log-prompt.py`.

---

## Prompt 1

**Modelo:** claude-opus-5-5
**Herramienta:** Claude Code

```
Haz una auditoría del código, mirándo sólo la funcionalidad de "comentarios". Se tiene que poder crear, editar, borrar y listar comentarios. Analiza
  todas las capas. Con tu análisis, genera una tabla en la que haya una fila por hallazgo y que tenga 6 columnas:


<pasted_content id="2f6e">
Qué es. Una frase. Si necesitas tres, es que hay dos hallazgos.

Dónde está, con la línea que lo demuestra. Sin esta casilla, la fila no se escribe.

Qué se rompe, y quién lo nota. El efecto concreto y a quién le llega. "Nadie todavía" es una respuesta válida y buena.

Cuántos sitios hay que tocar. Contados, no estimados.

¿Se deshace con un revert? Sí o no, y si es que no, por qué no.

¿Bloquea o desbloquea a otro hallazgo? Nombrando al otro. "A ninguno" es la respuesta más frecuente
</pasted_content id="2f6e">

Los hallazgos quiero que los ordenes por prioridad. Para establecer la prioridad te indico los criterios de mayor a menor:
-  la precedencia (lo que abarata o desbloquea a otro hallazgo va antes), y dentro de cada nivel, el daño(¿qué se rompe, y quién lo nota?) dividido por el radio (¿Cuántos sitios hay que tocar a la vez?
Si un hallazgo tiene el daño más alto y la precedencia más baja a la vez, se parte en dos filas, una contención que corta el daño hoy y el arreglo de fondo que espera su turno
```

**Respuesta:**

He auditado los comentarios en todas las capas: rutas, migración, semilla, gestor de errores, tests y los dos componentes del front. Salen **26 hallazgos**. Los tres más graves son estos:

- **Ningún endpoint aprueba comentarios.** Todo comentario se guarda como `pendiente` y no hay forma de cambiarlo, así que ninguno nuevo se llega a ver nunca.
- **Los correos de quien comenta salen en la API pública.** Ese correo es justo lo único que piden el borrado y la edición, así que cualquiera puede borrar el comentario de otro.
- **Los recuentos no cuadran con lo que se ve.** Lo he comprobado contra la base sembrada: el post 1 anuncia 4 comentarios y enseña 2; el post 6 anuncia 1 y no enseña ninguno.

**Cómo leer el orden:** el nivel 1 es lo que desbloquea o abarata a otro hallazgo, en orden de dependencia. Dentro de cada nivel manda daño ÷ radio. Las filas de contención (#1, #2, #3, #5) son los hallazgos graves que he partido en dos: la contención sube al nivel 1 y su arreglo de fondo (#22, #17, #16) espera en el nivel 2. Las rutas son relativas a `backend/` salvo las de `frontend/`.

| # | Qué es | Dónde | Qué se rompe y quién lo nota | Sitios | ¿Revert? | ¿Bloquea / desbloquea? |
|---|---|---|---|---|---|---|
| **N1** | | | | | | |
| 1 | **Contención:** el detalle del post devuelve los comentarios con `select *`, así que sale el correo de quien comenta. | `start/routes.ts:153` → `:171` | Cualquier visitante se lleva los correos de los comentaristas, y ese correo basta para borrar o editar (#9). | 1 | Sí | Abarata #9. Fondo: #22 |
| 2 | **Contención:** el listado cuenta `<> 'pendiente'` y el detalle solo enseña `= 'aprobado'`. | `routes.ts:75-76`, `:223` vs `:153` | Lectores de portada y relacionados: el post 1 anuncia 4 y enseña 2. Incumple la regla 5. | 2 | Sí | Abarata #17 |
| 3 | **Contención:** la clave de moderación está escrita en el código. | `routes.ts:25`, usada en `:351` | Quien lea el repo borra cualquier comentario sin dejar rastro. Nadie lo nota hoy. | 4 | **No:** ya está en el historial de git; hay que rotarla | Desbloquea #5. Fondo: #16 |
| 4 | Solo hay un test de comentarios: el camino feliz del POST, comprobando un campo. | `tests/functional/comentarios.spec.ts:41-42` | Nadie todavía, pero cualquier refactor va a ciegas. | 1 | Sí | Desbloquea #5, #6 y todo refactor |
| 5 | **Contención:** ningún endpoint cambia el `estado`, así que nada se aprueba nunca. | `routes.ts:326`; ningún `update … estado` en `:295-393` | Todo el que comenta: su comentario no aparece nunca. | 5 | Sí | Lo bloquean #3 y #4. Abarata #6. Fondo: #16 |
| 6 | Todo vive en `routes.ts`: sin controlador, validador, servicio, repositorio, transformer ni modelo. | `routes.ts:295-393`; `app/models/` solo tiene `user.ts` | Nadie directo. Las reglas se duplican y se separan (#7). | 8 | Sí | Lo bloquea #4. Desbloquea #7, #8, #10, #12, #13, #22 |
| 7 | Borrar y editar tienen dos reglas de autorización distintas. | `routes.ts:353` vs `:379` | El autor puede borrar pero no editar si cambia mayúsculas; el moderador no puede editar. | 2 | Sí | Lo bloquea #6. Desbloquea #9, #16 |
| 8 | Cuatro formatos de error de validación, y uno es un **200** `{ok:false}`. | `routes.ts:311`, `:314`, `:317`, `:320`, `:385`; `handler.ts:15-17` sin tocar | Los clientes de la API toman el fallo por éxito. La web muestra un mensaje genérico (`Comentarios.jsx:44`). | 7 | Sí (avisar del cambio de contrato) | Lo bloquean #4 y #6. Desbloquea #15, #18, #19, #23 |
| 9 | Basta con conocer el correo para pasar por el autor. | `routes.ts:350-353`, `:378-379` | Se puede borrar (físicamente) o editar lo de otro. Los correos del equipo además salen en `posts` (`:74`). | 5 | **No:** los tokens ya entregados dejarían de valer | Lo bloquea #7. #1 lo abarata. Desbloquea #11 |
| 10 | No hay endpoint de listado; el front vuelve a pedir el post entero. | `Comentarios.jsx:19` (repite `DetallePost.jsx:34`) | Dos peticiones por visita. Nadie más lo nota. | 3 | Sí | Lo bloquea #6. Abarata #14 |
| **N2** | | | | | | |
| 11 | La web no deja editar ni borrar comentarios. | `Comentarios.jsx:28-48` (solo hay POST) | Quien comenta no puede corregir ni retirar lo que escribió. | 1 | Sí | Lo bloquea #9 |
| 12 | Al crear se devuelve «el último de la tabla», no el que se acaba de insertar. | `routes.ts:329-331` | Si dos comentan a la vez, uno puede recibir el comentario y el correo del otro. | 1 | Sí | A ninguno |
| 13 | Borrar un padre deja respuestas huérfanas. | `routes.ts:359`; sin FK | La respuesta deja de verse pero sigue contando en el listado. | 1 | **No:** el borrado es físico | A ninguno |
| 14 | El front lleva dos recuentos, y uno suma 1 al enviar un pendiente. | `Comentarios.jsx:39`, `DetallePost.jsx:38` | Quien comenta ve «3» en la sección, «2» en la cabecera y 2 comentarios. | 2 | Sí | A ninguno |
| 15 | Al editar no se exige la longitud mínima de 5 que sí se exige al crear. | `routes.ts:384` vs `:319` | Un comentario puede quedar en «a». Nadie todavía. | 1 | Sí | A ninguno |
| 16 | **Fondo #3/#5:** moderar es tener una clave compartida, no un rol autenticado. | `routes.ts:349-351`; `users` no tiene rol | No se sabe quién moderó qué. Al rotar la clave caen todos los moderadores. | 3 | **No:** migración en `users` | A ninguno |
| 17 | **Fondo #2:** `estado` es texto libre (`'aprobado'`, `'Aprobado'`, `'ok'`). | migración `…05_comentarios:18` (sin CHECK) | Los comentarios 3, 4 y 9 están aprobados pero no se ven. | 4 | **No:** se pierde el valor original | A ninguno |
| 18 | El correo solo se valida con `@`. | `routes.ts:316` | Los moderadores reciben correos basura. | 1 | Sí | A ninguno |
| 19 | Se acepta un cuerpo de solo espacios, sin longitud máxima. | `routes.ts:319`, `:305-307` | Spam. Nadie todavía. | 1 | Sí | A ninguno |
| 20 | La fecha va en UTC sin zona y se recorta a 10 caracteres. | `routes.ts:323`, `Comentarios.jsx:59`, `:68` | Un comentario de las 00:30 en Madrid sale con fecha del día anterior. | 3 | Sí | A ninguno |
| 21 | Solo se edita lo pendiente, una regla sin documentar. | `routes.ts:379` | Nadie todavía. Hay que decidir si es lo que se quiere. | 1 | Sí | A ninguno |
| 22 | **Fondo #1:** ninguna respuesta pasa por un transformer. | `routes.ts:171`, `:333`, `:392` | Salen `estado`, `post_id` y el correo (con #12, de otra persona). | 4 | Sí | A ninguno |
| 23 | `padre_id` entra sin validar. | `routes.ts:308` → `:326` | Nadie todavía (la web no lo envía). Esas respuestas no se verían nunca. | 2 | Sí | A ninguno |
| 24 | Solo se pintan respuestas de primer nivel, con `key={index}`. | `Comentarios.jsx:50`, `:63-64` | Nadie todavía. | 2 | Sí | A ninguno |
| 25 | `post_id` y `padre_id` no tienen FK ni índice. | migración `…05:14`, `:20` | Nadie todavía. Cada detalle recorre la tabla entera. | 1 | **No**, si antes hay que borrar huérfanos | A ninguno |
| 26 | La URL de la API está escrita a mano, con 6 copias. | `Comentarios.jsx:4` y 5 archivos más | Nadie todavía. Si cambia el puerto, no cargan. | 6 | Sí | A ninguno |

En el documento cada fila trae más detalle: los sitios contados uno a uno y por qué cada «No» no se deshace con un revert. Está guardado en `docs/auditoria/comentarios.md`, sin commitear; no he cambiado nada del código.

Si quieres pasárselo a alguien, puedo publicarlo como página.
