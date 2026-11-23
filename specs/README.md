# Specs (Spec-Driven Development)

Cada spec vive en `specs/NNN-nombre/` con `spec.md` (qué y por qué), `plan.md` (cómo) y `tasks.md` (tareas). La spec manda: si algo no está acá, no se implementa.

| Spec | Tema | Estado | Plan | Tareas |
|---|---|---|---|---|
| 001-base-y-acceso | PWA, deploy, ingreso con email o PIN | borrador | — | — |
| 002-motor-de-precios | Reglas de costo y precio, esquema y RLS | borrador | — | — |
| 003-revision-y-carga-manual | Revisión, alias, manual, repetir compra | borrador | — | — |
| 004-carga-con-gemini | Foto, PDF, Gemini, cola y reintentos | borrador | — | — |
| 005-rubros-y-margenes | Rubros, márgenes y recálculo | borrador | — | — |
| 006-consulta-de-precios | Buscador offline y ficha de producto | borrador | — | — |
| 007-fraccionados | Granel por fracción o por peso | borrador | — | — |
| 008-proveedores | Proveedores y sus días de pedido y entrega | borrador | — | — |

Orden de trabajo y backlog: `docs/fase-1/backlog.md`.

Flujo: `/sdd-clarify NNN` → aprobar la spec → `/sdd-plan NNN` → `/sdd-tasks NNN` → `/sdd-implement NNN Tn` (una por vez) → `/sdd-validate NNN`. O todo junto con `/sdd-coordinar NNN`.
