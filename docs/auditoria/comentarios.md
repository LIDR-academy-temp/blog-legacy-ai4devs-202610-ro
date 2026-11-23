# Auditoría priorizada — feature de comentarios

**Alcance:** crear, editar, borrar y listar comentarios en sus tres capas.

- **API:** `POST /posts/:slug/comentarios`, `PATCH /comentarios/:id` y `DELETE /comentarios/:id`, más la parte de comentarios de `GET /posts/:slug` y los recuentos `num_comentarios` de `GET /posts` y `GET /posts/:slug/relacionados`.
- **Base de datos:** la tabla `comentarios`.
- **Web:** `Comentarios.jsx` y los recuentos que enseñan `DetallePost.jsx`, `ListaPosts.jsx` y `PostsRelacionados.jsx`.

Queda fuera el resto del sistema, aunque se haya visto de paso (ver el final del documento).

**Fuentes leídas:**
- `backend/start/routes.ts` entero.
- La migración `1769000000005_create_comentarios_table.ts`.
- `blog_seeder.ts`, `comentarios.spec.ts` y `guard-tests.mjs`.
- `config/bodyparser.ts`, `start/env.ts` y `.env.example`.
- Los cuatro componentes de la web.

Además se consultó la base local `backend/tmp/db.sqlite3` en **modo solo lectura** para medir los estados reales y los recuentos.

## Cómo está ordenada

1. **Primero la precedencia**, por capas de dependencia. Una fila va en la capa siguiente a la de su dependencia más profunda. Así, lo que abarata o desbloquea a otro hallazgo siempre queda antes que él.
   - **Capa 1:** no depende de nada.
   - **Capa 2:** solo depende de la capa 1.
   - **Capa 3:** depende de algo de la capa 2.
   - **Capa 4:** depende de algo de la capa 3.
2. **Dentro de cada capa, daño ÷ radio.** El radio es el número de sitios que hay que tocar (columna «Sitios»). El daño va de 1 a 5:
   - **5:** filtra datos de terceros, permite actuar sobre lo ajeno o rompe el propósito de la feature, explotable hoy.
   - **4:** riesgo serio, pero con condición previa.
   - **3:** el lector ve datos erróneos o se puede abusar del recurso.
   - **2:** incoherencia visible o caso límite.
   - **1:** sin efecto observable hoy.
3. **Filas partidas.** Si un hallazgo tiene daño máximo y su arreglo de fondo tiene precedencia baja, se parte en dos filas. La **contención** corta el daño hoy y el **fondo** espera su turno. Pasa con H2/H17 (fuga del correo) y con H3/H16 (borrado con el correo como credencial).

> Por CLAUDE.md, toda fila después de H1 empieza por un test de caracterización que fija el comportamiento de **hoy**. Cada contención o arreglo es una tarea aparte, con su test y una decisión explícita. Ninguna se mete de tapadillo dentro de un refactor.

---

## Capa 1 — Desbloquea a todas las demás

| # · daño÷sitios | Qué es | Dónde está (línea que lo demuestra) | Qué se rompe, y quién lo nota | Sitios | ¿Revert? | ¿Bloquea o desbloquea? |
|---|---|---|---|---|---|---|
| **H1** · n/a | La feature tiene un único test: el camino feliz del POST, y solo comprueba `autor_nombre`. | `backend/tests/functional/comentarios.spec.ts:15` `test('deja un comentario en un post publicado'…` y `:42` `assert.equal(respuesta.body().comentario.autor_nombre, …)`. `.claude/hooks/guard-tests.mjs:32` `const COBERTURA_MINIMA = 0`. | Nadie todavía. Lo notará quien toque la feature, porque cualquier cambio de los que siguen puede romper GET, PATCH o DELETE sin que nada avise. | 1 (`comentarios.spec.ts`, ampliado con la matriz feliz / inválida / 404 / frontera de cada endpoint) | Sí. Solo añade tests. | **Desbloquea a todas (H2–H19).** CLAUDE.md exige caracterizar antes de cambiar nada. |

