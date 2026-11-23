# Spec 002 — Motor de costos y precios (dominio) y esquema base
Estado: borrador

## Contexto y objetivo
Todas las pantallas que muestran un costo o un precio sugerido usan las mismas reglas. Esta spec define esas reglas de forma verificable y crea el esquema de datos que las guarda. Es la base de la revisión, los porcentajes y los fraccionados.

## Usuarios
Ninguno directo (lo usan las specs 003, 005 y 007).

## Definiciones
- **Costo del renglón**: lo pagado por el renglón, IVA incluido y neto de bonificaciones.
- **Costo unitario**: costo del renglón ÷ unidades totales (packs × unidades por pack), redondeado a centavos (0,005 hacia arriba).
- **Redondeo a la decena**: múltiplo de 10 más cercano; el 5 va hacia arriba (D2).
- **Precio fijo efectivo**: el producto está marcado precio fijo, o su rubro lo está.

## Requisitos funcionales
- RF-1: EL SISTEMA calcula el neto del renglón como `packs × precio_unitario × (1 − bonif% / 100) − bonif$`.
- RF-2: SI la factura está marcada "sin IVA", ENTONCES EL SISTEMA calcula el costo del renglón como `neto × (1 + alícuota / 100)`, con la alícuota del renglón o 21 si no tiene.
- RF-3: SI la factura está marcada "con IVA", ENTONCES EL SISTEMA usa el neto como costo del renglón, sin sumar IVA.
- RF-4: EL SISTEMA calcula el costo unitario como costo del renglón ÷ (packs × unidades por pack), redondeado a centavos.
- RF-5: EL SISTEMA calcula el precio sugerido como `redondearDecena(costo_unitario × (1 + margen / 100))` usando aritmética entera, de modo que costo 100 y margen 15 % den 120, y costo 100 y margen 13 % den 110.
- RF-6: MIENTRAS un producto tiene precio fijo efectivo, EL SISTEMA no sugiere ni recalcula su precio y solo registra el costo.
- RF-7: EL SISTEMA calcula la variación de costo contra la compra anterior como porcentaje entero con signo ("+12 %", "−3 %"); sin compra anterior no hay variación.
- RF-8: SI packs, unidades por pack o precio unitario son cero, negativos o no numéricos, ENTONCES EL SISTEMA devuelve un error de validación en lugar de un costo.
- RF-9: SI la bonificación deja el neto en negativo, ENTONCES EL SISTEMA devuelve un error de validación.
- RF-10: EL SISTEMA normaliza textos de proveedor para alias: mayúsculas, sin tildes, sin signos repetidos, espacios colapsados ("Gall. Oreo  118g x24" → "GALL. OREO 118G X24").
- RF-11: EL SISTEMA interpreta números escritos en formato argentino ("1.234,50" → 1234,50) en todos los campos de carga.
- RF-12: SI un renglón tiene costo 0 (precio $0 o bonificación 100 %) y en la misma factura hay un renglón pago del mismo producto, ENTONCES EL SISTEMA suma sus unidades a ese renglón y calcula un solo costo unitario: costo pagado ÷ (unidades pagas + unidades de regalo) (D17).
- RF-13: SI un renglón de regalo no tiene un renglón pago del mismo producto en la factura, ENTONCES EL SISTEMA lo devuelve como "muestra sin cargo", que no se toma en cuenta: sin costo, sin precio y sin compra (D19).
- RF-14: CUANDO se aplica la migración inicial, EL SISTEMA crea las tablas, políticas y RPC de `docs/modelo-datos.md` y los 11 rubros iniciales con sus márgenes.

## Requisitos no funcionales
- Funciones puras en `src/dominio/`, sin React ni Supabase, con cobertura de tests de todos los RF de cálculo.

## Casos límite
- Costo unitario con decimales largos (1000 ÷ 3).
- Margen 0 %. Margen con decimales (17,5 %).
- Precio resultante menor a 5 (redondea a 0): se marca para revisión en lugar de sugerir $0.
- Alícuota 10,5 % y 27 %.
- Bonificación 100 % (unidad de regalo) con y sin renglón pago del mismo producto.
- Dos renglones pagos del mismo producto con precios distintos más uno de regalo.

## Fuera de alcance
- Descuentos generales al pie de la factura: se ignoran (D13).

## Criterios de finalización
- `npm run test` en verde con un caso por RF de cálculo y por caso límite.
- Migración aplicada en el proyecto de desarrollo; `get_advisors` de Supabase sin alertas de seguridad.

## Dudas abiertas
- (ninguna)
