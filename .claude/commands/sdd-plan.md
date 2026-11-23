---
description: SDD · Genera el plan técnico de una spec aprobada
argument-hint: "[NNN]"
---

Usá el subagente `planner`. NO escribas código.

Que lea `docs/constitution.md`, `AGENTS.md`, `docs/arquitectura.md`, `docs/modelo-datos.md`, `specs/$0*/spec.md` y el bloque de esa spec en `docs/fase-1/backlog.md`, y genere `specs/$0*/plan.md` con:
- Archivos que se crean o modifican y la responsabilidad de cada uno.
- Funciones puras de `src/dominio/` y sus firmas.
- Migraciones, políticas RLS y RPC, si corresponde.
- Cómo se ve en la interfaz, con sus estados de carga, error y vacío.
- Decisiones técnicas justificadas, con la alternativa descartada.
- Estrategia de tests con Vitest y verificación con Chrome DevTools.
- Qué RF cubre cada parte.

Si la spec no está aprobada o tiene [NECESITA ACLARACIÓN] sin resolver, pará y avisame.
