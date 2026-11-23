---
name: gemini-facturas
description: Úsala siempre que trabajes en la lectura de facturas de El Cuaderno - Edge Function extraer-factura, prompt, esquema JSON de Gemini, compresión de fotos, PDF con pdf.js, cola de facturas, reintentos, pg_cron o el set de prueba.
---

# Lectura de facturas con Gemini

Referencia completa: `docs/gemini-extraccion.md` (flujo, esquema, prompt y validaciones). Si cambiás el prompt o el esquema, actualizá ese documento en el mismo cambio y volvé a correr el set de prueba.

## Reglas
- Gemini se llama **solo** desde `supabase/functions/extraer-factura`. La clave y el modelo salen de `Deno.env` (`GEMINI_API_KEY`, `GEMINI_MODEL`).
- Structured output: `responseMimeType: "application/json"` + esquema. El `enum` de rubros se arma en cada llamada desde `categorias`. Verificá con Context7 el formato vigente de la API antes de escribir la llamada.
- `temperature` 0 y pensamiento al mínimo.
- Gemini transcribe; no calcula. Costos y precios se calculan en `src/dominio/` (skill `reglas-precios`).
- Guardar siempre `extraccion_cruda` y `modelo_ia`.
- La factura nace con `id` generado en el celular. Toda operación es idempotente: reintentar no duplica facturas ni renglones (borrar y recrear los renglones de una factura que todavía no está confirmada).
- Errores transitorios (429, 5xx, timeout 60 s) → `en_cola` con backoff 1, 2, 5, 15, 60 min. 8 intentos → `error`. Errores de contenido (no es factura, JSON inválido dos veces) → `error` directo.
- La foto o el PDF se guardan en IndexedDB antes de cualquier intento y se borran del dispositivo solo cuando la base confirma archivo y fila.
- Compresión: canvas, lado mayor 1600 px, WebP 0,7 → 0,5 hasta ≤ 300 KB; JPEG si el navegador no codifica WebP.
- PDF: menos de 40 caracteres útiles por página = escaneado → rasterizar.

## Set de prueba
`supabase/functions/extraer-factura/fixtures/`: cada factura con su `esperado.json`. Sin CUIT de personas ni datos personales de clientes. Métricas: renglones encontrados, precio unitario exacto, unidades por bulto correctas, IVA sugerido correcto, costo en tokens.

## Checklist
- [ ] ¿La clave aparece en algún archivo del cliente o en el repo?
- [ ] ¿El rubro sugerido puede salir de fuera de la lista?
- [ ] ¿Un 429 deja la factura en cola sin mostrarle un error a la usuaria?
- [ ] ¿Reintentar dos veces crea renglones duplicados?
- [ ] ¿Se actualizó `docs/gemini-extraccion.md`?
