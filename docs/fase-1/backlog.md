# Backlog de la fase 1

Tareas chicas (20–40 min con el agente), en orden de dependencia, cada una verificable por separado. Cuando arranca una spec, `/sdd-plan` y `/sdd-tasks` convierten su bloque en `specs/NNN-*/plan.md` y `tasks.md` (pueden afinar o dividir tareas, no agregar alcance).

**Orden recomendado:** 001 → 002 → 006 → 003 → 005 → 008 → 004 → 007. La consulta de precios (006) va antes que la revisión porque es lo que más rápido le sirve a la familia y prueba el modo offline temprano; Gemini (004) va después de la revisión porque necesita una pantalla donde caer.

**Meta para la visita (~1 mes):** 001, 002, 006, 003, 005 y 008 en producción; 004 funcionando con el set de prueba.

## 001 · Base y acceso
- [ ] **T001-1. Proyecto Vite + React + TS** con ESLint, `npm run dev/test/lint/typecheck/build`. — Hecho cuando: los 5 comandos corren en verde en un proyecto vacío.
- [ ] **T001-2. PWA instalable** (vite-plugin-pwa, manifest, íconos, `registerType: autoUpdate`). — Hecho cuando: Lighthouse marca "instalable" y la app se instala en un Android real.
- [ ] **T001-3. Deploy en Vercel** conectado al repo, con `VITE_SUPABASE_URL` y `VITE_SUPABASE_PUBLISHABLE_KEY`. — Hecho cuando: cada push a `main` publica y cada PR tiene vista previa.
- [ ] **T001-4. Cuenta nueva de Supabase con proyectos `dev` y `prod`** + CLI por `npx supabase` (sin Docker, Node 22). — Hecho cuando: `npx supabase migration list` responde en ambos y el MCP apunta a `dev`.
- [ ] **T001-5. Migración: `perfiles`, `rol_actual()`, `tiene_acceso()`** + alta de las 3 cuentas desde el dashboard. — Hecho cuando: un usuario con perfil lee `perfiles` y uno sin perfil no.
- [ ] **T001-6. Pantalla de ingreso** (dos opciones, email/contraseña, teclado PIN, errores, sin conexión). RF-2 a RF-6. — Hecho cuando: los 3 roles entran y la sesión persiste al reabrir.
- [ ] **T001-7. Ruteo con sesión y registro de autor**. RF-7, RF-10. — Hecho cuando: sin sesión toda ruta lleva al ingreso, y los cambios guardan `usuario_id`.
- [ ] **T001-8. Script de verificación RLS** (`supabase/tests/rls_acceso.sql`). RF-8. — Hecho cuando: corre con el MCP; sin sesión todo falla, con cada rol todo funciona.

## 002 · Motor de precios y esquema
- [ ] **T002-1. `normalizar.ts`**: texto de alias y números en formato argentino. RF-10, RF-11. — Hecho cuando: tests en verde.
- [ ] **T002-2. `redondeo.ts`**: decena con 5 hacia arriba, aritmética entera. RF-5. — Hecho cuando: tests 113→110, 115→120, 117→120, 100×15 %→120, 1235→1240.
- [ ] **T002-3. `costos.ts`**: neto, IVA, bonificaciones, costo unitario, validaciones. RF-1 a RF-4, RF-8, RF-9. — Hecho cuando: tests de cada RF y caso límite.
- [ ] **T002-3b. Unidades de regalo**: unir renglones de regalo con el renglón pago del mismo producto. RF-12, RF-13. — Hecho cuando: test "24 pagas + 2 de regalo" da costo ÷ 26, y una muestra sin producto pago se descarta.
- [ ] **T002-4. `precios.ts`**: precio sugerido, precio fijo efectivo, variación %. RF-5 a RF-7. — Hecho cuando: tests en verde.
- [ ] **T002-5. Migración del esquema** (tablas, índices, triggers) según `docs/modelo-datos.md`. RF-12. — Hecho cuando: aplicada en dev y tipos generados en `src/datos/tipos.ts`.
- [ ] **T002-6. Migración de RLS, Storage y RPC de sincronización** (`lista_precios`). — Hecho cuando: script RLS de T001-8 ampliado en verde y `get_advisors` sin alertas.
- [ ] **T002-7. Migración de RPC de escritura** (`confirmar_factura` con regla de factura vieja, `aplicar_precios`, `eliminar_rubro`) + semilla de rubros. — Hecho cuando: prueba SQL de cada RPC con sesión (OK), sin sesión (falla) y con una factura vieja (no cambia precio).

