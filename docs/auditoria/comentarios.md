# Auditoría de la feature de comentarios

Alcance: solo la feature de comentarios tal como está en [mapeo.md](mapeo.md). Cada fila se
contrasta con las reglas de negocio de [README.md](../../README.md#reglas-de-negocio). No se ha
cambiado código. «Qué se rompe y quién lo nota» la he rellenado yo; «¿Bloquea o desbloquea a
otro?» la rellenó el agente a partir de su análisis de relaciones (prompts 5-7). Las filas 5 y 6
no están comprobadas.

La tabla se cerró pasados los 45 minutos; la parte B se escribió después.

Las rutas de los enlaces son relativas a este archivo (`docs/auditoria/`).

| # | Qué es | Dónde está | Qué se rompe y quién lo nota | Cuántos sitios hay que tocar | ¿Se deshace con un revert? | ¿Bloquea o desbloquea a otro? |
|---|---|---|---|---|---|---|
| 1 [Hecho] | ✔ El detalle público del post devuelve el correo de quien comenta, en contra de la regla 6, porque [routes.ts:153](../../backend/start/routes.ts#L153) hace `"select * from comentarios where post_id = ? and estado = 'aprobado' order by creado_en asc"` y [routes.ts:171](../../backend/start/routes.ts#L171) lo envía tal cual con `comentarios: comentarios,`. | [backend/start/routes.ts:153](../../backend/start/routes.ts#L153), [backend/start/routes.ts:171](../../backend/start/routes.ts#L171) | Cualquiera que abra la respuesta de GET /posts/:slug (red del navegador o API) ve el correo de todas las personas que han comentado ese post. No se ve en pantalla, así que hoy no lo nota nadie, pero el dato ya está expuesto. | 1: la forma de salida de los comentarios en `GET /posts/:slug` ([routes.ts:152-155](../../backend/start/routes.ts#L152-L155) o [routes.ts:171](../../backend/start/routes.ts#L171)) | Sí, el cambio es solo de código. Lo que ya se ha servido no se recupera | Bloquea a la 3 y a la 2: si cualquiera de las dos se arregla antes, el detalle empieza a mostrar los ids 3, 4 y 9 y expone tres correos más. Comparte con la 2 la línea [routes.ts:153](../../backend/start/routes.ts#L153). |
| 3 [Hecho] | ✔ La columna `estado` admite cualquier texto ([migración:18](../../backend/database/migrations/1769000000005_create_comentarios_table.ts#L18) `table.string('estado').notNullable().defaultTo('pendiente')`) y la semilla guarda variantes que las reglas 2 y 5 no contemplan: `estado: 'Aprobado',` ([seeder:510](../../backend/database/seeders/blog_seeder.ts#L510), [seeder:575](../../backend/database/seeders/blog_seeder.ts#L575)) y `estado: 'ok',` ([seeder:522](../../backend/database/seeders/blog_seeder.ts#L522)). | [1769000000005_create_comentarios_table.ts:18](../../backend/database/migrations/1769000000005_create_comentarios_table.ts#L18), [blog_seeder.ts:510](../../backend/database/seeders/blog_seeder.ts#L510), [blog_seeder.ts:522](../../backend/database/seeders/blog_seeder.ts#L522), [blog_seeder.ts:575](../../backend/database/seeders/blog_seeder.ts#L575) | Los comentarios con estado `'Aprobado'` o `'ok'` no se muestran en el detalle del post, aunque sí se cuentan en los recuentos. Podría notarlo el usuario, aunque no lo ve. | 5: una migración nueva que restrinja los valores (la de [L18](../../backend/database/migrations/1769000000005_create_comentarios_table.ts#L18) ya está aplicada), la normalización de los datos ya guardados, y las 3 líneas de la semilla ([510](../../backend/database/seeders/blog_seeder.ts#L510), [522](../../backend/database/seeders/blog_seeder.ts#L522), [575](../../backend/database/seeders/blog_seeder.ts#L575)) | No del todo: la restricción y la semilla sí, pero normalizar las filas reescribe datos y un revert de código no devuelve los valores originales (`'Aprobado'`, `'ok'`) salvo que el `down` de la migración los guarde | La bloquea la 1: normalizar `'Aprobado'` y `'ok'` hace visibles los ids 3, 4 y 9 en el detalle y expone sus correos. Desbloquea a la 2: con los datos normalizados, `= 'aprobado'` y `<> 'pendiente'` devuelven lo mismo y el síntoma desaparece. |
| 2 [Hecho] | ✔ La feature usa dos criterios distintos de «aprobado»: el detalle exige el texto exacto `"select * from comentarios where post_id = ? and estado = 'aprobado' order by creado_en asc"` ([routes.ts:153](../../backend/start/routes.ts#L153)), mientras que los recuentos aceptan todo lo que no sea pendiente con `c.estado <> ' +` `"'pendiente') as num_comentarios from posts"` ([routes.ts:75-76](../../backend/start/routes.ts#L75-L76)) y `"select count(*) as total from comentarios where post_id = ? and estado <> 'pendiente'"` ([routes.ts:223](../../backend/start/routes.ts#L223)), así que los aprobados guardados como `'Aprobado'` u `'ok'` cuentan en el recuento de la regla 5 pero no se muestran en el detalle. | [backend/start/routes.ts:153](../../backend/start/routes.ts#L153) (`GET /posts/:slug`), [backend/start/routes.ts:75-76](../../backend/start/routes.ts#L75-L76) (`GET /posts`), [backend/start/routes.ts:223](../../backend/start/routes.ts#L223) (`GET /posts/:slug/relacionados`) | Quien comenta: 3 comentarios aprobados (ids 3, 4 y 9) no se publican nunca. El lector: la portada anuncia más comentarios de los que enseña el post (post 1: 4 frente a 2; post 6: 1 frente a 0). | 3: [routes.ts:153](../../backend/start/routes.ts#L153), [routes.ts:75-76](../../backend/start/routes.ts#L75-L76) y [routes.ts:223](../../backend/start/routes.ts#L223) (o 1 si solo se alinea el detalle con los recuentos) | Sí, el cambio es solo de código | La bloquea la 1: mostrar los ids 3, 4 y 9 expone sus correos. La desbloquea la 3: hecha la 3, aquí puede bastar 1 sitio o ninguno. Comparte con la 1 la línea [routes.ts:153](../../backend/start/routes.ts#L153). |
| 4 [Hecho] | ✔ Al enviar un comentario, que nace `pendiente`, la web suma uno al recuento visible con `setTotal(total + 1)` ([Comentarios.jsx:39](../../frontend/src/componentes/Comentarios.jsx#L39)), en contra de la regla 5. | [frontend/src/componentes/Comentarios.jsx:39](../../frontend/src/componentes/Comentarios.jsx#L39) | Quien acaba de comentar: tras enviar, el título de la sección suma uno aunque su comentario está pendiente y no aparece en la lista. Solo él, y solo hasta recargar. | 1: [Comentarios.jsx:39](../../frontend/src/componentes/Comentarios.jsx#L39) | Ni bloquea ni desbloquea a la 1, la 2 ni la 3. Arreglar la 2 o la 3 cambia el número del que parte el contador ([Comentarios.jsx:23](../../frontend/src/componentes/Comentarios.jsx#L23)), pero el `+1` sigue estando mal. |
| 5 [Riesgo]| La web calcula el recuento de la regla 5 en cuatro sitios sin una fuente común: dos contadores locales con `setNumComentarios((d.comentarios \|\| []).length)` ([DetallePost.jsx:38](../../frontend/src/componentes/DetallePost.jsx#L38)) y `setTotal((d.comentarios \|\| []).length)` ([Comentarios.jsx:23](../../frontend/src/componentes/Comentarios.jsx#L23)), y dos que pintan el `num_comentarios` de la API con `{p.num_comentarios} comentarios` ([ListaPosts.jsx:57](../../frontend/src/componentes/ListaPosts.jsx#L57), [PostsRelacionados.jsx:32](../../frontend/src/componentes/PostsRelacionados.jsx#L32)). | [DetallePost.jsx:30](../../frontend/src/componentes/DetallePost.jsx#L30), [:38](../../frontend/src/componentes/DetallePost.jsx#L38), [:71](../../frontend/src/componentes/DetallePost.jsx#L71); [Comentarios.jsx:11](../../frontend/src/componentes/Comentarios.jsx#L11), [:23](../../frontend/src/componentes/Comentarios.jsx#L23), [:54](../../frontend/src/componentes/Comentarios.jsx#L54); [ListaPosts.jsx:57](../../frontend/src/componentes/ListaPosts.jsx#L57); [PostsRelacionados.jsx:32](../../frontend/src/componentes/PostsRelacionados.jsx#L32) | | 4 componentes, 8 líneas: [DetallePost.jsx:30, 38, 71](../../frontend/src/componentes/DetallePost.jsx#L30); [Comentarios.jsx:11, 23, 54](../../frontend/src/componentes/Comentarios.jsx#L11); [ListaPosts.jsx:57](../../frontend/src/componentes/ListaPosts.jsx#L57); [PostsRelacionados.jsx:32](../../frontend/src/componentes/PostsRelacionados.jsx#L32) | Sí, el cambio es solo de código | |
| 6 [Riesgo]| La web solo pinta las respuestas cuyo padre es un comentario raíz visible (`const raices = comentarios.filter((c) => !c.padre_id)` en [Comentarios.jsx:50](../../frontend/src/componentes/Comentarios.jsx#L50) y `.filter((r) => r.padre_id === c.id)` en [Comentarios.jsx:64](../../frontend/src/componentes/Comentarios.jsx#L64)), así que una respuesta aprobada a un padre no aprobado o a otra respuesta nunca se muestra, aunque la regla 2 solo oculta los pendientes. | [frontend/src/componentes/Comentarios.jsx:50](../../frontend/src/componentes/Comentarios.jsx#L50), [frontend/src/componentes/Comentarios.jsx:63-64](../../frontend/src/componentes/Comentarios.jsx#L63-L64) | | 1: el bloque de pintado de [Comentarios.jsx:50-74](../../frontend/src/componentes/Comentarios.jsx#L50-L74) | Sí, el cambio es solo de código | |

## Parte B

1. **Hallazgos devueltos por el agente: 18** (17 en la primera pasada). **Comprobados abriendo el archivo: 4** (filas 1, 2, 3 y 4).
2. **Fila 1.** Chocaban dos razones para colocarla: tiene el daño más alto y bloquea a la 3 y a la 2 (arreglarlas antes expondría tres correos más), pero su arreglo de fondo, elegir qué campos salen, se escribe contra el modelo de `estado` que la 3 va a cambiar.
3. **El borrado.** El agente afirmó que cualquiera puede borrar un comentario ajeno, porque el DELETE autoriza con el correo de quien comentó ([routes.ts:353](../../backend/start/routes.ts#L353)) y ese correo es público por la fila 1. No llegué a abrir esa línea ni a probar un borrado, así que no puedo confirmarlo ni descartarlo, y por eso se queda fuera de la tabla.

## Comprobaciones hechas

Medido el 2026-11-22 con peticiones `GET` contra la API local (`localhost:3401`), con los datos de
la semilla. No se ha hecho ningún POST, PATCH ni DELETE.

| Post | `num_comentarios` en `GET /posts` | Comentarios en `GET /posts/:slug` | Aprobados que no salen en el detalle | Filas 1-6 afectadas |
|---|---|---|---|---|
| 1 `diez-tendencias-…` | 4 (ids 1-4) | 2 (ids 1 y 2) | id 3 (`'Aprobado'`), id 4 (`'ok'`) | 2, 3, 5 |
| 6 `que-mide-de-verdad-…` | 1 (id 9) | 0 | id 9 (`'Aprobado'`) | 2, 3, 5 |
| 2 y 4 | 1 | no medido | — | — |

Valores de `estado` en la base local (`select estado, count(*) … group by estado`, solo lectura):
`aprobado` (ids 1, 2, 6, 8), `Aprobado` (ids 3, 9), `ok` (id 4) y `pendiente` (ids 5, 7). No hay
ningún otro valor, así que hoy `<> 'pendiente'` cuenta exactamente los aprobados.

El detalle del post 1 devuelve `"autor_email":"ana.cortes@ejemplo.es"` y
`"autor_email":"marta.ruiz@corriente.es"` dentro de `comentarios` (fila 1).

Las filas 4 y 6 no se han medido. La 4 necesita un POST, y la 6 un dato que la semilla no tiene
(una respuesta aprobada con un padre no aprobado).

## Reglas del README que la feature sí cumple

- **Regla 2**: el detalle filtra con `estado = 'aprobado'`
  ([routes.ts:153](../../backend/start/routes.ts#L153)), así que los pendientes (ids 5 y 7) no
  salen. Medido en el post 1.
- **Regla 5**, en los recuentos de la API: con los datos actuales, `estado <> 'pendiente'`
  ([routes.ts:75-76](../../backend/start/routes.ts#L75-L76),
  [routes.ts:223](../../backend/start/routes.ts#L223)) cuenta solo aprobados, si `'Aprobado'` y
  `'ok'` se leen como aprobados. Así lo indican los comentarios de la semilla
  ([seeder:509](../../backend/database/seeders/blog_seeder.ts#L509) «entró por el panel viejo de
  moderación», [seeder:521](../../backend/database/seeders/blog_seeder.ts#L521)) y el de
  [routes.ts:150-151](../../backend/start/routes.ts#L150-L151). Medido en la base local. La
  fragilidad de ese filtro está en la propuesta 3.
- **Regla 1**: no se puede comentar un borrador. `POST /posts/:slug/comentarios` busca con
  `estado = 'publicado'` ([routes.ts:297](../../backend/start/routes.ts#L297)) y responde 404.
  Leído en el código, no medido.
- **Reglas 3 y 4**: no aplican a comentarios.

## Propuestas sin medir (fuera de la tabla)

Son hipótesis. Ninguna se ha observado.

1. **El POST puede devolver el comentario de otra persona.** Tras el insert, el endpoint lee la
   última fila de toda la tabla con `'select * from comentarios order by id desc limit 1'`
   ([routes.ts:330](../../backend/start/routes.ts#L330)). Si dos peticiones se intercalan, una
   podría recibir la fila, y con ella el correo, de la otra.
   - **Cómo medirlo:** un test funcional que lance N POST simultáneos (`Promise.all`) a posts
     distintos y afirme que cada respuesta trae el `autor_email` y el `post_id` que envió. Repetirlo
     varias veces y contar los cruces.
2. **Los recuentos pueden ser lentos sin índice en `post_id`.** La columna no tiene índice
   ([migración:14](../../backend/database/migrations/1769000000005_create_comentarios_table.ts#L14)),
   y la subconsulta de [routes.ts:75](../../backend/start/routes.ts#L75) se ejecuta una vez por
   post del listado.
   - **Cómo medirlo:** `EXPLAIN QUERY PLAN` de esa consulta en `backend/tmp/db.sqlite3` para ver si
     hace `SCAN comentarios`, y medir el tiempo de `GET /posts` con una tabla de comentarios
     sembrada a 10⁴ y 10⁵ filas, antes y después de un índice.
3. **El recuento dejaría de cumplir la regla 5 si aparece un estado nuevo.** `<> 'pendiente'`
   ([routes.ts:75-76](../../backend/start/routes.ts#L75-L76),
   [routes.ts:223](../../backend/start/routes.ts#L223)) contaría como aprobado cualquier otro
   valor, por ejemplo `'rechazado'` o `'spam'`. La columna lo admite (fila 3). Hoy no existe
   ninguno en la base local.
   - **Cómo medirlo:** `select estado, count(*) from comentarios group by estado` sobre los datos
     reales, no sobre la semilla, para ver si hay algún valor que no sea una variante de aprobado
     ni `'pendiente'`.

## Recuento

Hay **18 hallazgos** en total:

- **6 en la tabla**, contrastados con una regla del README.
- **3 propuestas sin medir**, en la sección anterior.
- **9 que no están en la tabla**, porque no hay una regla del README contra la que contrastarlos.
  Algunos sí chocan con reglas de `CLAUDE.md`. Quedan anotados para decidir si entran:
  1. **Cualquiera puede borrar un comentario.** El DELETE autoriza a quien traiga el correo de
     quien comentó
     (`String(comentario.autor_email).toLowerCase().trim() === String(email).toLowerCase().trim()`,
     [routes.ts:353](../../backend/start/routes.ts#L353)), y ese correo es público por la fila 1.
  2. **Clave de moderación en el código.** Está escrita en el fuente y compartida con las sesiones:
     `const SECRETO_SESION = 'dev-secret-cambiar'`
     ([routes.ts:25](../../backend/start/routes.ts#L25)).
  3. **DELETE y PATCH autorizan de forma distinta.** El PATCH compara el correo distinguiendo
     mayúsculas y no admite la clave de moderación:
     `if (comentario.autor_email !== email || comentario.estado !== 'pendiente')`
     ([routes.ts:379](../../backend/start/routes.ts#L379)).
  4. **Un cuerpo corto responde 200.** El POST devuelve 200 para un cuerpo demasiado corto
     (`return response.send({ ok: false, motivo: 'comentario demasiado corto' })`,
     [routes.ts:320](../../backend/start/routes.ts#L320)). Además, sus errores tienen cuatro formas
     distintas ([routes.ts:310-321](../../backend/start/routes.ts#L310-L321)).
  5. **El PATCH acepta un cuerpo de un carácter.** Solo comprueba `if (!cuerpo) {`
     ([routes.ts:384](../../backend/start/routes.ts#L384)), mientras que el POST exige 5 caracteres.
  6. **`padre_id` no se valida.** Se acepta sin comprobar que exista ni que sea del mismo post:
     `const padre = request.input('padre_id', null)`
     ([routes.ts:308](../../backend/start/routes.ts#L308)).
  7. **POST y PATCH devuelven la fila completa con `autor_email`**
     ([routes.ts:333](../../backend/start/routes.ts#L333) y
     [routes.ts:392](../../backend/start/routes.ts#L392)). Al ser la respuesta a quien envió ese
     correo, no está claro que la regla 6 aplique.
  8. **La web pide el post entero dos veces.** `Comentarios` vuelve a pedirlo solo para sacar los
     comentarios: `fetch(API + '/posts/' + encodeURIComponent(slug))`
     ([Comentarios.jsx:19](../../frontend/src/componentes/Comentarios.jsx#L19)).
  9. **La feature tiene un solo test.** Solo cubre el camino feliz del POST
     ([comentarios.spec.ts:15](../../backend/tests/functional/comentarios.spec.ts#L15)).
