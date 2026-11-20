# Prompts

Aquí van **todos los prompts que lanzaste** para hacer el ejercicio, en el orden en que los
lanzaste, con el modelo y la herramienta de cada uno.

Esto no es papeleo. Lo que se revisa es **cómo pediste las cosas**, no solo lo que salió: un
resultado flojo con un prompt bueno y un resultado flojo con un prompt vago necesitan feedback
distinto, y sin este archivo no se distinguen.

## Cómo rellenarlo

- Un apartado `## Prompt N` por cada prompt.
- **Pega el prompt tal cual lo lanzaste**, dentro del bloque de código, aunque ocupe diez líneas
  y aunque tenga faltas. No lo reescribas para que quede bien: el que arreglaste mentalmente
  después no es el que lanzaste.
- Incluye también los que **no funcionaron**. Suelen ser los más útiles de leer.
- `Modelo` y `Herramienta` en todos. Si cambiaste de una a otra a mitad, se nota aquí.

Borra el ejemplo de abajo cuando escribas el primero.

---

## Prompt 1

**Modelo:** Opus 5.5
**Herramienta:** Claude Code (extensión de VS Code)

```
Audita SOLO la feature de comentarios del blog, de punta a punta, en sus tres capas:
- API: las rutas POST /posts/:slug/comentarios, PATCH /comentarios/:id, DELETE /comentarios/:id y cómo se listan/cuentan los comentarios dentro de GET /posts, GET /posts/:slug y GET /posts/:slug/relacionados (backend/start/routes.ts).
- Base de datos: backend/database/migrations/1769000000005_create_comentarios_table.ts, backend/database/schema.ts y los comentarios de backend/database/seeders/blog_seeder.ts.
- Web: frontend/src/componentes/Comentarios.jsx, DetallePost.jsx, ListaPosts.jsx y PostsRelacionados.jsx (solo lo relativo a comentarios).
- Tests existentes: backend/tests/functional/comentarios.spec.ts.

No audites posts, autores, categorías ni etiquetas salvo donde toquen comentarios. No modifiques código.

Contrasta cada hallazgo con las reglas de negocio del README (sobre todo 2, 5 y 6) y con la arquitectura objetivo de CLAUDE.md. Lee los archivos antes de afirmar nada; si no puedes señalar la línea exacta, no escribas el hallazgo.

Escribe el resultado en docs/auditoria/comentarios.md (no en el chat) con:
1. Título "# Auditoría — feature de comentarios".
2. Una tabla markdown, ordenada por prioridad (P1 primero), con UNA fila por hallazgo y estas columnas exactas:
   | # | Prioridad | Qué es | Dónde está (archivo:línea + la línea citada) | Qué se rompe y quién lo nota | Sitios a tocar (contados) | ¿Se deshace con un revert? | ¿Bloquea o desbloquea a otro? |
   - "Qué es": una sola frase. Si necesitas más, son dos hallazgos.
   - "Dónde está": ruta:línea y la línea de código literal entre backticks.
   - "Qué se rompe": efecto concreto y a quién le llega ("Nadie todavía" es válido).
   - "Sitios a tocar": número contado, listando los archivo:línea.
   - "¿Revert?": Sí / No, y si es No, por qué.
   - "¿Bloquea/desbloquea?": nombra el # del otro hallazgo o "A ninguno".
3. Debajo de la tabla, una sección "## Parte B" vacía con tres viñetas en blanco (la relleno yo).

Al terminar, dime en el chat solo: cuántos hallazgos escribiste y la lista de archivo:línea de cada uno.
```

**Qué salió:** 24 hallazgos (5 P1, 12 P2, 7 P3) en docs/auditoria/comentarios.md, cada uno con archivo:línea y línea citada; sin tocar código.