## 006 · Consulta de precios offline
- [ ] **T006-1. Base local Dexie** (productos, rubros, metadatos de sincronización). — Hecho cuando: test de lectura/escritura en el navegador.
- [ ] **T006-2. Sincronización incremental** con `lista_precios(desde)`. RF-8, RF-9. — Hecho cuando: un cambio en la base aparece en el celular sin recargar todo.
- [ ] **T006-3. `busqueda.ts`** con MiniSearch: tildes, prefijo, tipeo, alias, filtro por rubro, orden. RF-1 a RF-4. — Hecho cuando: tests en verde.
- [ ] **T006-4. Pantalla Precios**: buscador, desplegable, grilla de rubros, tarjetas con "hace X". RF-5 a RF-7, RF-14 a RF-16. — Hecho cuando: verificada en 360 px con Chrome DevTools.
- [ ] **T006-5. Modo offline y aviso de fecha**. RF-10. — Hecho cuando: demo con modo avión.
- [ ] **T006-6. Ficha de producto**: costo, margen, historial, editar precio, cambiar rubro, precio fijo, modo lectura offline. RF-11 a RF-13, RF-17. — Hecho cuando: editar un precio deja historial `manual` con el usuario que lo hizo.

## 003 · Revisión, alias y carga manual
- [ ] **T003-1. `emparejar.ts`**: alias exacto por proveedor y candidatos por similitud. RF-2, RF-5. — Hecho cuando: tests en verde.
- [ ] **T003-2. `clasificar.ts`**: verde/amarillo con motivo. RF-4, RF-6. — Hecho cuando: un test por motivo.
- [ ] **T003-3. Componente Revisión (solo lectura)** con datos de ejemplo. RF-1, RF-3. — Hecho cuando: se ve bien en 360 px con 3 renglones verdes y 2 amarillos.
- [ ] **T003-4. Edición de renglón amarillo**: emparejar, producto nuevo, rubro, números. RF-5. — Hecho cuando: todos los motivos amarillos se pueden resolver.
- [ ] **T003-5. Cabecera**: proveedor (nuevo solo con nombre), fecha obligatoria, con/sin IVA con recálculo. RF-9, RF-13, RF-20. — Hecho cuando: cambiar IVA recalcula todo al instante.
- [ ] **T003-5b. Avisos de factura repetida y vieja** (`facturaVieja.ts` + consulta de repetidas). RF-18, RF-19. — Hecho cuando: tests de dominio (misma fecha no es vieja) y demo cargando una factura dos veces y una vieja.
- [ ] **T003-6. "Confirmar todo"** con `confirmar_factura`, idempotente. RF-7, RF-8. — Hecho cuando: doble toque no duplica y el historial queda bien.
- [ ] **T003-7. Borrador local y confirmación offline**. RF-14, RF-15. — Hecho cuando: cerrar la app no pierde ediciones; confirmar en modo avión se aplica al volver.
- [ ] **T003-8. Manual: elegir proveedor + cargar renglones**. RF-12. — Hecho cuando: una factura cargada a mano llega a la revisión.
- [ ] **T003-9. Inicio con avisos y descartar factura**. RF-16, RF-17. — Hecho cuando: los avisos aparecen solo cuando hay algo.
- [ ] **T003-10. Repetir última compra** (prioridad baja, D9). RF-10, RF-11. — Hecho cuando: test de dominio + demo con un proveedor.

