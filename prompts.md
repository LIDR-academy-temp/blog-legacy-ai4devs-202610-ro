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

---

## Prompt 1

**Modelo:** Claude Opus 5.5
**Herramienta:** Claude Code

```
Actúa como auditor técnico.

Voy a auditar únicamente la feature de comentarios.

No busques incidencias todavía.

Construye un mapa completo de la feature de comentarios de punta a punta:

- Rutas API relacionadas con comentarios
- Controladores involucrados
- Servicios, casos de uso o acciones llamados por esas rutas
- Modelos y tablas de base de datos implicados
- Componentes React que crean, muestran, editan o eliminan comentarios
- Tests relacionados

Para cada elemento indica:

1. Archivo
2. Responsabilidad
3. Dependencias directas

Devuelve el resultado como una lista estructurada.

No infieras nada que no puedas demostrar leyendo código.
```

**Qué salió:** a la primera. Mapa con archivo y línea, y la constatación de que no hay controladores, servicios ni modelo de comentarios: todo vive en `routes.ts`.

## Prompt 2

**Modelo:** Claude Opus 5.5
**Herramienta:** Claude Code

```
Actúa como auditor técnico.

He observado que:

- GET /posts/:slug filtra comentarios con estado = 'aprobado'
- GET /posts calcula num_comentarios con estado <> 'pendiente'
- El seeder contiene estados: aprobado, Aprobado, ok y pendiente

Analiza si esta diferencia puede producir comportamientos inconsistentes.

1. Localiza las consultas exactas.
2. Cita las líneas.
3. Enumera los estados posibles encontrados en el código y los datos semilla.
4. Construye ejemplos concretos de qué comentarios se muestran y cuáles se contabilizan.
5. Indica si existe una discrepancia observable para un usuario.
6. Clasifica el resultado como:
   - Hallazgo confirmado
   - No confirmado
   - Necesita más evidencia

No propongas soluciones.
```

**Qué salió:** hallazgo confirmado (C-01), con las consultas ejecutadas en solo lectura sobre la base local: el post 1 da 4 frente a 2, y el post 6 da 1 frente a 0.

## Prompt 3

**Modelo:** Claude Opus 5.5
**Herramienta:** Claude Code

```
Actúa como auditor técnico.

A partir del hallazgo confirmado sobre el criterio de conteo y visualización de comentarios:

Analiza su prioridad siguiendo exclusivamente estos criterios:

1. Qué comportamiento incorrecto produce.
2. Cuántas pantallas o funcionalidades afecta.
3. Cuántos usuarios podrían verlo.
4. Si la inconsistencia es temporal o permanente.
5. Si puede provocar pérdida de datos.
6. Si puede propagarse a otros cálculos o métricas.
7. Dependencias con otros posibles hallazgos de la feature.

No propongas correcciones.

Devuelve una valoración razonada de prioridad:
- Crítica
- Alta
- Media
- Baja

Explicando cada factor con evidencia.
```

**Qué salió:** prioridad Media, justificada factor por factor. Comprobó en la base en qué página de la portada salen los posts afectados y reprodujo el algoritmo de relacionados.

## Prompt 4

**Modelo:** Claude Opus 5.5
**Herramienta:** Claude Code

```
Guarda el hallazgo en docs/auditoria
```

**Qué salió:** lo guardó en un archivo propio con formato libre. No usó el formato del ejercicio porque no se lo pedí; lo corregí en el prompt 6.

## Prompt 5

**Modelo:** Claude Opus 5.5
**Herramienta:** Claude Code

```
Actúa como auditor técnico.

Analiza el ciclo de vida completo de un comentario creado desde la aplicación.

Empieza en:

POST /posts/:slug/comentarios

y sigue todas las rutas, consultas, actualizaciones de base de datos y componentes frontend relacionados.

Responde únicamente a estas preguntas:

1. En qué estado se crea un comentario nuevo.
2. Qué operaciones pueden modificar ese estado.
3. Qué estados son visibles para los lectores.
4. Si existe un flujo completo que permita que un comentario enviado por un usuario llegue a ser visible públicamente.
5. Si no existe dicho flujo, demuestra cada paso con archivo y líneas concretas.
6. Clasifica la conclusión como:
   - Hallazgo confirmado
   - No confirmado
   - Necesita más evidencia

No propongas soluciones.
No priorices todavía.
No hagas inferencias no respaldadas por código.
```

**Qué salió:** hallazgo confirmado (C-02). Para demostrar que nada cambia `estado`, buscó todas las escrituras sobre `comentarios` en el repo.

## Prompt 6

**Modelo:** Claude Opus 5.5
**Herramienta:** Claude Code

```
Guarda el hallazgo en docs/auditoria. Utiliza el formato que pide el ejercicio. Si es necesario reescribe el primer hallazgo con el mismo formato
```

**Qué salió:** encontró el formato en el `README.md` (tabla de seis casillas en `docs/auditoria/comentarios.md`), reescribió el primer hallazgo y borró el archivo anterior. Dejó la Parte B sin rellenar.

## Prompt 7

**Modelo:** Claude Opus 5.5
**Herramienta:** Claude Code

