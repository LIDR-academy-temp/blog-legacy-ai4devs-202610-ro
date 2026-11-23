# Auditoría de la feature de comentarios

Alcance: solo comentarios (API de crear, editar, borrar y listar; base de datos; lo que muestra la web).
Rama `auditoria-HALR`. No se ha cambiado código. Rutas relativas a `backend/` salvo que empiecen por
`frontend/`, `README.md`, `CLAUDE.md` o `prompts.md`.

> **ESTADO PARCIAL (se acabó el tiempo).** La investigación se detuvo sin abrir más archivos ni buscar más evidencia.
> La tabla conserva solo hallazgos cuya evidencia (ruta y línea) se leyó en el código. Todo se obtuvo por lectura
> de código y consultas de solo lectura a la base local; **la API no se ejecutó**. Las columnas «Qué se rompe y quién
> lo nota» son inferencias de esa lectura. B1 se retiró de la tabla porque su efecto es una hipótesis sin reproducir
> y pasó a la sección de propuestas sin medir. Las casillas incompletas siguen marcadas como PENDIENTE.

## Cómo leer la tabla

- **Orden:** primero por precedencia (lo que abarata o desbloquea a otro va antes) y, dentro de cada nivel
  (`N0` a `N4`), por daño dividido por sitios. Cada fila empieza por su nivel y su ID.
- **Daño:** juicio del auditor (3 alto, 2 medio, 1 bajo), no medido. **Sitios:** ubicaciones concretas, existentes o nuevas.
  La columna de sitios muestra `sitios (daño/sitios = ratio)`.
- **Contención y fondo:** donde un mismo problema combina daño alto y precedencia baja se divide en dos filas
  (C1 y C2 cortan el daño ahora; A1 es el arreglo de fondo).
- **PENDIENTE:** dato que no se pudo comprobar. No está completado de memoria.
- `CLAUDE.md:43` exige la arquitectura por capas en cada endpoint que se toque y `CLAUDE.md:95-104` exige caracterizar antes
  de refactorizar y tratar el arreglo de un bug como una tarea aparte. Por eso T1 y A2 van antes de casi todo.

## Hallazgos

