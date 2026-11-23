# Fase 2 — Ventas y stock real (planificación, no se construye todavía)

Se construye para la visita (~1 mes). Esta página solo deja anotado lo que la fase 1 ya tiene que prever.

## Lo que la fase 1 deja listo

- Historial completo de compras con producto, proveedor, fecha, pack, cantidad de packs y costo unitario (`factura_items` confirmados).
- Días de pedido y entrega de cada proveedor.
- `productos.modo_venta` (unidad, fracción, peso) para que la venta sepa cómo cobrar.
- `lista_precios` offline y el PIN de la familia: la misma base y el mismo acceso sirven para la pantalla de venta.

## A construir

| Tema | Notas |
|---|---|
| Pantalla de venta en computadora | Buscador + botones de los más vendidos (con historial de ventas; al principio, los más comprados), toque para sumar, "Importe suelto" que no descuenta stock. |
| Cobro | Descuenta stock (`movimientos_stock`), medio de pago opcional, número correlativo interno (secuencia de Postgres). |
| Ticket | Nombre del almacén, fecha y hora, número, detalle, total y "Documento no válido como factura". |
| Impresora térmica Bluetooth | Probablemente protocolo propio, no ESC/POS. Investigar el modelo exacto antes de la visita; Web Bluetooth en Chrome de escritorio. Plan B: imprimir desde el celular con la app del fabricante. |
| Códigos de barras | Tabla `codigos_barras`; los asocia la desarrolladora escaneando góndolas. En venta, código desconocido → "¿Asociar ahora?" con "Después", sin trabar. |
| Conteo inicial | "Día cero" de los productos principales. Pantalla de conteo rápido por rubro. |
| Alerta de stock bajo | Un aviso por proveedor el día anterior a su día de pedido, con sus productos bajos. Stock mínimo a mano. pg_cron + Edge Function + Resend (verificar dominio propio). Evaluar push de la PWA. |
| Familia cobra | Con PIN compartido (D3) las ventas quedan a nombre de "familia". |

## Preguntas para la visita
- Modelo exacto de impresora y si imprime desde el navegador.
- Qué computadora se usa para cobrar y si tiene lector de código de barras.
- Cuáles son los "productos principales" para el conteo inicial.
- Medios de pago que usa (efectivo, transferencia, QR).
