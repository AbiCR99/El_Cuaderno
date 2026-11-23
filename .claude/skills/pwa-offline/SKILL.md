---
name: pwa-offline
description: Úsala siempre que toques el service worker, vite-plugin-pwa, el manifest, IndexedDB (Dexie), la sincronización de precios, la cola local de facturas o confirmaciones, o cualquier comportamiento sin conexión de El Cuaderno.
---

# PWA y funcionamiento sin conexión

## Qué tiene que funcionar offline
- **Consulta de precios** completa (admin y familia).
- **Sacar fotos y subir PDF**: quedan en cola.
- **Revisión**: se puede revisar una factura ya descargada; "Confirmar todo" queda en cola.
- No offline: ingreso por primera vez, Porcentajes (cambiar márgenes recalcula en la base).

## Reglas
- vite-plugin-pwa con `registerType: 'autoUpdate'`: la versión nueva se aplica sola al reabrir. Precache del shell de la app; las llamadas a Supabase no se cachean con Workbox (los datos viven en Dexie).
- Dexie: una sola base `el-cuaderno` con versiones numeradas. Cambiar el esquema local = nueva `version()` con migración; nunca borrar la base del usuario.
- Tablas locales previstas: `productos`, `rubros`, `meta` (última sincronización), `cola` (facturas y confirmaciones pendientes), `borradores` (revisiones en curso).
- Sincronización incremental con `lista_precios(desde)` usando el `actualizado_en` máximo recibido (no la hora del celular). Disparadores: abrir la app, volver a ella (`visibilitychange`), evento `online` y cada 5 min con la app abierta.
- Cola: cada ítem con `id` (uuid), `tipo`, `intentos`, `creadoEn`. Se procesa en orden; un ítem que falla no bloquea a los siguientes. Background Sync cuando el navegador lo soporta; si no, al abrir la app.
- Nunca mostrar un error por estar offline: mostrar el estado ("Sin conexión · precios del …", "1 factura esperando conexión").
- La familia y la admin pueden compartir dispositivo: al salir, borrar los datos locales de la sesión anterior (precios incluidos, porque la admin también guarda costos de su ficha).

## Cómo verificar
Chrome DevTools MCP: Application → Service Workers (activo), Network → Offline, recargar y buscar un producto; tomar una "foto" de prueba offline y volver online; revisar IndexedDB.

## Checklist
- [ ] ¿Funciona la búsqueda en modo avión después de un primer ingreso?
- [ ] ¿Una versión nueva se aplica sin que la usuaria haga nada?
- [ ] ¿Un cambio de esquema local migra en lugar de borrar?
- [ ] ¿Se guarda algún costo en el dispositivo de la familia?
