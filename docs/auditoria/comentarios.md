# Auditoría técnica: funcionalidad de comentarios

**Alcance:** solo comentarios (frontend, backend y base de datos). No se ha modificado código de la aplicación ni se ha corregido nada.

**Método (dos fases):**

1.  **Lectura estática** del repositorio el 2026-11-22.
2.  **Verificación en ejecución** el mismo día, contra la API local en `localhost:3401`, con la aplicación levantada y desde una sola IP.
    *   Se crearon 22 comentarios de prueba (`autor_nombre` = `ZZ-auditoria*`) y se borraron todos por la API.
    *   La base `backend/tmp/db.sqlite3` se leyó en solo lectura. Al terminar quedaban las 9 filas originales, con la misma distribución de estados.
    *   No he comprobado si el contador de ids de SQLite avanzó.
    *   El frontend no se probó en un navegador.

**Etiquetas en las celdas:**

*   **\[verificado\]** = probado contra la aplicación en marcha.
*   **\[deducido\]** = sale de leer el código, sin ejecutar.

**Abreviaturas de archivos:**

*   `routes.ts` = `backend/start/routes.ts`
*   `Comentarios.jsx` = `frontend/src/componentes/Comentarios.jsx`
*   `DetallePost.jsx` = `frontend/src/componentes/DetallePost.jsx`
*   `migración` = `backend/database/migrations/1769000000005_create_comentarios_table.ts`
*   `seeder` = `backend/database/seeders/blog_seeder.ts`

**Criterio de orden:**

1.  Primero la precedencia: van antes los hallazgos que desbloquean o abaratan otros.
2.  Dentro de cada nivel, el daño dividido por el radio de cambio.

El «daño» es una valoración cualitativa mía, no una medición. Niveles:

*   N0: H12.
*   N1: H3 (contención), H2, H5, H6.
*   N2: H7 (contención), H4, H1, H9.
*   N3: H10, H8.
*   N4: H3 y H7 (soluciones definitivas).

H7 (contención) no depende de ningún otro cambio; está en N2 por aplicar el criterio de precedencia al pie de la letra.

H11 no cumple el requisito de línea que lo demuestre y queda en «Pendientes de medir o verificar».

## Hallazgos

