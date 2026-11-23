---
description: SDD · Nuevo requisito o cambio - primero la spec, después el código (uso - /sdd-change 003 requisito…)
argument-hint: "[NNN] [cambio]"
---

Cambio pedido para `specs/$0*/`: $ARGUMENTS

NO toques código. Usá el subagente `planner` con la skill `sdd`:
1. Actualizar `spec.md`: RF nuevo o modificado en EARS, sus casos límite y lo que queda fuera.
2. Si el cambio contradice una decisión de `MEMORY.md` o `docs/riesgos-y-preguntas.md`, señalarlo.
3. Indicar qué partes de `plan.md` y `tasks.md` habría que cambiar después.
4. Mostrarme el diff de la spec y esperar mi aprobación.
