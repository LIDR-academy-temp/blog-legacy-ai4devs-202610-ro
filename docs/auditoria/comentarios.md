# Auditoría priorizada — feature «Comentarios»

**Alcance:** crear (`POST /posts/:slug/comentarios`), editar (`PATCH /comentarios/:id`),
borrar (`DELETE /comentarios/:id`) y listar (comentarios dentro de `GET /posts/:slug` y el
recuento `num_comentarios` en `GET /posts` y relacionados), en sus tres capas: API
(`backend/start/routes.ts`), base de datos (`comentarios`) y web (`Comentarios.jsx`,
`DetallePost.jsx`, `ListaPosts.jsx`, `PostsRelacionados.jsx`).

**Fecha:** 2026-11-22 · Rama auditada: `auditoria-XHM` (commit `35c93cc`). No se ha cambiado código.

**Orden:** primero por precedencia (lo que abarata o desbloquea otro hallazgo va antes); dentro
de cada nivel, por daño ÷ radio (lo que hace mucho daño tocando poco, arriba). Los hallazgos
de daño máximo pero precedencia baja se parten en dos filas: una **contención** inmediata y el
**arreglo de fondo** en su nivel. Los IDs (C1…) sirven para referenciarse en la columna 6.

## Nivel 0 — Red de seguridad (desbloquea todo lo demás)

| ID | Qué es | Dónde está | Qué se rompe y quién lo nota | Cuántos sitios hay que tocar | ¿Se deshace con un revert? | ¿Bloquea o desbloquea a otro hallazgo? |
|---|---|---|---|---|---|---|
| C1 | La feature tiene un único test, del camino feliz de crear, que solo afirma `status 201` y `autor_nombre`. | `backend/tests/functional/comentarios.spec.ts:41-42` | Ningún comportamiento de editar, borrar, listar, validación, 404 ni filtración está fijado: cualquier refactor o arreglo puede romper el contrato sin que nadie se entere (lo notaría el lector en producción). | 1 archivo (`comentarios.spec.ts`) | Sí | Desbloquea a todos: sin caracterización no se puede tocar ninguno de C2–C19 según `CLAUDE.md`. |

## Nivel 1 — Contención de daño máximo (filas partidas)

| ID | Qué es | Dónde está | Qué se rompe y quién lo nota | Cuántos sitios hay que tocar | ¿Se deshace con un revert? | ¿Bloquea o desbloquea a otro hallazgo? |
|---|---|---|---|---|---|---|
| C2a | *(Contención)* El detalle público de un post devuelve las filas crudas de comentarios, con `autor_email` de cada comentarista. | `backend/start/routes.ts:152-153` (`select *`) y `:171` | Cualquier visitante obtiene el correo de todos los que han comentado (incumple la regla de dominio 6). Lo notan los comentaristas, sin aviso: nadie lo detecta. | 1 sitio (`routes.ts:153`, quitar `autor_email` del SELECT) | Sí | Desbloquea a C3: sin correos públicos, el borrado por correo deja de ser explotable por cualquiera. Su arreglo de fondo es C2b. |
| C3 | El borrado se autoriza solo con conocer el correo del autor, que viaja en el cuerpo sin ninguna verificación. | `backend/start/routes.ts:350-355` | Cualquiera que sepa (o lea, ver C2a) el correo de un comentarista borra sus comentarios. Lo nota el comentarista cuando su comentario desaparece; nadie recibe aviso. | 1 sitio (`routes.ts:350-355`) | Sí | Bloqueado por C2a (mientras se filtran los correos, el exploit es trivial). Comparte regla con C9. |
| C4 | La clave de moderación está escrita en el código fuente con valor de desarrollo. | `backend/start/routes.ts:25` y `:351` | Quien tenga acceso al repositorio puede borrar cualquier comentario enviando `x-clave-moderacion: dev-secret-cambiar`. Lo nota el autor del comentario borrado; nadie más. | 2 sitios (`routes.ts:25` → variable de entorno en `start/env.ts`; `.env.example`) = 3 archivos | No: el secreto ya está en el historial de git; revertir el commit lo vuelve a exponer y hay que rotarlo igualmente. | A ninguno |
| C5 | Tras insertar, se devuelve «el último comentario de toda la tabla», no el recién creado. | `backend/start/routes.ts:329-331` | Con dos envíos concurrentes, un usuario recibe en la respuesta el comentario de otro, **con su correo**. Lo nota (o no) quien envía; el afectado nunca lo sabe. | 1 sitio (`routes.ts:329-331`, usar el id devuelto por el insert) | Sí | A ninguno |

