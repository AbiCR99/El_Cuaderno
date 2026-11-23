---
name: "buena-ux"
description: "Úsala siempre que diseñes, construyas, revises o pidas a una IA cualquier pantalla, componente, formulario o flujo de una app o web: estados de carga, error, vacío, parcial y éxito, mensajes, validación, layout y leyes de UX."
---

# Build for Good UX

Actúa como diseñador/a UI/UX senior. Las herramientas de IA generan por defecto solo el "camino feliz": sin estados de error, con fallos silenciosos y con la misma estética genérica de siempre. Esta skill existe para que eso no pase. Fuentes base: serie "Build for Good UX" (Katherine Gilligan), Nielsen Norman Group, Baymard Institute, Laws of UX, Apple HIG y guías de accesibilidad (WCAG / ARIA).

## 1. Regla madre: toda pantalla tiene 5 estados

Antes de dar por terminada una pantalla o sección, define y construye **los cinco**:

| Estado | Pregunta que responde | Si falta, el usuario piensa… |
|---|---|---|
| Carga | "¿Está pasando algo?" | "Se colgó" |
| Éxito | "¿Funcionó?" | "¿Lo aprieto de nuevo?" |
| Error | "¿Qué falló y qué hago?" | "Está roto" |
| Vacío | "¿Por qué no hay nada y cómo empiezo?" | "No cargó" |
| Parcial | "¿Puedo usar lo que sí cargó?" | "Se cayó todo" |

El principio de fondo es la heurística n.º 1 de Nielsen, **visibilidad del estado del sistema**: informar siempre al usuario de lo que pasa con una respuesta adecuada y en un tiempo razonable.

## 2. Estados de carga

### Umbrales de tiempo (Nielsen, basados en percepción humana; no cambian con el hardware)
- **< 0,1 s**: se siente instantáneo. Ningún indicador.
- **< 1 s**: no muestres loader; mostrar uno que "parpadea" hace que se sienta *más* lento. Muestra el resultado directo.
- **1–10 s**: el usuario tolera la espera si ve actividad. Animación en bucle (spinner o skeleton).
- **> 10 s**: la atención se va. Hace falta un indicador **determinado** (barra de %, pasos, tiempo estimado).

### Anti-parpadeo (implementación)
- No muestres el loader hasta que pasen **~300 ms**.
- Una vez visible, mantenlo un **mínimo de ~400–500 ms** para que no aparezca y desaparezca de golpe.
- En React: hook tipo `useSpinDelay` (delay + minDuration). En CSS puro: `animation-delay` en la opacidad del spinner.

### Qué loader va dónde (no son intercambiables)
| Situación | Usa | Por qué |
|---|---|---|
| Página o sección grande con contenido estructurado (feeds, listas, dashboards) | **Skeleton screen** | Muestra el layout antes que los datos; el cerebro procesa la estructura mientras espera |
| Acción chica y acotada (botón, card, módulo que se refresca) | **Spinner inline** | Señal mínima de "estamos en eso" en el lugar exacto de la acción |
| Duración conocida o medible (subida/descarga de archivos, instalación, exportación) | **Barra de progreso** | El usuario necesita saber cuánto falta; un spinner en una subida parece trabado |
| Acción de alta probabilidad de éxito y reversible (like, toggle, mover card, marcar leído) | **Optimistic UI** (sin loader) | Se siente instantáneo |
| Proceso largo de varias etapas | **Indicador de pasos** | Da progreso real y control |
| Video | Spinner/buffer propio del reproductor | Es la convención esperada |

### Reglas de skeleton
- Debe parecerse en tamaño y posición al contenido real (si no, genera saltos de layout y confusión).
- Animación lenta y suave (pulse o shimmer lento), nunca rápida.
- Solo para contenido dinámico, nunca para contenido estático.
- No pongas varios spinners por toda la pantalla: si las secciones cargan juntas, un solo indicador; si cargan por separado, skeleton por sección.

### Reglas de spinner según duración
- 1–5 s: spinner solo sirve.
- Más de ~5 s: agrega texto ("Guardando…").
- Mejor aún: texto que cambia con pasos **reales** ("Conectando con tu cuenta… Procesando pago…"). Mostrar el trabajo que el sistema está haciendo aumenta la paciencia y el valor percibido ("labor illusion", Buell & Norton, 2011). **No inventes pasos falsos**: describe lo que realmente ocurre.
- Más de 10 s: los loops dejan de funcionar y empiezan a frustrar. Pasa a barra de progreso o pasos.
- La barra de progreso nunca retrocede y no debe "mentir" (90 % en 5 s y el último 10 % en 5 minutos genera desconfianza).
- Si algo falla, muestra el error **apenas** lo sepas. Nunca hagas esperar 20 s para decir "no funcionó". Pon timeouts.

