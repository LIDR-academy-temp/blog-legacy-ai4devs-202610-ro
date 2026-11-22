# Auditoría priorizada: comentarios

Rebanada: la feature de comentarios. Las rutas `GET /posts/:slug` (parte de comentarios), `POST /posts/:slug/comentarios`, `DELETE /comentarios/:id` y `PATCH /comentarios/:id`, el recuento en los listados (`backend/start/routes.ts`), la migración de `comentarios` y `frontend/src/componentes/Comentarios.jsx` y `DetallePost.jsx`. Vara de medir: las reglas de negocio 2, 5 y 6 de `CLAUDE.md`.

Cada fila marcada «probado» se ha comprobado además contra la API levantada en local (`make api`, base sembrada con `make db` y resembrada después). Suite de partida: `make test` → 1 test, en verde.

Orden: primero la precedencia (lo que desbloquea a otro va antes); dentro de cada nivel, daño entre radio.

## Nivel 1 · No dependen de nada, o desbloquean a otro

| # | Qué es | Dónde está | Qué se rompe y quién lo nota | Sitios | ¿Revert? | Bloquea / desbloquea |
|---|---|---|---|---|---|---|
| 1 | La clave de moderación está escrita en el código, en un repo público | `routes.ts:25` `const SECRETO_SESION = 'dev-secret-cambiar'`, usada en `:351` | Cualquiera que lea el repo borra cualquier comentario con la cabecera `x-clave-moderacion` (probado: `DELETE /comentarios/2` → 200). Lo notan los autores cuando su comentario desaparece | 2 | Sí el código; no la clave: ya está publicada y hay que rotarla | A ninguno |
| 2a | **Contención:** el detalle del post devuelve el correo de quien comenta | `routes.ts:153` `select * from comentarios …`, devuelto tal cual en `:171` | Cualquiera que lea la API obtiene el correo de cada comentarista (probado). En la web no se ve: nadie lo nota todavía | 1 | Sí | Desbloquea la 4: sin el correo publicado, borrar comentarios ajenos deja de ser trivial |
| 3 | La regla de quién puede tocar un comentario está escrita dos veces, y distinta | Borrado `routes.ts:348-356` (mayúsculas indiferentes y clave de moderación) frente a edición `:378-381` (correo exacto, sin clave, solo pendientes) | Probado: el autor puede borrar con `A@B.ES` y no puede editar con `prueba@test.es`; el moderador borra pero no edita. Lo notan autores y moderadores | 2 | Sí | Desbloquea la 4: la autorización se cambia una sola vez |
| 5 | `estado` es texto libre y conviven `aprobado`, `Aprobado` y `ok` | Migración `1769000000005_create_comentarios_table.ts` (`table.string('estado')`, sin enum ni CHECK); el detalle filtra `= 'aprobado'` (`routes.ts:153`) y los listados cuentan `<> 'pendiente'` (`:75-76`, `:223`) | Los comentarios `Aprobado`/`ok` cuentan en el listado y no salen en el detalle. Probado: «diez-tendencias…» dice 4 en el listado y enseña 2; «que-mide…» dice 1 y enseña 0. Lo notan los lectores | 4: tres consultas y los datos guardados | **No**: normalizar reescribe filas guardadas; necesita una persona delante | Desbloquea la 6 |

## Nivel 2 · Esperan a otro hallazgo

| # | Qué es | Dónde está | Qué se rompe y quién lo nota | Sitios | ¿Revert? | Bloquea / desbloquea |
|---|---|---|---|---|---|---|
| 4 | **Arreglo:** el correo de quien comenta funciona como credencial para borrar | `routes.ts:352-353` (`esAutor` compara el correo recibido con el guardado) | Probado: con el correo leído de la propia API se borra un comentario aprobado ajeno (`DELETE /comentarios/1` → 200). Lo notan los comentaristas | 2 (borrado y edición) | Sí | Bloqueado por la 3; la 2a abarata el riesgo mientras tanto |
| 6 | El recuento de comentarios se calcula en tres sitios | `routes.ts:75-76`, `:223` y `DetallePost.jsx:38` (`(d.comentarios \|\| []).length`) | Tres números para el mismo post (listado 4, cabecera del detalle 2). Lo notan los lectores | 3 | Sí | Bloqueado por la 5 |
| 2b | **Arreglo:** las respuestas de comentarios no pasan por un transformer | `routes.ts:171` y `:333` devuelven filas crudas | Cualquier columna nueva sale sola por la API sin que nadie lo decida. Nadie todavía | 3 (dos respuestas y un transformer nuevo) | Sí | Sustituye a la contención 2a; a ninguno más |

## Nivel 3 · Sin dependencias, menor daño

| # | Qué es | Dónde está | Qué se rompe y quién lo nota | Sitios | ¿Revert? | Bloquea / desbloquea |
|---|---|---|---|---|---|---|
| 7 | Borrar un comentario deja sus respuestas huérfanas | `routes.ts:359` borra solo esa fila; `Comentarios.jsx:50` y `:64` pintan respuestas solo bajo una raíz | Probado: tras borrar el 1, la respuesta 2 sigue en la API, la web no la pinta y la cabecera dice «1 comentarios». Lo notan los lectores | 2 | Depende del arreglo: pintar huérfanas sí; borrar en cascada no (pierde datos) | A ninguno; mismo síntoma que la 8 |
| 8 | `padre_id` no se valida | `routes.ts:308` y `:326` | Probado: se acepta `padre_id: 99999`; la respuesta queda invisible. Nadie todavía | 1 | Sí | A ninguno |
| 9 | Cada error de validación usa un formato y un código distinto | `routes.ts:310-321`: `400 {error}`, `422 {message}`, `400 {errores}`, `200 {ok:false}` | Probado. Un consumidor de la API ve un comentario corto como éxito (200). La web lo absorbe leyendo `d.error \|\| d.message \|\| d.motivo` (`Comentarios.jsx:44`): unificarlo cambia lo que la web lee | 4 | Sí | A ninguno |
| 10 | La edición no valida la longitud del cuerpo | `routes.ts:383-386` (solo comprueba que exista) frente al mínimo de 5 al crear (`:320`) | Probado: se edita a `"x"` → 200. Nadie todavía | 1 | Sí | A ninguno |
| 11 | La respuesta del alta lee «el último comentario», no el que se acaba de crear | `routes.ts:329-331` `select * from comentarios order by id desc limit 1` | Riesgo, no probado: con dos altas simultáneas, a uno le puede volver el comentario del otro, con su correo | 1 | Sí | A ninguno |

## Fuera de la lista: opiniones, con su experimento

- **Relacionados con N+1 consultas** (`routes.ts:217-229`). «Conviene una sola consulta» es una solución antes de medir. Experimento: medir cuánto tarda `GET /posts/:slug/relacionados` y dónde se va el tiempo.
- **`post_id` sin índice ni clave foránea** (comentario de la migración). Experimento: medir el detalle con miles de comentarios antes de añadir el índice.

---

## Las tres líneas

1. **El agente devolvió 13 (11 hallazgos y 2 opiniones); yo comprobé 3 abriendo el archivo** (filas 1, 2a y 3).
2. **No supe decidir la prioridad de la 1, la clave en el código:** el daño es enorme, pero la clave ya está publicada, rotarla no se deshace con un revert, y no sé si moverla a una variable de entorno es la contención o el arreglo.
3. **No pude verificar la 11:** que con dos altas simultáneas a uno le vuelva el comentario del otro. Se deduce de la consulta, pero en local no se puede provocar ni descartar.