## Capa 2 — Dependen solo de H1

| # · daño÷sitios | Qué es | Dónde está (línea que lo demuestra) | Qué se rompe, y quién lo nota | Sitios | ¿Revert? | ¿Bloquea o desbloquea? |
|---|---|---|---|---|---|---|
| **H2** · 5÷1 = 5,0 · *contención de H17* | El detalle público de un post devuelve el correo de cada comentarista aprobado. | `backend/start/routes.ts:153` `"select * from comentarios where post_id = ? and estado = 'aprobado' …"`, enviado tal cual en `:171` `comentarios: comentarios`. | Cualquier visitante o scraper lee hoy el correo de quienes comentan, contra la regla 6. Lo sufren los comentaristas. La web no usa ese campo, así que quitarlo no lo nota nadie. | 1 (la consulta de `:153`: columnas explícitas sin `autor_email`; el resto del contrato sigue igual) | Sí el código. Lo que ya se haya raspado no se recupera. | **Abarata H3:** sin correo público, el borrado ajeno exige conocer el correo por otra vía. Su fondo es H17. |
| **H3** · 5÷1 = 5,0 · *contención de H16* | Cualquiera borra un comentario ajeno con solo enviar el correo de su autor, y ese correo es público (H2). | `backend/start/routes.ts:352-353` `const esAutor = String(comentario.autor_email).toLowerCase().trim() === String(email)…` y `:355` `if (!esModerador && !esAutor)`. | Un tercero puede borrar comentarios aprobados ajenos copiando el correo de `GET /posts/:slug`. Lo notan el comentarista (su comentario desaparece) y el lector. Si se corta la rama `esAutor`, desde la web no lo nota nadie: no existe UI de borrar (ningún `DELETE` en `frontend/src`). | 1 (el bloque de `:350-356`: solo la clave de moderación autoriza) | Sí. | Su fondo es H16. |
| **H4** · 3÷1 = 3,0 | La respuesta del POST devuelve la última fila de toda la tabla, no la que acaba de insertar. | `backend/start/routes.ts:330` `'select * from comentarios order by id desc limit 1'` → `:333` `send({ comentario: nuevos[0] })`. | Con dos envíos casi simultáneos, una persona recibe el comentario de otra, con su correo. Nadie lo ha visto todavía: con este tráfico no ocurre, y **no se ha reproducido**, solo se ha deducido leyendo el código. | 1 (`:324-331`: `insert … returning *`, que el SQLite 3.53.4 local admite) | Sí. | A ninguno. Tras H8 cae de forma natural en el repositorio. |
| **H5** · 4÷3 = 1,3 | La clave de moderación está escrita a fuego en el código y versionada en git. | `backend/start/routes.ts:25` `const SECRETO_SESION = 'dev-secret-cambiar'`, usada en `:351` `const esModerador = clave === SECRETO_SESION` (es el único uso: el comentario de `:21-24` dice que firma sesiones, pero no lo hace). | Cualquiera con acceso de lectura al repo o a un fork borra cualquier comentario con la cabecera `x-clave-moderacion`. Lo nota el comentarista. | 3 (`routes.ts:25`, `start/env.ts` y `.env.example`) | **No del todo:** el valor ya está en el historial de git. Solo deja de valer si se rota. | **Desbloquea H12:** la moderación necesita una identidad de moderador fiable. |
| **H6** · 2÷2 = 1,0 | La web lleva dos recuentos de comentarios separados, y el de la sección suma 1 al enviar aunque el comentario quede pendiente e invisible. | `frontend/src/componentes/DetallePost.jsx:30` `const [numComentarios, setNumComentarios]` (pintado en `:71`) frente a `frontend/src/componentes/Comentarios.jsx:11` `const [total, setTotal]` y `:39` `setTotal(total + 1)`. | El lector: tras comentar, la cabecera dice «2 comentarios» y la sección «3 comentarios», con dos visibles. | 2 archivos (7 líneas: `DetallePost.jsx:30,38,71` y `Comentarios.jsx:11,23,39,54`) | Sí. | A ninguno. |
| **H7** · 3÷5 = 0,6 | `estado` es texto libre y en los datos conviven `aprobado`, `Aprobado` y `ok`. | Migración `1769000000005_create_comentarios_table.ts:18` `table.string('estado').notNullable().defaultTo('pendiente')`, sin CHECK. En la base local: `aprobado` 4, `Aprobado` 2, `ok` 1 y `pendiente` 2. | 3 de los 7 comentarios no pendientes no salen nunca en el detalle, porque `routes.ts:153` compara con `= 'aprobado'` exacto. El post 6 enseña 0 comentarios. Lo notan el lector y quien escribió el comentario. | 5 (1 migración nueva, 1 semilla con las filas 3, 4 y 9 de `COMENTARIOS`, y 3 consultas: `routes.ts:75`, `:153` y `:223`) | **No:** la normalización reescribe datos y el `down` no sabe qué fila era `ok` y cuál `Aprobado`. Hay que guardar una copia antes. Además hay que **decidir** si `ok` cuenta como aprobado. | **Desbloquea H11** (fija qué es «aprobado»). **Abarata H13 y H15:** en SQLite, añadir CHECK o FK exige reconstruir la tabla, y eso se hace una sola vez. |
| **H8** · 1÷8 ≈ 0,13 | Las rutas de comentarios mezclan SQL, validación, autorización y forma de la respuesta dentro de `routes.ts`: no hay controlador, validador, servicio, repositorio ni transformer. | `backend/start/routes.ts:295` `router.post('/posts/:slug/comentarios', async ({ params, request, response }) => {` (igual en `:339` y `:366`); `db.rawQuery` directo en `:324`, `:359` y `:388`. | Nadie todavía: es deuda de estructura. Lo nota el equipo, que tiene que copiar reglas a mano, y ya salieron distintas (H3 frente a `:379`). | 8 (5 archivos nuevos: controlador, validador, servicio, repositorio y transformer; 3 editados: `routes.ts`, el alias `#repositories/*` en `backend/package.json` y `app/exceptions/handler.ts`) | Sí, si se hace por pasos con la suite verde. | **Desbloquea H9, H10, H12, H14, H16, H17 y H18.** |

