---
name: supabase-rls
description: Úsala siempre que escribas o revises migraciones, tablas, políticas RLS, funciones RPC, Storage, Edge Functions o consultas de Supabase en El Cuaderno, o cuando algo dependa de quién está logueado.
---

# Supabase y seguridad por rol

Referencia: `docs/modelo-datos.md`. Si una migración se aparta del documento, primero se actualiza el documento.

## Reglas
- Todo cambio de esquema es un archivo nuevo en `supabase/migrations/` (`supabase migration new <nombre>`). Nunca editar una migración ya aplicada.
- Migraciones sin pérdida de datos: nada de `drop table` ni `drop column` sobre datos reales sin pregunta explícita.
- `enable row level security` en **toda** tabla nueva, en la misma migración que la crea.
- Políticas con `(select public.tiene_acceso())` (entre paréntesis con `select`, para que Postgres la evalúe una vez).
- Los tres roles (`admin`, `dev`, `familia`) tienen los mismos permisos (D5). `anon` y los usuarios sin perfil no tienen acceso a nada. El rol se guarda para saber quién hizo cada cambio, no para restringir.
- `historial_precios` y los `factura_items` confirmados no se editan ni se borran (son la base de la fase 3).
- Funciones `security definer` siempre con `set search_path = ''`, nombres calificados (`public.tabla`), `revoke execute ... from anon, public` y `grant ... to authenticated`.
- RPC de escritura `security invoker`, atómicas e idempotentes (repetir la llamada no duplica).
- El cliente usa la clave publicable. `service_role` solo dentro de Edge Functions.
- Secretos (`GEMINI_API_KEY`, `GEMINI_MODEL`, `CRON_SECRET`) con `supabase secrets set`. Nunca en el repo ni en `VITE_*`.
- Después de cada migración: regenerar tipos (`supabase gen types typescript --project-id <dev> > src/datos/tipos.ts`) y correr `get_advisors` (seguridad y rendimiento).
- Aplicar migraciones al proyecto remoto, desplegar funciones y cambiar secretos: siempre con permiso de la desarrolladora.

## Cómo probar RLS (con el MCP, dentro de una transacción)
```sql
begin;
set local role authenticated;
set local request.jwt.claims = '{"sub":"<uuid-sin-perfil>","role":"authenticated"}';
select count(*) from public.productos;          -- esperado: 0
insert into public.categorias (nombre, margen_pct) values ('x', 1);  -- esperado: error
rollback;
```
Repetir con `set local role anon` (todo falla) y con el uuid de cada rol con perfil (todo permitido). Guardar el script en `supabase/tests/`.

## Checklist
- [ ] ¿RLS activa en todas las tablas nuevas?
- [ ] ¿Alguna política `to anon` o `using (true)`?
- [ ] ¿Un usuario sin perfil o `anon` puede leer o escribir algo?
- [ ] ¿Funciones definer con `search_path = ''` y `revoke` a `anon`?
- [ ] ¿Tipos regenerados y `get_advisors` sin alertas?