## Nivel 2 — Datos y esquema (desbloquean el listado y el recuento)

| ID | Qué es | Dónde está | Qué se rompe y quién lo nota | Cuántos sitios hay que tocar | ¿Se deshace con un revert? | ¿Bloquea o desbloquea a otro hallazgo? |
|---|---|---|---|---|---|---|
| C6 | Hay comentarios aprobados guardados como `'Aprobado'` y `'ok'`. | `backend/database/seeders/blog_seeder.ts:510`, `:522`, `:575` | El detalle solo muestra `estado = 'aprobado'` exacto: esos comentarios aprobados no se ven nunca. Lo notan sus autores (su comentario «no sale»); los lectores no. | 3 filas de semilla + 1 migración de datos (`UPDATE`) = 2 archivos | No: el `UPDATE` sobrescribe el valor original; revertir el commit no devuelve `'Aprobado'`/`'ok'` a las filas ya normalizadas. | Desbloquea a C7 y C8 (con un único valor de estado, las dos consultas pueden unificarse). |
| C7 | La columna `estado` es texto libre, sin CHECK ni enum. | `backend/database/migrations/1769000000005_create_comentarios_table.ts:18` | Cualquier escritura futura puede volver a meter un estado no previsto y reproducir C6. Nadie lo nota hasta que un comentario «desaparece». | 1 migración nueva | No: revertir el commit no deshace la migración ya aplicada; hace falta ejecutar su `down()`. | Bloqueado por C6 (el CHECK falla si quedan valores sucios). |
| C8 | El recuento de comentarios usa una regla distinta a la del listado: `<> 'pendiente'` frente a `= 'aprobado'`. | `backend/start/routes.ts:75-76`, `:223` frente a `:153` | El listado y los relacionados dicen «3 comentarios» y el detalle muestra 1 (cuenta `'Aprobado'`, `'ok'` y cualquier valor raro). Lo nota el lector. Incumple la regla de dominio 5. | 3 sitios (`routes.ts:76`, `:223`, `:153`) | Sí | Bloqueado por C6 (si se arregla antes, el listado bajará de número y parecerá una regresión). |
| C9 | `post_id` no tiene clave foránea ni índice, y `padre_id` tampoco. | `backend/database/migrations/1769000000005_create_comentarios_table.ts:14` y `:20` | Se pueden guardar comentarios de posts inexistentes y respuestas a padres inexistentes; al borrar un padre (`routes.ts:359`) sus respuestas quedan huérfanas. Nadie lo nota: quedan invisibles (ver C14). | 1 migración nueva | No: revertir el commit no deshace la migración aplicada; además, añadir la FK exige limpiar antes los huérfanos (borrado de datos). | Desbloquea a C14 (deja de haber huérfanos que ocultar). |

## Nivel 3 — Arquitectura (desbloquea los arreglos de reglas y de contrato)

| ID | Qué es | Dónde está | Qué se rompe y quién lo nota | Cuántos sitios hay que tocar | ¿Se deshace con un revert? | ¿Bloquea o desbloquea a otro hallazgo? |
|---|---|---|---|---|---|---|
| C10 | Los tres endpoints de escritura viven enteros en el archivo de rutas: SQL, validación, autorización y forma de respuesta. | `backend/start/routes.ts:295-393` | Nada visible hoy; cada regla está copiada a mano por endpoint y diverge (C11, C12, C13). Lo nota el equipo al mantener. | 7 archivos: `routes.ts`, controlador, validador, servicio, repositorio, modelo `Comentario`, transformer; + `package.json` (alias `#repositories/*`) = 8 | Sí | Bloqueado por C1. Desbloquea a C2b, C11, C12, C13. |
| C2b | *(Arreglo de fondo)* Crear y editar devuelven la fila cruda, incluido `autor_email` y `estado`, en vez de un transformer. | `backend/start/routes.ts:333` y `:392` | El correo sale en la respuesta de crear/editar y el contrato depende de las columnas de la tabla. Lo nota quien consume la API. | 2 sitios + 1 transformer nuevo = 3 | Sí | Bloqueado por C10 (necesita la capa de transformer). |

