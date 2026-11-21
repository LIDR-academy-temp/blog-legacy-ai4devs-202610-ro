# Auditoría — funcionalidad «comentarios»

**Alcance:** la funcionalidad de comentarios de punta a punta (crear, editar, borrar y listar)
en sus tres capas: rutas de la API, tabla en la base de datos y lo que la web muestra.
Lo que no es de comentarios queda fuera.

**Método:**
- Un agente de solo lectura recorrió el código.
- Después comprobé a mano, una por una, las líneas citadas.
- Un hallazgo sin la línea que lo demuestra no está en la lista.
- No se ha modificado código ni el esquema.

**Fuentes leídas:**
- `backend/start/routes.ts`
- `backend/database/migrations/1769000000005_create_comentarios_table.ts`
- `backend/database/seeders/blog_seeder.ts`
- `backend/config/cors.ts` y `backend/config/shield.ts`
- `backend/start/env.ts`
- `backend/app/exceptions/handler.ts`
- `backend/tests/functional/comentarios.spec.ts`
- `frontend/src/componentes/Comentarios.jsx` y `DetallePost.jsx`, más el resto de `frontend/src/` para buscar usos

**Sin priorizar.** La numeración H1…H33 solo sirve para identificar cada hallazgo y poder citarlo
en la columna «Bloquea / desbloquea».

**Datos comprobados en la BD local** (consulta de solo lectura):

| Post | `num_comentarios` en el listado (`estado <> 'pendiente'`) | Comentarios en el detalle (`estado = 'aprobado'`) |
|---|---|---|
| 1 | 4 | 2 |
| 2 | 1 | 1 |
| 4 | 1 | 1 |
| 6 | 1 | 0 |

La BD local tiene además dos comentarios `pendiente` (ids 10 y 11). Se crearon usando la app y no
vienen de la semilla.

---

## API y base de datos

