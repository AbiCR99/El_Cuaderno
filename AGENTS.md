# AGENTS.md — El Cuaderno

PWA para un almacén de barrio en Mar del Plata: carga de facturas de proveedores (foto, PDF o manual), cálculo de costos y precios de venta, y consulta de precios offline para la familia. Lo usa una sola persona que atiende sola: **todo tiene que ser más rápido que hoy; ante la duda, menos pasos y botones grandes.**

## Stack y estructura
- React + Vite + TypeScript (PWA con vite-plugin-pwa), Dexie (IndexedDB), MiniSearch. Deploy en Vercel.
- Supabase: Postgres + RLS, Auth, Storage, Edge Functions (Deno) que llaman a Gemini. pg_cron para reintentos.
- `src/dominio/` lógica pura (precios, IVA, bultos, redondeo, búsqueda): sin React, sin Supabase. `supabase/migrations/` esquema y RLS. `supabase/functions/` Edge Functions. `docs/` decisiones y modelo. `specs/NNN-*/` specs SDD.

## Comandos
- `npm run dev` · `npm run test` (Vitest) · `npm run lint` · `npm run typecheck` · `npm run build`

## Convenciones
- Interfaz en español rioplatense (vos), textos cortos. Código: identificadores de dominio en español sin tildes ni ñ (`precioVenta`, `anio`); comentarios en español.
- Plata: precios de venta en pesos enteros; costos con 2 decimales; márgenes en % con hasta 2 decimales. Formato `es-AR`. Zona horaria `America/Argentina/Buenos_Aires`.

## Reglas de dominio (fácil equivocarse)
- Una sola lógica de precios: `src/dominio/precios.ts`. La base no calcula precios; la UI tampoco. Ver skill `reglas-precios`.
- Redondeo a la decena, el 5 va hacia arriba (115 → 120). Calcular con enteros, nunca `Math.round(costo * 1.15)`.
- Recalcular un rubro o cargar una factura **pisa** los precios editados a mano. Solo "precio fijo" (rubro o producto) nunca se recalcula.
- La familia nunca ve costos, márgenes ni proveedores: solo accede por las RPC `lista_precios` y `lista_rubros`. Ver skill `supabase-rls`.
- La API key de Gemini vive solo en secretos de Supabase. Jamás en el cliente ni en el repo.
- No mostrar "stock" en la interfaz de la fase 1.

## Forma de trabajar
- Lee `docs/constitution.md` y la spec activa antes de tocar código. Flujo SDD con `/sdd-*`; cambios chicos con `/feature`.
- Diagnóstico y plan antes de implementar. Una tarea por vez, cambios chicos, rama desde `main` + PR.
- Al terminar: qué cambió, cómo se verificó y qué decisiones tomaste vos.

## Límites
- ✅ Siempre: tests de `src/dominio` en verde, textos en español, actualizar `MEMORY.md` al cerrar cada tarea.
- ⚠️ Preguntá antes: dependencias nuevas, migraciones o cambios en Supabase remoto, cambios de RLS, cambios al esquema JSON de Gemini, cualquier paso extra para la usuaria.
- 🚫 Nunca: secretos en el código, `service_role` en el cliente, desactivar RLS, inventar requisitos, mostrar stock en fase 1.

## Verificación
- `npm run test && npm run typecheck && npm run lint`. UI: Chrome DevTools MCP en vista móvil 360 px, modo offline incluido. RLS: probar como `familia` y como `admin`.
