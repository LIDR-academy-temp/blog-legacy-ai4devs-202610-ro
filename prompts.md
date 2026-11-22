# Prompts

## Prompt 1

**Modelo:** Opus 5.5
**Herramienta:** Claude Code (app de escritorio)

```
@"/home/noureddine/Baixades/BORRAR/Modulo_6_AI_For_Legacy.pdf"
ese el el modolo 6 analesa el documento y el ejersisio 6 antes de haerlo deme lo que vas hacer
```

**Qué salió:** resumen de las cuatro lecciones y un plan de 12 pasos, marcando qué hacía el agente, qué tenía que hacer yo y dónde pedía confirmación. No tocó nada.

## Prompt 2

**Modelo:** Opus 5.5
**Herramienta:** Claude Code (app de escritorio)

```
has el ejersisio y y un testeng y pensa antes en mas de una upcion y usa la mejor la mas correcta
```

**Qué salió:** fork, clon, `make setup`/`db`/`api`/`web` y `make test` (1 test, verde). Comparó tres formas de auditar (pedir la lista entera; un subagente que encuentra y yo verifico; leer la rebanada entera citando cada línea y probar cada hallazgo contra la API) y eligió la tercera. Leyó las 5 rutas de comentarios, la migración, la semilla y los componentes, y probó 10 de los 11 hallazgos con `curl` contra la API levantada, resembrando la base después. Corrigió dos citas que apuntaban a una línea equivocada. Las tres líneas salen de mis respuestas a un formulario: comprobé 3 hallazgos yo con `sed -n` sobre las líneas citadas.

## Prompt 3

**Modelo:** Opus 5.5
**Herramienta:** Claude Code (app de escritorio)

```
analisa el block y peraso que si ha complitado el egersisio o no
```

**Qué salió:** repasó el checklist. Todo estaba hecho salvo abrir el PR, que abro yo.