| # | Qué es | Dónde (línea que lo demuestra) | Qué se rompe y quién lo nota | Sitios a tocar (contados) | ¿Revert? | Bloquea / desbloquea |
|---|---|---|---|---|---|---|
| H1 | El listado y los relacionados cuentan los comentarios con un criterio distinto al que usa el detalle para mostrarlos. | `routes.ts:75-76` `c.estado <> 'pendiente') as num_comentarios`; `routes.ts:223` `estado <> 'pendiente'`; frente a `routes.ts:153` `estado = 'aprobado'` | El lector ve «4 comentarios» en la tarjeta del post 1 y solo 2 al abrirlo; en el post 6 ve 1 y luego 0. Incumple la regla 5. | 2: `routes.ts:75-76`, `routes.ts:223` | Sí | Lo bloquea **H2**: si se pasa a `= 'aprobado'` sin normalizar los datos antes, desaparecen los 'Aprobado' y 'ok'. |
| H2 | `comentarios.estado` es texto libre sin CHECK, y en los datos conviven 'aprobado', 'Aprobado' y 'ok'. | migración `…005:18` `table.string('estado').notNullable().defaultTo('pendiente')`; seeder `:510` `estado: 'Aprobado',`, `:522` `estado: 'ok',`, `:575` `estado: 'Aprobado',` | Los comentarios aprobados 3, 4 y 9 (Rubén, Silvia y Paula) nunca salen en el detalle, porque la comparación de `routes.ts:153` es exacta. Lo notan sus autores y los lectores. | 4: 1 migración nueva (UPDATE para normalizar y CHECK), seeder `:510`, `:522`, `:575` | **No**: al pasar 'ok' y 'Aprobado' a 'aprobado' se pierde cuál era el valor original. | Desbloquea **H1** y **H18**. |
| H3 | El detalle público de un post devuelve el `autor_email` y el `estado` de cada persona que comenta. | `routes.ts:153` `"select * from comentarios where post_id = ? and estado = 'aprobado' …"`; `routes.ts:171` `comentarios: comentarios,` | Cualquier lector o scraper puede recolectar los correos de quienes comentan. Incumple la regla 6. Lo sufren los autores de comentarios. | 2: `routes.ts:153` (columnas explícitas), `routes.ts:171` (pasar por un transformer) | **No**: los correos ya se han servido en público. | Desbloquea **H4**: le entrega el dato que H4 usa como credencial. |
| H4 | DELETE deja borrar a quien envíe el email del autor del comentario, y ese email es público. | `routes.ts:350` `const email = request.input('autor_email', '')`; `routes.ts:352-353` `String(comentario.autor_email).toLowerCase().trim() === String(email).toLowerCase().trim()` | Cualquiera puede borrar cualquier comentario aprobado: le basta leer el correo en el detalle (H3). Lo notan el autor del comentario y el moderador. | 5: `routes.ts:350`, `routes.ts:352-353`, `routes.ts:324-326` (emitir un token al crear), 1 migración (columna del token), `Comentarios.jsx:37-42` (guardar el token) | **No**: el borrado es físico (H6) y lo borrado no vuelve. | **H3** lo hace explotable. Bloquea **H16** y **H28**. |
| H5 | La clave de moderación está escrita en el código y está en el historial de git. | `routes.ts:25` `const SECRETO_SESION = 'dev-secret-cambiar'`; `routes.ts:351` `const esModerador = clave === SECRETO_SESION`; commit `e0e0943` | Quien lea el repo puede borrar cualquier comentario enviando la cabecera `x-clave-moderacion`. Lo sufren el moderador y los autores. | 5: `routes.ts:25`, `routes.ts:351`, `start/env.ts` (declarar la variable), `.env`, `.env.example` | **No**: el secreto queda en el historial; hay que rotarlo. | A ninguno. |
| H6 | El borrado es físico y deja huérfanas las respuestas de ese comentario. | `routes.ts:359` `await db.rawQuery('delete from comentarios where id = ?', [params.id])`; migración `…005:20` `table.integer('padre_id').nullable()` (sin FK) | Las respuestas quedan con un `padre_id` que ya no existe y la web deja de mostrarlas (H23). No queda rastro para el moderador. Lo nota quien respondió. | 2: `routes.ts:359` (borrado lógico o en cascada), 1 migración (`deleted_at` o FK con `on delete`) | **No**: las filas borradas no vuelven. | Agrava **H4** y **H5**. Desbloquea **H23**. |
| H7 | El POST devuelve el último comentario de toda la tabla, no el que acaba de insertar. | `routes.ts:329-331` `'select * from comentarios order by id desc limit 1'` | Con dos envíos a la vez, un usuario puede recibir el comentario del otro, con su correo incluido. Con el tráfico actual no lo nota nadie todavía. | 2: `routes.ts:324-327` (usar el id insertado), `routes.ts:329-331` | El código sí; lo que ya se haya devuelto a otra persona, no. | A ninguno. Conviene arreglarlo junto con **H8**. |
| H8 | El 201 del POST devuelve la fila cruda (`autor_email`, `estado`, `post_id`, `padre_id`). | `routes.ts:333` `return response.status(201).send({ comentario: nuevos[0] })` | Expone campos internos y el correo, y deja el front atado a la forma de la tabla. Lo nota el desarrollador. | 2: `routes.ts:333`, 1 transformer nuevo | Sí | A ninguno. Va junto con **H7**. |
| H9 | Cada error de validación del POST tiene un código HTTP y una forma de cuerpo distintos. | `routes.ts:311` `status(400).send({ error: … })`; `:314` `status(422).send({ message: 'Falta el email' })`; `:317` `status(400).send({ errores: ['email no válido'] })`; `:320` `send({ ok: false, motivo: … })` | Si el email no tiene '@', quien comenta ve «No se ha podido enviar.»: `Comentarios.jsx:44` no lee `errores`. Cualquier cliente de la API tiene que manejar cuatro contratos. | 7: `routes.ts:310-312`, `:313-315`, `:316-318`, `:319-321`, 1 validador VineJS nuevo, `app/exceptions/handler.ts:15-17`, `Comentarios.jsx:44` | Sí | Desbloquea **H10**, **H11**, **H12** y **H17**. |
| H10 | Un cuerpo demasiado corto responde 200, como si hubiera ido bien. | `routes.ts:320` `return response.send({ ok: false, motivo: 'comentario demasiado corto' })` | La monitorización y los clientes HTTP lo cuentan como éxito. Lo nota el desarrollador u operador. | 1: `routes.ts:320` | Sí | Lo bloquea **H9** (mismo validador). |
| H11 | El cuerpo se valida sin recortar espacios y sin longitud máxima. | `routes.ts:319` `if (!cuerpo \|\| String(cuerpo).length < 5) {` | Se aceptan cinco espacios como comentario, y textos enormes hasta el límite del bodyparser. Lo nota el moderador. | 1: `routes.ts:319` | El código sí; las filas basura ya guardadas, no. | Lo bloquea **H9**. |
| H12 | La validación del email solo comprueba que haya un '@'. | `routes.ts:316` `if (String(email).indexOf('@') === -1) {` | Se guardan direcciones como «a@». Como el email es la credencial para borrar y editar (H4, H16), quien lo escriba mal no podrá gestionar su comentario. | 1: `routes.ts:316` | El código sí; los emails mal formados ya guardados, no. | Lo bloquea **H9**. |
| H13 | `padre_id` se acepta sin comprobar que exista, que sea del mismo post ni que sea numérico. | `routes.ts:308` `const padre = request.input('padre_id', null)`; `routes.ts:326` `[post.id, nombre, email, cuerpo, 'pendiente', creado, padre]` | Se pueden crear respuestas a comentarios de otro post o inexistentes, que nunca se mostrarán. Nadie lo nota todavía: el front no envía `padre_id` (`Comentarios.jsx:33`). | 3: `routes.ts:308`, 1 comprobación antes de `routes.ts:324`, 1 migración (FK) | El código sí; las filas inválidas ya escritas, no. | Comparte la FK de `padre_id` con **H6**. Bloquea **H23**. |
| H14 | `comentarios.post_id` no tiene índice ni clave foránea. | migración `…005:14` `table.integer('post_id').notNullable()` (y el comentario de las líneas 4-6 lo admite) | La subconsulta de `routes.ts:75` recorre la tabla una vez por post, y al borrar un post sus comentarios quedan huérfanos. Hoy no lo nota nadie (11 filas). | 1: 1 migración nueva | Sí (con su `down()`) | A ninguno. |
| H15 | En relacionados, los comentarios se cuentan con una consulta por cada post (N+1). | `routes.ts:220-225` `for (…) { const cuenta: any = await db.rawQuery("select count(*) as total from comentarios where post_id = ? …` | Viajes extra a la BD; con `MAX_RELACIONADOS = 2` son pocos. No lo nota nadie todavía. | 1: bloque `routes.ts:217-232` | Sí | A ninguno. Se toca el mismo bloque que en **H1**. |
| H16 | PATCH comprueba la autoría distinto que DELETE: distingue mayúsculas, no recorta espacios y no acepta la clave de moderación. | `routes.ts:379` `if (comentario.autor_email !== email \|\| comentario.estado !== 'pendiente') {` frente a `routes.ts:352-353` (`toLowerCase().trim()`) | Quien escribió «Ana@…» puede borrar su comentario pero no editarlo, y el moderador no puede corregir. Lo notan el autor y el moderador. | 3: `routes.ts:379`, `routes.ts:352-353`, 1 política o servicio compartido nuevo | Sí | Lo bloquea **H4** (la credencial es la misma). |
| H17 | PATCH acepta cualquier cuerpo que no esté vacío y se salta el mínimo de 5 caracteres del POST. | `routes.ts:384` `if (!cuerpo) {` | Un comentario puede editarse hasta dejarlo en «x». Lo nota el moderador. | 1: `routes.ts:384` | El código sí; los cuerpos ya editados, no. | Lo bloquea **H9**. |
| H18 | No hay ningún endpoint para aprobar o rechazar comentarios. | `routes.ts:326` (solo se escribe `'pendiente'`); `routes.ts:388` `'update comentarios set cuerpo = ? where id = ?'` es el único UPDATE | Los pendientes nuevos (ids 10 y 11) no se publicarán nunca sin tocar la BD a mano. Lo notan el autor del comentario y el moderador. | 4 (nuevos): 1 ruta en `routes.ts`, 1 controlador, 1 validador, 1 servicio | Sí | Lo bloquea **H2** (antes hay que fijar los valores válidos de `estado`). |
| H19 | `creado_en` se guarda en UTC, mientras que los datos de la semilla son horas sin zona. | `routes.ts:323` `const creado = new Date().toISOString().slice(0, 19).replace('T', ' ')` | El orden (`routes.ts:153`) y la fecha que se muestra (`Comentarios.jsx:59`) pueden desfasarse según la zona horaria; cerca de medianoche cambia el día. Lo nota el lector. *Alcance no verificado: depende de `TZ`.* | 1: `routes.ts:323` | El código sí; las filas ya escritas, no. | A ninguno. |
| H20 | El POST no tiene límite de peticiones ni defensa contra spam. | `routes.ts:295` `router.post('/posts/:slug/comentarios', async ({ params, request, response }) => {` (sin `.use(...)`; no hay limiter en `config/` ni en `start/`). La semilla ya trae spam: seeder `:551` `autor_email: 'promo@plantillas-baratas.example'` | Se puede inundar la cola de pendientes. Lo nota el moderador. | 2: `routes.ts:295`, 1 configuración de limiter nueva | El código sí; el spam ya insertado, no. | A ninguno. |
| H21 | En producción, CORS no admite ningún origen, así que la web no puede llamar a la API de comentarios. | `config/cors.ts:21` `origin: app.inDev ? true : [],` | En producción no se pueden cargar ni enviar comentarios. Nadie lo nota todavía (solo hay entorno de desarrollo). | 2: `cors.ts:21`, `start/env.ts` (variable nueva) | Sí | A ninguno. Mismo problema de despliegue que **H31**. |
| H22 | Los comentarios no siguen la arquitectura objetivo: SQL, validación y reglas viven en las rutas, sin modelo, servicio, repositorio ni transformer. | `routes.ts:295`, `:339`, `:366` (handlers en línea con `db.rawQuery`); `app/models/` solo tiene `user.ts` | Ninguna regla vive en un solo sitio, y de ahí salen H1, H9 y H16. Lo nota el desarrollador. | 10: las 3 rutas, 6 ficheros nuevos (controlador, validador, servicio, repositorio, modelo y transformer) y el alias `#repositories/*` en `backend/package.json` | Sí | Desbloquea **H9**, **H16** y **H17**. |

