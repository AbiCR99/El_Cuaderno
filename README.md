# El Cuaderno

PWA para un almacén de barrio: carga de facturas de proveedores (foto, PDF o manual), costos y precios de venta automáticos, y consulta de precios offline. La usan la dueña y su familia.

**Estado:** fase 1 en planificación. Todavía no hay código de la aplicación.

## Por dónde empezar
1. `docs/brief-original.md`: qué se pidió.
2. `docs/riesgos-y-preguntas.md`: decisiones tomadas y preguntas abiertas.
3. `docs/arquitectura.md` y `docs/modelo-datos.md`: cómo se construye.
4. `docs/fase-1/backlog.md` y `specs/`: qué se construye y en qué orden.

## Trabajar con Claude Code
- `CLAUDE.md` importa `AGENTS.md` (reglas) y `MEMORY.md` (estado).
- `/sdd-status` dice en qué punto está cada spec y qué comando sigue.
- `/sdd-coordinar 001` corre el flujo completo de una spec con los subagentes `planner`, `implementer` y `reviewer`.

## Requisitos
Node 22 LTS (la Mac de desarrollo tiene Monterey; Node 24 no corre ahí). Supabase CLI con `npx supabase` (sin Docker).

## Variables de entorno
Copiá `.env.example` a `.env.local` (no se sube al repo). Para los MCP, exportá en tu terminal `SUPABASE_PROJECT_REF_DEV` y, si tenés, `CONTEXT7_API_KEY`. Los secretos del servidor (`GEMINI_API_KEY`, `GEMINI_MODEL`, `CRON_SECRET`) van con `supabase secrets set`, nunca en archivos.

## Skills

**Incluidas en el repo** (`.claude/skills/`): `sdd`, `reglas-precios`, `supabase-rls`, `gemini-facturas`, `pwa-offline`, `ux-almacen` y `buena-ux`.

**Para instalar a nivel proyecto** (desde la carpeta del repo, con Node 22; revisá cada `SKILL.md` antes de usarla):

| Skill | Para qué en este proyecto | Comando |
|---|---|---|
| `supabase` | Migraciones, Auth, Storage, Edge Functions y RLS bien hechas | `npx skills add supabase/agent-skills --skill supabase` |
| `supabase-postgres-best-practices` | Índices, consultas y políticas RLS eficientes | `npx skills add supabase/agent-skills --skill supabase-postgres-best-practices` |
| `grill-me` | Te interroga antes de aprobar cada spec | `claude plugin install mattpocock-skills@claude-plugins-official` |
| `frontend-design` | Que las pantallas no se vean genéricas | `npx skills add https://github.com/anthropics/skills --skill frontend-design` |
| `web-design-guidelines` | Revisión de accesibilidad y UX de lo construido | `npx skills add vercel-labs/agent-skills --skill web-design-guidelines` |
| `systematic-debugging` | Depurar con método cuando algo falle en el celular | `npx skills add obra/superpowers --skill systematic-debugging` |
