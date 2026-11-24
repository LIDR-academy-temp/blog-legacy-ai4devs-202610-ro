# Auditoría priorizada — feature de comentarios

**Alcance:** los tres endpoints de comentarios (`POST /posts/:slug/comentarios`,
`DELETE /comentarios/:id`, `PATCH /comentarios/:id`), la parte de comentarios de
`GET /posts/:slug` y de los recuentos `num_comentarios`, su tabla, su semilla, su test y los
componentes del frontend que los pintan. Leído en `s6/start` el 2026-11-24.

## Cómo está ordenada

1. **Precedencia primero.** Lo que abarata o desbloquea otro hallazgo va antes:
   - **Nivel 0 — contención.** Filas que salen de partir un hallazgo de daño máximo (5) que
     cae en el nivel de precedencia más bajo: se contiene ya, con un cambio de 1–3 líneas, y
     el arreglo a fondo queda en su nivel (C1 → H8, C2 → H10, C3 → H10).
   - **Nivel 1:** desbloquea y no depende de nada.
   - **Nivel 2:** desbloquea, pero depende del nivel 1.
   - **Nivel 3:** no desbloquea nada.
   - **Nivel 4:** depende de un hallazgo del nivel 3.
2. **Dentro de cada nivel, daño ÷ radio.** Daño de 1 (cosmético) a 5 (fuga de datos personales
   o acción destructiva por un tercero). Radio = número de lugares a tocar (columna 4). En
   caso de empate, va primero el de mayor daño o el que destruye datos.

Hay una regla de `CLAUDE.md` que pesa en todo el orden: **ningún arreglo de backend se hace sin
su test de caracterización**. Por eso H1 desbloquea, de hecho, todas las filas del backend. Las
contenciones (nivel 0) son la excepción deliberada: llevan su propio test mínimo y no esperan a
la suite completa.

## Hallazgos

