# Extracción de facturas con Gemini

## Flujo

1. **Cámara**: el celular comprime la foto (lado mayor 1600 px, WebP calidad 0,7; si pasa de 300 KB, baja la calidad hasta 0,5; si el navegador no codifica WebP, JPEG). Genera `factura_id = crypto.randomUUID()`, guarda foto + id en la cola local (Dexie) y, si hay conexión, sube a Storage e inserta la fila `facturas` en estado `en_cola`. Después llama a `extraer-factura`.
2. **PDF**: pdf.js extrae el texto de cada página. Si el PDF tiene menos de 40 caracteres útiles por página, se considera escaneado: se rasteriza cada página a imagen y sigue el camino de la cámara. Si es digital, el texto va en `facturas.texto_pdf` y el PDF original también se guarda en Storage.
3. **Edge Function `extraer-factura`**: pasa la factura a `procesando`, arma el pedido (imagen o texto + prompt + esquema + lista de rubros actual) y llama a Gemini.
   - Respuesta válida → guarda `extraccion_cruda`, crea `factura_items`, empareja proveedor y alias, estado `revision`.
   - 429 o 5xx o timeout → `en_cola`, `intentos + 1`, `proximo_intento_en` según backoff.
   - JSON inválido o `es_factura = false` → `error` con mensaje para la usuaria ("No pude leer esta foto. ¿La cargás a mano o sacás otra?").
4. La app abre la revisión si la usuaria sigue en la pantalla; si no, la factura aparece en el aviso "Tenés N facturas para revisar".

## Modelo

- Variable `GEMINI_MODEL` (secreto de Supabase). Arranque: `gemini-3.1-flash-lite`. Candidato más barato a comparar: `gemini-2.5-flash-lite`.
- Pensamiento al mínimo (es extracción, no razonamiento) y `temperature` 0.
- Antes de fijarlo, correr el **set de prueba** (15–20 facturas reales de proveedores distintos, con el resultado correcto escrito a mano) con ambos modelos y comparar: renglones bien leídos, precios exactos, unidades por bulto, costo por factura. Verificar límites de requests por minuto del proyecto en AI Studio.
- Costo estimado: con ~170 facturas al mes y unos pocos miles de tokens por factura, el gasto debería quedar en el orden de 1 dólar mensual. Se mide con el set de prueba; no es un número garantizado.

## Esquema de respuesta (structured output)

Se envía como `responseJsonSchema` con `responseMimeType: "application/json"`. El `enum` de `rubro_sugerido` se arma en cada llamada con los nombres actuales de `categorias`, así Gemini no puede inventar rubros.

```json
{
  "type": "object",
  "properties": {
    "es_factura": { "type": "boolean", "description": "false si la imagen no es un comprobante de compra" },
    "proveedor": {
      "type": "object",
      "properties": {
        "nombre": { "type": "string" },
        "cuit": { "type": ["string", "null"], "description": "Solo dígitos, 11 caracteres" }
      },
      "required": ["nombre", "cuit"]
    },
    "fecha": { "type": ["string", "null"], "description": "AAAA-MM-DD" },
    "tipo_comprobante": { "type": "string", "enum": ["A", "B", "C", "X", "remito", "presupuesto", "otro"] },
    "numero": { "type": ["string", "null"] },
    "precios_incluyen_iva": { "type": "string", "enum": ["si", "no", "no_se"] },
    "renglones": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "descripcion": { "type": "string", "description": "Texto literal del renglón" },
          "cantidad": { "type": "number", "description": "Cantidad facturada (bultos o unidades, como figure)" },
          "unidades_por_bulto": { "type": ["number", "null"] },
          "unidades_por_bulto_fuente": { "type": "string", "enum": ["columna", "descripcion", "no_figura"] },
          "precio_unitario": { "type": "number", "description": "Precio por cada unidad facturada, como figura" },
          "bonificacion_porcentaje": { "type": ["number", "null"] },
          "bonificacion_monto": { "type": ["number", "null"], "description": "Descuento total del renglón en pesos" },
          "alicuota_iva": { "type": ["number", "null"], "enum": [0, 2.5, 5, 10.5, 21, 27, null] },
          "importe": { "type": ["number", "null"], "description": "Importe total del renglón, como figura" },
          "rubro_sugerido": { "type": "string", "enum": ["<se completa con los rubros actuales>"] },
          "legibilidad": { "type": "string", "enum": ["clara", "dudosa"] },
          "nota": { "type": ["string", "null"], "description": "Por qué es dudosa, en pocas palabras" }
        },
        "required": ["descripcion", "cantidad", "unidades_por_bulto", "unidades_por_bulto_fuente",
                     "precio_unitario", "bonificacion_porcentaje", "bonificacion_monto",
                     "alicuota_iva", "importe", "rubro_sugerido", "legibilidad", "nota"]
      }
    },
    "total_factura": { "type": ["number", "null"] }
  },
  "required": ["es_factura", "proveedor", "fecha", "tipo_comprobante", "numero",
               "precios_incluyen_iva", "renglones", "total_factura"]
}
```