### Optimistic UI
- Actualiza la interfaz al instante, envía la petición en segundo plano.
- Si falla: revierte el estado **y avisa** (ej.: "No se pudo guardar el like. Reintentar").
- Nunca para pagos, borrados irreversibles ni acciones con consecuencias legales o de dinero.
- React 19: `useOptimistic` + transiciones (rollback automático). Con React Query/SWR: `onMutate` + rollback en `onError`.

### Accesibilidad de la carga
- Contenedor que carga: `aria-busy="true"` mientras carga; quitarlo al terminar.
- Una región `role="status"` / `aria-live="polite"` con texto oculto "Cargando…" (una por contenedor, no una por cada barra gris).
- Formas del skeleton con `aria-hidden="true"`.
- Respeta `prefers-reduced-motion`:

```css
.skeleton { background: var(--skeleton-bg); }
@media (prefers-reduced-motion: no-preference) {
  .skeleton { animation: pulse 2s ease-in-out infinite; }
}
```

## 3. Estados de error

### Contenido: un buen mensaje hace tres cosas
1. **Qué pasó** ("Tu pago no se procesó").
2. **Por qué** ("La tarjeta fue rechazada").
3. **Qué hacer ahora**, con acción concreta ("Revisá los datos o usá otro medio de pago" + botón).

### Reglas
- **Nunca** muestres errores crudos del backend, stack traces, códigos SQL o nombres de tablas: el usuario no los entiende y exponen detalles internos (riesgo de seguridad). Loguéalos para vos; al usuario, lenguaje humano.
- **"Algo salió mal" no alcanza**, sobre todo en acciones críticas. En un pago, el usuario necesita saber si se cobró o no.
- Lenguaje claro, preciso y sin culpar al usuario ("No encontramos esa dirección", no "Ingresaste una dirección inválida").
- No uses solo color para señalar un error (daltonismo): color + ícono + texto.
- Conserva lo que el usuario escribió. Nunca borres un formulario por un error.
- **El peor error es el silencioso**: botón que no hace nada, pantalla que no cambia. Todo `catch` debe terminar en algo visible para el usuario.
- Acciones de dinero: deshabilita el botón *mientras se envía* (evita doble envío) y usa idempotencia en el backend para no cobrar dos veces.

### Ubicación del error
| Tipo | Cuándo | Cuándo NO |
|---|---|---|
| **Inline** (junto al campo o botón) | La opción por defecto. Cuanto más cerca del problema, mejor. Campos de formulario, "Reintentar" al lado del botón que falló | — |
| **Toast** (aparece y se va solo) | Avisos transitorios, no críticos: "Sin conexión, reintentando…", "Copiado" | Errores importantes o que el usuario debe corregir. Prueba: si mira para otro lado y se lo pierde, ¿está bien? Si la respuesta es no, no es un toast |
| **Modal** (bloquea la pantalla) | Solo si el usuario no puede seguir sin resolverlo: pago fallido, falta de permisos, sesión vencida. **Siempre con una salida**: botón para actualizar el pago, pedir acceso, volver a iniciar sesión | Errores de un campo o de una sección |
| **Banner** persistente | Problemas que afectan a toda la app (sin conexión, mantenimiento) | Errores locales |

Accesibilidad de toasts: usa una región `aria-live` siempre en el mismo contenedor (`polite` para avisos, `assertive` solo para errores urgentes), deja que se puedan cerrar con teclado, no robes el foco y no pongas en un toast acciones que no existan en otro lugar de la app.

## 4. Formularios

