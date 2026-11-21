# Auditoría priorizada — feature de comentarios

**Alcance:** crear, editar, borrar y listar comentarios en sus tres capas.

- **API:** `backend/start/routes.ts`, que contiene `POST /posts/:slug/comentarios`, `PATCH /comentarios/:id`, `DELETE /comentarios/:id` y la parte de comentarios de `GET /posts/:slug`, `GET /posts` y `GET /posts/:slug/relacionados`.
- **Base de datos:** `backend/database/migrations/1769000000005_create_comentarios_table.ts` y la semilla `backend/database/seeders/blog_seeder.ts`.
- **Web:** `frontend/src/componentes/Comentarios.jsx` y `DetallePost.jsx`.

Queda fuera, aunque se vio durante la auditoría: el post también expone `autor_email` en `GET /posts/:slug` (`routes.ts:111`, `:167`). Es de la feature de posts.

**Criterio de orden:** primero la precedencia, es decir, lo que desbloquea o abarata a otro hallazgo va antes. Dentro de cada nivel se ordena por daño ÷ radio.

**Cómo leer la columna «¿Revert?»:** si se aplica el arreglo y después se revierte, ¿todo vuelve a quedar como estaba?

## Tabla

| # | Qué es | Dónde está | Qué se rompe y quién lo nota | Sitios | ¿Se deshace con un revert? | ¿Bloquea o desbloquea? |
|---|---|---|---|---|---|---|
| **Nivel 0 — contenciones: cortan el daño hoy** |||||||
| 1 | **Contención de #10.** El detalle del post devuelve el correo de cada comentarista, porque los comentarios salen con `select *`. | `routes.ts:153` (`select * from comentarios …`) y `:171` (`comentarios: comentarios`) | Cualquiera que haga `GET /posts/:slug` obtiene los correos de los comentaristas. Esto les afecta a ellos (spam, suplantación) y es la llave de #9. | 1 (la consulta de `:153`) | Sí en código. Los correos que ya se leyeron no se pueden retirar. | Reduce el radio de #9: sin correos públicos, borrar por correo deja de estar al alcance de cualquiera. El arreglo de fondo es #10. |
| 2 | La clave de moderación es un literal en el código y, por tanto, está en el historial de git. | `routes.ts:25` (`const SECRETO_SESION = 'dev-secret-cambiar'`) y `:351` | Cualquiera con acceso al repo borra cualquier comentario enviando la cabecera `x-clave-moderacion: dev-secret-cambiar`. Lo notan los comentaristas y el moderador, y no queda rastro. | 4 (`routes.ts:25`, `routes.ts:351`, `start/env.ts`, `.env.example`) | **No.** El valor sigue en el historial de git, así que un revert no lo retira: hay que rotar la clave. | A ninguno. |
| **Nivel 1 — desbloquean el resto** |||||||
| 3 | Solo hay un test de comentarios, y afirma un único campo del camino feliz. | `tests/functional/comentarios.spec.ts:41-42` | Nadie todavía. Quien refactorice puede cambiar el contrato sin que nada falle. | 1 archivo (ampliar con la matriz de los 3 endpoints de escritura más el listado de comentarios del detalle) | Sí. | **Desbloquea** #5, #6, #7, #9, #10, #11, #12 y #13. Ninguno se puede tocar con seguridad sin caracterizar antes. |
| 4 | `comentarios.estado` es texto libre sin `CHECK`, y conviven `'aprobado'`, `'Aprobado'` y `'ok'`. | migración `:18` (`table.string('estado')…`); semilla `:510`, `:522` y `:575` | Los comentarios aprobados 3, 4 y 9 no aparecen en el detalle, porque `routes.ts:153` compara exacto con `'aprobado'`. Lo notan los lectores y esos tres comentaristas. | 4 (una migración nueva con UPDATE y CHECK, más 3 líneas de la semilla) | **No.** Normalizar con UPDATE pierde el valor original (`'ok'` frente a `'Aprobado'`) si no se guarda copia. Además, decidir si `'ok'` cuenta como aprobado es una decisión de negocio. | **Desbloquea** #7: sin normalizar antes, contar solo `= 'aprobado'` escondería 3 comentarios más. |
| **Nivel 2 — estructura y reglas compartidas** |||||||
| 5 | Toda la lógica de comentarios vive en las rutas: no hay controlador, validador, servicio, repositorio ni transformer. | `routes.ts:295-393` | Nadie fuera del equipo. Cada regla se reescribe a mano y diverge (ver #6). | 10 (3 handlers, 5 archivos nuevos, el alias `#repositories/*` en `package.json` y la consulta de `:153`) | Sí. | Lo bloquea #3. **Desbloquea** #6, #10 y #11, que se resuelven dentro de esta capa. |
| 6 | Las reglas de autoría de DELETE y PATCH están escritas dos veces y no coinciden. | `routes.ts:351-353` (moderador o correo sin distinguir mayúsculas) frente a `:379` (correo exacto, sin moderador, solo si está pendiente) | El moderador no puede editar. Quien escribe `Ana@x.es` puede borrar su comentario pero no editarlo. Lo notan los comentaristas y el moderador. | 2 | Sí. | Lo bloquea #3. **Desbloquea** #9, porque deja la autorización en un solo sitio. |
| 7 | El recuento de comentarios cuenta todo lo que no es `'pendiente'`, no solo los aprobados (incumple la regla 5). | `routes.ts:75-76` y `:223` (`estado <> 'pendiente'`) | En el post 1, el listado dice 4 comentarios y el detalle muestra 2. Lo notan los lectores del listado y de los relacionados. | 2 | Sí. | Lo bloquea #4. |
| **Nivel 3 — arreglos de la API, por daño ÷ radio** |||||||
| 8 | El POST devuelve el último comentario de la tabla, no el que acaba de insertar. | `routes.ts:329-331` (`select * from comentarios order by id desc limit 1`) | Si llegan dos envíos a la vez, uno recibe el comentario del otro, con su correo incluido. Nadie todavía, por el poco tráfico. | 1 | Sí. | A ninguno. |
| 9 | La única prueba de autoría para borrar es el correo enviado en el cuerpo de la petición. | `routes.ts:350-353` | Junto con #1, cualquier visitante puede borrar cualquier comentario aprobado. El borrado es físico (`:359`). Lo notan los comentaristas y los lectores. | 4 (migración con el secreto de autoría, POST `:326/:333`, DELETE `:350-353` y PATCH `:378-379`) | El código sí. **Los comentarios borrados no vuelven**: no hay papelera. | Lo bloquean #3 y #6. **Bloquea** #20. |
| 10 | **Fondo de #1.** Ninguna respuesta de comentarios pasa por un transformer: salen filas crudas. | `routes.ts:171`, `:333` y `:392` | Cualquier columna nueva de `comentarios` se publicará sola. Hoy se ve en #1. | 4 (3 respuestas y el transformer nuevo) | Sí. | Lo bloquea #3 y se hace dentro de #5. Sustituye a #1. |
| 11 | La validación del POST son cuatro `if` con tres códigos de estado y tres formas de error distintas, y un cuerpo corto devuelve 200. | `routes.ts:310-321` (400 `{error}`, 422 `{message}`, 400 `{errores}`, 200 `{ok:false}`) | Los clientes de la API no pueden tratar los errores de forma uniforme. La web lo esquiva leyendo `d.error \|\| d.message \|\| d.motivo` (`Comentarios.jsx:44`). | 4 (`routes.ts:310-321`, validador nuevo, `exceptions/handler.ts:15`, `Comentarios.jsx:44`) | Sí. | Lo bloquea #3, que debe caracterizar el `200 {ok:false}` antes de cambiarlo. **Desbloquea** #12 y #13, que reutilizan el mismo validador. |
| 12 | El PATCH acepta cualquier cuerpo no vacío y se salta el mínimo de 5 caracteres del POST. | `routes.ts:384` (`if (!cuerpo)`) frente a `:319` (`length < 5`) | Un comentarista puede dejar su comentario en `"a"`. Nadie todavía. | 1 | Sí. | Lo bloquea #11. |
| 13 | `padre_id` se guarda sin validar: puede apuntar a un comentario inexistente, de otro post o pendiente. | `routes.ts:308` y `:326` | La respuesta no aparece nunca en la web, que solo enlaza un nivel (`Comentarios.jsx:50`, `:63-64`). Lo nota quien responde. | 2 (validación en la API y el render en la web) | Sí. | Lo bloquea #11. Se abarata con #15. |
| 14 | Borrar un comentario deja sus respuestas huérfanas: invisibles en la web, pero contadas en el listado. | `routes.ts:359`; `Comentarios.jsx:50` (`!c.padre_id`); `routes.ts:75` | Si se borra el comentario 1 del post 1, la respuesta 2 desaparece de la web y sigue sumando en el recuento. La web muestra "1 comentarios" con la lista vacía. Lo notan los lectores. | 1 (decidir la política en el DELETE) | El código sí. Lo borrado no vuelve. | Se abarata con #15. |
| 15 | `post_id` y `padre_id` no tienen clave foránea ni índice. | migración `:14` y `:20` | Pueden quedar comentarios huérfanos de posts o padres que ya no existen. Nadie todavía, porque no hay endpoint para borrar posts. | 1 (migración nueva) | **No del todo.** En SQLite, añadir una FK obliga a recrear la tabla y limpiar antes los huérfanos, y los que se limpien no vuelven. | Abarata #13 y #14, pero no los bloquea. |
| 16 | DELETE y PATCH no comprueban si el post del comentario sigue publicado. | `routes.ts:340-342` y `:367-369` | Se pueden editar o borrar comentarios de un post que ha vuelto a borrador. Nadie todavía. | 2 | Sí. | A ninguno. |
| **Nivel 4 — web** |||||||
| 17 | Los comentarios se piden dos veces en cada vista del post. | `Comentarios.jsx:19` y `DetallePost.jsx:34` | Dos peticiones idénticas a `GET /posts/:slug`. Nadie lo nota. | 2 | Sí. | **Desbloquea** #18: con una sola fuente de datos hay un solo recuento. |
| 18 | Hay dos recuentos de comentarios en la misma página y divergen. | `DetallePost.jsx:30`, `:38`, `:71`; `Comentarios.jsx:11`, `:23`, `:39`, `:54` | Después de comentar, la cabecera dice «2 comentarios» y la sección dice «3». Lo nota el lector. | 7 líneas en 2 archivos | Sí. | Lo abarata #17. |
| 19 | Al enviar un comentario, el contador suma uno que está pendiente y no se muestra. | `Comentarios.jsx:39` (`setTotal(total + 1)`) | Quien comenta ve «3 comentarios» con solo 2 en pantalla. | 1 | Sí. | Se resuelve junto a #18. |
| 20 | La web no permite editar ni borrar comentarios, aunque la API sí lo permita. | `Comentarios.jsx:76-88` (solo el formulario de alta) | Los comentaristas no pueden corregir ni retirar lo que escribieron. | 1 archivo | Sí. | Lo bloquea #9: no tiene sentido construir la interfaz sobre una autorización por correo. |
| 21 | La fecha se guarda en UTC sin zona horaria y la web muestra sus 10 primeros caracteres. | `routes.ts:323`; `Comentarios.jsx:59` y `:68` | Un comentario hecho a las 00:30 en Madrid aparece con la fecha del día anterior. Nadie todavía. | 3 | Sí. | A ninguno. |
| 22 | Las listas de comentarios y respuestas usan el índice del array como `key`. | `Comentarios.jsx:57` y `:66` | Nadie todavía: la lista no se reordena en caliente. | 2 | Sí. | A ninguno. |

## Propuestas sin medir (fuera de la tabla)

| Propuesta | Dónde | Qué habría que medir |
|---|---|---|
| Falta un índice en `comentarios.post_id`. El recuento es una subconsulta por fila del listado y una consulta extra por cada relacionado. | `routes.ts:75-76`, `:222-225`; migración `:14` | `EXPLAIN QUERY PLAN` y el tiempo de `GET /posts` y `/relacionados` con 10 000 comentarios sembrados. |
| El POST no tiene límite de frecuencia ni filtro antispam. | `routes.ts:295` | Cuántos pendientes entran al día y qué porcentaje es spam. La semilla (1 de 2 pendientes es spam) no es una muestra. |
| No hay longitud máxima para el cuerpo del comentario. | `routes.ts:319`, `:383` | El límite real de `config/bodyparser.ts` y cómo se ve la página con un comentario de ese tamaño. |
| CSRF desactivado y CORS con `origin: true` y `credentials: true` en desarrollo. | `config/shield.ts:33`, `config/cors.ts` | Si algún endpoint de comentarios llega a depender de una cookie de sesión. Hoy no usan autenticación, así que el riesgo no está demostrado. |

## Las tres líneas

1. Hallazgos que devolvió el agente: 22 filas (21 hallazgos; el 1 y el 10 son el mismo, partido en contención y fondo) y 4 propuestas sin medir. Hallazgos que comprobé yo abriendo el archivo: 14
2. Hallazgo cuya prioridad no supe decidir, y por qué: 14 Borrar un comentario deja sus respuestas huérfanas: al revisar en web esto dejaba como si existieran comentarios donde no, que a nivel usuario puede ser mas incomodo que a nivel codigo.
3. Algo que el agente afirmó y no pude verificar: 15  no hay claves foráneas y DELETE y PATCH no miran el estado del post: no pude comprobar la 15 y la 16 se me termino el tiempo.
