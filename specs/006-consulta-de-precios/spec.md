# Spec 006 — Consulta de precios offline y ficha de producto
Estado: borrador

## Contexto y objetivo
Hoy la familia no sabe los precios cuando atiende. Esta pantalla les da el precio en segundos, aunque no haya señal en el almacén. Todos los usuarios ven lo mismo y pueden editar (D5).

## Usuarios
Familia y administradora, con los mismos permisos (D5).

## Historias de usuario
- HU-1. Como familiar quiero escribir "oreo" o "galle" y ver el precio.
- HU-2. Como familiar quiero ver todos los lácteos sin escribir nada.
- HU-3. Como administradora o familiar quiero corregir un precio desde la consulta.

## Requisitos funcionales
- RF-1: EL SISTEMA encuentra productos a partir de 2 letras, sin distinguir mayúsculas ni tildes, buscando en nombre y alias.
- RF-2: EL SISTEMA tolera un error de tipeo en palabras de 4 letras o más ("galletits" encuentra "galletitas").
- RF-3: CUANDO hay un rubro elegido en el desplegable, EL SISTEMA busca solo dentro de ese rubro.
- RF-4: MIENTRAS no hay texto buscado, EL SISTEMA ordena alfabéticamente; con texto, por relevancia.
- RF-5: MIENTRAS no hay texto ni rubro elegido, EL SISTEMA muestra una grilla de botones grandes, uno por rubro, con su cantidad de productos.
- RF-6: EL SISTEMA muestra en cada producto el precio y cuándo se actualizó ("hoy", "ayer", "hace 3 días").
- RF-7: EL SISTEMA muestra los fraccionados como "bolsita 100 g · $350" o "$3.200 el kg · $320 los 100 g".
- RF-8: EL SISTEMA guarda la lista de precios en el dispositivo y funciona sin conexión.
- RF-9: CUANDO hay conexión al abrir la app, al volver a ella o cada 5 minutos con la app abierta, EL SISTEMA descarga solo los cambios desde la última sincronización.
- RF-10: MIENTRAS no hay conexión, EL SISTEMA muestra "Sin conexión · precios del {fecha y hora}".
- RF-11: EL SISTEMA guarda en el dispositivo también el costo, el último proveedor y la última fecha de compra, para que la ficha funcione sin conexión (solo lectura offline).
- RF-12: CUANDO un usuario toca un producto, EL SISTEMA abre su ficha con costo, margen, último proveedor, últimos 10 cambios de precio, edición de precio, cambio de rubro y marca de precio fijo.
- RF-13: CUANDO un usuario edita un precio con conexión, EL SISTEMA lo guarda con historial de origen `manual` (hasta el próximo recálculo, D1).
- RF-14: SI la búsqueda no encuentra nada, ENTONCES EL SISTEMA muestra "No encontré «texto». Probá con otras letras o elegí un rubro."
- RF-15: SI el catálogo está vacío, ENTONCES EL SISTEMA muestra "Todavía no hay productos. Se cargan solos con las facturas."
- RF-16: EL SISTEMA no muestra productos desactivados.
- RF-17: SI no hay conexión, ENTONCES EL SISTEMA muestra la ficha en modo lectura y avisa "Necesitás conexión para cambiar precios".

## Requisitos no funcionales
- Resultados en menos de 100 ms por tecla con 2.000 productos en un Android de gama baja.
- La primera descarga completa de 2.000 productos en menos de 5 s en 4G.

## Casos límite
- Búsqueda con espacios extra o "ñ" (`"ñoquis"`, `"noquis"`).
- Producto sin precio todavía (precio fijo sin cargar): "Sin precio".
- Sincronización interrumpida a la mitad.
- Rubro eliminado mientras el familiar tenía la lista offline.

## Fuera de alcance
- Stock (fase 1 no lo muestra).
- Código de barras (fase 2).

## Criterios de finalización
- Tests de búsqueda (normalización, prefijo, tipeo, filtro por rubro).
- Demo en Android con modo avión, entrando con PIN y con email.

## Dudas abiertas
- (ninguna)
