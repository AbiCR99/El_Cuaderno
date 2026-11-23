---
name: ux-almacen
description: Úsala siempre que diseñes, construyas o revises una pantalla, componente, texto o flujo de El Cuaderno, para que respete el principio rector (más rápido que hoy) y el uso real en el mostrador con un Android.
---

# UX para el mostrador

La usuaria atiende sola, con clientes esperando, en un celular Android, muchas veces con una mano. Cada toque extra es una razón para dejar de usar la app.

## Reglas
- **Contar toques.** Para cada flujo, escribí en el plan cuántos toques lleva el camino feliz. Si una tarea sube ese número, se justifica en la spec.
- Botones de al menos 56 px de alto, texto base 18 px, contraste AA. Acción principal abajo, a lo ancho.
- Una acción principal por pantalla. Lo destructivo (descartar, eliminar), en un menú secundario y con confirmación.
- Teclado numérico (`inputmode="decimal"`) en todo campo de número; aceptar "1.234,50".
- Colores con significado fijo: verde = listo, amarillo = necesita un toque, rojo solo para errores. Nunca solo color: siempre un texto ("Producto nuevo").
- Textos en español rioplatense, cortos, sin jerga técnica: "Guardada. La leo apenas haya conexión.", no "Error 429: rate limit".
- Toda espera tiene texto; toda espera de más de 20 s ofrece seguir con otra cosa.
- Todo error dice qué pasó y ofrece una salida.
- Estados obligatorios de cada pantalla: cargando, vacío, error, sin conexión y éxito. Escribilos en el plan.
- Plata con `Intl.NumberFormat('es-AR')`, sin centavos en precios de venta. Fechas relativas con `Intl.RelativeTimeFormat('es-AR')`.
- Nada de "stock" en la fase 1.
- Si la skill `buena-ux` está instalada, aplicala además de esta.

## Cómo verificar
Chrome DevTools MCP en 360 × 740, captura de cada estado. Recorrer el camino feliz contando toques.

## Checklist
- [ ] ¿Cuántos toques lleva el camino feliz? ¿Subió respecto del plan?
- [ ] ¿Hay algún texto técnico o en inglés visible?
- [ ] ¿Están los 5 estados?
- [ ] ¿Se puede usar con el pulgar en 360 px?
