# Fase 1 — Pantallas, flujos y estados

Mapa general. El detalle verificable (requisitos EARS) está en cada `specs/NNN-*/spec.md`. Lo marcado **[propuesta]** no está en el brief: es una consecuencia necesaria o una ubicación de UI que hay que aprobar (ver `docs/riesgos-y-preguntas.md`).

## Principios de interfaz (para todas las pantallas)

- Celular Android en vertical, ancho de diseño 360 px. Botones de al menos 56 px de alto, texto base 18 px.
- Una acción principal por pantalla, abajo y al alcance del pulgar.
- Nunca un spinner sin texto: siempre qué está pasando ("Leyendo la factura…").
- Ningún error deja a la usuaria trabada: cada error dice qué pasó y ofrece una salida (reintentar, cargar a mano, seguir después).
- Nada se pierde al cerrar la app: borradores de revisión y cola se guardan en el dispositivo.

## Navegación

```
Ingreso ─┬─ admin/dev ─→ Inicio ─┬─ Subir productos ─┬─ Cámara ──→ (cola) ─→ Revisión
         │                       │                   ├─ Subir PDF ─→ (cola) ─→ Revisión
         │                       │                   ├─ Manual ─┬─ Repetir última compra ─→ Revisión
         │                       │                   │          ├─ Cargar renglones ─→ Revisión
         │                       │                   │          └─ Fraccionado ─→ Revisión
         │                       │                   ├─ Rubros ─→ Editar rubro / Nuevo rubro
         │                       │                   └─ Proveedores ─→ Editar proveedor
         │                       └─ Precios ─→ Ficha de producto
         └─ familia ───→ Inicio (mismos permisos, D5)
```

## 1. Ingreso

- **Primera vez en el dispositivo:** dos botones grandes: "Soy la administradora" (email + contraseña) y "Familia" (teclado numérico de 6 dígitos).
- **Después:** la sesión queda abierta; la app abre directo en Inicio. La familia ve y puede hacer lo mismo que la administradora (D5).
- **Estados:** cargando ("Entrando…"); error de PIN ("PIN incorrecto"), sin revelar si existe la cuenta; sin conexión en el primer ingreso ("Necesitás internet la primera vez").

## 2. Inicio

- Dos botones grandes: **Subir productos** y **Precios**.
- Encima de los botones, solo si hay algo (D10): "Tenés 2 facturas para revisar" (toca y abre la primera), "1 factura esperando conexión", "1 factura no se pudo leer".
- **Vacío:** solo los dos botones.

## 3. Subir productos

Tres botones grandes de carga, uno debajo del otro: **Cámara**, **Subir PDF**, **Manual**. Abajo, dos botones más chicos lado a lado: **Rubros** (D20) y **Proveedores** (D8), según D21.

### 3a. Cámara
1. Abre la cámara del sistema (`<input type="file" accept="image/*" capture="environment">`).
2. Comprime y muestra "Leyendo la factura…" con la miniatura.
3. Con conexión: en 5–20 s abre la Revisión. Si tarda más de 20 s: "Sigue leyendo. Podés seguir con otra cosa; te aviso en Inicio."
4. Sin conexión o 429: "Guardada. La leo apenas haya conexión." y vuelve a Inicio. Botón secundario "Cargar a mano ahora".
5. Error de lectura: "No pude leer esta foto." con "Sacar otra" y "Cargar a mano".

### 3b. Subir PDF
Selector de archivos (`accept="application/pdf"`). Igual que la cámara desde el paso 2. Mensaje si el PDF no se puede abrir: "Este PDF no se puede abrir. ¿Probás con una foto?".

### 3c. Manual
1. Elegir proveedor: buscador + proveedores recientes como botones. "Proveedor nuevo" pide solo el nombre.
2. Tres botones: **Cargar renglones**, **Fraccionado** y, más chico, **Repetir última compra** (se usa poco, D9; deshabilitado con leyenda si ese proveedor no tiene compras).
3. **Cargar renglones:** por renglón, producto (buscador del catálogo o "nuevo"), packs, unidades por pack, precio del pack, bonificación. Teclado numérico. Termina en la Revisión.
4. **Fraccionado:** ver spec 007. Termina en la Revisión.

### 3d. Rubros
Lista de rubros con su margen ("Lácteos · 18 %", "Cigarrillos · precio fijo"). Tocar abre edición: nombre, margen o "precio fijo", y "Eliminar rubro". Botón "Nuevo rubro". Ver spec 005.