| Qué es | Dónde está (ruta y línea que lo demuestra) | Qué se rompe y quién lo nota | Cuántos sitios hay que tocar | ¿Se deshace con un revert? | ¿Bloquea o desbloquea a otro hallazgo? |
|---|---|---|---|---|---|
| **[N0] T1.** El único test cubre el camino feliz y no fija el contrato. | `tests/functional/comentarios.spec.ts:15-43` (solo afirma 201 y `autor_nombre`, `:41-42`); `:5-10` lo admite. | Un campo perdido o una fuga de email pasa sin aviso. Lo nota el equipo al romper algo. | Ampliar 1 archivo, unos 4 endpoints por la matriz mínima. Número de tests: PENDIENTE de estimar. | Sí, es aditivo. | Desbloquea a todos los demás (`CLAUDE.md:95-104`). |
| **[N1] C1 (contención).** Dejar de emitir el `autor_email` de quien comenta en el detalle público. | `routes.ts:152-153` (`select *` de comentarios), `:171` (se devuelve); `README.md:66` (regla 6). | Cualquiera con DevTools o curl obtiene los correos de los comentarios aprobados. Lo notan los comentaristas (privacidad). El front no lo usa: `frontend/src/componentes/Comentarios.jsx:58-70` pinta nombre, fecha y cuerpo. | 1 (proyección en `routes.ts:152`). Daño 3: 3/1 = 3.0. | El código sí. Lo ya servido no se puede retirar. | Corta el camino de explotación de A1. El fondo de la fuga (transformer) cae en A2. Pendiente: consumidores externos del campo. |
| **[N1] C2 (contención).** Sacar la clave de moderación del código y rotarla. | `routes.ts:25` (literal con TODO de 2024-11), `:351` (`===` con la cabecera `x-clave-moderacion`). | Quien tenga el repo puede borrar cualquier comentario con esa cabecera. Se nota solo tras el abuso. | 4 (`routes.ts:25` y `:351`, `start/env.ts`, `.env.example`). Daño 3: 3/4 = 0.75. | El código sí. La clave vieja queda en el historial de git y el revert no la invalida. Uso en producción y historial de git: PENDIENTE. | El fondo es A1. PENDIENTE: si hay clientes externos que envíen la cabecera (el front no: sin resultados en `frontend/src`). |
| **[N2] A2.** Las reglas de comentarios viven en `routes.ts` sin validador, servicio, repositorio ni transformer. | `routes.ts:6-11`, `:295-393`; `CLAUDE.md:43`; `database/schema.ts:73-90` (`ComentarioSchema` sin uso en las rutas). | No lo nota el lector. El equipo tiene SQL, validación y forma de respuesta mezclados en cada arreglo. | 6 ubicaciones existentes (`routes.ts:295`, `:339`, `:366`, `:152`, `:75`, `:222`) más archivos nuevos. Daño 1: 1/6 = 0.2. | Sí por commits si no cambia la conducta. Si añade el alias `#repositories/*` en `package.json`, hay que revertirlo también. | Desbloquea A1, R1, M1, D1, O1, P1, W1, E1 y X1. Requiere T1 antes. PENDIENTE: si C1 y C2 pueden aplicarse en `routes.ts` antes del refactor. |
| **[N2] V1.** `estado` no tiene vocabulario: conviven `aprobado`, `Aprobado` y `ok`. | `database/migrations/1769000000005_create_comentarios_table.ts:4-7,18`; `database/seeders/blog_seeder.ts:509-510`, `:521-522`, `:575`; base local, comentarios 3, 4 y 9. | Los comentarios 3, 4 y 9 están aprobados de hecho, pero el detalle los oculta (`routes.ts:152-153`). Lo notan sus autores y los lectores. | 2 (migración nueva y semilla) más datos de producción: PENDIENTE. Daño 2: 2/2 = 1.0. | No limpio. Normalizar pierde la distinción `ok` frente a `Aprobado`, y un CHECK en SQLite obliga a recrear la tabla. | Desbloquea R1 y M1. PENDIENTE: si `ok` cuenta como aprobado (`README.md:64` solo dice «aprobados»). |
| **[N2] A1 (fondo).** Borrar y editar se autoriza con el correo del autor, que no es un secreto, y con una clave de moderación compartida. | `routes.ts:349-353` (DELETE), `:378-381` (PATCH). | Quien conozca el email de un comentarista puede borrar su comentario, o editarlo mientras esté pendiente. Lo nota el comentarista al perderlo. | 5 (DELETE, PATCH, POST que emite el justificante, migración y front si se añade UI). Daño 3: 3/5 = 0.6. El mecanismo está sin decidir. | El código sí. Una columna nueva con datos no se revierte limpia. | Es el fondo de C1 y C2. Desbloquea M1 y D1. Depende de A2. PENDIENTE: mecanismo (token por comentario o cuenta del starter kit, `routes.ts:399-418`). |
| **[N3] R1.** El recuento de comentarios difiere entre el listado y los relacionados, y el detalle. | `routes.ts:75-76` y `:222-224` (`<> 'pendiente'`) frente a `:152-153` (`= 'aprobado'`); `README.md:64`; `frontend/src/componentes/ListaPosts.jsx:57`, `frontend/src/componentes/PostsRelacionados.jsx:32`. | Post 1: 4 en el listado y 2 en el detalle. Post 6: 1 y 0. Calculado con el mismo predicado en SQLite, no por HTTP (reproducción por HTTP: PENDIENTE). Lo nota el lector. | 2 (`routes.ts:75-76`, `:222-224`). Daño 2: 2/2 = 1.0. | Sí. | Depende de V1 (qué estados cuentan) y de A2. |
| **[N3] M1.** No hay endpoint que apruebe un comentario. | `routes.ts:324`, `:359`, `:388` (únicas escrituras sobre `comentarios`); `README.md:50-52` (endpoints listados). | Todo comentario nuevo nace pendiente y no se publica por la API. Lo notan comentaristas y moderación. | 4 (ruta, controlador, servicio y validador nuevos). Daño 3: 3/4 = 0.75. | El código sí. Los estados ya cambiados no se deshacen. | Depende de V1, A1 y A2. PENDIENTE: si existe un panel externo (`database/seeders/blog_seeder.ts:509` habla de un «panel viejo»; no apareció en los archivos que mencionan comentarios, pero no se buscó fuera de ellos). |
| **[N3] D1.** PATCH y DELETE autorizan con reglas distintas. | `routes.ts:349-357` frente a `:378-381` (el comentario de `:375-377` lo reconoce). | Un autor con el email en otra capitalización puede borrar pero no editar. El moderador borra pero no edita. Nadie edita tras la aprobación. | 2 (`routes.ts:349-357`, `:378-381`). Daño 1: 1/2 = 0.5. | Sí. | Se resuelve con A1. PENDIENTE: si «editar solo pendientes» es intencional (`routes.ts:379`). |
| **[N4] O1.** DELETE deja las respuestas huérfanas. | `routes.ts:359`; `database/migrations/1769000000005_create_comentarios_table.ts:20` (sin clave foránea); `frontend/src/componentes/Comentarios.jsx:63-65`; `database/seeders/blog_seeder.ts:500` (el comentario 2 cuelga del 1). | Borrar el comentario 1 deja el 2 en la tabla e invisible en la web. Lo nota el moderador, no el lector. Hoy hay 0 huérfanos en la base local. | 1 (handler). La clave foránea sumaría 1 migración. Daño 1: 1/1 = 1.0. | El código sí. Las filas ya borradas no se recuperan (DELETE físico). | Depende de A2. |
| **[N4] P1.** `padre_id` se acepta sin validar. | `routes.ts:308`, `:326`. | Una respuesta puede colgar de un id inexistente o de otro post y la web no la pinta. El front nunca envía `padre_id` (`frontend/src/componentes/Comentarios.jsx:33`), así que solo ocurre vía API. | 1 (`routes.ts:308`). Daño 1: 1/1 = 1.0. | Sí. | Depende de A2. |
| **[N4] W1.** PATCH no exige la longitud mínima que POST sí exige. | `routes.ts:383-386` frente a `:319`. | Un comentario válido puede editarse a un carácter mientras está pendiente. Lo nota el lector tras la aprobación. | 1 (`routes.ts:383-386`). Daño 1: 1/1 = 1.0. | Sí. | Depende de A2. |
| **[N4] E1.** El email solo se valida con «contiene @». | `routes.ts:316`. | Entran correos como `a@`. Pesa más porque el correo hace de credencial (A1). | 1 (`routes.ts:316`). Daño 1: 1/1 = 1.0. | Sí. | Depende de A2. |
| **[N4] F2.** El front tiene dos contadores de comentarios independientes. | `frontend/src/componentes/DetallePost.jsx:30`, `:38`, `:71`; `frontend/src/componentes/Comentarios.jsx:11`, `:23`, `:54`. | La cabecera del artículo y la sección pueden mostrar números distintos. Lo nota el lector. | 2 (un archivo cada uno). Daño 1: 1/2 = 0.5. | Sí. | Va antes que F1: si el estado se eleva a un único origen, F1 desaparece. |
| **[N4] F1.** La sección suma 1 al enviar un comentario que queda pendiente. | `frontend/src/componentes/Comentarios.jsx:38-39`, `:54`. | Tras enviar se ve «N+1 comentarios» sin que aparezca ninguno nuevo, y el aviso dice «pendiente». Lo nota quien comenta. | 1 (`Comentarios.jsx:39`). Daño 1: 1/1 = 1.0. | Sí. | Subsumido por F2. |
| **[N4] X1.** La misma entrada inválida devuelve cuatro formas de error distintas. | `routes.ts:311`, `:314`, `:317`, `:320`; `frontend/src/componentes/Comentarios.jsx:44`. | Hay que mirar `error`, `message`, `errores` o `motivo`. `200 {ok:false}` parece un éxito a quien mire solo el status. | 5 (cuatro ramas de `routes.ts` y `Comentarios.jsx:44`). Daño 1: 1/5 = 0.2. | El código sí, pero cambia el contrato público (`CLAUDE.md:104`). | Depende de A2 y de T1. PENDIENTE: consumidores externos del contrato. |

