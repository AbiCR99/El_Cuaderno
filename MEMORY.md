# MEMORY.md — El Cuaderno

Memoria del proyecto entre sesiones. Máximo ~50 líneas: resumí o eliminá lo que ya no aporte. Nunca guardes claves, tokens, PIN ni datos personales.

## Estado actual
- Fase 1 (costos y precios): entorno preparado, sin código todavía. Próximo paso: spec 001 (`/sdd-plan 001-base-y-acceso`).
- Specs escritas en borrador: 001 a 007. Backlog en `docs/fase-1/backlog.md`.

## Decisiones (y por qué)
- 2026-11-23 · Precios editados a mano: **se pisan** al recalcular un rubro o al confirmar una factura nueva. La única excepción es "precio fijo". (Decisión de la dueña del proyecto.)
- 2026-11-23 · Redondeo: decena más cercana, **el 5 va hacia arriba** (115 → 120).
- 2026-11-23 · Familia: **un solo PIN compartido** (una cuenta `familia`). En fase 2 no se sabrá qué familiar cobró cada venta.
- 2026-11-23 · Frontend: React + Vite + **TypeScript**. Tipos generados desde Supabase.
- Cálculo de precios solo en `src/dominio/precios.ts` (TS). La base recibe precios ya calculados por RPC atómicas.
- Sin `margen_override` por producto: el brief no lo pide; el escape es "precio fijo".
- Código de barras sale de `productos` y pasa a tabla propia en fase 2 (un producto puede tener varios).

## Aprendizajes y errores a evitar
- `100 * 1.15` da `114.99999999999999` en JS: por eso el redondeo se hace con enteros.

## Próximos pasos
- Resolver las preguntas abiertas de `docs/riesgos-y-preguntas.md` antes de cerrar las specs 003 y 004.
- Juntar 15–20 facturas reales (fotos y PDF) para el set de prueba de Gemini.