## Nivel 4 — Reglas y validación (dependen de C1 y C10)

| ID | Qué es | Dónde está | Qué se rompe y quién lo nota | Cuántos sitios hay que tocar | ¿Se deshace con un revert? | ¿Bloquea o desbloquea a otro hallazgo? |
|---|---|---|---|---|---|---|
| C11 | La validación de crear devuelve cuatro formatos y códigos distintos (400 `error`, 422 `message`, 400 `errores[]`, 200 `ok:false`). | `backend/start/routes.ts:310-321` | Un cuerpo corto responde **200** y el cliente cree que fue bien; el front tiene que adivinar el campo del error (`Comentarios.jsx:44`). Lo nota quien comenta y quien integra. | 2 sitios (`routes.ts:310-321` → validador; `Comentarios.jsx:44`) | Sí | Bloqueado por C10. |
| C12 | Editar y borrar aplican reglas de autorización distintas (editar compara el correo sensible a mayúsculas y no acepta la clave de moderación). | `backend/start/routes.ts:379` frente a `:351-353` | Quien escribió `Ana@x.es` puede borrar con `ana@x.es` pero no editar; un moderador no puede editar. Lo nota el comentarista o el moderador. | 2 sitios (`routes.ts:353`, `:379`) | Sí | Bloqueado por C10 y por la decisión de C3. |
| C13 | Editar no exige longitud mínima del cuerpo y crear sí (5 caracteres). | `backend/start/routes.ts:384` frente a `:319` | Un comentario puede quedarse en 1 carácter tras editarlo. Lo nota el moderador. | 1 sitio (`routes.ts:384`) | Sí | Bloqueado por C10 (mismo validador). |
| C15 | `padre_id` se acepta sin validar que exista ni que sea del mismo post. | `backend/start/routes.ts:308` y `:326` | Se pueden colgar respuestas de comentarios de otro post o inexistentes. Nadie lo nota: no se pintan. | 1 sitio (`routes.ts:308` → validador) | Sí | Bloqueado por C10; relacionado con C9. |

## Nivel 5 — Web (sin dependencias de backend salvo la indicada)

| ID | Qué es | Dónde está | Qué se rompe y quién lo nota | Cuántos sitios hay que tocar | ¿Se deshace con un revert? | ¿Bloquea o desbloquea a otro hallazgo? |
|---|---|---|---|---|---|---|
| C16 | El contador del bloque de comentarios suma 1 al enviar un comentario que queda pendiente y no se muestra. | `frontend/src/componentes/Comentarios.jsx:39` | El lector ve «3 comentarios» con solo 2 visibles hasta recargar. Lo nota quien acaba de comentar. | 1 sitio | Sí | A ninguno |
| C17 | Hay dos recuentos de comentarios independientes en la misma página. | `frontend/src/componentes/DetallePost.jsx:30`, `:38` y `Comentarios.jsx:11`, `:23` | La cabecera y el bloque muestran cifras distintas tras comentar (agravado por C16). Lo nota el lector. | 2 archivos (4 líneas) | Sí | Desbloquea a C18 (un solo dueño del dato). |
| C18 | El bloque de comentarios vuelve a pedir el post entero que su padre ya pidió. | `frontend/src/componentes/Comentarios.jsx:19` y `DetallePost.jsx:34` | Dos peticiones idénticas por visita. Nadie lo nota a simple vista. | 2 archivos | Sí | Bloqueado por C17. |
| C14 | La web solo pinta un nivel de respuestas y descarta las que cuelgan de un padre no visible. | `frontend/src/componentes/Comentarios.jsx:50` y `:63-64` | Una respuesta aprobada a un comentario pendiente, borrado o con estado `'Aprobado'` nunca se ve. Lo nota su autor. | 1 archivo | Sí | Bloqueado por C9 y C6 (depende de qué padres existen y cuáles se muestran). |
| C19 | La web no ofrece editar ni borrar comentarios: el único envío es el `POST` de crear. | `frontend/src/componentes/Comentarios.jsx:30-34` | Dos de las cuatro operaciones de la feature solo existen en la API. Lo nota el comentarista, que no puede corregir ni retirar lo que escribió. | 1 archivo (+ componente nuevo si se separa) | Sí | Bloqueado por C3 y C12 (no se debe exponer en la web una autorización por correo rota). |
| C20 | Las listas de comentarios usan el índice del array como `key`. | `frontend/src/componentes/Comentarios.jsx:57` y `:66` | Al insertar o borrar un comentario React reutiliza nodos equivocados. Hoy nadie lo nota (la lista no cambia sin recargar); lo notará el lector cuando exista C19. | 2 sitios | Sí | A ninguno |