```
Actúa como auditor técnico.

A partir de toda la evidencia obtenida sobre la feature de comentarios, construye una tabla de candidatos a hallazgo.

No confirmes ni descartes todavía ninguno nuevo.

Para cada candidato indica:

- ID provisional
- Título corto
- Dónde aparece
- Evidencia inicial
- Qué habría que comprobar
- Riesgo percibido
- Estado:
  - Pendiente
  - Confirmado
  - Descartado

Incluye también los dos hallazgos ya investigados y marca claramente cuáles están confirmados.

No propongas soluciones.
No priorices todavía.
```

**Qué salió:** 24 candidatos: 2 confirmados (C-01, C-02) y 22 pendientes, con las dependencias que ya se veían entre ellos.

## Prompt 8

**Modelo:** Claude Opus 5.5
**Herramienta:** Claude Code

```
Actúa como auditor técnico.

Comprueba el candidato C-03.

Hipótesis:
El endpoint GET /posts/:slug expone autor_email de los comentarios a cualquier visitante.

Necesito:

1. Localizar la consulta exacta.
2. Seguir el flujo completo hasta la respuesta HTTP.
3. Determinar qué campos de comentarios se devuelven realmente.
4. Identificar si existe algún filtrado, serialización o transformación previa.
5. Determinar si autor_email llega al cliente.
6. Clasificar:
   - Confirmado
   - Descartado
   - Evidencia insuficiente

Aporta únicamente evidencias demostrables.
No propongas soluciones.
No priorices.
```

**Qué salió:** confirmado. Levantó la API y vio `autor_email` en la respuesta HTTP real. Al pararla quedó un proceso `node ace serve` escuchando en el 3401, que no mató porque no tenía claro si era suyo.

## Prompt 9

**Modelo:** Claude Opus 5.5
**Herramienta:** Claude Code

```
Actúa como auditor técnico.

Comprueba el candidato C-10.

Hipótesis:
Un usuario puede borrar comentarios ajenos si conoce el correo del autor.

Necesito:

1. Seguir el flujo completo de autorización de DELETE /comentarios/:id.
2. Indicar exactamente qué datos utiliza para autorizar.
3. Construir un escenario reproducible.
4. Analizar si la explotación depende de conocer autor_email.
5. Indicar si C-03 influye en la explotabilidad.
6. Clasificar:
   - Confirmado
   - Descartado
   - Evidencia insuficiente

No propongas soluciones.
No priorices.
```

**Qué salió:** confirmado con un test Japa temporal dentro de una transacción (la base no cambió y el archivo se borró después). Un DELETE anónimo con el correo sacado del GET público devolvió 200; los controles devolvieron 403.

## Prompt 10

**Modelo:** Claude Opus 5.5
**Herramienta:** Claude Code

```
Actúa como auditor técnico.

Tenemos estos hallazgos confirmados:

- C-01: El recuento no coincide con los comentarios mostrados.
- C-02: Ningún comentario enviado llega a publicarse.
- C-03: El detalle de un post expone el correo de quien comenta.
- C-10: Un visitante puede borrar comentarios ajenos utilizando el correo del autor.

Priorízalos siguiendo exclusivamente los criterios definidos por el ejercicio.

Para cada hallazgo indica:

1. Qué comportamiento se rompe.
2. Alcance funcional.
3. Alcance para usuarios.
4. Persistencia del problema.
5. Riesgo de pérdida de datos.
6. Dependencias con otros hallazgos.
7. Prioridad resultante.

Después genera una tabla resumen:

| ID | Prioridad | Justificación corta |

No propongas soluciones.
No introduzcas nuevos hallazgos.
No cambies el estado de confirmación.
```

**Qué salió:** C-10 Crítica, C-03 Alta, C-02 Alta y C-01 Media. Varias afirmaciones de alcance y dependencias decían más de lo demostrado; las cazó la revisión del prompt 11.

## Prompt 11

**Modelo:** Claude Opus 5.5
**Herramienta:** Claude Code

```
Revisa la tabla de priorización resultante.

Comprueba que cada nivel de prioridad está respaldado por evidencia ya obtenida y que no existe ninguna contradicción entre:

- Estado de confirmación
- Alcance
- Dependencias
- Prioridad asignada

Si encuentras inconsistencias, señálalas.

Si no encuentras ninguna, indica que la tabla está lista para incorporarse a docs/auditoria/comentarios.md.

No reevalúes los hallazgos.
No busques nuevos candidatos.
```

**Qué salió:** cinco inconsistencias de redacción, ninguna de prioridad. La principal: decía que C-02 acotaba a C-10, y la autorización de DELETE no mira `estado`.

## Prompt 12

**Modelo:** Claude Opus 5.5
**Herramienta:** Claude Code

```
Sí. Aplica las correcciones identificadas y actualiza docs/auditoria/comentarios.md.
```

**Qué salió:** tabla con los 4 hallazgos en el formato del ejercicio, identificadores `C-xx` unificados y tabla resumen. La Parte B sigue por rellenar.

## Prompt 13

**Modelo:** Claude Opus 5.5
**Herramienta:** Claude Code

```
Rellena el archivo de prompts
```

**Qué salió:** este archivo.