| Qué es | Dónde está | Qué se rompe y quién lo nota | Cuántos sitios hay que tocar | ¿Se deshace con un revert? | ¿Bloquea o desbloquea a otro hallazgo? |
| --- | --- | --- | --- | --- | --- |
| **H12.** Un único test cubre solo el camino feliz del POST de comentarios. | `backend/tests/functional/comentarios.spec.ts:15-43` (único `test(...)`, línea 15) | Lectura, edición, borrado y validación no están protegidos por ningún test. Lo nota quien modifique esas rutas. |   | Sí: solo añade archivos de test. | Desbloquea H1–H10. `CLAUDE.md` exige caracterizar antes de refactorizar; es una regla del proyecto, no una dependencia técnica. |
| **H3 (contención).** Dejar de devolver `autor_email` en las dos respuestas que lo exponen. | `routes.ts:152` (`select *`), `:171` (`comentarios: comentarios`), `:329-333` (el POST devuelve la fila completa) | **\[verificado\]** `GET /posts/:slug` devuelve `autor_email` de cada comentario público (por ejemplo el de los ids 1 y 2 del post 1), y el POST devuelve el correo en su respuesta 201. Lo notan los comentaristas (privacidad). El frontend de este repo no lee `autor_email` de la respuesta; solo lo envía (`Comentarios.jsx:33`). Clientes externos: sin comprobar. | 2 (`routes.ts:152` y `:329-330`) | Sí: solo código. | Rebaja el riesgo de H7: el correo es lo que permite pasar `routes.ts:352-353` (**\[verificado\]** que el borrado acepta el correo del cuerpo; **\[deducido\]** que el correo filtrado se use así). Comparte líneas con H4 (`:329-333`). Prepara H3 (definitiva). |
| **H2.** `estado` es texto libre y ninguna ruta lo cambia. | `migración:18`. `seeder:510, 522, 575` (`Aprobado`, `ok`, `Aprobado`). La única sentencia `update comentarios` de `routes.ts` es `:388` y solo cambia `cuerpo`. | **\[verificado\]** La base real contiene `aprobado` ×4, `Aprobado` ×2, `ok` ×1 y `pendiente` ×2. **\[verificado\]** Un PATCH con `estado:"aprobado"` en el body respondió 200 y el comentario siguió `pendiente`. Los `Aprobado` y `ok` no salen en el detalle (`routes.ts:153`) y los nuevos nacen `pendiente` (`:326`); en el repo no hay ruta que los apruebe. Lo notan lectores, comentaristas y moderación. | 4 en el repo (`migración:18`; `seeder:510, 522, 575`). Las filas de la base real: 3 con estado distinto de `aprobado`/`pendiente`, sin contar el resto de datos. | No. Corregir datos es tocar `backend/tmp/db.sqlite3`, ignorado por Git (`backend/.gitignore:4`). El `down()` de la migración (`:24-26`) hace `dropTable` y borraría los comentarios. | Desbloquea H1 y H10: hay que decidir qué estados son visibles antes. |
| **H5.** La validación del POST responde con cuatro formatos distintos, uno de ellos con 200. | `routes.ts:311` (400 `{error}`), `:314` (422 `{message}`), `:317` (400 `{errores}`), `:320` (200 `{ok:false}`). `Comentarios.jsx:44` solo lee `error`, `message` y `motivo`. | **\[verificado\]** Sin nombre: 400 `{error}`. Sin email: 422 `{message}`. Email sin `@`: 400 `{errores:[...]}`. Cuerpo corto: **200** `{ok:false,motivo}`. **\[deducido\]** Que el frontend muestre «No se ha podido enviar.» ante `errores` (`Comentarios.jsx:44`); la interfaz no se probó. Cambiar códigos altera el contrato. | 5 (`routes.ts:311, 314, 317, 320`; `Comentarios.jsx:44`) | Sí: solo código. Clientes externos que dependan de estos códigos: sin comprobar. | Abarata H8: un validador común para POST y PATCH (diseño propuesto, no obligatorio). |
| **H6.** `padre_id` se guarda sin validar y el frontend no lo envía. | `routes.ts:308` (lectura), `:326` (insert). `Comentarios.jsx:33` (el body no lo incluye), `:50` y `:63-64` (solo raíces y un nivel). `migración:20` (columna sin clave foránea). | **\[verificado\]** Un POST con `padre_id` 99999 (inexistente) y otro con `padre_id` 1 (de otro post) devolvieron 201. **\[deducido\]** Que esos comentarios no se pintarían (`Comentarios.jsx:50` y `:64`). Desde la interfaz no se pueden crear respuestas. Lo notaría quien use la API directamente. Si otro cliente envía `padre_id`: sin comprobar. | 4 (`routes.ts:308, 326`; `Comentarios.jsx:33`; `migración:20`) | Código: sí. Corregir filas existentes es un cambio de datos en `db.sqlite3` (ignorado por Git): no. | Desbloquea H9: hay que limpiar respuestas antes de cualquier restricción. |
| **H7 (contención).** Sacar del repositorio la clave de moderación. | `routes.ts:25` (`'dev-secret-cambiar'`), `:351` (`clave === SECRETO_SESION`) | **\[verificado\]** La aplicación en marcha acepta ese valor: un DELETE con la cabecera `x-clave-moderacion: dev-secret-cambiar` borró un comentario ajeno (200); con una clave incorrecta devolvió 403. Quien tenga el repositorio puede borrar cualquier comentario. Que la misma clave firme sesiones (lo dice el comentario de `:21-24`): **sin verificar**. Lo notan comentaristas y moderación. | 1 en código (`routes.ts:25`). La variable de entorno queda sin contar: no he comprobado si existe un `.env`. | Parcial. El código sí. La clave actual queda en el historial de Git y rotarla no se deshace con un revert. | A ninguno como dependencia. Su explotación práctica pasa por H3 (el correo en `:352-353`). |
| **H4.** El POST devuelve la última fila de la tabla en lugar de la insertada. | `routes.ts:329-330` (`select * from comentarios order by id desc limit 1`) | **\[verificado\]** La consulta devuelve la última fila, no la insertada (lectura). **\[no reproducido\]** En 20 POST simultáneos cada petición recibió su propio comentario (20 ids distintos, 0 respuestas ajenas). El riesgo de que alguien reciba el comentario de otro, con su correo (ver H3), sigue siendo teórico. | 1 (`routes.ts:329-330`) | Sí: solo código. | Comparte líneas con H3 (contención) en `:329-333`: conviene hacerlos juntos. No bloquea a ninguno. |
| **H1.** «Comentario visible» se define de tres formas distintas. | `routes.ts:75` y `:223` (`estado <> 'pendiente'`), `:153` (`estado = 'aprobado'`) | **\[verificado\]** Con los datos reales: el post 1 sale con `num_comentarios` 4 en el listado (`?pagina=2`) y 2 comentarios en el detalle. El post 6 sale con 1 en el listado y 0 en el detalle. Los posts 2 y 4 coinciden (1 y 1). Los relacionados usan la misma definición del listado (`routes.ts:223`): en el post 1, el post 6 aparece con 1. Lo notan lectores. | 3 (`routes.ts:75, 153, 223`) | Sí: solo consultas. | Depende de H2: hay que decidir antes qué estados cuentan. Desbloquea H10. |
| **H9.** Borrar un comentario no trata sus respuestas. | `routes.ts:359` (`delete from comentarios where id = ?`). `migración:20` (sin clave foránea ni cascada). | **\[verificado\]** Tras crear un padre y una hija (`padre_id` = padre) y borrar el padre, la hija siguió en la tabla con `padre_id` apuntando a una fila inexistente. **\[deducido\]** Que cuente en `routes.ts:75` y `:223` si no está pendiente y que la pantalla no la muestre (`Comentarios.jsx:50`, `:64`). Lo notan lectores. | 1 (`routes.ts:359`). 2 si se opta por clave foránea (`migración:20`). | Código: sí. Las filas ya borradas u huérfanas no se recuperan ni se limpian con un revert (datos en `db.sqlite3`, ignorado por Git). | Depende de H6 (limpiar antes). Afecta a la cifra de H1. |
| **H10.** La sección y la cabecera usan dos contadores de comentarios distintos. | `Comentarios.jsx:11, 23, 39, 54`; `DetallePost.jsx:30, 38, 71` | **\[deducido\]** `Comentarios.jsx:39` suma 1 al enviar un comentario que queda pendiente, que no se pinta. La sección diría «N+1 comentarios» sin ningún comentario nuevo visible y la cabecera (`DetallePost.jsx:71`) no cambiaría. **No probado en el navegador.** Lo notan lectores. | 7 líneas en 2 archivos (`Comentarios.jsx:11, 23, 39, 54`; `DetallePost.jsx:30, 38, 71`) | Sí: solo código. | Depende de H1: hay que decidir qué cifra es la correcta. |
| **H8.** PATCH no aplica las mismas reglas que el POST. | `routes.ts:379` (compara el correo sin normalizar, exige `pendiente`, sin clave de moderación), `:384` (solo comprueba que haya `cuerpo`), frente a `:319` (el POST exige 5 caracteres) | **\[verificado\]** Una edición a 1 carácter con el correo exacto devolvió 200. El correo en mayúsculas devolvió 403, y la clave de moderación sin correo también 403. El frontend no tiene interfaz de edición (no hay `PATCH` en `frontend/src`), así que solo lo nota un cliente de la API. | 2 (`routes.ts:379, 384`) | Sí: solo código. | Se abarata si H5 introduce un validador común. |
| **H3 (solución definitiva).** La respuesta pública de comentarios debería salir de un transformer, no de la fila cruda. | `routes.ts:152, 171, 333`. En `backend/app/transformers/` solo existe `user_transformer.ts`. | Mismo daño que H3 (contención), **\[verificado\]** arriba. Sin esa capa, cada consulta debe recordar excluir el correo (opinión). |   | Sí: solo código. | Depende de H12. Hacen falta capas que no existen todavía: `backend/app/services` y `backend/app/repositories` no existen. |
| **H7 (solución definitiva).** Una misma política de autorización para borrar y editar, que no dependa del correo ni de una clave en el código. | `routes.ts:351-353` (DELETE), `:379` (PATCH) | **\[verificado\]** El DELETE lee el correo del cuerpo y lo acepta aunque lleve mayúsculas y espacios (`" ZZ@Auditoria.TEST "` borró el comentario de `zz@auditoria.test`). Quien conozca el correo de un autor puede borrar su comentario (H3 lo expone). Lo notan comentaristas. |   | Sí: solo código. | Depende de H3 y H12. Relacionada con H8 (misma regla en PATCH). |

