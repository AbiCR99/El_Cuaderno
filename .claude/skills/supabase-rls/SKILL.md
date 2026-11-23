---
name: supabase-rls
description: Úsala siempre que escribas o revises migraciones, tablas, políticas RLS, funciones RPC, Storage, Edge Functions o consultas de Supabase en El Cuaderno, o cuando algo dependa del rol (admin, dev, familia).
---

# Supabase y seguridad por rol

Referencia: `docs/modelo-datos.md`. Si una migración se aparta del documento, primero se actualiza el documento.

## Reglas
- Todo cambio de esquema es un archivo nuevo en `supabase/migrations/` (`supabase migration new <nombre>`). Nunca editar una migración ya aplicada.
- Migraciones sin pérdida de datos: nada de `drop table` ni `drop column` sobre datos reales sin pregunta explícita.
- `enable row level security` en **toda** tabla nueva, en la misma migración que la crea.
- Políticas con `(select public.es_admin())` (entre paréntesis con `select`, para que Postgres la evalúe una vez).
- `anon` no tiene políticas. La familia no tiene políticas sobre tablas de negocio: solo lee con `lista_precios` y `lista_rubros`.
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
set local request.jwt.claims = '{"sub":"<uuid-familia>","role":"authenticated"}';
select count(*) from public.productos;          -- esperado: 0
select * from public.lista_precios() limit 1;   -- esperado: filas, sin columnas de costo
insert into public.categorias (nombre, margen_pct) values ('x', 1);  -- esperado: error
rollback;
```
Repetir con el uuid de admin (todo permitido) y sin `sub` (todo falla). Guardar el script en `supabase/tests/`.

## Checklist
- [ ] ¿RLS activa en todas las tablas nuevas?
- [ ] ¿Alguna política `to anon` o `using (true)`?
- [ ] ¿`lista_precios` sigue sin costo, margen, proveedor ni alias de otro rol?
- [ ] ¿Funciones definer con `search_path = ''` y `revoke` a `anon`?
- [ ] ¿Tipos regenerados y `get_advisors` sin alertas?