1. **Marca los campos obligatorios** (o los opcionales, si son minoría). El usuario nunca debe adivinar qué falta.
2. **Validación inline al salir del campo (on blur), no mientras escribe.** Mostrar "email inválido" cuando recién tipeaste dos letras es "validación prematura" y frustra. Patrón recomendado: *reward early, punish late* → el primer error aparece al salir del campo; una vez marcado, se limpia **en cuanto** se corrige, mientras escribe.
3. **Contador de caracteres** visible si hay límite ("38/280").
4. **Precarga lo que ya sabés**: email del usuario logueado, país detectado, último domicilio.
5. **Requisitos de contraseña visibles mientras escribe**, con check en los que ya cumple.
6. **Sé flexible con el formato**: acepta teléfonos con guiones, espacios o paréntesis, tarjetas con o sin espacios, y normaliza en el backend.
7. **Botón de enviar**: la tendencia del material es "gris hasta que esté completo", pero la investigación de accesibilidad recomienda lo contrario. **Por defecto, deja el botón habilitado**; al enviar con errores, muestra un resumen arriba, marca cada campo y lleva el foco al primero. Si igualmente lo deshabilitás, explicá al lado qué falta y usá `aria-disabled` en vez de `disabled` para que siga siendo enfocable.
8. Formularios largos (más de ~7 campos): divídelos en pasos con indicador de progreso. Hay casos publicados de grandes mejoras en conversión (Venture Harbour), pero son casos de estudio, no estudios controlados: **testéalo**.
9. Una sola columna, labels visibles arriba del campo (el placeholder no es label), `type`, `inputmode` y `autocomplete` correctos.

## 5. Estados vacíos

Suele ser lo **primero** que ve un usuario nuevo: es la primera impresión.

Un buen estado vacío: **dice por qué está vacío, muestra qué hacer y no parece roto.**

| Tipo | Qué hacer |
|---|---|
| **Primer uso** (dashboard nuevo) | Explicá para qué sirve la sección + CTA principal ("Creá tu primer proyecto"). Opcional: guía paso a paso o checklist de onboarding |
| **Cada sección sin contenido** | Nunca en blanco: qué va acá y cómo empezar a usarla |
| **Sin resultados de búsqueda** | Mostrá el término buscado, sugerí correcciones ("¿Quisiste decir *purple shoes*?" con link), filtros a quitar o búsquedas populares |
| **Vaciado por el usuario como meta** (inbox cero, tareas hechas) | Celebralo: mensaje positivo, ilustración o animación sutil |
| **Vacío por error o permisos** | No es un vacío: tratalo como error (qué pasó + acción) |

Solo ilustración si hay espacio; si no, texto y CTA.

## 6. Estados parciales y degradación elegante

Una página casi nunca viene de una sola fuente: el perfil, el feed y los gráficos cargan a velocidades distintas y pueden fallar por separado.

- **Cada sección es responsable de sus propios datos, su propia carga y su propio error** (con su propio botón "Reintentar"). El resto de la página sigue usable.
- No bloquees toda la pantalla hasta que cargue el último componente. Muestra lo que ya está.
- Si una sección falla, nunca tires abajo la página entera con un error de pantalla completa.
- **Stale-while-revalidate**: mostrá datos en caché (marcados como desactualizados si importa) mientras traés los nuevos y luego reemplazalos sin saltos.
- Implementación React: un `<ErrorBoundary>` + `<Suspense>` por región lógica (no uno solo para toda la app). Secciones críticas vs. no críticas: si fallan las reseñas, el producto sigue a la venta.
- Al diseñar, dibujá explícitamente cómo se ve la pantalla cuando **algunas** partes funcionan y otras no.

## 7. Estados de éxito

Toda acción necesita confirmación. Sin ella aparece la duda ("¿se cobró? ¿lo aprieto otra vez?"), que es la peor sensación para el usuario.

- **Proporcional al peso de la acción**:
  - Micro (like, mover card a "Hecho", toggle): la acción misma es la confirmación. La card se queda en "Hecho" y listo.
  - Media (guardar, enviar): check inline, cambio de texto del botón ("Guardado ✓") o toast breve.
  - Alta (pago, reserva, envío de pedido): pantalla o bloque de confirmación con resumen, número de referencia y próximo paso.
  - Hitos (primer proyecto, meta alcanzada): celebración (confeti, animación). **Con moderación**: si todo celebra, nada celebra.
- Las acciones destructivas: confirmación + opción de deshacer cuando sea posible.

## 8. Ley de Jakob: no reinventes lo que ya funciona

Los usuarios pasan la mayor parte de su tiempo en *otros* sitios, por eso esperan que el tuyo funcione igual que los que ya conocen. Lo predecible le permite al usuario enfocarse en lo que importa (comprar, completar la tarea), no en encontrar el carrito.