## Web

| # | Qué es | Dónde (línea que lo demuestra) | Qué se rompe y quién lo nota | Sitios a tocar (contados) | ¿Revert? | Bloquea / desbloquea |
|---|---|---|---|---|---|---|
| H23 | Solo se pintan los comentarios raíz y sus hijos directos; se pierden las respuestas de segundo nivel y las huérfanas. | `Comentarios.jsx:50` `const raices = comentarios.filter((c) => !c.padre_id)`; `:63-64` `comentarios.filter((r) => r.padre_id === c.id)` | Una respuesta aprobada cuyo padre no esté visible no aparece nunca. Lo notaría quien respondió; nadie todavía (la semilla solo tiene la respuesta 2 → 1). | 1: bloque `Comentarios.jsx:50-74` | Sí | Lo bloquean **H6** y **H13**, que son los que generan huérfanas. |
| H24 | La página guarda dos recuentos de comentarios independientes que se desincronizan. | `DetallePost.jsx:30` `const [numComentarios, setNumComentarios] = useState(0)`; `Comentarios.jsx:11` `const [total, setTotal] = useState(0)` | Al enviar un comentario, la cabecera mantiene el número anterior y la sección suma uno. El lector ve dos cifras distintas en la misma página. | 6: `DetallePost.jsx:30`, `:38`, `:71`, `:123`; `Comentarios.jsx:11`, `:23` | Sí | Desbloquea **H25** y **H26**. |
| H25 | Al enviar, el total sube aunque el comentario quede pendiente y no se muestre. | `Comentarios.jsx:39` `setTotal(total + 1)` | El título dice «3 comentarios» y debajo solo hay 2. Contradice las reglas 2 y 5. Lo notan el lector y quien comenta. | 1: `Comentarios.jsx:39` | Sí | Lo bloquea **H24**. |
| H26 | El componente de comentarios vuelve a pedir el post entero, que `DetallePost` ya ha pedido. | `Comentarios.jsx:19` `fetch(API + '/posts/' + encodeURIComponent(slug))`, duplicado de `DetallePost.jsx:34` | Dos peticiones idénticas, y cada una lanza 5 o 6 consultas en el backend. Lo nota el operador. | 2: `Comentarios.jsx:17-26`, `DetallePost.jsx:123` | Sí | Lo bloquea **H24** (se resuelven juntos). |
| H27 | Los errores de carga se ignoran y la página muestra «0 comentarios». | `Comentarios.jsx:25` `.catch(() => {})` | Con la API caída, el lector cree que el post no tiene comentarios. | 1: `Comentarios.jsx:25` | Sí | A ninguno. |
| H28 | La web no permite editar, borrar ni responder comentarios. | `Comentarios.jsx:33` `body: JSON.stringify({ autor_nombre: nombre, autor_email: email, cuerpo: cuerpo })` (sin `padre_id`); no hay ningún `fetch` con PATCH o DELETE en `frontend/src` | El autor no puede corregir ni retirar su comentario, y el moderador no tiene interfaz. | 3: `Comentarios.jsx:33`, 1 handler de edición, 1 handler de borrado | Sí | Lo bloquea **H4**: poner el borrado en la web antes de arreglar la credencial facilita el abuso. |
| H29 | Las listas de comentarios usan el índice como `key`. | `Comentarios.jsx:57` `<div className="comentario" key={index}>`; `:66` `key={i}` | Si cambia el orden o se borra un comentario, React reutiliza nodos que no corresponden. No lo nota nadie todavía. | 2: `Comentarios.jsx:57`, `:66` | Sí | A ninguno. |
| H30 | El formulario no tiene `label`, `type="email"` ni `required`. | `Comentarios.jsx:78` `<input placeholder="Tu nombre" …`; `:79` `<input placeholder="Tu correo" value={email} …` | Quien usa lector de pantalla no tiene etiquetas, y no hay validación en el navegador antes de llamar a la API. | 3: `Comentarios.jsx:78`, `:79`, `:80-85` | Sí | A ninguno. |
| H31 | La URL de la API está escrita a mano en cada componente. | `Comentarios.jsx:4` `const API = 'http://localhost:3401'` (igual en `DetallePost.jsx:7`, `ListaPosts.jsx:4`, `PostsRelacionados.jsx:4`, `BarraLateral.jsx:4`, `App.jsx:7`) | Desplegar en otro host obliga a tocar 6 ficheros; si se olvida uno, ese componente deja de funcionar. Lo nota el desarrollador. | 7: los 6 ficheros y 1 módulo de configuración nuevo | Sí | A ninguno. Relacionado con **H21**. |