Hallazgos devueltos por el agente: 17; comprobados por mí abriendo el archivo: 16.
No supe decidir la prioridad de C1 (contener ya la exposición de emails) frente a T1/A2 (caracterizar y respetar el refactor test-first), porque cortar el daño de inmediato chocaba con priorizar la caracterización y la arquitectura antes de cambiarlo.
El agente dejó pendiente si hay consumidores externos de `autor_email` o de las formas actuales de error; no pude comprobarlo ni a favor ni en contra.

## Propuestas aún no medidas (fuera de la tabla)

No hay medición detrás de ellas. Esto es lo que habría que medir para evaluarlas.

| Propuesta | Qué habría que medir |
|---|---|
| Anti-spam o límite de frecuencia en POST. No hay `throttle` ni `limiter` en el `.ts` ni en `package.json` del backend. | POST por hora reales; el porcentaje de pendientes que son spam (la semilla tiene una sola muestra, el comentario 7); si la infraestructura ya limita. |
| Índice en `comentarios.post_id` (`database/migrations/1769000000005_create_comentarios_table.ts:14`). | `EXPLAIN QUERY PLAN` y tiempos con volumen real. La base local tiene 9 filas. |
| Consultas N+1 en relacionados (`routes.ts:222-231`). | Latencia con datos reales. |
| Paginar los comentarios de un post. | Comentarios por post. El máximo local es 4. |
| Segunda petición del post entero en `frontend/src/componentes/Comentarios.jsx:19`. | Tamaño de la respuesta y tiempo frente a la petición de `DetallePost.jsx:34`. |
| `cuerpo` que no sea texto o sea enorme (`routes.ts:319`, `:325`). | Reproducir si devuelve 500 y cuál es el límite del body parser (no leído). |
| Borrado lógico en lugar de físico (`routes.ts:359`). | Frecuencia de borrados y necesidad de auditoría. |
| Interfaz para responder, editar o borrar. No existe: sin resultados de `PATCH`, `DELETE` ni `padre_id` de lectura en `frontend/src`. | Demanda real de esas acciones. |
| Hipótesis B1 (retirada de la tabla): POST devuelve la última fila global y no la insertada (`routes.ts:329-331`, `:333`), por lo que con peticiones casi simultáneas podría devolver el comentario y el email de otro usuario. | Reproducirlo con dos POST concurrentes y comprobar qué fila recibe cada cliente. Sin esa prueba, el efecto es PENDIENTE. |

