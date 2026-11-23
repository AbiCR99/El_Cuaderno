# MEMORY.md — El Cuaderno

Memoria del proyecto entre sesiones. Máximo ~50 líneas: resumí o eliminá lo que ya no aporte. Nunca guardes claves, tokens, PIN ni datos personales.

## Estado actual
- Fase 1 (costos y precios): entorno preparado, sin código todavía. Próximo paso: spec 001 (`/sdd-plan 001-base-y-acceso`).
- Specs en borrador: 001 a 008. Backlog en `docs/fase-1/backlog.md`. Sin preguntas abiertas.

## Decisiones (y por qué) — detalle en `docs/riesgos-y-preguntas.md`
- D1 · Precios editados a mano: **se pisan** al recalcular un rubro o al confirmar una factura. Solo "precio fijo" se respeta.
- D2 · Redondeo a la decena, **el 5 hacia arriba** (115 → 120), con enteros.
- D3 · Familia: **un PIN compartido** (una cuenta `familia`).
- D4 · React + Vite + **TypeScript**.
- D5 · La familia tiene **exactamente los mismos permisos** que la administradora (ve costos, carga y confirma facturas, edita precios y rubros). Reemplaza lo que decía el brief.
- D6 · Supabase gratis, cuenta nueva solo para el proyecto: dos proyectos (`dev` y `prod`).
- D7 · "Con IVA" = los precios de la factura ya traen IVA. "Sin IVA" = se suma la alícuota (21 % por defecto).
- D8 · Proveedores (nombre, días de pedido y entrega) se editan desde un botón **Proveedores** en Subir productos, como Rubros. La revisión no pide días.
- D9 · Factura repetida (mismo proveedor y número) o vieja (algún producto tiene una compra con fecha posterior): aviso amarillo en la cabecera, no bloquea. Al confirmar una vieja se guarda la compra pero no cambia costo ni precio de esos productos.
- D10 · Descuentos generales de la factura (al pie): se ignoran.
- D11 · Unidades de regalo (renglón a $0 o con 100 % de bonificación) se suman al mismo producto y bajan su costo unitario.
- D12 · Facturas de una sola hoja: no hay "agregar otra hoja".
- D18 · Si baja el costo, se sugiere el precio más bajo ("antes $X") y se aplica solo al confirmar.
- D19 · Muestras gratis (regalo de un producto que no viene pago en la factura): no se toman en cuenta; renglón gris, no se guarda.
- D20 · El botón "Porcentajes" se llama **Rubros**.
- D21 · Subir productos: Cámara, PDF y Manual grandes; abajo Rubros y Proveedores chicos, lado a lado.
- Cálculo de precios solo en `src/dominio/precios.ts`. La base recibe precios ya calculados por RPC atómicas.
- Sin `margen_override` por producto: el escape es "precio fijo". Código de barras pasa a tabla propia en fase 2.

## Aprendizajes y errores a evitar
- `100 * 1.15` da `114.99999999999999` en JS: por eso el redondeo se hace con enteros.

## Próximos pasos
- Juntar 15–20 facturas reales (fotos y PDF) para el set de prueba de Gemini.
