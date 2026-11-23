# Spec 007 — Fraccionados
Estado: borrador

## Contexto y objetivo
Productos que se compran a granel (cereales, snacks, condimentos) y se venden en bolsitas o por peso. Hoy calcular su precio es especialmente lento.

## Usuarios
Administradora y familia (mismos permisos, D5).

## Definiciones
- **Por fracción**: bolsitas de peso fijo; se vende cada bolsita.
- **Por peso**: se vende al peso; precio por kg y por 100 g.
- El modo de venta es una propiedad del producto, no un rubro.

## Historias de usuario
- HU-1. Como administradora quiero cargar "compré 5 kg de maní a $15.000, hago bolsitas de 100 g" y ver el precio de cada bolsita.
- HU-2. Como administradora quiero vender orégano por peso y ver el precio del kg y de 100 g.

## Requisitos funcionales
- RF-1: CUANDO la administradora elige Manual → Fraccionado, EL SISTEMA pide producto (existente o nuevo, con rubro), precio de compra, peso de compra y modo de venta.
- RF-2: MIENTRAS el modo es "por fracción", EL SISTEMA permite indicar la fracción por peso (gramos por bolsita) o por unidades (cantidad de bolsitas) y calcula el otro valor.
- RF-3: EL SISTEMA calcula para "por fracción" el costo por bolsita como costo de compra ÷ cantidad de bolsitas y su precio con el motor de la spec 002.
- RF-4: EL SISTEMA calcula para "por peso" el costo por kg y por 100 g, y redondea el precio de cada uno por separado con el motor de la spec 002.
- RF-5: CUANDO se termina la carga, EL SISTEMA abre la revisión de la spec 003 con el renglón fraccionado y el botón con IVA / sin IVA.
- RF-6: CUANDO se confirma, EL SISTEMA guarda el fraccionamiento, el modo de venta del producto, el costo y el precio con historial de origen `fraccionado`.
- RF-7: CUANDO se carga otra vez un producto ya fraccionado, EL SISTEMA precarga el modo y la fracción de la última vez.
- RF-8: SI el peso de compra no es divisible exactamente en fracciones, ENTONCES EL SISTEMA usa la cantidad entera de bolsitas y avisa cuántos gramos sobran.

## Casos límite
- Peso en kg con decimales ("2,5 kg").
- Fracción mayor que el peso de compra.
- Cambio de modo de un producto ya fraccionado.

## Fuera de alcance
- Fraccionados que llegan en facturas fotografiadas (D14: se resuelve después).
- Merma por fraccionado.

## Criterios de finalización
- Tests de dominio para ambos modos y los casos límite.
- Demo con los dos ejemplos de las historias.

## Dudas abiertas
- (ninguna: D14 y D15 resueltas)
