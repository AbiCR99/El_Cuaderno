# Spec 003 — Pantalla de revisión, alias y carga manual
Estado: borrador

## Contexto y objetivo
Es la pantalla donde la administradora pasa más tiempo. Recibe renglones de cualquier origen (Gemini, PDF, manual, repetir compra), los empareja con el catálogo y le deja confirmar todo de un toque, tocando solo lo que necesita atención. El catálogo se arma solo a partir de lo que se confirma acá.

## Usuarios
Administradora (y desarrolladora).

## Historias de usuario
- HU-1. Como administradora quiero confirmar una factura conocida con un solo toque.
- HU-2. Como administradora quiero que un producto que ya asocié una vez se reconozca solo en la próxima factura del mismo proveedor.
- HU-3. Como administradora quiero repetir la compra de ayer del lechero y solo cambiar cantidades.
- HU-4. Como administradora quiero cargar una factura a mano cuando no hay foto o la lectura falló.

## Requisitos funcionales
- RF-1: EL SISTEMA muestra la revisión con el mismo componente sin importar el origen de los renglones.
- RF-2: CUANDO el texto normalizado de un renglón coincide con un alias del mismo proveedor, EL SISTEMA lo empareja con ese producto y precarga las unidades por pack del alias si el renglón no las trae.
- RF-3: EL SISTEMA muestra por renglón: producto, rubro, "packs × unidades = total", costo unitario, variación contra la compra anterior y precio sugerido editable, usando el motor de la spec 002.
- RF-4: EL SISTEMA marca en amarillo, con su motivo, los renglones con: producto sin emparejar, rubro "Sin categoría" en producto nuevo, lectura dudosa, importe que no coincide, unidades por pack desconocidas en producto sin alias, o precio vacío en producto de precio fijo. El resto, en verde.
- RF-5: CUANDO la administradora toca un renglón amarillo sin emparejar, EL SISTEMA ofrece hasta 3 productos parecidos del catálogo y la opción "Producto nuevo" con el nombre prellenado y el rubro sugerido.
- RF-6: MIENTRAS quede algún renglón amarillo o el botón con/sin IVA sin elegir, EL SISTEMA deshabilita "Confirmar todo" y muestra cuántos faltan.
- RF-7: CUANDO la administradora toca "Confirmar todo", EL SISTEMA guarda en una sola operación: productos nuevos, alias nuevos, costo y precio de cada producto, historial y la factura como confirmada.
- RF-8: CUANDO se confirma una factura, EL SISTEMA reemplaza el precio de cada producto por el precio mostrado en la revisión, aunque antes tuviera un precio editado a mano (D1), excepto los de precio fijo, que solo cambian si ella editó el precio.
- RF-9: CUANDO la administradora cambia el botón con IVA / sin IVA, EL SISTEMA recalcula al instante todos los costos y precios sugeridos de la factura que no fueron editados a mano en esta revisión.
- RF-10: CUANDO la administradora elige un proveedor en Manual y toca "Repetir última compra", EL SISTEMA abre la revisión con los renglones de la última factura confirmada de ese proveedor, con los precios de entonces, listos para corregir.
- RF-11: SI el proveedor no tiene compras confirmadas, ENTONCES EL SISTEMA muestra "Repetir última compra" deshabilitado con la leyenda "Todavía no hay compras de este proveedor".
- RF-12: CUANDO la administradora carga renglones a mano, EL SISTEMA pide producto, packs, unidades por pack, precio del pack y bonificación, con teclado numérico, y termina en la revisión.
- RF-13: CUANDO el proveedor de la factura es nuevo, EL SISTEMA permite cargar sus días de pedido y de entrega sin que sean obligatorios. [NECESITA ACLARACIÓN P4]
- RF-14: SI se cierra la app durante la revisión, ENTONCES EL SISTEMA conserva las ediciones y las muestra al volver.
- RF-15: SI no hay conexión al confirmar, ENTONCES EL SISTEMA guarda la confirmación en la cola local, informa "Se guarda apenas haya conexión" y la aplica sola después.
- RF-16: CUANDO la administradora descarta una factura y lo confirma, EL SISTEMA la marca como descartada sin tocar productos ni precios.
- RF-17: EL SISTEMA muestra en Inicio, solo si existen, la cantidad de facturas para revisar, en cola y con error. [NECESITA ACLARACIÓN P6]

## Requisitos no funcionales
- Confirmar una factura de 10 renglones todos verdes: un toque y menos de 3 s con conexión.
- Toda la pantalla usable con una mano en 360 px.

## Casos límite
- Dos renglones de la misma factura con el mismo producto.
- Un alias que apunta a un producto desactivado.
- El mismo texto con distintas unidades por pack en dos facturas.
- Confirmar dos veces (doble toque o cola que reintenta): no duplica nada.
- Producto nuevo con nombre igual a uno existente: se ofrece el existente primero.

## Fuera de alcance
- Unir productos duplicados (fase 2 / visita).
- Descuentos generales de la factura (P9).
- Editar facturas ya confirmadas.

## Criterios de finalización
- Tests de dominio de clasificación verde/amarillo y de reconstrucción de "repetir compra".
- Demo en Android: factura nueva (todo amarillo) → confirmar → misma factura otra vez → todo verde, un toque.

## Dudas abiertas
- [NECESITA ACLARACIÓN] P3, P4, P5, P6, P8, P9 y P12 de `docs/riesgos-y-preguntas.md`.
