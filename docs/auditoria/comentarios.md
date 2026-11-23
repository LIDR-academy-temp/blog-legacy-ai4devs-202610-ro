# Auditoría — feature de comentarios (crear · editar · borrar · listar)

Fecha: 2026-11-24 · Rama: `auditoria-ims`

## Alcance

Según el README, una feature es «sus rutas en la API, su tabla en la base de datos y lo que la web enseña de ella». Para comentarios eso es:

- **API:** `POST /posts/:slug/comentarios`, `PATCH /comentarios/:id` y `DELETE /comentarios/:id`, más el bloque `comentarios` de `GET /posts/:slug`, que es el listado.
- **Tabla:** la migración `1769000000005_create_comentarios_table.ts` y las filas de `COMENTARIOS` en la semilla.
- **Web:** `Comentarios.jsx`, y el recuento de comentarios que enseñan `DetallePost.jsx`, `ListaPosts.jsx` y `PostsRelacionados.jsx`.

Queda fuera todo lo demás de posts, autores, categorías y etiquetas, aunque viva en el mismo archivo.

Rutas relativas a `backend/` salvo las de `frontend/`.

## Cómo está ordenada

1. **Precedencia.** Nivel 1: lo que desbloquea o abarata a otro hallazgo, en orden de dependencia. Nivel 2: hojas.
2. **Dentro de cada nivel:** daño ÷ radio. Escala de daño: alto 3, medio 2, bajo 1, «nadie todavía» 0,5.
3. **Hallazgos partidos en dos:** si un hallazgo tiene mucho daño y poca precedencia, su contención va en el nivel 1 y el fondo espera en su sitio. Aun así, ninguna contención va antes que la caracterización (#1), porque `CLAUDE.md` exige que esté en verde antes de tocar nada.

**Comprobado a mano:**

- Con la base sembrada: el post 1 anuncia 4 comentarios en el listado y enseña 2 en el detalle; el post 6 anuncia 1 y enseña 0.
- Con `better-sqlite3`: enlazar un objeto en el insert lanza un `TypeError`, que llega al cliente como 500.

## Tabla

| # | Qué es | Dónde (línea que lo demuestra) | Qué se rompe y quién lo nota | Sitios a tocar | ¿Se deshace con revert? | ¿Bloquea / desbloquea? |
|---|---|---|---|---|---|---|
| **Nivel 1 — desbloquean o abaratan** ||||||
| 1 | Solo hay un test de comentarios: el camino feliz del POST, que comprueba un campo. | `tests/functional/comentarios.spec.ts:41-42` | Nadie todavía. Pero no hay ningún test de PATCH, DELETE, 404, validación ni de que no salga el correo, así que cualquier cambio de #2 a #22 va a ciegas. | 1 (el spec) | Sí | Desbloquea a todos los demás. |
| 2 | **Contención:** el listado de comentarios del detalle se lee con `select *` y devuelve el correo de quien comenta. | `start/routes.ts:153` → `:171` | Cualquier visitante o *scraper* se lleva los correos de los comentaristas. Incumple la regla 6. Con ese correo se borra o se edita el comentario (#8). | 1 (`:153`: columnas explícitas, conservando `id` y `padre_id`, que usa `Comentarios.jsx:50,64`) | Sí | Lo bloquea #1. Abarata #8. El fondo es #19. |
| 3 | **Contención:** la clave que autoriza a borrar cualquier comentario está escrita en el código. | `start/routes.ts:25` (`'dev-secret-cambiar'`), comparada en `:351` | Cualquiera que lea el repo (los forks son públicos) borra cualquier comentario con la cabecera `x-clave-moderacion`. | 4 (`routes.ts:25`, `start/env.ts`, `.env.example`, `.env.test`) | **No:** el valor ya está en el historial de git. Hay que rotarlo, y un revert lo devolvería al código. | Lo bloquea #1. Abarata #8, del que es contención. |
| 4 | `estado` es texto libre y conviven `'aprobado'`, `'Aprobado'` y `'ok'`. | migración `1769000000005_…:18` (sin CHECK). En la semilla: `estado: 'Aprobado'` (ids 3 y 9) y `'ok'` (id 4). | Lectores: el detalle esconde los comentarios 3, 4 y 9 (`routes.ts:153` compara exacto), y los listados sí los cuentan (#13). Antes hay que decidir qué valores significan «aprobado». | 4 (1 migración que normaliza y añade el CHECK, y 3 valores de la semilla) | **No:** el UPDATE pierde el valor original y el `down` no sabe cuáles eran `'ok'`. | Lo bloquea #1. Desbloquea #13. |
| 5 | Los tres endpoints y el listado viven en *closures* de `routes.ts`: no hay controlador, validador, servicio, repositorio, transformer ni modelo. | `start/routes.ts:295-393` y `:152-155`. En `app/models/` solo está `user.ts`. | Nadie de forma directa. Por esto las reglas se han copiado y han acabado distintas (#6). | 8 (`routes.ts`, controlador, validador, servicio, repositorio, transformer, modelo y alias `#repositories/*` en `package.json`) | Sí | Lo bloquea #1. Desbloquea #6, #7 y #19. |
| 6 | Editar y borrar tienen cada uno su regla de autorización, y no coinciden. | `start/routes.ts:353` (minúsculas, `trim`, admite la clave) frente a `:379` (correo exacto, sin clave, y solo si está pendiente). El propio comentario de `:375-377` lo reconoce. | Quien comenta y escribe su correo con otra capitalización puede borrar, pero no editar. Ninguna regla escrita decide si editar un comentario aprobado está permitido. | 2 → 1 política en el servicio | Sí | Lo bloquea #5. Desbloquea #8. |
| 7 | Cada error de validación tiene un código y una forma distintos, y el del cuerpo corto es un 200. | `start/routes.ts:311` (400 `error`), `:314` (422 `message`), `:317` (400 `errores[]`), `:320` (**200** `{ok:false}`), `:385`. `app/exceptions/handler.ts:15-17` está sin tocar. | Un cliente de la API da por bueno un comentario corto porque recibe 200. En la web, un correo sin `@` enseña «No se ha podido enviar.», porque `Comentarios.jsx:44` no lee `errores`. | 7 (4 ramas del POST, 1 del PATCH, `handler.ts`, `Comentarios.jsx:44`) | Sí | Lo bloquea #5. Desbloquea #11 y #14. |
| 8 | **Fondo de #2 y #3:** para editar o borrar no se comprueba ninguna identidad; basta con un correo o con una clave compartida. | `start/routes.ts:349-353` y `:378-379` (`request.input('autor_email')`, `request.header('x-clave-moderacion')`) | Quien sepa o adivine un correo, o tenga la clave, borra (de forma física) o edita el comentario de otro. | 5 (migración con token de gestión, política, validador, `auth` en las rutas y `Comentarios.jsx`) | **No:** los tokens que se entreguen a la gente dejan de valer si se revierte. | Lo bloquea #6. #2 y #3 lo abaratan. Desbloquea #9 y #16. |
| **Nivel 2 — hojas** ||||||
| 9 | La web no deja editar ni borrar comentarios. | `frontend/src/componentes/Comentarios.jsx:28-48`: solo hay un `POST` | Quien comenta no puede corregir ni retirar lo que escribió. Dos de las cuatro operaciones no se pueden usar desde la web. | 1 (`Comentarios.jsx`) | Sí | Lo bloquea #8. |
| 10 | Al crear un comentario se devuelve «el último de la tabla», no el que se acaba de insertar. | `start/routes.ts:329-331` (`order by id desc limit 1`) | Si dos personas comentan a la vez, una puede recibir el comentario de la otra, con su correo. Nadie todavía, con el tráfico actual. *(Plausible, no reproducido.)* | 1 | Sí | A ninguno. |
| 11 | El POST no valida el tipo, el formato ni la longitud de la entrada. | `start/routes.ts:305-307` (`request.input` sin tipo), `:316` (solo busca `@`), `:319` (sin `trim` ni máximo) | Un cliente que manda `cuerpo` como objeto recibe un **500** (comprobado). Los moderadores reciben correos como `a@`, cuerpos de 5 espacios y textos sin límite. | 1 (el validador de #7) | Sí | Lo bloquea #7. |
| 12 | Al borrar un comentario se quedan sus respuestas con un `padre_id` que ya no existe. | `start/routes.ts:359` (`delete` sin mirar los hijos) | Quien respondió: su respuesta desaparece de la web (`Comentarios.jsx:50,63-64`), pero sigue en la tabla y en los recuentos. | 1 (servicio: cascada o borrado lógico) | **No:** el borrado es físico y revertir el código no devuelve las filas. | A ninguno. |
| 13 | El recuento de los listados y el listado del detalle usan criterios distintos de «aprobado». | `start/routes.ts:75-76` y `:223` (`<> 'pendiente'`) frente a `:153` (`= 'aprobado'`) | Lectores de la portada y de los relacionados: el post 1 anuncia 4 y enseña 2. Incumple la regla 5. | 2 (`:76` y `:223` alineados con lo que decida #4) | Sí | Lo bloquea #4. |
| 14 | Al editar no se exige la longitud mínima que sí se exige al crear. | `start/routes.ts:384` (solo `!cuerpo`) frente a `:319` (`< 5`) | El autor puede dejar su comentario en «a». Nadie todavía, porque la web no edita (#9). | 1 (reutilizar el validador de #7) | Sí | Lo bloquea #7. |
| 15 | El front lleva dos recuentos por separado, y uno suma 1 al enviar un comentario pendiente. | `frontend/src/componentes/Comentarios.jsx:39` (`setTotal(total + 1)`) y `DetallePost.jsx:38` | Quien comenta: la sección dice 3, la cabecera 2 y se ven 2. Incumple la regla 5 en pantalla. | 2 | Sí | A ninguno. |
| 16 | Ningún endpoint cambia el `estado` de un comentario, así que los nuevos no llegan nunca al listado. | `start/routes.ts:326` (siempre `'pendiente'`). En `:295-393` no hay ningún `update … estado`. | Toda persona que comenta: su comentario no aparece nunca, aunque la web le dice «pendiente de moderación» (`Comentarios.jsx:38`). Crear un endpoint de moderación es una decisión de producto, no un arreglo. | 4 (ruta, controlador, validador y servicio) | Sí | Lo bloquea #8, que tiene que decir quién modera. |
| 17 | `post_id` y `padre_id` no tienen ni clave foránea ni índice. | migración `1769000000005_…:14` y `:20` | Nadie todavía. Cada listado del detalle recorre la tabla entera, y nada impide un `padre_id` que no existe (#20). | 1 (migración) | **No**, si para crear la FK antes hay que borrar huérfanos. | A ninguno. |
| 18 | La fecha se guarda en UTC sin zona y la web la recorta a 10 caracteres. | `start/routes.ts:323` (`toISOString().slice(0,19)`), `Comentarios.jsx:59` y `:68` | Lectores: un comentario de las 00:30 en Madrid aparece fechado el día anterior. | 3 | Sí | A ninguno. |
| 19 | **Fondo de #2:** ninguna respuesta de comentarios pasa por un transformer. | `start/routes.ts:171`, `:333`, `:392` (filas enteras) | Después de #2 solo lo nota el autor, que recibe su propio correo, `estado` y `post_id`. Con #10, se lo llevaría otra persona. | 4 (transformer y 3 respuestas) | Sí | Lo bloquea #5. |
| 20 | `padre_id` entra sin validar: puede apuntar a un comentario que no existe, a uno de otro post o a uno pendiente. | `start/routes.ts:308` → `:326` | Nadie todavía (la web no envía `padre_id`, `Comentarios.jsx:33`). Cuando lo haga, esas respuestas no se verán nunca. | 2 (validador y comprobación en el repositorio) | Sí | A ninguno. |
| 21 | La web solo pinta las respuestas de primer nivel. | `frontend/src/componentes/Comentarios.jsx:50` y `:63-64` | Nadie todavía (la semilla solo tiene un nivel). Una respuesta a una respuesta no se vería. | 2 | Sí | A ninguno. |
| 22 | La web vuelve a pedir el post entero solo para sacar los comentarios. | `frontend/src/componentes/Comentarios.jsx:19`, que repite `DetallePost.jsx:34` | Hay dos peticiones del detalle por cada visita. Nadie más lo nota. | 2 (`Comentarios.jsx:19` y pasar los comentarios desde `DetallePost.jsx`) | Sí | A ninguno. |

## Lo que se revisó y está bien

- Todo el SQL de comentarios va con parámetros (`?`): no hay inyección.
- No se puede comentar en un borrador (`routes.ts:297`).
- La regla 2 se cumple: el detalle no enseña los pendientes (`:153`).
- React escapa `cuerpo` y `autor_nombre`: no hay XSS.

PARTE B
1. Cuántos hallazgos devolvió el agente: 22. Cuántos he comprobado hasta ahora abriendo el archivo: 7.
2. En los analizados, todavía no he encontrado ningun contradicción.
3. En el hallazgo 13 se indica que no se cumple la regla 5, y se podría considerar cierto. Al no tener contexto del significado real 
   del estado "ok", no podemos afirmar al 100% que ese estado sea equivalente a "Aprobado" y se está mostrando.