## Tests

| # | Qué es | Dónde (línea que lo demuestra) | Qué se rompe y quién lo nota | Sitios a tocar (contados) | ¿Revert? | Bloquea / desbloquea |
|---|---|---|---|---|---|---|
| H32 | El único test solo comprueba el status y un campo, sin aserciones negativas. | `tests/functional/comentarios.spec.ts:41-42` `respuesta.assertStatus(201)` / `assert.equal(respuesta.body().comentario.autor_nombre, 'Lucía Ferrer')` | No detecta que se devuelva `autor_email` (H8), que `estado` no sea 'pendiente' ni que se devuelva otro comentario (H7). Lo nota el desarrollador al refactorizar. | 1: `comentarios.spec.ts:41-42` | Sí | Desbloquea **H7** y **H8** (hay que caracterizarlos primero). |
| H33 | No hay tests de DELETE, de PATCH, de las ramas de validación, del 404, del filtro de pendientes ni de los recuentos. | `tests/functional/comentarios.spec.ts:15`: es el único `test(…)` del fichero, como reconoce su comentario de las líneas 8-10 | Ninguno de los hallazgos de API (H1-H19) está cubierto, así que cualquier arreglo se hace a ciegas. Lo nota el desarrollador. | 1 fichero con **18 tests**: POST 5 (4 validaciones + 404), DELETE 4 (autor, moderador, 403, 404), PATCH 5 (feliz, 403 email, 403 no pendiente, 400, 404), detalle 2 (oculta pendientes, sin email), recuentos 2 (`/posts`, relacionados) | Sí | Bloquea el arreglo de **H1-H19**: CLAUDE.md exige caracterizar antes de cambiar. |

