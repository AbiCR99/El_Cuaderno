# Spec 005 — Porcentajes: rubros y márgenes
Estado: borrador

## Contexto y objetivo
Con la inflación, la administradora ajusta márgenes por rubro. Cambiar un margen tiene que actualizar todos los precios del rubro de una vez, con una confirmación clara.

## Usuarios
Administradora y familia (mismos permisos, D5).

## Historias de usuario
- HU-1. Como administradora quiero subir el margen de Lácteos y que se actualicen todos los lácteos.
- HU-2. Como administradora quiero crear, renombrar y eliminar rubros sin perder productos.

## Requisitos funcionales
- RF-1: EL SISTEMA muestra la lista de rubros con su margen o la leyenda "precio fijo", con "Sin categoría" al final.
- RF-2: CUANDO la administradora cambia el margen de un rubro, EL SISTEMA muestra "Esto cambia el precio de N productos. ¿Confirmar?", donde N son los productos del rubro sin precio fijo efectivo.
- RF-3: CUANDO lo confirma, EL SISTEMA recalcula con el motor de la spec 002 el precio de esos N productos a partir de su último costo, los guarda con historial de origen `recalculo_rubro` y reemplaza los precios editados a mano (D1).
- RF-4: SI un producto del rubro no tiene costo registrado, ENTONCES EL SISTEMA no lo recalcula ni lo cuenta en N.
- RF-5: CUANDO la administradora crea un rubro, EL SISTEMA pide nombre y margen (o precio fijo) y rechaza nombres repetidos sin importar mayúsculas.
- RF-6: CUANDO la administradora renombra un rubro, EL SISTEMA cambia el nombre sin tocar precios.
- RF-7: CUANDO la administradora elimina un rubro con productos, EL SISTEMA pregunta adónde moverlos, con "Sin categoría" como opción por defecto, y no recalcula al moverlos salvo que ella lo pida.
- RF-8: EL SISTEMA impide eliminar o renombrar "Sin categoría".
- RF-9: CUANDO la administradora cambia un producto de rubro, EL SISTEMA pregunta "¿Recalcular con el margen de {rubro} (X %)? Precio nuevo: $Y".
- RF-10: CUANDO la administradora marca o desmarca un rubro como precio fijo, EL SISTEMA no cambia precios; al desmarcar, pregunta si recalcular como en RF-2.
- RF-11: SI no hay conexión, ENTONCES EL SISTEMA no permite editar rubros y muestra "Necesitás conexión para cambiar porcentajes".

## Requisitos no funcionales
- Recalcular 200 productos tarda menos de 3 s y es todo o nada.

## Casos límite
- Margen 0 %.
- Rubro con 0 productos (sin confirmación).
- Dos dispositivos cambiando el mismo rubro (gana el último; queda en el historial).

## Fuera de alcance
- Subcategorías.
- Margen por producto (ver `docs/modelo-datos.md`).

## Criterios de finalización
- Tests del cálculo de N y del lote de cambios.
- Demo: cambiar Lácteos de 18 a 20 % y verificar precios e historial.

## Dudas abiertas
- (ninguna)
