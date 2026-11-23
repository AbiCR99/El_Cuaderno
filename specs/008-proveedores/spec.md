# Spec 008 — Proveedores
Estado: borrador

## Contexto y objetivo
En la fase 2 la alerta de stock bajo se manda el día anterior al día de pedido de cada proveedor. Para eso, desde la fase 1 hay que tener cargados los días de pedido y de entrega. Se cargan aparte, en una pantalla propia, para no sumar pasos a la revisión de facturas (D8).

## Usuarios
Administradora y familia (mismos permisos, D5).

## Historias de usuario
- HU-1. Como administradora quiero anotar que La Serenísima toma pedidos los lunes y entrega los miércoles.
- HU-2. Como administradora quiero ver qué proveedores todavía no tienen días cargados.

## Requisitos funcionales
- RF-1: CUANDO se toca Subir productos → Proveedores, EL SISTEMA muestra los proveedores en orden alfabético con sus días ("pide lun · entrega mié") o la leyenda "sin días".
- RF-2: CUANDO se toca un proveedor, EL SISTEMA permite editar su nombre y elegir días de pedido y de entrega con botones L M M J V S D (varios días posibles, ninguno obligatorio).
- RF-3: CUANDO se toca "Nuevo proveedor", EL SISTEMA pide el nombre y opcionalmente los días.
- RF-4: SI el nombre nuevo o editado coincide con otro proveedor sin importar mayúsculas ni tildes, ENTONCES EL SISTEMA lo avisa y no lo guarda.
- RF-5: EL SISTEMA incluye en la lista los proveedores creados automáticamente desde la revisión de facturas.
- RF-6: SI no hay conexión, ENTONCES EL SISTEMA muestra la lista guardada y no permite editar ("Necesitás conexión para cambiar proveedores").
- RF-7: SI la lista está vacía, ENTONCES EL SISTEMA muestra "Todavía no hay proveedores. Se crean solos con las facturas." y el botón "Nuevo proveedor".

## Casos límite
- Proveedor que pide todos los días (lechero).
- Renombrar un proveedor que ya tiene facturas: sus facturas y alias siguen asociados.

## Fuera de alcance
- Eliminar o unir proveedores (fase 2 / visita).
- Uso de los días (alertas de stock, fase 2).
- Datos de contacto.

## Criterios de finalización
- Demo: crear un proveedor, cargarle días, renombrarlo y verlo en la revisión de una factura.

## Dudas abiertas
- (ninguna: D21)