- **Desktop/web**: carrito y cuenta arriba a la derecha; logo arriba a la izquierda que lleva al inicio; búsqueda arriba.
- **Mobile**: navegación principal y carrito **abajo** (tab bar). El pulgar vive abajo; arriba a la derecha es de las zonas más difíciles de alcanzar con una mano (Steven Hoober: ~49 % usa el teléfono con una sola mano). Acciones primarias en la zona del pulgar y con targets grandes (Ley de Fitts).
- **Idiomas RTL** (árabe, hebreo): todo se espeja; el carrito pasa arriba a la izquierda. Usá propiedades lógicas de CSS (`margin-inline-start`, `inset-inline-end`) para soportarlo sin reescribir.
- Respetá las convenciones de cada plataforma (Apple HIG, Material Design).
- Dónde **sí** innovar: identidad visual, tono, ilustración, microinteracciones, contenido. Dónde **no**: navegación, carrito, login, búsqueda, formularios, checkout, iconografía estándar.
- Si de verdad hay que cambiar un patrón conocido, dale al usuario un período de transición o la opción de volver a la versión anterior.

## 9. Ley de Hick y divulgación progresiva

**Ley de Hick**: el tiempo para decidir crece con la cantidad y la complejidad de las opciones. (El famoso estudio de las mermeladas, 24 vs. 6 frascos, es buena ilustración pero no se replicó con solidez: un meta-análisis halló un efecto promedio cercano a cero, muy dependiente del contexto. Usalo como ejemplo, no como prueba.)

- Una acción principal clara por pantalla (Google: buscar).
- Menús largos → pocas opciones visibles + búsqueda y filtros.
- Recomendaciones y curaduría en lugar de catálogos enteros (según Netflix, alrededor del 80 % de lo que se mira llega por recomendaciones).
- No se trata de quitar funciones, sino de que las opciones sean manejables.

**Divulgación progresiva**: mostrá lo que el usuario necesita *ahora*; el resto, un nivel más abajo. Nada se elimina, solo no se tira todo de golpe. Como un GPS que te avisa giro por giro en vez de leerte los 35 de entrada.
- Buen ejemplo: Notion (el menú de bloques aparece al tipear `/` y se filtra mientras escribís; la IA aparece cuando la pedís).
- Mal ejemplo: dropdowns gigantes donde la opción que buscás está escondida al fondo.
- Patrones: acordeones, "Opciones avanzadas", menús contextuales, comandos con `/`, formularios por pasos, onboarding gradual.
- **Límite**: no escondas tanto las funciones clave que haga falta un tutorial para encontrarlas. Lo que aparece en la primera vista le dice al usuario qué es importante.
- Pregunta de control: "¿Todo lo que hay en esta pantalla hace falta *en este momento*?"

## 10. Detalles tipográficos

Evitá palabras huérfanas (la última palabra sola en una línea):

```css
h1, h2, h3, h4 { text-wrap: balance; } /* títulos cortos: líneas parejas */
p, li, blockquote { text-wrap: pretty; } /* texto largo: sin huérfanas */
```

- `balance` solo para bloques cortos (títulos, pocas líneas): es costoso y los navegadores lo limitan a pocas líneas.
- `pretty` no tiene soporte universal todavía: es **mejora progresiva**, el texto se ve bien igual donde no funcione.
- Ancho de lectura cómodo: `max-width: 60–75ch` para párrafos.

## 11. Que no parezca "vibe-coded"

Señales típicas de app hecha con IA sin dirección: gradientes violetas, emojis decorativos al azar, UI genérica igual a todas.

1. **Referencias**: buscá inspiración real (Mobbin para patrones de apps, 21st.dev para componentes), hacé capturas y pasáselas a la herramienta. Google Stitch sirve para explorar conceptos.
2. **Prompts específicos**: detalles visuales, referencias y **qué evitar**. Ej.: "UI limpia y minimalista estilo Notion, mucho espacio en blanco, divisores gris claro, acentos de color suaves; sin gradientes, sin emojis, sin sombras pesadas".
3. **Guía de estilo fija**: un prompt maestro o archivo de tokens con colores, tipografías, espaciado, radios y sombras, que se reutiliza en cada pedido.
4. **Control total**: diseñá los wireframes en Figma y pasalos a código con el MCP de Figma.
5. Si está disponible, combiná con la skill `frontend-design` para la dirección estética.

## Bloque para pegar en prompts a herramientas de IA