### 3e. Proveedores
Lista alfabética ("La Serenísima · pide lun · entrega mié"; "Distribuidora X · sin días"). Tocar abre edición: nombre y días de pedido y de entrega (chips L M M J V S D). Botón "Nuevo proveedor". Los proveedores también se crean solos desde la revisión, solo con el nombre. Ver spec 008.

## 4. Revisión (componente único)

**Cabecera:** proveedor (amarillo si es nuevo o no se reconoció), fecha, tipo, botón **Precios con IVA / Precios sin IVA (se suma)** (preseleccionado si Gemini lo sugirió; sin elegir y amarillo si dudó).

**Avisos amarillos de cabecera que no bloquean (D12):**
- "Esta factura ya se cargó el 12/11 (mismo proveedor y número)."
- "Factura vieja: 3 productos tienen una compra más nueva. Se guarda la compra pero no cambia su precio." Esos renglones muestran "Hay una compra del 20/11: el precio no cambia" en lugar del precio sugerido.

**Renglón (tarjeta):**
```
[verde]  Galletitas Oreo 118 g                       Galletitas
         2 packs × 24 = 48 unidades
         Costo $512,40 c/u   (+12 %)
         Precio  [ $670 ]                       antes $600
```
- Amarillo con el motivo arriba ("Producto nuevo", "No se leyó bien: precio borroso", "El importe no coincide").
- Tocar un renglón amarillo abre su edición: emparejar con producto existente (candidatos) o crear nuevo (nombre prellenado, rubro sugerido), corregir números, elegir rubro.
- Precio fijo: muestra el costo y el precio actual sin sugerencia; si el producto es nuevo, el precio es obligatorio y el renglón queda amarillo hasta cargarlo.
- Unidades de regalo (renglón a $0 o con 100 % de bonificación): se muestran unidas al renglón pago del mismo producto ("24 + 2 de regalo = 26 unidades") y bajan el costo (D17). Regalo de un producto que no viene pago en la factura (muestra gratis): renglón gris "Muestra sin cargo: no se carga", no se guarda (D19).
- Si el costo bajó, el precio sugerido también baja y se ve "antes $X"; se aplica solo al confirmar (D18).
- El precio siempre es editable. Editarlo no cambia el color.

**Pie:** "Confirmar todo" (deshabilitado mientras quede amarillo, con el texto "Faltan 2 renglones"). "Descartar factura" en un menú secundario, con confirmación.

**Estados:**
- Cargando datos de la factura: esqueleto de tarjetas.
- Guardando: "Guardando…" y el botón bloqueado; sin conexión: "Se guarda apenas haya conexión" (la confirmación entra a la cola local y se aplica sola).
- Error al guardar: "No se pudo guardar. Tus cambios siguen acá." + "Reintentar".
- Factura sin renglones de mercadería: "No encontré productos en esta factura." + "Cargar a mano" / "Descartar".
- El borrador (ediciones hechas) se guarda en el dispositivo a cada cambio.

## 5. Precios

**Barra superior:** buscador (foco automático) + desplegable "Todos los rubros".

**Sin búsqueda ni rubro elegido:** grilla 2 columnas de botones grandes, uno por rubro, con la cantidad de productos.

**Resultados:** lista de tarjetas ordenadas alfabéticamente (por relevancia si hay texto buscado):
```
Galletitas Oreo 118 g                   $670
Galletitas · hace 3 días
```
Fraccionados: "Maní · bolsita 100 g · $350" o "Maní · $3.200 el kg · $320 los 100 g".

**Tocar una tarjeta** (cualquier usuario, D5): ficha con costo, margen del rubro, último proveedor, historial (últimos 10 cambios), editar precio, cambiar de rubro (pregunta si recalcula), marcar precio fijo.

**Estados:**
- Sin resultados: "No encontré «galletits». Probá con otras letras o elegí un rubro."
- Catálogo vacío (primer uso): "Todavía no hay productos. Se cargan solos con las facturas." con botón "Subir productos".
- Offline: funciona igual; arriba, discreto: "Sin conexión · precios del 21/11 18:40".
- Primera sincronización: "Descargando precios…" con barra.
- Error de sincronización: se siguen mostrando los precios guardados y el aviso de fecha.
