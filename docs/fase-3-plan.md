# Fase 3 — Informe de compras (planificación, ~5 meses)

## Objetivo
Recomendar cuánto comprar, en qué tamaño de pack, a qué proveedor y cada cuánto.

## Datos necesarios y de dónde salen

| Dato | Fuente |
|---|---|
| Compras por producto, proveedor, fecha, pack y costo unitario | `factura_items` confirmados (desde la fase 1) |
| Ventas | `venta_items` (desde la fase 2) |
| Stock inicial y final | `conteos` inicial (visita 1) y de control (visita 2) |
| Consumo real estimado | `stock inicial + compras − stock final` en el período |
| Días de pedido y entrega | `proveedores` (desde la fase 1) |

## Análisis previstos
- Precio por unidad según pack y proveedor (¿conviene el bulto de 24 o el de 12?).
- Frecuencia de compra vs. consumo: sobrestock y faltantes.
- Capital inmovilizado por rubro.
- Vencimientos y espacio: no hay datos todavía; se relevan en la visita 2 si hacen falta.

## Lo que la fase 1 tiene que cuidar
- No editar ni borrar `factura_items` confirmados (son el historial).
- Unidades por pack correctas: son la base de todo el análisis por unidad.
- Un producto = una fila: los duplicados distorsionan el consumo (ver riesgo de alias).
