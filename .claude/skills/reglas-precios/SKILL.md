---
name: reglas-precios
description: Úsala siempre que escribas, modifiques o revises código que calcule o muestre costos, IVA, bultos, bonificaciones, márgenes, redondeo, precio fijo, fraccionados o variación de costo en El Cuaderno, incluida la pantalla de revisión, Porcentajes y la ficha de producto.
---

# Reglas de costos y precios

Esta es la regla de negocio central. Una sola implementación en `src/dominio/` (`costos.ts`, `precios.ts`, `redondeo.ts`, `fraccionados.ts`). Ninguna pantalla, RPC ni migración hace estas cuentas.

## Unidades
- Costos: pesos con 2 decimales (centavos). Precios de venta: pesos enteros, múltiplos de 10.
- Márgenes: porcentaje con hasta 2 decimales (`30`, `17.5`). Internamente, puntos básicos enteros (`3000`, `1750`).
- Toda entrada de la usuaria pasa por `parsearNumeroAR` ("1.234,50" → 1234.5).

## Orden del cálculo (un renglón)
1. `neto = packs × precioUnitario × (1 − bonifPct/100) − bonifMonto`
2. Factura **sin IVA** (los precios no lo traen): `costoRenglon = neto × (1 + alicuota/100)`, con `alicuota = renglon.alicuota ?? 21`.
   Factura **con IVA** (ya lo traen): `costoRenglon = neto`.
3. `costoUnitario = redondearCentavos(costoRenglon / (packs × unidadesPorPack))`
4. Si el **precio fijo es efectivo** (producto o rubro): no hay precio sugerido.
5. Si no: `precio = redondearDecena(costoUnitario, margenPct)`.

## Redondeo (decisión D2: el 5 va hacia arriba)
Nunca `Math.round(costo * 1.15)`: `100 * 1.15` da `114.99999999999999` y redondea a 110. Calcular con enteros:

```ts
// costoUnitario en pesos con 2 decimales, margenPct con hasta 2 decimales
const centavos = Math.round(costoUnitario * 100);          // entero
const bp = Math.round(margenPct * 100);                    // 30 % → 3000
const n = centavos * (10000 + bp);                         // pesos × 1.000.000, entero exacto
const decenas = Math.floor((n + 5_000_000) / 10_000_000);  // mitad hacia arriba
return decenas * 10;
```

## Recalcular pisa lo manual (decisión D1)
- Cambiar el margen de un rubro recalcula **todos** sus productos con costo y sin precio fijo efectivo, aunque tengan un precio editado a mano.
- Confirmar una factura reemplaza el precio por el que muestra la revisión.
- Precio fijo: se guarda el costo; el precio solo cambia si ella lo escribe.

## Variación de costo
`variacion = round((costoNuevo − costoAnterior) / costoAnterior × 100)` → "+12 %", "−3 %". Sin compra anterior o costo anterior 0: sin variación.

## Fraccionados
- Por fracción: `costoBolsita = costoCompra / cantidadBolsitas`; precio con `redondearDecena`.
- Por peso: `costoKg = costoCompra / pesoKg` y `costo100g = costoCompra / (pesoGramos / 100)`, cada uno redondeado por separado (no `precioKg / 10`).
- Si el peso no divide exacto: bolsitas enteras y aviso de gramos sobrantes.

## Casos de prueba obligatorios
| Caso | Esperado |
|---|---|
| costo 100, margen 13 % | 110 |
| costo 100, margen 15 % | 120 |
| costo 100, margen 17 % | 120 |
| costo 950, margen 30 % (1235 exacto) | 1240 |
| costo 949,99, margen 30 % (1234,987) | 1230 |
| bulto $12.000 × 1 pack, 24 u, con IVA | costo 500,00 |
| bulto $10.000 × 1, 24 u, sin IVA, 21 % | costo 504,17 |
| alícuota 10,5 % sin IVA, neto 1000, 1 u | costo 1105,00 |
| bonif 10 % sobre 2 × $1.000 | neto 1800 |
| bonif 100 % (unidad de regalo) | costo 0 → renglón amarillo, sin precio sugerido (P14) |
| neto negativo | error de validación |
| precio calculado menor a 5 | 0 → renglón amarillo, no sugerir $0 |
| precio fijo efectivo | sin sugerencia |

## Checklist de revisión
- [ ] ¿Alguna cuenta de costo o precio fuera de `src/dominio/`?
- [ ] ¿Algún `Math.round`, `toFixed` o `* (1 +` sobre precios fuera de `redondeo.ts`?
- [ ] ¿Se respeta el precio fijo del **rubro** además del del producto?
- [ ] ¿El IVA se suma solo cuando la factura es "sin IVA"?
- [ ] ¿La bonificación se resta antes de sumar IVA?
- [ ] ¿Todo cambio de costo o precio deja historial con su origen?

## Al terminar
Indicá qué casos de la tabla quedaron cubiertos por tests.
