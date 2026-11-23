# Prompts

Aquí se registran **todos los prompts** para hacer el ejercicio, en el orden en que se ejecutan,
con el modelo y la herramienta de cada uno. Los que aún no hayas ejecutado están preparados para
lanzarlos; completa sus datos después de cada ejecución.

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

### **Parte A: la auditoría**

---

Los siguientes prompts están preparados para ejecutarse **uno a uno**, en orden. Registra el
modelo y la herramienta que uses al ejecutar cada uno; no los edites después de lanzarlos.

## Prompt 1 — reunir el mapa de evidencia

```text
Audita exclusivamente la feature de comentarios de este blog, de punta a punta: API (crear, editar, borrar y listar), base de datos y lo que muestra la web. No audites el resto del sistema, no implementes cambios y no escribas todavía el informe.

Inspecciona el código y reúne evidencia verificable para posibles hallazgos. Para cada evidencia anota la ruta del archivo, la línea exacta y qué demuestra. No afirmes nada que no puedas respaldar con esa evidencia. Mantén el alcance limitado a comentarios.

Devuélveme un mapa breve de evidencias y una lista provisional de hallazgos candidatos. Señala explícitamente lo que no hayas podido comprobar; no lo rellenes de memoria.
```

**Modelo:** Claude Sonnet 5.5 (`claude-sonnet-5-5`)  
**Herramienta:** Claude Code (CLI), en la rama `auditoria-HALR`  
**Qué salió:** Mapa de 24 evidencias con archivo:línea (`backend/start/routes.ts`, migración, semilla, SQLite local en solo lectura, `Comentarios.jsx`, `DetallePost.jsx`, test y README) y 14 hallazgos candidatos provisionales. Los principales: el `autor_email` sale en respuestas públicas y se usa como credencial para borrar o editar, la clave de moderación está hardcodeada, no existe ningún endpoint que apruebe comentarios, los recuentos se contradicen (`<> 'pendiente'` frente a `= 'aprobado'`) con estados `'Aprobado'` y `'ok'` mezclados, y hay dos contadores distintos en el front. Quedó marcado como no comprobado todo lo que requiere ejecución (la API no se arrancó), `config/cors.ts`, `kernel.ts`, `handler.ts`, rate-limiting, producción y el historial git. No se escribió el informe ni se cambió código.

## Prompt 2 — ordenar los hallazgos candidatos

```text
A partir de la evidencia reunida, prepara los hallazgos de la auditoría, siempre limitados a comentarios.

Ordénalos primero por precedencia: lo que abarata o desbloquea otro hallazgo va antes; nombra el hallazgo afectado. Dentro de cada nivel, ordénalos por daño dividido por radio. Si un mismo problema combina daño alto y precedencia baja, sepáralo en dos hallazgos: la contención que corta el daño ahora y el arreglo de fondo para su turno.

Cada hallazgo debe ser una sola idea y tener evidencia con ruta y línea exacta. Para cada uno determina el efecto concreto y quién lo nota, cuántos sitios concretos habría que tocar y si se deshace con un revert; explica por qué si no. Si un dato no está verificado, indícalo como pendiente, no lo supongas.

Las propuestas que nadie haya medido todavía no van en la tabla: ponlas en una sección aparte y escribe qué habría que medir para evaluarlas. No escribas aún el archivo final; muéstrame el borrador y cualquier dato que siga pendiente.
```

**Modelo:** Claude Sonnet 5.5 (`claude-sonnet-5-5`)  
**Herramienta:** Claude Code (CLI), en la rama `auditoria-HALR`  
**Qué salió:** Borrador de 17 hallazgos en 5 niveles de precedencia (0 tests de caracterización; 1 contenciones; 2 desbloqueadores de fondo; 3 dependientes; 4 independientes), con ratio daño/sitios, y 8 propuestas sin medir fuera de la tabla. Se separó en contención y fondo el correo expuesto (C1 y A1) y la clave de moderación (C2 y A1). Antes de ordenar se cerraron comprobaciones pendientes del Prompt 1: el front no tiene llamadas PATCH ni DELETE, no hay rate-limiting en el backend, la BD local no tiene fechas nulas ni respuestas huérfanas, y `start/env.ts` no define variable para la clave. Se corrigió una evidencia del Prompt 1: el `pragma foreign_keys = 0` era de la conexión CLI y no vale; lo que demuestra la ausencia de FK es el DDL sin `REFERENCES`. Quedan pendientes y sin completar de memoria: si `'ok'` cuenta como aprobado, la reproducción por HTTP de los recuentos y de la carrera de B1, si existe un panel de moderación externo, si "editar solo pendientes" es intencional, si la contención puede aplicarse en `routes.ts` antes del refactor, y los consumidores externos del contrato. No se escribió el archivo final ni se cambió código.

## Prompt 3 — escribir el entregable

