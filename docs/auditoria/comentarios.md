# Auditoría de la feature de comentarios
 
Alcance: comentarios de punta a punta: API (`POST /posts/:slug/comentarios`, `PATCH` y `DELETE /comentarios/:id`, comentarios y recuentos en `GET /posts`, `GET /posts/:slug` y `GET /posts/:slug/relacionados`), tabla `comentarios` con su semilla, y lo que `frontend/src` muestra o envía. Contrastado con las reglas 2, 5 y 6 y con la arquitectura de `CLAUDE.md`.

| # | Qué es | Dónde está | Qué se rompe, y quién lo nota | Cuántos sitios hay que tocar | ¿Se deshace con un revert? | ¿Bloquea o desbloquea a otro hallazgo? |
|---|---|---|---|---|---|---|
| C1 | `GET /posts/:slug` devuelve la fila cruda de cada comentario, con el correo de quien comenta (incumple la regla 6). | `backend/start/routes.ts:153` `"select * from comentarios where post_id = ? and estado = 'aprobado' order by creado_en asc",` y `backend/start/routes.ts:171` `comentarios: comentarios,` | Cualquier lector o *scraper* obtiene el `autor_email` de todos los comentaristas aprobados (comprobado: el post 1 devuelve `ana.cortes@ejemplo.es` y `marta.ruiz@corriente.es`). Lo sufren los comentaristas. | 2 (routes.ts:153, routes.ts:171) | No: los correos ya servidos no se pueden retirar. | Alimenta C2: es la fuente pública de la credencial de borrado. Arreglarlo mitiga C2, pero no lo cierra. |
| C2 | `DELETE /comentarios/:id` considera autor a quien envíe en el cuerpo el mismo correo que tiene el comentario, sin más prueba de identidad. | `backend/start/routes.ts:353` `String(comentario.autor_email).toLowerCase().trim() === String(email).toLowerCase().trim()` | Con el correo que publica C1, cualquier lector borra cualquier comentario aprobado (comprobado: se borró el id 1 con el correo sacado del GET). Lo nota el comentarista cuando su comentario desaparece. | 1 (routes.ts:353) | No: el borrado es físico y no hay papelera, así que lo borrado se pierde. | Depende de C1. Es lo que dispara C4. |
| C3 | La clave de moderación que permite borrar cualquier comentario está escrita como literal en el código fuente. | `backend/start/routes.ts:25` `const SECRETO_SESION = 'dev-secret-cambiar'` (se usa en routes.ts:351 `const esModerador = clave === SECRETO_SESION`) | Quien tenga acceso al repo borra cualquier comentario enviando `x-clave-moderacion: dev-secret-cambiar` (comprobado con el id 3). Lo notan los comentaristas y el operador. | 2 (routes.ts:25, routes.ts:351) | No: la clave ya está en el historial de git y hay que rotarla. | A ninguno. |
| C4 | Al borrar un comentario raíz, sus respuestas se quedan con un `padre_id` que ya no existe, y la web las oculta. | `backend/start/routes.ts:359` `await db.rawQuery('delete from comentarios where id = ?', [params.id])`; `frontend/src/componentes/Comentarios.jsx:50` `const raices = comentarios.filter((c) => !c.padre_id)` | Tras borrar el id 1, la API sigue devolviendo la respuesta aprobada id 2 con `padre_id: 1` (comprobado), pero la web solo pinta respuestas de raíces presentes, así que el lector deja de verla. Lo nota también quien escribió la respuesta. | 3 (routes.ts:359, migración `1769000000005_create_comentarios_table.ts:20`, Comentarios.jsx:50) | No: el padre borrado no se recupera. | Lo dispara C2 (cualquiera puede borrar un padre). |
| C5 | El recuento de comentarios del listado y de relacionados cuenta todo estado que no sea `'pendiente'`, no solo los aprobados (incumple la regla 5). | `backend/start/routes.ts:75` `'select *, (select count(*) from comentarios c where c.post_id = posts.id and c.estado <> ' +`; `backend/start/routes.ts:223` `"select count(*) as total from comentarios where post_id = ? and estado <> 'pendiente'",` | El lector ve «4 comentarios» en la portada para el post 1 y luego encuentra 2 en el detalle; el post 6 marca 1 y en el detalle tiene 0 (comprobado). Un futuro estado como `'rechazado'` también sumaría. | 2 (routes.ts:75, routes.ts:223) | Sí. | Bloqueado por C7: hasta normalizar `estado` no hay una única definición de «aprobado». |
| C6 | El detalle filtra por `estado = 'aprobado'` exacto, así que oculta los comentarios guardados como `'Aprobado'` u `'ok'`, que la semilla documenta como aprobados. | `backend/start/routes.ts:153` `"select * from comentarios where post_id = ? and estado = 'aprobado' order by creado_en asc",` | El lector no ve los comentarios 3, 4 y 9, y el post 6 aparece sin ninguno (comprobado). Lo notan sus autores, que ven desaparecer un comentario aprobado. | 1 (routes.ts:153) | Sí. | Bloqueado por C7. |
| C7 | La columna `comentarios.estado` es texto libre, sin CHECK ni enum, y la semilla ya mezcla `'aprobado'`, `'Aprobado'` y `'ok'`. | `backend/database/migrations/1769000000005_create_comentarios_table.ts:18` `table.string('estado').notNullable().defaultTo('pendiente')`; `backend/database/seeders/blog_seeder.ts:510` `estado: 'Aprobado',` | Es la causa de C5 y C6: cada consulta interpreta «aprobado» a su manera. Lo nota el desarrollador al escribir cualquier filtro, y el operador al moderar. | 4 (migración:18, blog_seeder.ts:510, blog_seeder.ts:522, blog_seeder.ts:575), más una migración de datos nueva | No: normalizar exige migrar los datos de las filas existentes, y revertir el código no devuelve los valores originales. | Desbloquea C5 y C6. |
| C8 | `POST /posts/:slug/comentarios` responde con el último comentario de toda la tabla, no con el que acaba de insertar. | `backend/start/routes.ts:330` `'select * from comentarios order by id desc limit 1'` | Si dos peticiones se cruzan entre el `insert` y el `select`, cada comentarista recibe el comentario del otro, con su correo. Nadie todavía; con tráfico concurrente, el comentarista. | 1 (routes.ts:330) | Sí para el código. Si la carrera ya ha ocurrido, el correo devuelto a otra persona no se recupera. | Lo desbloquea C16 (un repositorio que devuelva la fila creada). |
| C9 | Al enviar un comentario, la web suma 1 al recuento aunque el comentario queda `pendiente` (incumple la regla 5). | `frontend/src/componentes/Comentarios.jsx:39` `setTotal(total + 1)` | El comentarista ve «3 comentarios» sobre una lista de 2. La cabecera del artículo (`DetallePost.jsx:71`) no cambia, así que en la misma página aparecen dos cifras distintas. | 1 (Comentarios.jsx:39) | Sí. | A ninguno. |
| C10 | La validación del POST son `if`s escritos a mano que devuelven cuatro formas de error distintas, una de ellas un `200 {ok:false}`. | `backend/start/routes.ts:320` `return response.send({ ok: false, motivo: 'comentario demasiado corto' })` (además `400 {error}` en :311, `422 {message}` en :314 y `400 {errores:[...]}` en :317) | Un cliente que mire el código HTTP da por bueno un comentario corto (comprobado: 200). La web no lee `errores` (Comentarios.jsx:44 `setAviso(d.error \|\| d.message \|\| d.motivo \|\| 'No se ha podido enviar.')`), así que con un email sin `@` el comentarista solo ve el mensaje genérico. | 5 (routes.ts:311, :314, :317, :320, Comentarios.jsx:44) | Sí. | Lo desbloquea C16 (validador VineJS + `handler.ts`). |
| C11 | La única comprobación del correo es que contenga `@`. | `backend/start/routes.ts:316` `if (String(email).indexOf('@') === -1) {` | Se aceptó `autor_email: "@"` con un 201 (comprobado). El moderador no puede contactar con el comentarista. | 1 (routes.ts:316) | Sí para el código. Los correos inválidos ya guardados se quedan. | Lo desbloquea C16 (validador). |
| C12 | El POST acepta cualquier `padre_id` sin comprobar que exista ni que pertenezca al mismo post. | `backend/start/routes.ts:308` `const padre = request.input('padre_id', null)` | Se crearon respuestas con `padre_id` 99999 (no existe) y 8 (es del post 4) en el post 1, ambas con 201 (comprobado). Cuando se aprueben, la web no las pintará (Comentarios.jsx:50), así que su autor nunca las verá publicadas. | 1 (routes.ts:308) | Sí para el código. Las filas ya guardadas con padres inválidos se quedan. | Lo desbloquea C16 (validador). Agrava C4. |
| C13 | `PATCH` y `DELETE` aplican reglas de autoría distintas: el PATCH distingue mayúsculas en el correo, no acepta la clave de moderación y exige que el comentario siga `pendiente`. | `backend/start/routes.ts:379` `if (comentario.autor_email !== email \|\| comentario.estado !== 'pendiente') {` | Con un comentario guardado como `Aud@b.c`, se puede borrar enviando `aud@b.c` pero no editar (403). El moderador recibe 403 al editar (comprobado). Lo notan el comentarista y el moderador. | 2 (routes.ts:353, routes.ts:379) | Sí. | Lo desbloquea C16 (una sola regla en el servicio). |
| C14 | El PATCH no aplica la longitud mínima de 5 caracteres que sí exige el POST. | `backend/start/routes.ts:384` `if (!cuerpo) {` (frente a routes.ts:319 `if (!cuerpo \|\| String(cuerpo).length < 5) {`) | Se editó un comentario a `"x"` con un 200 (comprobado). Un comentario rechazado por corto al crearlo puede quedar corto después de editarlo. | 2 (routes.ts:319, routes.ts:384) | Sí. | Lo desbloquea C16 (un validador compartido). |
| C15 | `POST` y `PATCH` devuelven la fila cruda del comentario (`autor_email`, `estado`, `post_id`) en vez de un transformer. | `backend/start/routes.ts:333` `return response.status(201).send({ comentario: nuevos[0] })` y `backend/start/routes.ts:392` `return response.send({ comentario: actualizados[0] })` | La respuesta expone campos internos que el contrato no pide (comprobado en ambas). Con C8, el correo que sale puede ser de otra persona. Por ahora lo nota el desarrollador; con C8, también el comentarista. | 2 (routes.ts:333, routes.ts:392) | Sí. | Lo desbloquea C16 (transformer). |
| C16 | Los tres endpoints de comentarios son closures en `start/routes.ts`, con SQL, validación y autorización dentro, y sin modelo, controlador, validador, servicio, repositorio ni transformer. | `backend/start/routes.ts:295` `router.post('/posts/:slug/comentarios', async ({ params, request, response }) => {` (y :339, :366); `backend/app/models/` solo contiene `user.ts` | Lo nota el desarrollador: cada regla se duplica a mano y acaba siendo distinta (C13, C14), y no hay un sitio único para la validación ni para la forma de la respuesta. | 3 (routes.ts:295, routes.ts:339, routes.ts:366), más las capas que faltan | Sí. | Bloqueado por C17. Desbloquea C8, C10, C11, C12, C13, C14 y C15. |
| C17 | La única prueba de comentarios cubre el camino feliz del POST y solo afirma `autor_nombre`. | `backend/tests/functional/comentarios.spec.ts:42` `assert.equal(respuesta.body().comentario.autor_nombre, 'Lucía Ferrer')` | Ningún test cubre la entrada inválida, el 404, PATCH, DELETE, el ocultado de pendientes, los recuentos ni la ausencia de correos. Un refactor podría romper o filtrar algo sin que la suite se ponga en rojo. Lo nota el desarrollador. | 1 (comentarios.spec.ts:42) | Sí. | Bloquea C16: no se refactoriza sin caracterización previa. |
| C18 | `comentarios.post_id` no tiene clave foránea ni índice. | `backend/database/migrations/1769000000005_create_comentarios_table.ts:14` `table.integer('post_id').notNullable()` | La subconsulta de recuento de routes.ts:75 recorre la tabla entera por cada post. Además, nada impide comentarios de posts inexistentes. Nadie todavía, porque la semilla tiene 9 filas. | 1 (migración:14) | No: en SQLite, añadir la FK obliga a recrear la tabla y a limpiar antes los huérfanos, es decir, una migración con datos. | A ninguno. |