```
Para cada pantalla y sección implementa los 5 estados: carga, éxito, error, vacío y parcial.
- Carga: nada si tarda < 1 s (delay ~300 ms y mínimo ~500 ms visible); skeleton para secciones; spinner inline para acciones; barra de progreso si dura > 10 s o es medible.
- Error: mensaje humano con qué pasó, por qué y qué hacer + acción. Nunca errores crudos del backend ni fallos silenciosos. Inline por defecto; modal solo si bloquea, siempre con salida; nunca un error importante solo en un toast.
- Vacío: explica por qué está vacío y ofrece un CTA.
- Parcial: cada sección carga y falla por separado, con su propio reintentar.
- Éxito: confirmación proporcional a la acción.
- Formularios: obligatorios marcados, validación al salir del campo, contador si hay límite, requisitos de contraseña en vivo, formato flexible.
- Accesibilidad: aria-busy, aria-live, foco al primer error, prefers-reduced-motion, nunca solo color.
```

## Criterios de aceptación en EARS (para specs SDD)

Si el proyecto usa la skill `sdd`, agrega a cada spec con UI requisitos como estos:
- RF-x: MIENTRAS <sección> carga datos por más de 300 ms, EL SISTEMA muestra un skeleton con su forma.
- RF-x: SI la carga de <sección> falla, ENTONCES EL SISTEMA muestra un mensaje con la causa y un botón "Reintentar" en esa sección, sin afectar al resto de la página.
- RF-x: SI <lista> no tiene elementos, ENTONCES EL SISTEMA muestra qué contiene esa sección y un botón para crear el primero.
- RF-x: CUANDO el usuario sale de un campo con un valor inválido, EL SISTEMA muestra junto al campo qué está mal y cómo corregirlo.
- RF-x: CUANDO <acción> se completa, EL SISTEMA muestra una confirmación visible antes de 1 s.

## Checklist de revisión

- [ ] ¿Cada pantalla y sección tiene sus 5 estados (carga, éxito, error, vacío, parcial)?
- [ ] ¿Algún loader que parpadea en cargas de menos de 1 s?
- [ ] ¿El tipo de loader corresponde a la situación (skeleton / spinner / barra / optimista)?
- [ ] ¿Algo con loop de más de 10 s sin progreso ni texto?
- [ ] ¿Hay timeouts? ¿Los errores aparecen apenas se conocen?
- [ ] ¿Algún `catch` vacío o error que solo va a la consola (fallo silencioso)?
- [ ] ¿Algún mensaje que exponga detalles del backend o que solo diga "Algo salió mal"?
- [ ] ¿Cada error dice qué pasó, por qué y qué hacer, y está cerca del problema?
- [ ] ¿Algún error importante que solo aparece en un toast? ¿Algún modal sin salida?
- [ ] ¿La validación espera a que el usuario salga del campo y se limpia al corregir?
- [ ] ¿Obligatorios marcados, contador de caracteres, requisitos de contraseña en vivo, formato flexible, precarga?
- [ ] ¿Un botón deshabilitado sin explicación de qué falta?
- [ ] ¿Los pagos evitan el doble envío y el doble cobro?
- [ ] ¿Algún estado vacío en blanco o sin CTA? ¿Búsquedas sin resultado sin sugerencias?
- [ ] ¿Una sección que falla tira abajo toda la página?
- [ ] ¿Toda acción tiene confirmación proporcional? ¿Celebraciones exageradas?
- [ ] ¿Carrito, navegación y login están donde el usuario los espera en esa plataforma (desktop, mobile, RTL)?
- [ ] ¿Acciones principales en la zona del pulgar en mobile?
- [ ] ¿Demasiadas opciones a la vez? ¿Alguna función clave escondida de más?
- [ ] ¿`text-wrap: balance` en títulos y `pretty` en párrafos?
- [ ] ¿aria-busy / aria-live / foco al primer error / `prefers-reduced-motion` / nada comunicado solo con color?
- [ ] ¿Gradientes violetas, emojis decorativos o UI genérica sin guía de estilo?
- [ ] ¿Probado en vista móvil (375 px) y con red lenta (DevTools → throttling) para ver los estados de carga y error de verdad?

## Al terminar

Indica qué puntos del checklist comprobaste y cómo (por ejemplo: simulando red lenta, forzando un error del servidor, vaciando los datos, probando en 375 px). Lista los estados que quedaron pendientes o las decisiones de UX que el usuario debe revisar.