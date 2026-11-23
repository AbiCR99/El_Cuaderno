# Brief original del proyecto

Copia del encargo inicial, sin cambios de alcance. Las decisiones posteriores están en `MEMORY.md` y `docs/riesgos-y-preguntas.md`; si contradicen este archivo, mandan ellas.

> **Cambio importante posterior (D5):** la familia ya **no** es solo consulta. Tiene los mismos permisos que la administradora: ve costos, carga facturas y edita precios. Lo que dice abajo sobre la familia quedó superado.

## Contexto

- Almacén de barrio en Mar del Plata. Compra en cantidades chicas (6 paquetes de galletitas, 3 de shampoo) y recibe **entre 35 y 42 facturas de proveedores por semana**.
- La dueña atiende **sola**, de 9 a 14 y de 17 a 22 hs. Su mayor carga es calcular costos y precios de venta y controlar el stock.
- **Principio rector: todo lo que haga en la app tiene que ser más rápido que lo que hace hoy.** Si una función le suma trabajo, no la va a usar. Ante la duda, menos pasos y botones grandes.
- La desarrolladora vive en Córdoba y desarrolla a distancia. Visitas espaciadas: la próxima en ~1 mes, la siguiente 5 meses después.
- La usuaria usa un celular **Android**.

## Usuarios y roles

1. **Administradora (la dueña):** carga facturas, ve costos, edita precios y márgenes.
2. **Familia (hermana, cuñado y papá):** **solo consultan precios**. Atienden cuando ella no está. Entran con un PIN, no ven costos ni márgenes y no pueden modificar nada.
3. **Desarrolladora:** administra todo a distancia.

## Fase 1 (ahora): costos y precios

**Inicio (administradora): dos botones grandes**
- **Subir productos** → cuatro botones grandes: **Cámara** (foto → Gemini), **Subir PDF**, **Manual** (ver fraccionados), **Porcentajes** (márgenes por rubro).
- **Precios** → buscador + desplegable de categorías.

**Carga de facturas**
- **Cámara:** imagen comprimida en el celular (~200–300 KB, JPEG o WebP), guardada en Supabase Storage y enviada a Gemini (pago) desde una Edge Function. La API key nunca va al cliente.
- **PDF:** si es digital, se extrae el texto con pdf.js y se le pasa a Gemini para estructurarlo. Si es escaneado, se trata como imagen.
- Gemini devuelve JSON estructurado (structured output): proveedor, fecha, tipo de factura, renglones (descripción, cantidad, unidades por bulto, precio unitario, descuento por renglón, alícuota de IVA si figura). Sugiere una categoría por renglón **de la lista cerrada de rubros**, sin inventar rubros.
- **Cola y reintentos:** sin conexión o con error 429, la factura queda en cola y se reintenta sola. Nunca bloquea. La carga manual siempre está disponible.

**Pantalla de revisión (una sola, compartida por cámara, PDF y manual)**
- Un único componente que recibe renglones, vengan de donde vengan. Una sola lógica de cálculo de precios.
- Por renglón: producto reconocido, pack × cantidad = unidades, costo unitario, precio sugerido y aviso si el costo cambió respecto de la compra anterior ("+12%").
- Reconocidos con seguridad en verde; los que requieren atención (producto nuevo, sin categoría, dudas de lectura) en amarillo.
- Botón único **"Confirmar todo"**: la usuaria solo toca lo amarillo.
- **Emparejamiento por alias:** el texto del proveedor ("GALL. OREO 118G X 24") se asocia una vez a un producto y queda en `alias_producto`. Las siguientes facturas se reconocen solas. Crítico para la velocidad.
- **"Repetir última compra"** por proveedor (proveedores diarios como el lechero): precarga la compra anterior y la usuaria solo corrige cantidades.
- El catálogo se arma solo a partir de las facturas. No hay carga inicial.