Criterio de prioridad: primero lo que expone datos personales o destruye contenido de forma irreversible (C1–C4). Después, las violaciones de las reglas 2, 5 y 6 que ve el lector (C5–C9). Luego la validación y la autorización incoherentes (C10–C15). Al final, la deuda estructural que no rompe nada visible hoy (C16–C18).

## Orden de ataque por capas

**Primera clave: precedencia (orden topológico).** A va antes que B si arreglar A abarata arreglar B. Las aristas salen de la columna «¿Bloquea o desbloquea?»:

- C17 → C16 (no se refactoriza sin caracterización).
- C7 → C5, C6 (sin un único valor de «aprobado» no hay filtro correcto).
- C16 → C8, C10, C11, C12, C13, C14, C15 (validador, servicio, repositorio y transformer dan un sitio único donde arreglarlos).

No cuentan como precedencia, porque **bajan el daño, no el coste del arreglo**: C1 «mitiga» C2, C2 «dispara» C4 y C12 «agrava» C4. Arreglar uno no hace más barato el código del otro.

**Segunda clave, dentro de cada capa: daño ÷ radio.** El radio es la columna «Cuántos sitios hay que tocar». El daño va en una escala de 1 a 5:

- 5: datos personales o contenido perdidos de forma irreversible, al alcance de cualquier lector.
- 4: incumple una regla de negocio y lo ve cualquier lector.
- 3: efecto visible y erróneo para algunos usuarios.
- 2: incoherencia o riesgo que hoy notan pocos (comentarista, operador o desarrollador).
- 1: nadie todavía.

