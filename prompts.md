# Prompts

## Prompt 1

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
resume los últimos commits, y que busca en qué archivos aparece una palabra user del dominio
```

**Qué salió:** Indica que user solo aparece en la parte de autenticación. y que el código del blog nunca la usa.

## Prompt 2

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
busca en qué archivos aparece una palabra autor del dominio.
```

**Qué salió:** Indica que el autor aparece en 16 archivos, los lista e indica que el que el autor todavía no tiene su propio modelo, servicio ni transformer.

## Prompt 3

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
Con un agente, produce la auditoría priorizada de la feature de comentarios y déjala escrita en docs/auditoria/comentarios.md.

Una feature es una funcionalidad del blog de punta a punta: sus rutas en la API, su tabla en la base de datos y lo que la web enseña de ella. Este blog tiene cinco (posts, comentarios, autores, categorías y etiquetas), y la que se audita es comentarios: crear, editar, borrar y listar comentarios, en esas tres capas. Solo esa feature. No el sistema entero.

Un hallazgo sin la línea que lo demuestra no es un hallazgo.

Cada afirmación debe venir con su sitio: archivo y línea, o el fragmento citado tal cual está escrito. Sin sitio, el hallazgo no se escribe.

Debe existir una fila por hallazgo, con estas seis casillas.

Qué es. Una frase. Si necesitas tres, es que hay dos hallazgos.

Dónde está, con la línea que lo demuestra. Sin esta casilla, la fila no se escribe.

Qué se rompe, y quién lo nota. El efecto concreto y a quién le llega. "Nadie todavía" es una respuesta válida y buena.

Cuántos sitios hay que tocar. Contados, no estimados.

¿Se deshace con un revert? Sí o no, y si es que no, por qué no.

¿Bloquea o desbloquea a otro hallazgo? Nombrando al otro. "A ninguno" es la respuesta más frecuente.
```

**Qué salió:** creó el archivo comentarios.md y escribio los 24 hallazgos que encontro el agente.

## Prompt 4

**Modelo:** Opus 5.5
**Herramienta:** Claude Code

```
Cuando se pregunta si se deshace con un revert, se hacia referencia a que si al ajustar el hallazgo se puede hacer el revert y si no explicar porque
```

**Qué salió:** Ajusta la informacion de la tabla con la respuesta a 'se deshace con un revert'.