**Reglas de negocio**
- **IVA:** monotributista → el IVA forma parte del costo. Cada factura tiene botón **con IVA / sin IVA**. Gemini puede sugerirlo y ella confirma. Alícuota por defecto 21%; si la factura discrimina otra por renglón (10,5%), se usa esa.
- **Bultos:** costo unitario = precio del bulto ÷ unidades por bulto. Se restan las bonificaciones por renglón.
- **Precio de venta:** `precio = redondear(costo_unitario × (1 + margen_del_rubro))`.
- **Redondeo:** a la decena más cercana (113 → 110, 117 → 120). Siempre sugerencia editable en la revisión.
- **Precio fijo:** un rubro o producto puede ser "precio fijo" (cigarrillos). Se registra el costo, pero el precio se ingresa a mano y nunca se recalcula.
- **Historial de costos y precios** por producto.

**Fraccionados (carga manual)**
- Productos a granel que se fraccionan (cereales, snacks, condimentos).
- Campos: producto, precio de compra, peso de compra, cantidad de fracciones (por peso o por unidades).
- Dos modos de venta: **por fracción** (bolsitas de peso fijo, precio por bolsita) o **por peso** (precio por kg y por 100 g).
- El modo de venta **no es una categoría**: el maní fraccionado va en su rubro (Snacks).

**Porcentajes (márgenes por rubro)**
- Un nivel de categorías, sin subcategorías (10 a 15 rubros).
- La administradora edita margen, agrega, renombra y elimina rubros. Si elimina uno con productos, se pregunta adónde moverlos o van a "Sin categoría".
- **Al cambiar el margen de un rubro se recalculan todos sus productos, incluidos los ya cargados**, con redondeo, previa confirmación ("Esto cambia el precio de 34 productos. ¿Confirmar?").
- Si se cambia un producto de rubro, se pregunta si recalcular con el margen del nuevo rubro.
- Valores iniciales: Almacén 30% · Lácteos 18% · Galletitas 30% · Golosinas y chocolates 40% · Snacks 35% · Bebidas sin alcohol 30% · Bebidas con alcohol 40% · Limpieza 30% · Perfumería e higiene 30% · Cigarrillos precio fijo · Sin categoría 30%.

**Precios (consulta)**
- Buscador tolerante: 2 o 3 letras, sin importar tildes, con errores de tipeo, y también por alias.
- Desplegable de categorías que **filtra**; el buscador busca dentro de lo filtrado. Orden alfabético dentro de la categoría.
- Pantalla vacía con grilla de botones grandes por categoría, además del desplegable.
- Cada precio muestra cuándo se actualizó ("hace 3 días").
- **Funciona sin internet** (service worker + IndexedDB) y sincroniza al haber conexión.
- La vista de familia es esta misma pantalla, sin costos ni márgenes y sin edición.

**Stock en fase 1:** no se muestra "stock" en la interfaz. Se guarda el historial completo de compras.

**Proveedores:** cargar desde ahora sus **días de pedido y de entrega**, aunque se usen en fase 2.

## Fase 2 (~1 mes, en la visita): ventas y stock real — solo planificar
Pantalla de venta (buscador + más vendidos + "importe suelto"), descuento de stock, medio de pago opcional, ticket "Documento no válido como factura", impresora térmica Bluetooth (protocolo a resolver), códigos de barras asociados por la desarrolladora, conteo físico inicial, alerta de stock bajo por proveedor el día anterior al pedido (pg_cron + Edge Function + Resend; evaluar push), la familia usa el mismo acceso para cobrar.

## Fase 3 (~5 meses): informe de compras
Recomendaciones de cuánto, qué pack, a qué proveedor y cada cuánto comprar. Consumo real = stock inicial + compras − stock final. Desde la fase 1 cada renglón guarda producto, proveedor, fecha, tamaño del pack, cantidad de packs y costo unitario.

## Fuera de alcance
Comparación de precios con el mercado (SEPA/Precios Claros/web), facturación electrónica (ARCA), entrenar modelos propios.
