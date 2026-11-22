# Mapeo de la feature de comentarios

Mapeo de punta a punta (API, base de datos y web) de la feature de comentarios del blog «Corriente».
Solo describe lo que hay en el código: no valora si algo está bien o mal.

Las rutas de los enlaces son relativas a este archivo (`docs/auditoria/`).

## 1. Endpoints propios de la feature

Todos están en [backend/start/routes.ts](../../backend/start/routes.ts). No hay controlador,
validador, servicio ni transformer de comentarios en `backend/app/`.

| Verbo y ruta | Líneas | Qué hace | Columnas que usa |
|---|---|---|---|
| `POST /posts/:slug/comentarios` | [295-334](../../backend/start/routes.ts#L295-L334) | Busca el post por `slug` y `estado='publicado'` (404 si no existe). Valida en la propia ruta (L310-321) e inserta el comentario con `estado='pendiente'`. Devuelve `201 {comentario}` con la fila leída con `select * … order by id desc limit 1` (L329-331) | Lee `posts.id`, `slug`, `estado`. Escribe `comentarios.post_id`, `autor_nombre`, `autor_email`, `cuerpo`, `estado`, `creado_en`, `padre_id` |
| `DELETE /comentarios/:id` | [339-361](../../backend/start/routes.ts#L339-L361) | 404 si no existe. Autoriza con la cabecera `x-clave-moderacion` comparada con `SECRETO_SESION`, o con un `autor_email` que coincida sin distinguir mayúsculas ni espacios. Si no, 403. Si pasa, borra y devuelve `{borrado:true}` | `comentarios.id`, `autor_email` |
| `PATCH /comentarios/:id` | [366-393](../../backend/start/routes.ts#L366-L393) | 404 si no existe. Exige que el `autor_email` coincida exactamente y que `estado` sea `'pendiente'`; si no, 403. Responde 400 si falta `cuerpo`. Actualiza `cuerpo` y devuelve `{comentario}` | `comentarios.id`, `autor_email`, `estado`, `cuerpo` |

El DELETE depende de la constante `SECRETO_SESION`
([routes.ts:25](../../backend/start/routes.ts#L25)). Según su propio comentario, esa clave también
firma las sesiones, así que es compartida y no exclusiva de esta feature.

## 2. Endpoints de otras features que devuelven o usan comentarios

| Verbo y ruta | Líneas | Uso de comentarios |
|---|---|---|
| `GET /posts/:slug` (detalle del post) | [150-155](../../backend/start/routes.ts#L150-L155) y [171](../../backend/start/routes.ts#L171) | Devuelve el array `comentarios` con `select *` y el filtro `estado = 'aprobado'` exacto, ordenado por `creado_en asc`. Incluye todas las columnas, también `autor_email` |
| `GET /posts` (listado y búsqueda) | [74-80](../../backend/start/routes.ts#L74-L80) | Subconsulta `num_comentarios` con el filtro `estado <> 'pendiente'` |
| `GET /posts/:slug/relacionados` | [217-229](../../backend/start/routes.ts#L217-L229) | Cuenta los comentarios de cada relacionado con el filtro `estado <> 'pendiente'` y lo devuelve como `num_comentarios` |

`GET /posts/recientes`, `/categorias`, `/etiquetas` y `/autores/:id` no tocan la tabla `comentarios`.

## 3. Base de datos

### Migración

[1769000000005_create_comentarios_table.ts:12-21](../../backend/database/migrations/1769000000005_create_comentarios_table.ts#L12-L21)

| Columna | Tipo |
|---|---|
| `id` | increments |
| `post_id` | integer, not null. Sin FK ni índice |
| `autor_nombre` | string, not null |
| `autor_email` | string, not null |
| `cuerpo` | text, not null |
| `estado` | string, not null, por defecto `'pendiente'`. Texto libre, sin CHECK ni enum |
| `creado_en` | dateTime, nullable |
| `padre_id` | integer, nullable. Sin FK |

La tabla no tiene `created_at` ni `updated_at`. Hay un esquema Lucid generado en
[database/schema.ts:73-91](../../backend/database/schema.ts#L73-L91) (`ComentarioSchema`), pero no
existe un modelo en `backend/app/models/`.

### Semilla

[blog_seeder.ts](../../backend/database/seeders/blog_seeder.ts)

- Vacía la tabla en L18 e inserta los datos en L128. El array `COMENTARIOS` ocupa
  [L479-579](../../backend/database/seeders/blog_seeder.ts#L479-L579) y tiene 9 filas.
- Valores de `estado`:
  - `'aprobado'`: ids 1, 2, 6 y 8
  - `'Aprobado'`: ids 3 y 9
  - `'ok'`: id 4
  - `'pendiente'`: ids 5 y 7
- Solo hay una respuesta: el id 2 tiene `padre_id = 1`.
- Hay comentarios en los posts 1, 2, 4 y 6. Ninguno en los borradores (posts 9 y 10).

## 4. Web (React)

| Componente | Qué hace con los comentarios |
|---|---|
| [Comentarios.jsx](../../frontend/src/componentes/Comentarios.jsx) | Lista los comentarios: vuelve a pedir `GET /posts/:slug` (L17-26) y pinta los raíz y sus respuestas por `padre_id` (L50-74). Tiene el formulario que llama a `POST /posts/:slug/comentarios` (L28-48 y L76-88). Lleva su propio contador `total` (L11, L39 y L54) |
| [DetallePost.jsx](../../frontend/src/componentes/DetallePost.jsx) | Calcula `numComentarios` a partir de `d.comentarios.length` (L30 y L38) y lo pinta en la cabecera (L71). Monta `<Comentarios>` en L123 |
| [ListaPosts.jsx:57](../../frontend/src/componentes/ListaPosts.jsx#L57) | Muestra `p.num_comentarios` de `GET /posts` |
| [PostsRelacionados.jsx:32](../../frontend/src/componentes/PostsRelacionados.jsx#L32) | Muestra `p.num_comentarios` de `/relacionados` |
| [index.css](../../frontend/src/index.css) | Estilos `.recuento-cabecera` (L73) y `.comentarios`, `.comentario`, `.respuesta` y `.comentario-autor` (L109-116) |

En la web no hay ninguna interfaz que llame a `PATCH` ni a `DELETE /comentarios/:id`. Esos dos
endpoints solo se pueden usar directamente contra la API.

## 5. Tests que ya existen

[tests/functional/comentarios.spec.ts](../../backend/tests/functional/comentarios.spec.ts) tiene un
solo test: el camino feliz del POST.

## 6. Lo que no se ha asignado a la feature

1. **`GET /posts/:slug`.** Es el único sitio donde se listan comentarios, y el README define la
   feature como "crear, editar, borrar y **listar**". Pero el endpoint pertenece a la feature de
   posts y devuelve mucho más (autor, categoría, etiquetas, anterior y siguiente). Falta decidir si
   el bloque de L150-155 y la clave `comentarios` de L171 cuentan como de comentarios o de posts.
2. **Los recuentos `num_comentarios`** de `GET /posts` y `/relacionados`, y los contadores del front
   en `DetallePost` y `ListaPosts`. Leen la tabla de comentarios, pero forman parte de los endpoints
   y componentes de posts. Están como "otras features que usan comentarios" sin asignarlos.
3. **`SECRETO_SESION`.** El DELETE lo usa, pero es configuración global compartida con las sesiones.
4. **`ComentarioSchema` en `database/schema.ts`.** Es un archivo generado que ningún código del blog
   importa, por lo que se ha visto.