## Fuera de la tabla — propuestas sin medir

Estas propuestas aparecen al leer el código pero nadie ha medido su efecto; no entran en la
priorización hasta tener el dato.

| Propuesta | Qué habría que medir |
|---|---|
| Índice sobre `comentarios.post_id` por rendimiento (`routes.ts:75-76`, subconsulta correlacionada por post en el listado). | Tiempo de `GET /posts` con el volumen real de comentarios (p50/p95) con y sin índice, con `EXPLAIN QUERY PLAN` en SQLite. |
| Eliminar el N+1 del recuento en relacionados (`routes.ts:219-229`, una consulta por relacionado). | Número de consultas y latencia de `GET /posts/:slug/relacionados`; con `MAX_RELACIONADOS` pequeño puede ser irrelevante. |
| Limitación de frecuencia / antispam en crear (la semilla ya tiene spam pendiente, `blog_seeder.ts:547-553`). | Volumen de comentarios `pendiente` por día y proporción que acaban siendo spam. |
| Paginar los comentarios del detalle (`routes.ts:152-155` los devuelve todos). | Distribución de comentarios por post y tamaño de la respuesta de `GET /posts/:slug`. |


## PARTE B

1. **Cuántos hallazgos devolvió el agente, y cuántos comprobaste tú abriendo el archivo.** Los dos números, tal cual salieron. No los redondees ni los expliques.

    21 / 9.

2. **El hallazgo del que no supiste decidir la prioridad**, y en una frase, por qué. Lo que interesa es **qué dos cosas se contradecían**: lo grave contra lo que bloquea, lo urgente contra lo caro, lo que te pedían contra lo que veías.

    El hallazgo C10. No acabé de enteder qué significaba y es el que se llevo parte de los 45 minutos del ejercicio, finalmente preguntando a Claude lo llegué a entender. El codigo en teoria es correcto en este punto, pero el problema es que en Claude.md hay especificada una regla de `Arquitectura Objetivo` que provoca que la parte de comentarios no la cumpla y que los endpoints no estan separados en diferentes capas. Por lo que sé, arreglar este punto implica tocar diversos ficheros (7) con el respectivo riesgo de introducir bugs sobre un código en teoria correcto o mas probado. Priorizar este cambio sobre otros hallazgos que vienen luego, y que tienen solucion mas rápida y sencilla y un impacto en el uso del programa, hacen que me plantee si la criticidad de este punto se deberíua bajar a un nivel inferior.

3. **Algo que el agente afirmó y no pudiste verificar**. Ojo al matiz: no es lo que dijo mal. Es lo que **no pudiste comprobar ni a favor ni en contra**, y te tocó dejar en el aire.

    El hallazgo C5. Segun dice el agente, tras insertar se devuelve el último comentario de la tabla pero que con 2 inserciones concurrentes, un usuario podria llegar a ver el comentario de otro usuario. 

    Revisando el código, aunque tiene todo el sentido lo que dice ya que se hace una `SELECT` ordenando por `ID` de forma descendiente y con un `LIMIT` de 1, no he podido verificar si puede llegar a pasar o si esto ya está controlado por el propio motor de la web. 
    