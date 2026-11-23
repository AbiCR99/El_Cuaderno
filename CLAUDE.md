@AGENTS.md
@MEMORY.md

## Específico de Claude Code
- Subagentes en `.claude/agents/`: `planner` (specs, planes y tareas; solo escribe en `specs/` y `docs/`), `implementer` (una tarea por vez, tests primero) y `reviewer` (solo lectura, da veredicto).
- Comandos: `/sdd-coordinar` (flujo completo con los tres subagentes), `/sdd-spec`, `/sdd-clarify`, `/sdd-plan`, `/sdd-tasks`, `/sdd-implement`, `/sdd-validate`, `/sdd-change`, `/sdd-status`, `/feature`.
- Skills del proyecto en `.claude/skills/`: `sdd`, `reglas-precios`, `supabase-rls`, `gemini-facturas`, `pwa-offline`, `ux-almacen`.
- MCP en `.mcp.json`: Supabase (apuntando al proyecto de desarrollo), Context7 y Chrome DevTools. Usá Context7 antes de escribir código contra Supabase, Vite PWA, Dexie, pdf.js o la API de Gemini.
- Para cambios grandes, entrá en modo plan (Shift+Tab) antes de editar.
