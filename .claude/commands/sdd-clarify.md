---
description: SDD · Revisa una spec como QA (solo detecta, no resuelve)
argument-hint: "[NNN]"
---

Usá el subagente `reviewer` para revisar `specs/$0*/spec.md` como un QA muy profesional, con la skill `sdd`.

Que liste:
1. Ambigüedades (requisitos que no se pueden verificar).
2. Contradicciones entre requisitos.
3. Casos límite no cubiertos.
4. Conflictos con `docs/constitution.md` o `docs/brief-original.md`.
5. Requisitos que no salen del brief ni de una decisión registrada.
6. Preguntas de `docs/riesgos-y-preguntas.md` que bloquean esta spec.

No propongas soluciones: solo detectá. Formato: lista numerada.