```text
Escribe el informe final en docs/auditoria/comentarios.md. Si hace falta, crea los directorios necesarios.

El informe debe cubrir solo comentarios y contener una tabla con exactamente estas seis columnas: Qué es; Dónde está (ruta y línea que lo demuestra); Qué se rompe y quién lo nota; Cuántos sitios hay que tocar; ¿Se deshace con un revert?; ¿Bloquea o desbloquea a otro hallazgo?

Usa una fila por hallazgo y una frase para explicar qué es. No incluyas en la tabla hallazgos sin evidencia con líneas exactas. Respeta el orden por precedencia y, dentro de cada nivel, daño dividido por radio. Divide en dos filas la contención y el arreglo de fondo cuando corresponda. Pon las propuestas aún no medidas fuera de la tabla, junto con lo que habría que medir. Conserva explícitamente como pendiente cualquier casilla que no se haya podido comprobar; no la completes de memoria.

No cambies código ni hagas una auditoría general. Al terminar, confirma la ruta del informe y qué partes quedaron pendientes.
```

**Modelo:** Claude Sonnet 5.5 (`claude-sonnet-5-5`)  
**Herramienta:** Claude Code (CLI), en la rama `auditoria-HALR`  
**Qué salió:** Informe escrito en `docs/auditoria/comentarios.md` con una tabla de exactamente seis columnas y 17 filas (T1, C1, C2, A2, V1, A1, R1, M1, D1, B1, O1, P1, W1, E1, F2, F1, X1), ordenadas por nivel de precedencia (N0 a N4) y por daño/sitios dentro de cada nivel. Como la tabla no admite columnas extra, el nivel, el ID y el ratio van dentro de las celdas. Las propuestas sin medir van fuera de la tabla con lo que habría que medir. Quedan marcados como PENDIENTE: si `ok` cuenta como aprobado, la reproducción por HTTP de los recuentos y de la carrera de B1, el panel de moderación externo, si editar solo pendientes es intencional, si C1 y C2 pueden ir en `routes.ts` antes del refactor, los consumidores externos del contrato, el mecanismo de A1, el número de tests de T1 y todo lo no leído (producción, historial de git, `config/cors.ts`, `kernel.ts`, `handler.ts`, `bodyparser`). No se cambió código y no se hizo commit.

## Prompt opcional — guardar el estado al acabarse el tiempo

Ejecuta este prompt solo si se acaba el tiempo antes de completar la auditoría.

```text
Se acabó el tiempo. Detén la investigación ahora: no abras más archivos, no busques más evidencia y no completes nada de memoria. Guarda en docs/auditoria/comentarios.md únicamente los hallazgos y evidencias que ya estaban comprobados. Deja las casillas incompletas marcadas como pendientes y conserva fuera de la tabla las propuestas no medidas con sus mediciones pendientes. No cambies código. Después de guardar ese estado parcial, detente.
```

**Modelo:** Claude Sonnet 5.5 (`claude-sonnet-5-5`)  
**Herramienta:** Claude Code (CLI), en la rama `auditoria-HALR`  
**Qué salió:** Sin abrir ni buscar nada más, se actualizó `docs/auditoria/comentarios.md` con un aviso de estado parcial al principio. Se retiró B1 de la tabla porque su efecto (devolver la fila de otro usuario) nunca se reprodujo; ahora figura entre las propuestas sin medir, con la medición pendiente (dos POST concurrentes). La tabla queda con 16 hallazgos, y se quitó B1 de la lista de lo que desbloquea A2. Se conservaron las casillas marcadas como PENDIENTE y las propuestas no medidas. No se cambió código ni se hizo commit.

### **Parte B: las tres líneas**

## Prompt 4 — comprobar y dejar la reflexión al final del informe

```text
En docs/auditoria/comentarios.md, asegúrate de que, inmediatamente debajo de la tabla de hallazgos y antes de «Propuestas aún no medidas», aparezcan exactamente estas tres líneas, sin duplicarlas si ya están presentes:

Hallazgos devueltos por el agente: 17; comprobados por mí abriendo el archivo: 16.
No supe decidir la prioridad de C1 (contener ya la exposición de emails) frente a T1/A2 (caracterizar y respetar el refactor test-first), porque cortar el daño de inmediato chocaba con priorizar la caracterización y la arquitectura antes de cambiarlo.
El agente dejó pendiente si hay consumidores externos de `autor_email` o de las formas actuales de error; no pude comprobarlo ni a favor ni en contra.

No vuelvas a investigar ni cambies la tabla, otras secciones o archivos. Si las tres líneas ya están exactamente así, no hagas cambios. Confirma si las encontraste o las añadiste.
```

**Modelo:** Claude Sonnet 5.5 (`claude-sonnet-5-5`)  
**Herramienta:** Claude Code (CLI), en la rama `auditoria-HALR`  
**Qué salió:** Las tres líneas ya estaban en `docs/auditoria/comentarios.md` (líneas 46-48), exactamente iguales, justo debajo de la tabla de hallazgos y antes de «Propuestas aún no medidas», y una sola vez. No se añadió ni se cambió nada en el informe. Las cifras cuadran con la tabla actual: 16 filas, tras retirar B1 de los 17 hallazgos iniciales. No se investigó nada más ni se cambió código.
