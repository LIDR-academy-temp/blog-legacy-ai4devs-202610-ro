# Prompts

## Prompt 1

**Modelo:** Sonnet 5.5
**Herramienta:** Claude Code

```
Necesito realizar una auditoría técnica del código heredado de este proyecto.

Analiza exclusivamente la funcionalidad de comentarios, incluyendo frontend, backend y base de datos.

Investiga cómo se crean, consultan, editan y eliminan los comentarios.

Identifica posibles bugs, riesgos y problemas de mantenimiento.

Para cada hallazgo, indícame:
- Qué problema has encontrado.
- Archivo y línea exacta donde se demuestra.
- Qué podría romperse y a qué usuarios afectaría.
- Cuántos lugares del código habría que modificar.
- Si se podría revertir el cambio con Git.
- Si bloquea o desbloquea la solución de otro problema.

No modifiques el código ni arregles los problemas.

Presenta primero los hallazgos para que pueda comprobarlos personalmente.

No inventes información. Si falta contexto, deja el criterio sin resolver e indica qué habría que comprobar.
```

## Prompt 2

**Modelo:** Sonnet 5.5
**Herramienta:** Claude Code

```
Necesito realizar una auditoría técnica del código heredado de este proyecto.

Analiza exclusivamente la funcionalidad de comentarios, incluyendo frontend, backend y base de datos.

Investiga cómo se crean, consultan, editan y eliminan los comentarios.

Identifica posibles bugs, riesgos y problemas de mantenimiento.

Para cada hallazgo, indícame:
1. Clasifica cada hallazgo como hecho, riesgo u opinión.
2. Identifica el daño real y quién resulta afectado.
3. Enumera los componentes que habría que modificar.
4. Indica si cada cambio es reversible.
5. Identifica las dependencias entre los cambios.
6. Propón un orden que respete las dependencias y considere el daño frente al radio.
7. Cuando un problema grave dependa de otros cambios, propón una contención inmediata y una solución definitiva.

No modifiques el código ni arregles los problemas.

Presenta primero los hallazgos para que pueda comprobarlos personalmente.

No inventes información. Si falta contexto, deja el criterio sin resolver e indica qué habría que comprobar.
```

## Prompt 3

**Modelo:** Sonnet 5.5
**Herramienta:** Claude Code

```
Quiero que guardes la auditoría de la funcionalidad de comentarios que hemos realizado en el archivo:

docs/auditoria/comentarios.md

Utiliza los 12 hallazgos (H1 a H12) que hemos identificado anteriormente, respetando las instrucciones siguientes.

1. Alcance

Audita exclusivamente la funcionalidad de comentarios.

Incluye frontend, backend y base de datos.

No modifiques el código de la aplicación.

No soluciones los problemas encontrados.

No inventes información ni des por verificado algo que únicamente has deducido.

2. Formato de la auditoría

Crea una tabla Markdown con una fila por hallazgo y exactamente estas seis columnas:

Qué es: descripción del problema en una frase.

Dónde está: archivo y número de línea exacto que demuestra el hallazgo.

Qué se rompe y quién lo nota: consecuencia concreta y usuarios afectados.

Cuántos sitios hay que tocar: número de ubicaciones que habría que modificar, contadas y no estimadas.

¿Se deshace con un revert?: sí o no, explicando por qué cuando no sea reversible.

¿Bloquea o desbloquea a otro hallazgo?: identifica los hallazgos relacionados o indica "A ninguno".

Si no puedes demostrar un hallazgo con una ubicación y línea concreta, no lo incluyas como hallazgo confirmado.

Si falta información para completar alguna columna, deja la celda vacía. No inventes respuestas.

3. Priorización

Ordena los hallazgos utilizando exactamente estos criterios:

Primero, la precedencia: los hallazgos que desbloquean o abaratan la solución de otros deben ir antes.

Dentro del mismo nivel de precedencia, prioriza considerando el daño dividido por el radio de cambio.

Si un problema tiene un daño elevado pero su solución definitiva depende de otros cambios, sepáralo en dos filas: contención inmediata y solución definitiva.

Las propuestas de mejora que no tengan evidencia suficiente deben quedar fuera de la tabla, en una sección de pendientes de medir o verificar, indicando qué habría que comprobar.

Mantén la tabla breve y centrada en los hallazgos más relevantes. No fuerces la inclusión de los 12 si no cumplen los requisitos.

4. Parte B: las tres líneas obligatorias

Debajo de la tabla incluye exactamente estos tres apartados:

Hallazgos: el agente identificó 12 hallazgos. Deja pendiente el número de hallazgos que he verificado personalmente, porque ese dato debo proporcionarlo yo.

Duda de priorización: deja un espacio para que escriba qué hallazgo no supe priorizar y qué criterios entraban en conflicto.

Afirmación sin verificar: deja un espacio para que escriba algo que afirmaste y que yo no pude comprobar ni confirmar ni descartar.

No respondas estas preguntas en mi nombre.
```

## Prompt 4

**Modelo:** Sonnet 5.5
**Herramienta:** Claude Code

```
Veo que hay cosas pendientes de verificar, actualmente la aplicación está levantada puedes realizar peticiones y pruebas sobre ella
```

## Prompt 5

**Modelo:** Sonnet 5.5
**Herramienta:** Claude Code

```
Sí, actualiza la auditoría con estos resultados
```