## Capa 3 — Dependen de algo de la capa 2

| # · daño÷sitios | Qué es | Dónde está (línea que lo demuestra) | Qué se rompe, y quién lo nota | Sitios | ¿Revert? | ¿Bloquea o desbloquea? |
|---|---|---|---|---|---|---|
| **H9** · 3÷1 = 3,0 | El POST no limita el tamaño ni comprueba el tipo de nombre, correo y cuerpo; al correo solo le pide una `@`. | `backend/start/routes.ts:316` `if (String(email).indexOf('@') === -1)` y `:319` `if (!cuerpo \|\| String(cuerpo).length < 5)`, sin máximo; el único tope es `config/bodyparser.ts:69` `limit: '20mb'`. | Un anónimo puede dejar comentarios pendientes de hasta 20 MB y llenar el disco. Nadie lo ha notado todavía. Que un cuerpo no textual (un objeto JSON) acabe en 500 es una **deducción no ejecutada**. | 1 (el validador VineJS de creación) | Sí. | Depende de H8. **Abarata H14** (mismo validador). |
| **H10** · 2÷1 = 2,0 | `padre_id` se guarda sin comprobar que el padre exista, pertenezca al mismo post o sea de primer nivel. | `backend/start/routes.ts:308` `const padre = request.input('padre_id', null)`, insertado sin más en `:326`. | Una respuesta con un padre inválido o anidada a dos niveles no se pinta nunca: `Comentarios.jsx:50` solo pinta raíces y `:64` solo hijos directos. Aun así, cuenta en `num_comentarios`. Desde la web no lo nota nadie, porque el formulario no envía `padre_id`: solo pasa usando la API directamente. | 1 (el validador de creación) | Sí. | Depende de H8. |
| **H11** · 3÷2 = 1,5 | Los recuentos de los listados cuentan todo lo que no está pendiente, en vez de solo los aprobados. | `backend/start/routes.ts:75` `"… and c.estado <> 'pendiente') as num_comentarios"` y `:223` `"… estado <> 'pendiente'"`. | El lector, hoy. En la base local, el post 1 dice «4 comentarios» en el listado (`ListaPosts.jsx:57`) y en el detalle enseña 2. El post 6 dice 1 y enseña 0. Incumple la regla 5. | 2 (`routes.ts:75` y `:223`) | Sí. | Depende de H7: qué cuenta como aprobado se decide allí. |
| **H12** · 5÷5 = 1,0 | No existe ninguna ruta para aprobar o rechazar un comentario, así que nada pasa nunca de `pendiente` a publicado. | `backend/start/routes.ts:388` `'update comentarios set cuerpo = ? where id = ?'` es el **único** `update` de la tabla; la web promete lo contrario en `Comentarios.jsx:38` `'Gracias, tu comentario está pendiente de moderación.'`. | Todo el que comenta: su comentario no se publica jamás si nadie toca la base a mano (los ids 5 y 7 de la semilla siguen así). Contención actual: un `UPDATE` manual en SQLite. | 5 (ruta, método de controlador, validador, método de servicio y método de repositorio) | Sí. | Depende de H8 y de H5 (quién es moderador). **Desbloquea H19.** Es una feature ausente: necesita una decisión de producto. |
| **H13** · 2÷2 = 1,0 | Al borrar un comentario, sus respuestas quedan huérfanas: no hay cascada ni FK en `padre_id`. | `backend/start/routes.ts:359` `'delete from comentarios where id = ?'`; migración `:20` `table.integer('padre_id').nullable()`, sin `references`. | Las respuestas aprobadas desaparecen de la web (`Comentarios.jsx:50`), pero siguen contando en `routes.ts:75`. Nadie todavía: solo existe una respuesta (id 2 → padre 1) y la web no permite borrar. | 2 (`routes.ts:359` y la reconstrucción de la tabla de H7) | **No:** añadir una FK en SQLite obliga a recrear la tabla, y deshacerlo es otra migración. | Depende de H7 (comparten reconstrucción). |
| **H14** · 1÷1 = 1,0 | Al editar no se exige la longitud mínima que sí se exige al crear. | `backend/start/routes.ts:384` `if (!cuerpo) {`, frente a `:319` `String(cuerpo).length < 5`. | El autor puede dejar su comentario pendiente en un solo carácter. Nadie todavía: no hay UI de edición. | 1 (el validador de edición) | Sí. | Depende de H8 y H9. |
| **H15** · 1÷1 = 1,0 | `post_id` no tiene índice ni clave foránea. | Migración `:14` `table.integer('post_id').notNullable()`, y su propio comentario en `:6` `post_id no tiene índice ni clave foránea`. | Nadie todavía: con 9 filas no se nota, pero la subconsulta de `routes.ts:75` recorre la tabla entera por cada post del listado. | 1 (la misma reconstrucción de H7) | No (igual que H13). | Depende de H7. |
| **H16** · 3÷4 = 0,75 · *fondo de H3* | Quien comenta no tiene identidad: editar y borrar usan el correo como si fuera una contraseña. | `backend/start/routes.ts:379` `if (comentario.autor_email !== email \|\| comentario.estado !== 'pendiente')` (sin pasar a minúsculas, a diferencia del DELETE en `:353`). | Tras H3 queda abierto el PATCH sobre comentarios pendientes, para quien conozca o adivine el correo. El comentarista no puede gestionar lo suyo de forma segura. | 4 (1 migración con un token de autoría, más el POST que lo emite y el PATCH y el DELETE que lo exigen) | **No del todo:** si se revierte, los tokens ya entregados dejan de valer. | Depende de H3 y H8. |
| **H17** · 2÷4 = 0,5 · *fondo de H2* | Ninguna respuesta de comentarios pasa por un transformer: salen filas crudas con `autor_email`, `estado` y `post_id`. | `backend/start/routes.ts:333` `send({ comentario: nuevos[0] })` y `:392` `send({ comentario: actualizados[0] })`, además de `:171`. | Tras H2, lo que queda es que cualquier `select *` futuro vuelva a filtrar el correo, y que el contrato exponga campos internos. Nadie todavía. | 4 (1 transformer nuevo y 3 respuestas: `:171`, `:333` y `:392`) | Sí. | Depende de H2 y H8. |
| **H18** · 2÷6 ≈ 0,33 | La validación del POST responde con cuatro formas de error distintas, y una de ellas es un 200. | `backend/start/routes.ts:311` `status(400).send({ error … })`, `:314` `status(422).send({ message … })`, `:317` `status(400).send({ errores: [...] })` y `:320` `response.send({ ok: false, motivo: … })`. | Quien escribe mal el correo ve «No se ha podido enviar.», porque `Comentarios.jsx:44` `d.error \|\| d.message \|\| d.motivo` no lee `errores`. Un cliente de la API recibe 200 ante un cuerpo corto. | 6 (4 respuestas en `routes.ts`, `Comentarios.jsx:44` y `app/exceptions/handler.ts`) | Sí el código, pero cambia el contrato: es una decisión explícita que sustituye el test de caracterización. | Depende de H8. |

