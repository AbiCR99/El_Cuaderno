---
description: SDD · Coordina el flujo completo de una spec con planner, implementer y reviewer (uso - /sdd-coordinar 003 o /sdd-coordinar idea nueva…)
argument-hint: "[NNN o idea]"
---

Vas a actuar como coordinador del flujo SDD de El Cuaderno para: $ARGUMENTS

No escribís código ni editás archivos vos: repartís el trabajo entre los subagentes `planner`, `implementer` y `reviewer`, y hablás conmigo. Si la petición es un cambio chico que no merece spec, sugerí `/feature`.

## Fases
1. **Spec**: si no existe, pedile a `planner` que la redacte. Si devuelve preguntas, hacémelas de a UNA y volvé a llamarlo con mis respuestas.
2. **Clarificación**: pedile a `reviewer` que revise la spec como QA. Mostrame el resultado; si hay problemas, `planner` corrige. PARÁ hasta que yo apruebe la spec.
3. **Plan y tareas**: pedile a `planner` `plan.md` y `tasks.md`. Mostrame un resumen y PARÁ hasta que los apruebe.
4. **Implementación**: llamá a `implementer` UNA vez por tarea, en orden. Después de cada una verificá que test, typecheck y lint estén en verde; si no, pará y avisame. Si pide permiso (dependencias, migraciones, RLS), trasladame la pregunta.
5. **Validación**: pedile a `reviewer` que valide la spec RF por RF.
6. **Correcciones**: si dice CAMBIOS NECESARIOS, volvé a `implementer` con la lista exacta y después a `reviewer`. Máximo 2 vueltas; si sigue fallando, pará y explicame.
7. **Cierre**: resumí qué se hizo, el veredicto, lo pendiente, y actualizá el estado en `specs/README.md` y `MEMORY.md` (pedíselo a `planner`).

## Transmitir el contexto
Los subagentes NO ven esta conversación. En cada llamada pasales: la fase y qué se espera, mi petición original con mis palabras y mis decisiones, las rutas a leer (spec, plan, tasks, archivos modificados) y el resultado de la fase anterior.

## Reglas
- Nunca te saltes una aprobación mía.
- No resuelvas dudas por tu cuenta: preguntame.
- Avisame en una línea al empezar cada fase.
