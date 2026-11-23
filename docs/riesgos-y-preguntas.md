# Riesgos y preguntas abiertas

## Decisiones cerradas

| # | Fecha | Tema | Decisión |
|---|---|---|---|
| D1 | 2026-11-23 | Precios editados a mano al recalcular | Se pisan, al cambiar el margen de un rubro y al confirmar una factura nueva. Solo "precio fijo" se respeta. |
| D2 | 2026-11-23 | Redondeo del 5 exacto | Hacia arriba: 115 → 120. |
| D3 | 2026-11-23 | PIN de la familia | Uno compartido. |
| D4 | 2026-11-23 | Lenguaje del frontend | TypeScript. |
| D5 | 2026-11-23 | Permisos de la familia (antes P1) | **Exactamente los mismos que la administradora**: ven costos y márgenes, cargan y confirman facturas, editan precios, rubros y proveedores. La usan para calcular precios cuando llega un proveedor; normalmente lo hace la dueña. La hermana y la cuñada atienden a la tarde todos los días menos los martes. Reemplaza lo que decía el brief sobre la familia. |
| D6 | 2026-11-23 | Supabase (antes P2) | Plan gratis con una cuenta de email nueva solo para el proyecto. Dos proyectos: `dev` y `prod`. |
| D7 | 2026-11-23 | Botón con IVA / sin IVA (antes P3) | "Con IVA": los precios ya lo incluyen. "Sin IVA": se suma la alícuota (21 % salvo que el renglón diga otra). |
| D8 | 2026-11-23 | Días de los proveedores (antes P4) | Botón **Proveedores** dentro de Subir productos, junto a Porcentajes: lista con nombre y días de pedido y entrega. La revisión crea el proveedor nuevo solo con el nombre. |
| D9 | 2026-11-23 | "Repetir última compra" (antes P5) | Dentro de Manual. Es raro que se use: prioridad baja en el backlog. |
| D10 | 2026-11-23 | Avisos en Inicio (antes P6) | Sí, solo cuando hay algo, encima de los dos botones. |
| D11 | 2026-11-23 | Facturas de varias hojas (antes P7) | No existen. Una foto = una factura. |
| D12 | 2026-11-23 | Factura repetida o vieja (antes P8) | Aviso amarillo en la cabecera, no bloquea: (a) mismo proveedor y mismo número ya cargado; (b) algún producto de la factura tiene una compra confirmada con **fecha de factura** posterior. Se compara contra la fecha de la última factura del producto, no contra ediciones a mano. Si se confirma igual una factura vieja, la compra queda en el historial pero **no cambia el costo ni el precio** de esos productos. |
| D13 | 2026-11-23 | Descuentos generales al pie de la factura (antes P9) | Se ignoran. El costo puede quedar un poco más alto que lo pagado. |
| D14 | 2026-11-23 | Fraccionados e IVA (antes P10) | La carga de fraccionado tiene el botón con/sin IVA (pasa por la revisión). Si el granel llega en una factura, el producto se marca fraccionado una vez y las compras siguientes usan esa configuración. Fase 1: solo carga manual. |
| D15 | 2026-11-23 | Precio de 100 g (antes P11) | Se calcula aparte y se redondea a la decena; como todo precio, es una sugerencia editable. |
| D16 | 2026-11-23 | Desarrollo en Mac mini con Monterey (antes P13) | Node 22 LTS (Node 24 pide macOS 13.5 o más). Supabase CLI con `npx supabase`, sin Docker; base local no, se usa el proyecto `dev`. |
| D17 | 2026-11-23 | Unidades de regalo (antes P14) | Un renglón a $0 o con 100 % de bonificación ("llevás 12, pagás 10") suma sus unidades al mismo producto en la misma factura y baja su costo unitario. |

## Preguntas abiertas