---

## Descartados

- **CSRF desactivado** (`config/shield.ts:33` `enabled: false`): las rutas de comentarios no
  autorizan con sesión ni cookie, así que aquí no se puede explotar.
- **El `autor_email` del autor del post** en `GET /posts` y en el detalle (`routes.ts:74`, `:167`):
  incumple la regla 6, pero es del recurso posts, no de comentarios.
- **`app/exceptions/handler.ts` vacío** (`:15-17`): no es un hallazgo aparte; se cuenta como un
  sitio a tocar dentro de H9.

---

## Parte B: las tres líneas

1. **Cuántos hallazgos devolvió el agente, y cuántos comprobaste tú abriendo el archivo.** Los dos
   números, tal cual salieron. No los redondees ni los expliques.

   > 33 y 7

2. **El hallazgo del que no supiste decidir la prioridad, y en una frase, por qué.** Lo que interesa
   es qué dos cosas se contradecían: lo grave contra lo que bloquea, lo urgente contra lo caro, lo
   que te pedían contra lo que veías.

   > de muchos no supe decidir la prioridad, faltaria haber hecho la tabla de los ejes que dide la leccion y ordenar.

3. **Algo que el agente afirmó y no pudiste verificar.** Ojo al matiz: no es lo que dijo mal. Es lo
   que no pudiste comprobar ni a favor ni en contra, y te tocó dejar en el aire.

   > Que la web perdia respuestas huerfanas o de segundo nivel
