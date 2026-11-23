---
description: SDD · Entrevista y genera o completa una spec (uso - /sdd-spec 008-nombre idea inicial)
argument-hint: "[NNN-nombre] [idea]"
---

NO escribas código en ningún momento. Leé `docs/constitution.md`, `docs/brief-original.md`, `MEMORY.md` y `docs/riesgos-y-preguntas.md`, y usá la skill `sdd`.

Carpeta de la spec: `specs/$0/` (si existe, la completás; si no, la creás).
Petición completa: $ARGUMENTS

1. Hacéme preguntas de a UNA para eliminar ambigüedades (casos límite, errores, qué queda fuera). Máximo 5.
2. Con mis respuestas, escribí `specs/$0/spec.md` con la plantilla de la skill `sdd`, requisitos en EARS y "Estado: borrador".
3. Solo el QUÉ y el POR QUÉ: nada de stack, arquitectura ni archivos.
4. Agregá la spec a la tabla de `specs/README.md`.
