## Prompt 1

**Modelo:**  Opus 5.5
**Herramienta:** Claude Code

```
dame un resumen de los ultimos commits y busca en qué archivos aparece una palabra cualquiera del dominio
```

**Qué salió:** 
```
  Últimos commits (rama s6/start)

  El historial tiene solo 3 commits:

  Commit: 35c93cc
  Fecha: 2026-09-11
  Autor: LIDR-AI4Devs
  Qué hace: docs: cambia las instrucciones del README.md para que apunten a los repos del
    cohorte 202610-ro (3 líneas cambiadas).
  ────────────────────────────────────────
  Commit: b5da02a
  Fecha: 2026-09-11
  Autor: Earvin Saúl Pérez Ramos
  Qué hace: docs: amplía el README.md con los requisitos (Node 24 o superior y make), dónde
    funciona (macOS, Linux, WSL; no PowerShell), los puertos fijos, que el ejercicio es auditar
    la feature de comentarios, y que la rama de partida se trae de upstream (+32/−10).
  ────────────────────────────────────────
  Commit: e0e0943
  Fecha: 2026-09-02
  Autor: Earvin Saúl Pérez Ramos
  Qué hace: initial import: todo el proyecto. Incluye el backend AdonisJS (config, auth,
    migraciones de autores, categorías, etiquetas, posts y comentarios, semilla), la
    configuración de .claude (comando refactor, hook guard-tests.mjs), el Makefile y el
    CLAUDE.md.

  También tienes cambios locales sin commitear en backend/.adonisjs/*, backend/.env.example y backend/database/schema.ts. Seguramente los archivos de .adonisjs se regeneraron solos al arrancar el servidor.

  Dónde aparece «borrador»

  Elegí «borrador», el estado de un post que no debe publicarse. Aparece 8 veces en 5 archivos:

  ┌─────────────────────────────────────────────────────────────────┬─────────────────────────┐
  │                             Archivo                             │          Veces          │
  ├─────────────────────────────────────────────────────────────────┼─────────────────────────┤
  │ backend/database/seeders/blog_seeder.ts                         │ 3                       │
  ├─────────────────────────────────────────────────────────────────┼─────────────────────────┤
  │ CLAUDE.md                                                       │ 2                       │
  ├─────────────────────────────────────────────────────────────────┼─────────────────────────┤
  │ .claude/commands/refactor.md                                    │ 1                       │
  ├─────────────────────────────────────────────────────────────────┼─────────────────────────┤
  │ README.md                                                       │ 1                       │
  ├─────────────────────────────────────────────────────────────────┼─────────────────────────┤
  │ backend/database/migrations/1769000000004_create_posts_table.ts │ 1 (es un valor de la    │
  │                                                                 │ columna estado)         │
  └─────────────────────────────────────────────────────────────────┴─────────────────────────┘

  La palabra no aparece en backend/start/routes.ts, que es donde vive toda la lógica del blog. Eso no prueba que haya un bug, porque las rutas podrían filtrar por estado = 'publicado'. Pero conviene revisarlo contra la regla 1 del dominio: un borrador no aparece en listados, relacionados, recientes ni búsqueda. Si quieres, lo compruebo o busco otra palabra (slug, pendiente, autor_email…).
```


## Prompt 2

**Modelo:**  Opus 5.5
**Herramienta:** Claude Code

```
 genera la auditoria priorizada de la feature de comentarios, escribe el resultado en docs/auditoria/comentarios.md
```

**Qué salió:** 
```
 Write(docs\auditoria\comentarios.md)
Wrote 60 lines to docs\auditoria\comentarios.md
# Auditoría priorizada — feature de comentarios

**Alcance:** crear, editar, borrar y listar comentarios en sus tre
s capas.

- **API:** `backend/start/routes.ts`, que contiene `POST /posts/:s
lug/comentarios`, `PATCH /comentarios/:id`, `DELETE /comentarios/:
id` y la parte de comentarios de `GET /posts/:slug`, `GET /posts`
y `GET /posts/:slug/relacionados`.
- **Base de datos:** `backend/database/migrations/1769000000005_cr
… +55 lines (~11 KB)

He escrito la auditoría en docs/auditoria/comentarios.md: 22 filas (21 hallazgos, porque uno está partido en contención y fondo) y 4 propuestas sin medir fuera de la tabla. Saqué cada línea citada de los archivos que abrí. No he cambiado código ni hecho commits.

Lo más urgente (nivel 0):
- #1: GET /posts/:slug devuelve el correo de cada comentarista, porque los comentarios salen con select * (routes.ts:153). Es la contención; el arreglo de fondo es la #10, pasar las respuestas por un transformer.
- #2: la clave de moderación está escrita en el código y en el historial de git (routes.ts:25). Con ella cualquiera puede borrar cualquier comentario. Un revert no la retira, así que hay que cambiarla.

Lo que desbloquea el resto (nivel 1):
- #3: hoy hay un solo test de comentarios, y comprueba un único campo. Casi ningún otro arreglo se puede hacer con seguridad hasta tener tests que fijen el comportamiento actual.
- #4: el estado del comentario es texto libre, y conviven 'aprobado', 'Aprobado' y 'ok'. Por eso los comentarios 3, 4 y 9, que están aprobados, no salen en el detalle.

El resto de la tabla:
- Nivel 2: la lógica está toda en las rutas (#5); las reglas para editar y para borrar no coinciden (#6); y el recuento del listado incluye comentarios que no están aprobados (#7). En el post 1 eso hace que el listado diga 4 y el detalle muestre 2.
- Niveles 3 y 4: arreglos concretos en la API y en la web, como que el POST devuelve el último comentario de la tabla en vez del que acaba de crear (#8), o que la página muestra dos recuentos distintos (#18).

Las tres líneas (Parte B): solo he rellenado lo que me toca como agente, el número de hallazgos que devolví. El resto lo tienes que escribir tú, porque habla de lo que tú verificaste. Para la tercera línea hay un buen candidato: el comentario de routes.ts:21-24 dice que esa clave también firma las sesiones. Yo solo la encontré usada en la línea 351, y no pude confirmarlo ni descartarlo.
```