| # | Pregunta | Propuesta | Spec |
|---|---|---|---|
| P12 | Cuando el costo baja, el precio sugerido también baja (D1). ¿Está bien bajar precios automáticamente? | Se aplica tal cual (es lo que dice la fórmula) y la revisión muestra "antes $X" para que se vea. | 003 |
| P15 | Regalo de un producto que no viene pago en la misma factura (una muestra gratis). | Renglón amarillo "Regalo sin compra": guarda la compra pero no cambia costo; si el producto es nuevo, pide el precio a mano. | 003 |
| P16 | Subir productos pasa a tener 5 botones (Cámara, Subir PDF, Manual, Porcentajes, Proveedores). | Los tres de carga grandes, uno debajo del otro; Porcentajes y Proveedores más chicos, lado a lado abajo. | 003/008 |

## Riesgos

| Riesgo | Impacto | Mitigación |
|---|---|---|
| **Gemini lee mal precios o unidades por bulto** | Precios de venta mal calculados | Control cruzado con el importe del renglón, renglones dudosos en amarillo, precio siempre editable, set de prueba con facturas reales antes de elegir modelo, `extraccion_cruda` guardada. |
| **El emparejamiento por alias falla** (el proveedor cambia la descripción) | Productos duplicados, revisión lenta | Normalización del texto, candidatos por similitud al crear, varios alias por producto. **[fase 2/visita]** Herramienta para unir duplicados. |
| **Varias personas cargando** (D5) | Dos personas confirman la misma factura, o una carga una vieja | Confirmación idempotente por id de factura, aviso de factura repetida y vieja (D12), historial con quién hizo cada cambio. |
| **PIN compartido de 6 dígitos con permisos completos** (D3 + D5) | Quien adivine el PIN puede cambiar precios | Supabase limita intentos de login; todo cambio queda en el historial; cambiar el PIN es cambiar una contraseña desde el dashboard. No compartir el PIN fuera de la familia. |
| **La revisión no resulta más rápida que hoy** | Abandono de la app | Medir en la visita el tiempo por factura. Las primeras semanas casi todo será amarillo (catálogo vacío); avisarles que mejora sola. |
| **Recalcular pisa ajustes manuales (D1)** | Precios "redondos" elegidos a mano se pierden | La confirmación dice cuántos productos cambian; el historial muestra el precio anterior. |
| **Plan gratis de Supabase** | El proyecto se pausa tras una semana sin actividad; sin backups automáticos; 1 GB de archivos y 500 MB de base | `prod` se usa a diario, así que no se pausa; `dev` sí puede pausarse (se reactiva desde el dashboard). Backup semanal con `npx supabase db dump` guardado fuera de Supabase. Fotos de ~250 KB × ~170 por mes ≈ 45 MB/mes: el GB alcanza para unos 2 años; después se borran fotos viejas ya confirmadas. |
| **Desarrollo a distancia, visitas espaciadas** | Errores en producción sin ver el celular | Fase 1 en uso real antes de la visita; actualizaciones automáticas de la PWA; tabla `eventos_error` **[propuesta]**; como `dev` ves todas las facturas. |
| **Node 22 termina su soporte en abril de 2027** | Dependencias que dejen de soportarlo | Revisar antes de esa fecha; si cambiás de computadora, pasar a Node 24. |
| **Cola offline** (fotos que nunca suben) | Pérdida de facturas | Foto en IndexedDB antes de cualquier intento, Background Sync, aviso en Inicio, no se borra hasta que la base confirma. |
| **Límites o cambios de la API de Gemini** | Facturas sin leer | Modelo por variable de entorno, reintentos con backoff, carga manual siempre disponible. |
| **Redondeo en productos baratos** | Precio de 100 g o productos de menos de $100 con redondeo grosero | Regla única de decena; precio siempre editable (D15). Revisar en la visita. |
| **Lógica de precios duplicada por error** (SQL o UI) | Precios distintos según la pantalla | Regla en `AGENTS.md`, skill `reglas-precios`, el reviewer busca cuentas fuera de `src/dominio`. |
| **Fecha y hora** | "Hace 3 días" incorrecto, aviso de factura vieja equivocado | Fechas de factura como `date` sin zona; tiempos como `timestamptz`; mostrar siempre en `America/Argentina/Buenos_Aires`. |