| ID | Qué es | Dónde está | Qué se rompe y quién lo nota | Lugares a tocar | ¿Se puede hacer revert? | ¿Bloquea o desbloquea otro hallazgo? | Daño ÷ radio |
|---|---|---|---|---|---|---|---|
| **Nivel 0 — contención** | | | | | | | |
| C1 | **Contención** de la fuga del correo de quien comenta: el detalle hace `select *` de `comentarios` y lo devuelve tal cual. | `backend/start/routes.ts:152-155` (consulta), `:171` (respuesta) | Cualquier visitante ve en DevTools, en `GET /posts/:slug` → `comentarios[].autor_email`, el correo de cada persona que comentó (p. ej. `ana.cortes@ejemplo.es` en `diez-tendencias-…`). Las víctimas no lo notan. Incumple la regla 6. El frontend no usa ese campo, así que quitarlo no rompe nada visible. | 1 (proyectar columnas explícitas en `:153`) | Sí. El cambio se revierte con `git revert`; los correos que ya se filtraron no se pueden recuperar. | Abarata C2 (sin correo público, suplantar a un autor deja de ser trivial). Su arreglo a fondo es H8. | 5 ÷ 1 = 5,0 |
| C2 | **Contención** del borrado ajeno: `DELETE` autoriza a quien mande en el cuerpo el mismo `autor_email` que el comentario. | `backend/start/routes.ts:350-355` | Cualquiera copia el correo de C1 y borra cualquier comentario aprobado con `DELETE /comentarios/:id {autor_email}`. Lo notan el comentarista y el lector, porque el comentario desaparece, y no queda rastro. Contención: quitar la vía `esAutor` y dejar solo la de moderador. El frontend no tiene botón de borrar, así que nadie pierde nada. | 1 (`:350-355`) | Sí | La refuerza C1. Su arreglo a fondo es H10. | 5 ÷ 1 = 5,0 |
| C3 | **Contención** de la clave de moderación escrita en el código y commiteada (`'dev-secret-cambiar'`). | `backend/start/routes.ts:25`, `:351` | Quien haya visto el repo o su historial manda `x-clave-moderacion: dev-secret-cambiar` y borra cualquier comentario. No lo nota nadie hasta que faltan comentarios. Contención: leerla de `env` con un valor nuevo. | 3 (`routes.ts:25`, `start/env.ts`, `.env.example`) | **No.** El valor antiguo sigue en el historial de git (`e0e0943`): revertir el código vuelve a dejar activa una clave pública. Rotarla es irreversible por diseño. | Ninguno. Su arreglo a fondo, que elimina la clave compartida, es H10. | 5 ÷ 3 = 1,7 |
| **Nivel 1 — desbloquea, sin prerrequisitos** | | | | | | | |
| H1 | La feature tiene un único test: el camino feliz del `POST`, que solo afirma `autor_nombre`. | `backend/tests/functional/comentarios.spec.ts:42-70` (aserción única en `:69`) | Ningún cambio en comentarios tiene red. Hoy pasaría en verde un refactor que filtre el correo (no hay aserción negativa), que cambie un código de error o que rompa `DELETE`/`PATCH`, que no tienen ningún test. Lo nota el siguiente que toque el código, en producción. | 1 (el spec: caracterizar `POST`, `DELETE`, `PATCH` y la parte de comentarios de `GET /posts/:slug`, con su matriz feliz / inválida / 404 / frontera) | Sí | **Desbloquea todas las filas H del backend** (H2–H11 y H13). Es el requisito de `CLAUDE.md` antes de refactorizar o corregir. | n/a (único en su nivel) |
| **Nivel 2 — desbloquea, depende del nivel 1** | | | | | | | |
| H2 | `comentarios.estado` es texto libre sin CHECK, y conviven `'aprobado'`, `'Aprobado'` y `'ok'`. | `backend/database/migrations/1769000000005_create_comentarios_table.ts:18`; datos en `backend/database/seeders/blog_seeder.ts:510`, `:522`, `:575`; filtro exacto en `backend/start/routes.ts:153` | En el detalle de `diez-tendencias-…` solo salen 2 de los 4 comentarios aprobados: los ids 3 (`Aprobado`) y 4 (`ok`) no aparecen. En `que-mide-de-verdad-…` no sale el id 9. Lo nota el comentarista, que nunca ve publicado su comentario aprobado. | 4 (migración nueva que normaliza y añade CHECK, y las 3 filas de la semilla) | **No.** La normalización pierde información: después de pasar `'Aprobado'` y `'ok'` a `'aprobado'`, el `down()` no puede saber qué fila era cuál, salvo que se guarde una copia aparte. | Depende de H1. **Desbloquea H7**: el recuento solo se puede unificar cuando haya un único valor de "aprobado". | 4 ÷ 4 = 1,0 |
| H3 | Toda la lógica de comentarios vive en las rutas (SQL, validación, autorización y forma de la respuesta), sin controlador, validador, servicio, repositorio, modelo ni transformer. | `backend/start/routes.ts:295-393` | Lo nota el equipo: la regla de autorización ya se escribió dos veces y salió distinta (H13), y no hay ningún sitio único donde poner el transformer (H8) ni el validador (H9, H11). El usuario no ve nada directamente. | 8 (`routes.ts:295-393`, alias `#repositories/*` en `backend/package.json`, y 6 archivos nuevos: controlador, validador, servicio, repositorio, modelo `Comentario`, transformer) | Sí (refactor puro, protegido por H1) | Depende de H1. **Desbloquea H8, H9, H10 y H11**, que necesitan sus capas. | 3 ÷ 8 = 0,4 |
| **Nivel 3 — no desbloquea nada** | | | | | | | |
| H4 | Borrar un comentario padre deja sus respuestas con un `padre_id` que ya no existe, y el frontend deja de pintarlas. | `backend/start/routes.ts:359`; pintado en `frontend/src/componentes/Comentarios.jsx:50`, `:63-64` | Si se borra el comentario 1 de `diez-tendencias-…`, la respuesta aprobada id 2 sigue en la base, pero desaparece de la página: no es raíz (`padre_id` no nulo) y su padre ya no está en la lista. Lo nota quien respondió. El borrado es físico y no se recupera. | 1 (`:359`: decidir entre borrado en cascada, impedir el borrado con hijos o borrado lógico) | Sí para el arreglo. Los borrados que ya ocurrieron, no. | Ninguno (depende de H1) | 3 ÷ 1 = 3,0 |
| H5 | Tras el `insert`, el `POST` devuelve la última fila de toda la tabla, no la que acaba de insertar. | `backend/start/routes.ts:329-331` | Con dos envíos casi simultáneos, una persona recibe en el `201` el comentario de la otra, incluido su `autor_email` (otra fuga) y su texto. Lo nota el usuario en el aviso, o nadie si solo se mira `d.comentario`. | 1 (`:329-331`: usar el id devuelto por el insert) | Sí | Ninguno (depende de H1) | 3 ÷ 1 = 3,0 |
| H6 | `comentarios.post_id` no tiene clave foránea ni índice. | `backend/database/migrations/1769000000005_create_comentarios_table.ts:14` | Al borrar un post quedan comentarios huérfanos. El recuento correlacionado del listado (`routes.ts:75-76`) recorre toda la tabla por cada post de la página. Hoy no lo nota nadie; con volumen, lo nota quien carga el listado, en la latencia. | 1 (migración nueva) | Sí, mientras no haya huérfanos. Si los hay, crear la FK obliga a borrarlos y eso no se deshace. | Ninguno (depende de H1) | 2 ÷ 1 = 2,0 |
| H7 | `num_comentarios` cuenta todo lo que no es `'pendiente'` (`<> 'pendiente'`), mientras que el detalle muestra solo `= 'aprobado'`. | `backend/start/routes.ts:75-76` (listado), `:222-224` (relacionados) | El listado anuncia "4 comentarios" en `diez-tendencias-…` y el detalle muestra 2. En `que-mide-de-verdad-…`, 1 frente a 0. Cualquier estado futuro (`'rechazado'`, `'spam'`) también sumaría. Lo nota el lector al entrar en el post. Incumple la regla 5. | 2 (`:75-76`, `:222-224`) | Sí | Lo desbloquea H2 (y H1). No desbloquea ninguno. | 3 ÷ 2 = 1,5 |
| H8 | **Arreglo a fondo de C1**: no hay transformer de comentario, y las tres respuestas devuelven la fila cruda con `autor_email`, `estado` y `post_id`. | `backend/start/routes.ts:171` (detalle), `:333` (`POST`), `:392` (`PATCH`) | Lo mismo que C1, más los campos internos que el contrato no pide en las tres respuestas. Lo nota cualquier cliente de la API. | 4 (transformer nuevo, `:171`, `:333`, `:392`) | Sí | Lo desbloquea H3. No desbloquea ninguno. | 5 ÷ 4 = 1,25 |
| H9 | `padre_id` se acepta sin validar: puede apuntar a un comentario inexistente, a uno de otro post o a una respuesta. | `backend/start/routes.ts:308` (lectura), `:326` (insert) | Una respuesta a un id que no existe o que es de otro post se guarda y nunca se pinta, porque el frontend solo muestra un nivel (`Comentarios.jsx:63-64`). Lo nota quien respondió: su respuesta aprobada no aparece. | 2 (regla en el validador y en el servicio) | Sí | Lo desbloquea H3. No desbloquea ninguno. | 2 ÷ 2 = 1,0 |
| H10 | **Arreglo a fondo de C2 y C3**: la autorización de `DELETE`/`PATCH` se basa en conocer un correo o una clave compartida, en vez de en una identidad autenticada. | `backend/start/routes.ts:348-355` (`DELETE`), `:378-379` (`PATCH`), `:25` (clave) | Después de C2 y C3, un autor ya no puede borrar lo suyo, y la moderación depende de una clave compartida que se puede volver a filtrar. Lo nota el equipo de moderación. El starter kit ya trae `middleware.auth()` (`routes.ts:416`). | 5 (`middleware.auth()` en `:339` y `:366`, política en el servicio, columna de rol en `users`, eliminar `:25`) | Sí (la migración de rol tiene `down()` limpio) | Lo desbloquea H3. **Desbloquea H13**: con una sola política, `PATCH` deja de tener la suya propia. | 5 ÷ 5 = 1,0 |
| H11 | La validación del `POST` responde con cuatro códigos y cuatro formas distintas, y deja pasar entradas malas. | `backend/start/routes.ts:310-321`; el frontend lo parchea en `frontend/src/componentes/Comentarios.jsx:44`; `backend/app/exceptions/handler.ts:15-17` sin centralizar | Sin nombre: `400 {error}`. Sin email: `422 {message}`. Email sin `@`: `400 {errores:[…]}`, y el frontend no lee `errores`, así que muestra "No se ha podido enviar." Cuerpo corto: **`200 {ok:false}`**. El email solo se valida por contener `@`, y un cuerpo de 5 espacios pasa. Lo nota quien comenta, que recibe un mensaje genérico, y cualquier cliente que mire el código HTTP. | 4 (validador VineJS, `routes.ts:310-321`, `handler.ts`, `Comentarios.jsx:44`) | Sí en código, pero cambia el contrato: revertirlo rompe a los clientes ya adaptados a `422`. | Lo desbloquea H3 (y H1, que primero caracteriza el `200 {ok:false}`). No desbloquea ninguno. | 3 ÷ 4 = 0,75 |
| H12 | El frontend lleva dos recuentos independientes, y el del formulario suma un comentario que todavía está pendiente. | `frontend/src/componentes/DetallePost.jsx:30`, `:38`, `:71`; `frontend/src/componentes/Comentarios.jsx:11`, `:23`, `:39` | Tras enviar un comentario, la sección dice "3 comentarios" con 2 visibles, y la cabecera del artículo sigue en "2 comentarios". Lo nota el propio comentarista en la misma pantalla. | 3 (subir el recuento a `DetallePost` y quitar `Comentarios.jsx:11/23` y `:39`) | Sí | Ninguno (frontend, sin dependencia de backend; H7 lo empeora en el listado) | 2 ÷ 3 = 0,67 |
| **Nivel 4 — depende de un hallazgo del nivel 3** | | | | | | | |
| H13 | `PATCH` reescribe la autorización a mano y sale distinta de la de `DELETE`. Además, editar se salta la longitud mínima del cuerpo. | `backend/start/routes.ts:375-381` (correo exacto con mayúsculas, sin moderador, solo si está pendiente), `:383-386` (solo comprueba que exista) | Un autor que escribe su correo con otra capitalización puede borrar pero no editar. Un moderador no puede editar. Se puede editar un comentario a `"a"`, que el `POST` habría rechazado. Lo nota quien intenta corregir su comentario. | 2 (`:375-381`, `:383-386`) | Sí | Lo desbloquea H10 (política única). No desbloquea ninguno. | 3 ÷ 2 = 1,5 |


## Fuera de alcance (vistos pero no son de comentarios)

- `GET /posts/:slug` devuelve también `post.autor_email` (`routes.ts:167`) y `autor.email`
  (`:168`). Es la misma regla 6, pero pertenece al recurso post.
- No existe ningún endpoint para aprobar un comentario: los `pendiente` solo salen de ese estado
  tocando la base a mano. No tiene línea a la que apuntar, así que, según el formato pedido, no
  lleva fila.



## PARTE B
- El agente devolvio 16 hallazgos, de los cuales pude comprobar directamente 6 - el resto no fue posible por falta de tiempo.
- H1 Happy path test: Aunque no me parece tan crítico como parece estar tan alto, el agente lo considera bloqueante para otros arreglos criticos que quedan por debajo por tener esta dependencia.
- No pude revisar todos los hallazgos por el tiempo, pero hasta donde llegué pude verificar y coincidir con el criterio del agente.