Desempates: menos sitios primero; si persiste, el que más hallazgos desbloquea, y después el id.

### Capa 1: no depende de ninguno

| Orden | # | Qué es (resumen) | Daño | Sitios | Daño ÷ radio |
|---|---|---|---|---|---|
| 1 | C2 | Borrar un comentario solo exige conocer el correo del autor | 5 | 1 | **5,0** |
| 2 | C1 | El detalle del post expone el correo de quien comenta | 5 | 2 | **2,5** |
| 3 | C17 | El único test de comentarios solo afirma `autor_nombre` | 2 | 1 | **2,0** |
| 4 | C9 | La web suma 1 al recuento con el comentario aún pendiente | 2 | 1 | **2,0** |
| 5 | C3 | Clave de moderación escrita como literal en el código | 4 | 2 | **2,0** |
| 6 | C18 | `post_id` sin clave foránea ni índice | 1 | 1 | **1,0** |
| 7 | C4 | Las respuestas de un padre borrado quedan huérfanas y ocultas | 3 | 3 | **1,0** |
| 8 | C7 | `estado` es texto libre y la semilla mezcla tres valores | 2 | 5 | **0,4** |

C17 va por delante de C9 aunque empatan en daño y en sitios, porque desbloquea la capa 2 (C16) y con ella toda la capa 3.

