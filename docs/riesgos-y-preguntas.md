# Riesgos y preguntas abiertas

## Decisiones cerradas (2026-11-23)

| # | Tema | Decisión |
|---|---|---|
| D1 | Precios editados a mano al recalcular | Se pisan, tanto al cambiar el margen de un rubro como al confirmar una factura nueva. Solo "precio fijo" se respeta. |
| D2 | Redondeo del 5 exacto | Hacia arriba: 115 → 120. |
| D3 | PIN de la familia | Uno compartido. |
| D4 | Lenguaje del frontend | TypeScript. |

## Preguntas abiertas (decidir antes de cerrar la spec indicada)

| # | Pregunta | Propuesta | Spec |
|---|---|---|---|
| P1 | ¿Hay un celular compartido en el almacén? Si la familia usa el celular de tu mamá, vería costos. | Menú "Salir" + ingreso por PIN. Si siempre usan sus propios celulares, no hace falta nada. | 001 |
| P2 | ¿Un proyecto de Supabase o dos (dev y prod)? El plan gratuito tiene un límite de proyectos activos y Huddii ya ocupa uno. | Dos proyectos; verificar el límite en tu organización. Con uno solo, el agente nunca aplica migraciones sin tu aprobación. | 001 |
| P3 | El botón "con IVA / sin IVA": ¿"con IVA" significa "los precios de la factura ya incluyen IVA"? | Sí. Etiqueta en pantalla: "Precios con IVA" / "Precios sin IVA (se suma)". | 003 |
| P4 | ¿Dónde se cargan los días de pedido y entrega de cada proveedor? El inicio solo tiene dos botones. | Campos opcionales en la revisión cuando el proveedor es nuevo, y editables desde la ficha del proveedor dentro de Manual. | 003 |
| P5 | ¿Dónde va "Repetir última compra"? | Dentro de Manual, después de elegir el proveedor. | 003 |
| P6 | ¿Aviso en Inicio de facturas para revisar, en cola o con error? Sin él, una factura leída en segundo plano no se ve. | Sí, solo cuando hay algo, encima de los dos botones. | 003/004 |
| P7 | ¿Facturas de más de una hoja? | Botón "Agregar otra hoja" antes de enviar; todas las imágenes van en un solo pedido a Gemini. | 004 |
| P8 | ¿Avisar si una factura parece repetida (mismo proveedor y número)? | Aviso amarillo en la cabecera, no bloquea. | 003 |
| P9 | ¿Descuentos generales de la factura (no por renglón)? El brief solo habla de bonificación por renglón. | Fase 1: se ignoran. Si son frecuentes, se agrega con una spec de cambio. | 003 |
| P10 | Fraccionados: ¿el precio de compra a granel incluye IVA? ¿Y si el granel llega en una factura fotografiada? | La carga de fraccionado pasa por la revisión, que tiene el botón con/sin IVA. Si el granel llega en una factura, el producto se marca como fraccionado una vez y las siguientes compras usan su configuración. Fase 1: solo carga manual. | 007 |
| P11 | Fraccionados por peso: ¿el precio de 100 g se redondea a la decena como el resto? | Sí, se calcula aparte (`costo por 100 g × (1 + margen)`) y se redondea igual, no como precio del kg ÷ 10. | 007 |
| P12 | Cuando el costo baja, el precio sugerido también baja (decisión D1). ¿Está bien bajar precios automáticamente? | Se aplica tal cual (es lo que dice la fórmula) y se muestra "antes $X" para que lo vea. | 003 |
| P14 | Renglones bonificados al 100 % (unidades de regalo): ¿bajan el costo promedio del producto? | Fase 1: el renglón queda amarillo sin precio sugerido; si el mismo producto viene pagado en otro renglón, se suman las unidades gratis a ese renglón. Confirmar. | 002/003 |
| P13 | ¿Desarrollás en la Mac mini con Monterey? | El entorno evita Docker (Supabase local) y usa proyectos en la nube; Node 22 LTS funciona en Monterey. | 001 |

## Riesgos

| Riesgo | Impacto | Mitigación |
|---|---|---|
| **Gemini lee mal precios o unidades por bulto** | Precios de venta mal calculados | Control cruzado con el importe del renglón, renglones dudosos en amarillo, precio siempre editable, set de prueba con facturas reales antes de elegir el modelo, `extraccion_cruda` guardada para auditar. |
| **El emparejamiento por alias falla** (el proveedor cambia la descripción) | Productos duplicados, la revisión se vuelve lenta | Normalización del texto, candidatos por similitud al crear, alias múltiples por producto. **[fase 2/visita]** Herramienta para unir productos duplicados. |
| **La revisión no resulta más rápida que hoy** | Abandono de la app | Medir en la visita: tiempo por factura con y sin la app. Las primeras semanas casi todo será amarillo (catálogo vacío); avisarle que mejora sola. |
| **Recalcular pisa ajustes manuales (D1)** | Precios "redondos" que ella eligió se pierden | La confirmación muestra cuántos productos cambian; el historial permite ver el precio anterior. |
| **Desarrollo a distancia, visitas espaciadas** | Errores en producción sin poder ver el celular | Desplegar la fase 1 antes de la visita en uso real; actualizaciones de la PWA automáticas; registro de errores en una tabla `eventos_error` **[propuesta]**; vos como usuaria `dev` podés ver todas sus facturas. |
| **PIN compartido de 6 dígitos** | Alguien adivina el PIN | Solo da acceso a precios de venta (no costos). Supabase limita intentos de login por IP. Cambiar el PIN es cambiar una contraseña desde el dashboard. |
| **Cola offline** (fotos que nunca suben) | Pérdida de facturas | La foto se guarda en IndexedDB antes de cualquier intento; Background Sync en Chrome Android; aviso visible en Inicio; no se borra del dispositivo hasta que la base confirma. |
| **Límites o cambios de la API de Gemini** | Facturas sin leer | Modelo por variable de entorno, reintentos con backoff, carga manual siempre disponible. |
| **Inflación y redondeo** | Precio de 100 g o productos muy baratos con redondeo grosero | Regla única de decena; precio siempre editable. Revisar en la visita si hace falta otra regla para productos de menos de $100. |
| **Una sola lógica de precios duplicada por error** (SQL o UI) | Precios distintos según la pantalla | Regla en `AGENTS.md`, skill `reglas-precios`, el reviewer busca cuentas fuera de `src/dominio`. |
| **Fecha y hora** | "Hace 3 días" incorrecto, facturas con fecha corrida | Fechas de factura como `date` sin zona; tiempos como `timestamptz`; mostrar siempre en `America/Argentina/Buenos_Aires`. |