## 005 · Rubros y márgenes
- [ ] **T005-1. `recalculo.ts`**: productos afectados (N) y lote de cambios. RF-2 a RF-4. — Hecho cuando: tests (precio fijo excluido, sin costo excluido).
- [ ] **T005-2. Pantalla Rubros**: lista y edición. RF-1, RF-5, RF-6, RF-8, RF-11. — Hecho cuando: crear y renombrar funciona, "Sin categoría" protegido.
- [ ] **T005-3. Cambiar margen con confirmación** + `aplicar_precios`. RF-2, RF-3. — Hecho cuando: demo Lácteos 18 → 20 % con historial.
- [ ] **T005-4. Eliminar rubro moviendo productos**. RF-7. — Hecho cuando: los productos terminan en el destino elegido.
- [ ] **T005-5. Cambio de rubro de un producto y precio fijo del rubro**. RF-9, RF-10. — Hecho cuando: las preguntas aparecen y respetan la respuesta.

## 008 · Proveedores
- [ ] **T008-1. Pantalla Proveedores**: lista alfabética con días o "sin días", vacío y offline. RF-1, RF-5 a RF-7. — Hecho cuando: verificada en 360 px.
- [ ] **T008-2. Crear y editar proveedor** con días L–D y control de nombre repetido. RF-2 a RF-4. — Hecho cuando: demo de HU-1.
- [ ] **T008-3. Botón Proveedores en Subir productos** (disposición D21). — Hecho cuando: Subir productos muestra los 5 botones.

## 004 · Carga con Gemini
- [ ] **T004-1. Set de prueba**: 15–20 facturas reales con su resultado esperado en `supabase/functions/extraer-factura/fixtures/` (sin datos personales). — Hecho cuando: está en el repo y revisado por vos.
- [ ] **T004-2. `comprimirImagen.ts`** con canvas. RF-2. — Hecho cuando: una foto de 4 MB queda en 200–300 KB y legible.
- [ ] **T004-3. Cola local** (Dexie + Background Sync): guardar, subir a Storage, crear fila, reintentar. RF-3, RF-9. — Hecho cuando: demo en modo avión.
- [ ] **T004-4. Edge Function `extraer-factura`**: prompt, esquema, rubros dinámicos, guardado de renglones. RF-6 a RF-8. — Hecho cuando: el set de prueba corre y se mide la precisión.
- [ ] **T004-5. Elegir modelo**: comparar 3.1 Flash-Lite vs 2.5 Flash-Lite con el set. — Hecho cuando: tabla de resultados en `docs/gemini-extraccion.md` y `GEMINI_MODEL` fijado.
- [ ] **T004-6. Reintentos en servidor** (pg_cron + pg_net, backoff, 8 intentos). RF-10, RF-11. — Hecho cuando: test con 429 simulado.
- [ ] **T004-7. Pantalla Cámara** con estados ("Leyendo…", 20 s, error, cargar a mano). RF-1, RF-12, RF-13. — Hecho cuando: demo en Android.
- [ ] **T004-8. PDF**: texto con pdf.js, detección de escaneado y rasterizado. RF-4, RF-5. — Hecho cuando: un PDF digital y uno escaneado llegan a la revisión.

## 007 · Fraccionados
- [ ] **T007-1. `fraccionados.ts`**: ambos modos, sobrante, redondeo por separado. RF-2 a RF-4, RF-8. — Hecho cuando: tests en verde.
- [ ] **T007-2. Formulario de fraccionado** que termina en la revisión. RF-1, RF-5, RF-7. — Hecho cuando: demo con maní y orégano.
- [ ] **T007-3. Confirmación con `fraccionamientos` e historial**. RF-6. — Hecho cuando: la consulta muestra "bolsita 100 g · $X".

## Cierre de fase 1
- [ ] **T-F1. Validación completa** (`/sdd-validate` de cada spec) y prueba en el celular de tu mamá.
- [ ] **T-F2. Guía de una página para la familia** (cómo instalar, entrar con PIN y cargar una factura).