### Capa 2: depende de lo arreglado en la capa 1

| Orden | # | Qué es (resumen) | Depende de | Daño | Sitios | Daño ÷ radio |
|---|---|---|---|---|---|---|
| 1 | C6 | El detalle oculta los aprobados guardados como `'Aprobado'` u `'ok'` | C7 | 4 | 1 | **4,0** |
| 2 | C5 | El recuento del listado y de relacionados no cuenta solo los aprobados | C7 | 4 | 2 | **2,0** |
| 3 | C16 | Los tres endpoints son closures en `routes.ts`, sin ninguna capa | C17 | 2 | 10 | **0,2** |

El radio de C16 es 10: las 3 closures de `routes.ts:295`, `:339` y `:366`, más 7 piezas nuevas (modelo, controlador, validador, servicio, repositorio, transformer y el alias `#repositories/*` en `package.json`).

### Capa 3: depende de lo arreglado en la capa 2 (C16)

| Orden | # | Qué es (resumen) | Daño | Sitios | Daño ÷ radio |
|---|---|---|---|---|---|
| 1 | C12 | El POST acepta un `padre_id` inexistente o de otro post | 3 | 1 | **3,0** |
| 2 | C8 | El POST responde con el último comentario de la tabla, no con el creado | 2 | 1 | **2,0** |
| 3 | C11 | Como correo vale cualquier cosa que lleve `@` | 2 | 1 | **2,0** |
| 4 | C13 | PATCH y DELETE aplican reglas de autoría distintas | 2 | 2 | **1,0** |
| 5 | C14 | El PATCH no exige la longitud mínima del POST | 2 | 2 | **1,0** |
| 6 | C15 | POST y PATCH devuelven la fila cruda en vez de un transformer | 2 | 2 | **1,0** |
| 7 | C10 | Validación con `if`s y cuatro formatos de error, uno de ellos `200 {ok:false}` | 3 | 5 | **0,6** |

### Notas del orden

- **C4, C7 y C18 exigen recrear la tabla `comentarios`.** SQLite no añade un CHECK ni una FK con `ALTER TABLE`. No es precedencia, porque ninguno abarata a otro, pero conviene hacerlos en la misma migración para pagar la recreación una sola vez.
- **C2 y C13 tocan la misma línea (`routes.ts:353`).** Si C2 se arregla en capa 1 sobre la closure legacy, C13 lo reescribe en capa 3. Es un coste aceptado: el daño de C2 (5) no puede esperar a C16.
- **El daño es un juicio y el radio es un recuento.** Cambiar un daño en un punto puede mover un hallazgo dentro de su capa, pero nunca lo saca de ella.

## Parte B

1. **Hallazgos devueltos por el agente / comprobados por mí abriendo el archivo:** 18/14
2. **Hallazgo cuya prioridad no supe decidir, y por qué:** _pendiente_
3. **Algo que el agente afirmó y no pude verificar:** _pendiente_