## Pendientes de medir o verificar

| Pendiente | Qué habría que comprobar |
| --- | --- |
| **H4 (colisión).** No reproducida con 20 POST simultáneos desde una IP. | Repetir con más carga o con otro entorno. Sin reproducción, el riesgo sigue siendo teórico. |
| **H10 (contadores del frontend).** Solo deducido del código. | Probarlo en un navegador: enviar un comentario y comparar el título de la sección con la cabecera del post. |
| **H7 (alcance de la clave).** El comentario de `routes.ts:21-24` dice que firma sesiones. En los `.ts` leídos, `SECRETO_SESION` solo aparece en `routes.ts:25` y `:351`. | Revisar `backend/config/app.ts` y otras configuraciones. |
| **H2 (moderación externa).** En el repo no hay ruta que cambie `estado`. | Preguntar si la moderación se hace con otra herramienta o a mano en la base de datos. |
| **H11 (rendimiento).** **\[verificado\]** `comentarios` no tiene índices ni claves foráneas en la base real. `routes.ts:75` y `:222` filtran por `post_id`. | Volumen real de comentarios, `EXPLAIN QUERY PLAN` y tiempos con datos reales. Sin medición no se confirma el daño. |
| **H11 (límite de peticiones y spam).** **\[verificado\]** 20 POST simultáneos desde una IP, todos 201, sin ningún 429. No aparece `throttle`, `limiter` ni `rate` en `backend/start/kernel.ts`. | Revisar el resto de middleware y el despliegue (proxy). Probar con más volumen y desde varias IP. |
| **H11 (**`**key={index}**`**).** `Comentarios.jsx:57` y `:66`. | En la lectura no encuentro estado local en los hijos, así que no veo consecuencia. Comprobar si cambia la lista entre renders o si se añade interacción. |
| **H11 (URL de API duplicada).** `Comentarios.jsx:4` declara ser la cuarta copia. | Contar las copias reales en `frontend/src`. Es mantenimiento, sin daño medible. |
| **H3, H5 (clientes externos).** | Averiguar si algún cliente, además del frontend de este repo, consume la API de comentarios. |

## Parte B

### Hallazgos

El agente identificó 12 hallazgos. Hallazgos que he verificado personalmente: _Los 12_

### Duda de priorización

_Me ha costado decidir entre priorizar H12 (falta de tests) y H7 (problemas de autorización). Por un lado, considero importante tener pruebas antes de modificar código heredado para evitar romper otras funcionalidades. Por otro, H7 es un problema de seguridad que puede permitir borrar comentarios ajenos y que quizá debería resolverse cuanto antes. No tengo claro si debe pesar más la gravedad inmediata o la necesidad de preparar el código para corregirlo con seguridad._

### Afirmación sin verificar

_El agente identificó en H4 un posible problema de concurrencia: si dos usuarios publican comentarios prácticamente al mismo tiempo, uno podría recibir el comentario del otro. Aunque he revisado la consulta que utiliza el código y entiendo el riesgo, no he podido confirmar que ocurra realmente. El agente realizó 20 peticiones simultáneas y no consiguió reproducirlo, por lo que lo considero un riesgo posible, pero todavía no demostrado._