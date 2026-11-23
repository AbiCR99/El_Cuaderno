# MEMORY.md — El Cuaderno

Memoria del proyecto entre sesiones. Máximo ~50 líneas: resumí o eliminá lo que ya no aporte. Nunca guardes claves, tokens, PIN ni datos personales.

## Estado actual
- Fase 1 (costos y precios): entorno preparado, sin código todavía. Próximo paso: spec 001 (`/sdd-plan 001-base-y-acceso`).
- Specs en borrador: 001 a 008. Backlog en `docs/fase-1/backlog.md`. Sin preguntas abiertas.

## Decisiones (y por qué) — misma numeración y detalle en `docs/riesgos-y-preguntas.md`
- D1 · Precios editados a mano: **se pisan** al recalcular un rubro o al confirmar una factura. Solo "precio fijo" se respeta.
- D2 · Redondeo a la decena, **el 5 hacia arriba** (115 → 120), con enteros.
- D3 · Familia: **un PIN compartido** (una cuenta `familia`).
- D4 · React + Vite + **TypeScript**.
- D5 · La familia tiene **exactamente los mismos permisos** que la administradora. Reemplaza lo que decía el brief.
- D6 · Supabase gratis, cuenta nueva solo para el proyecto: proyectos `dev` y `prod`.
- D7 · "Con IVA" = los precios ya lo traen. "Sin IVA" = se suma la alícuota (21 % por defecto).
- D8 · Botón **Proveedores** (nombre, días de pedido y entrega). La revisión no pide días.
- D9 · "Repetir última compra" en Manual, prioridad baja.
- D10 · Avisos en Inicio solo si hay algo, encima de los dos botones.
- D11 · Facturas de una sola hoja.
- D12 · Factura repetida (mismo proveedor y número) o vieja (producto con compra de fecha posterior): aviso amarillo, no bloquea. Una vieja guarda la compra sin cambiar costo ni precio.
- D13 · Descuentos generales al pie: se ignoran.
- D14 · Fraccionados con botón con/sin IVA; solo carga manual en fase 1.
- D15 · Precio de 100 g calculado y redondeado aparte; editable.
- D16 · Node 22 LTS y `npx supabase`, sin Docker (Mac con Monterey).
- D17 · Unidades de regalo se suman al mismo producto y bajan su costo.
- D18 · Si baja el costo, se sugiere el precio más bajo ("antes $X") y se aplica al confirmar.
- D19 · Muestras gratis (regalo sin el producto pago en la factura): no se toman en cuenta; renglón gris.
- D20 · El botón "Porcentajes" se llama **Rubros**.
- D21 · Subir productos: Cámara, PDF y Manual grandes; abajo Rubros y Proveedores chicos.
- Cálculo de precios solo en `src/dominio/precios.ts`; la base recibe precios ya calculados por RPC atómicas.
- Sin `margen_override` por producto (el escape es "precio fijo"). Código de barras: tabla propia en fase 2.

## Aprendizajes y errores a evitar
- `100 * 1.15` da `114.99999999999999` en JS: por eso el redondeo se hace con enteros.

## Próximos pasos
- Juntar 15–20 facturas reales (fotos y PDF) para el set de prueba de Gemini.