## Capa 4 — Dependen de algo de la capa 3

| # · daño÷sitios | Qué es | Dónde está (línea que lo demuestra) | Qué se rompe, y quién lo nota | Sitios | ¿Revert? | ¿Bloquea o desbloquea? |
|---|---|---|---|---|---|---|
| **H19** · 2÷2 = 1,0 | El POST es anónimo y no tiene límite de frecuencia, y el spam ya está en los datos. | `backend/start/routes.ts:295` `router.post('/posts/:slug/comentarios', …)`, sin middleware; semilla `COMENTARIOS` id 7 `autor_email: 'promo@plantillas-baratas.example'`. `@adonisjs/limiter` no está en `backend/package.json`. | El moderador, en cuanto exista (H12): la cola se llena de spam. Hoy no lo nota nadie, porque nada se modera. | 2 (instalar y configurar el limiter, y aplicarlo a la ruta) | Sí. | Depende de H12. |

---

## Revisado, sin hallazgo

- **Inyección SQL:** todas las consultas de la feature usan parámetros `?`, por ejemplo `routes.ts:153`, `:325`, `:359` y `:388`.
- **XSS en comentarios:** el cuerpo se pinta como texto de React, `Comentarios.jsx:61` `<p>{c.cuerpo}</p>`. El único `dangerouslySetInnerHTML` (`DetallePost.jsx:92`) es del cuerpo del post, no de un comentario.
- **Comentar en un borrador:** el POST responde 404 porque filtra `estado = 'publicado'` (`routes.ts:297`).

## Fuera de alcance, visto de paso (sin auditar)

- `GET /posts`, `/posts/:slug`, `/posts/recientes` y `/autores/:id` devuelven `select * from posts`, y eso incluye el **correo del autor del post**.
- `/autores/:id` lista también los borradores (`routes.ts:285`).
- Ambos son de la feature de posts y de autores, no de esta.

---

## Las tres líneas

1. **Hallazgos devueltos por el agente:** 19 3
2. **Hallazgo con la prioridad más dudosa:** H12, la moderación que no existe. Tiene el daño máximo, pero es una feature ausente y no un bug;
3. **Afirmación que no se pudo verificar:** que en H4 la carrera entregue de verdad el comentario de otra persona, y que en H9 un cuerpo no textual acabe en 500. 