Antes de implementar, verificar con Context7 qué subconjunto de JSON Schema acepta la versión vigente de la API (uniones con `null`, `enum` numérico). Si alguno no se acepta, se ajusta el esquema y se documenta acá.

## Prompt (instrucción de sistema)

```text
Sos un lector de comprobantes de compra de un almacén de barrio en Argentina.
Tu única tarea es transcribir datos del comprobante al formato pedido. No calculás nada.

Reglas:
1. Copiá la descripción de cada renglón tal como figura, sin corregir ni completar abreviaturas.
2. Los números argentinos usan punto para miles y coma para decimales: "1.234,50" es 1234.50.
3. Si un dato no figura, devolvé null. No lo deduzcas ni lo inventes.
   Excepción: unidades_por_bulto puede leerse de la descripción ("X 24", "x12u", "C/6");
   en ese caso unidades_por_bulto_fuente = "descripcion".
4. precio_unitario es el precio de una unidad facturada tal como figura (si se factura por bulto,
   el precio del bulto). No lo dividas por las unidades del bulto.
5. bonificacion_porcentaje es el % de descuento del renglón; bonificacion_monto, el descuento en pesos
   del renglón. No apliques descuentos generales de la factura (al pie) a los renglones.
6. precios_incluyen_iva: "no" si la factura discrimina IVA (columna o subtotal de IVA aparte,
   típico de factura A); "si" si los precios ya lo incluyen (típico de B y C); "no_se" si dudás.
7. alicuota_iva: solo si figura para ese renglón; si no, null.
8. rubro_sugerido: elegí SIEMPRE uno de la lista dada. Si ninguno encaja con seguridad, "Sin categoría".
9. legibilidad = "dudosa" si algún número o la descripción del renglón no se lee con seguridad
   (borroso, tapado, cortado, escrito a mano). Explicá en "nota" qué no se lee.
10. Ignorá renglones que no son mercadería: subtotales, percepciones, recargos, envases retornables
    y leyendas.
    Sí incluí la mercadería sin cargo o bonificada al 100 % (precio 0 o bonificación 100), como un renglón más.
11. Si la imagen no es un comprobante de compra, es_factura = false y renglones vacío.
```

Mensaje de usuario: `Rubros disponibles: {lista}. Proveedores conocidos: {nombres y CUIT}.` + la imagen o el texto del PDF (`Texto extraído del PDF:\n{texto}`). Pasar los proveedores conocidos ayuda a que el nombre salga igual que la vez anterior.

## Validaciones después de Gemini (en `src/dominio`, no en la función)

Marcan el renglón en **amarillo** con un motivo visible:

- `legibilidad = "dudosa"` → "No se leyó bien: {nota}".
- `|cantidad × precio_unitario × (1 − bonif%) − bonif$ − importe| > 1 %` del importe → "El importe no coincide, revisá cantidad o precio".
- Sin alias para ese proveedor → "Producto nuevo" o "¿Es este producto?" con hasta 3 candidatos por similitud de nombre.
- `unidades_por_bulto_fuente = "no_figura"` en un producto sin alias → "¿Cuántas unidades trae?".
- Rubro "Sin categoría" en un producto nuevo → "Elegí el rubro".
- Suma de importes con diferencia > 2 % contra `total_factura` → aviso en la cabecera (no bloquea).
- `precios_incluyen_iva = "no_se"` → el botón con IVA / sin IVA aparece sin elegir y hay que tocarlo.
- Renglón de regalo (precio 0 o bonificación 100 %) sin renglón pago del mismo producto → "Regalo sin compra" (D17, P15).

Avisos de cabecera (no bloquean, D12): factura repetida (mismo proveedor y número) y factura vieja (productos con una compra de fecha posterior). Por eso el prompt pide el número y la fecha con cuidado.