## Pendientes y límites de la auditoría

- **No se ejecutó la API.** Todo está por lectura de código y consultas de solo lectura a la base local (`backend/tmp/db.sqlite3`, ignorada por git). Los recuentos de R1 salen de SQLite con el mismo predicado y no de una petición HTTP.
- **Pendientes de decisión o de dato:**
  1. Si `ok` cuenta como aprobado (V1, R1).
  2. Reproducción por HTTP de los recuentos (R1) y de la carrera (B1).
  3. Si existe un panel de moderación externo (M1).
  4. Si «editar solo pendientes» es intencional (D1).
  5. Si C1 y C2 pueden aplicarse en `routes.ts` antes del refactor o deben esperar a A2.
  6. Consumidores externos del campo `autor_email` y de los formatos de error (C1, X1).
  7. Mecanismo de autorización de autor y moderador (A1).
  8. Número de tests de T1.
- **No comprobado:** la base de producción y su motor, el historial de git de la clave de moderación, `config/cors.ts`, `start/kernel.ts`, `app/exceptions/handler.ts`, `config/bodyparser.ts`, la constante `COBERTURA_MINIMA` de `.claude/hooks/guard-tests.mjs` y cualquier límite de frecuencia a nivel de infraestructura.
- **Corrección a una evidencia anterior:** el `pragma foreign_keys = 0` de la conexión del CLI de sqlite no prueba nada sobre la aplicación. La ausencia de clave foránea la demuestra el DDL de `comentarios`, que no contiene `REFERENCES`.
- **Descartado por no tener efecto verificable:** `creado_en` nulo se pintaría como «null», pero hay 0 filas así; y el plural fijo «1 comentarios», que es cosmético.
