---
description: SDD · Implementa UNA tarea, tests primero (uso - /sdd-implement 002 T3)
argument-hint: "[NNN] [Tn]"
---

Usá el subagente `implementer` para implementar SOLO la tarea $1 de `specs/$0*/tasks.md`, siguiendo `plan.md`, `docs/constitution.md` y la skill `sdd`.

1. Primero los tests, y comprobar que fallan.
2. Después el código hasta que pasen.
3. `npm run test && npm run typecheck && npm run lint`, y mostrame el resultado.
4. Marcar $1 como hecha en `tasks.md` e indicar qué RF cubre.

Después PARÁ. No empieces la siguiente tarea. Si necesita una dependencia, una migración remota o un cambio de RLS, que me pregunte antes.
