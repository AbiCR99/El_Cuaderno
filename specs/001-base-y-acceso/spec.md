# Spec 001 — Base del proyecto y acceso por rol
Estado: borrador

## Contexto y objetivo
Antes de cualquier funcionalidad hace falta una PWA instalable en Chrome Android, publicada, con tres tipos de acceso: la administradora (ve todo), la desarrolladora (ve todo, a distancia) y la familia (solo precios, con PIN). Sin esta base no se puede probar nada en el celular real.

## Usuarios
Administradora, desarrolladora, familia.

## Historias de usuario
- HU-1. Como administradora quiero abrir la app desde un ícono y entrar directo, sin escribir contraseña cada vez.
- HU-2. Como familiar quiero entrar solo con un PIN para ver precios.
- HU-3. Como desarrolladora quiero que ningún familiar pueda ver costos ni modificar nada, aunque sepa usar herramientas técnicas.

## Requisitos funcionales
- RF-1: EL SISTEMA se puede instalar desde Chrome Android como app con ícono propio y nombre "El Cuaderno".
- RF-2: CUANDO se abre la app por primera vez en un dispositivo, EL SISTEMA muestra dos opciones: "Soy la administradora" y "Familia".
- RF-3: CUANDO la administradora ingresa email y contraseña válidos, EL SISTEMA abre Inicio y mantiene la sesión abierta en ese dispositivo hasta que ella salga.
- RF-4: CUANDO un familiar ingresa el PIN correcto de 6 dígitos, EL SISTEMA abre Precios en modo solo lectura y mantiene la sesión.
- RF-5: SI el PIN o la contraseña son incorrectos, ENTONCES EL SISTEMA muestra "PIN incorrecto" / "Email o contraseña incorrectos" sin borrar lo escrito salvo el PIN.
- RF-6: SI no hay conexión en el primer ingreso, ENTONCES EL SISTEMA muestra "Necesitás internet la primera vez".
- RF-7: MIENTRAS la sesión es de rol `familia`, EL SISTEMA no muestra Inicio, Subir productos, costos, márgenes, proveedores ni controles de edición.
- RF-8: EL SISTEMA rechaza en la base cualquier lectura de tablas de negocio y cualquier escritura hecha con una sesión `familia` o sin sesión.
- RF-9: CUANDO se publica una versión nueva, EL SISTEMA la aplica sola la próxima vez que se abre la app, sin pasos de la usuaria.
- RF-10: CUANDO alguien toca "Salir" en el menú, EL SISTEMA cierra la sesión y vuelve a la pantalla de RF-2. [NECESITA ACLARACIÓN P1]

## Requisitos no funcionales
- La pantalla de ingreso carga en menos de 3 s en 4G.
- Lighthouse PWA "instalable" en verde.
- Secretos solo en variables de entorno de Vercel (clave pública) y de Supabase (todo lo demás).

## Casos límite
- La sesión expira o se revoca: volver a la pantalla de ingreso sin perder la cola local.
- Un usuario de Auth sin fila en `perfiles`: no ve nada y se muestra "Tu usuario no tiene permisos".
- Cambio del PIN desde el dashboard: los familiares con sesión abierta siguen hasta que la sesión expire (aceptado).

## Fuera de alcance
- Alta de usuarios desde la app (se hacen desde el dashboard de Supabase).
- Recuperar contraseña desde la app.

## Criterios de finalización
- App instalada en un Android real, ingreso de los tres roles probado.
- Script de verificación de RLS: con sesión `familia`, `select` sobre cada tabla de negocio devuelve 0 filas y todo `insert/update/delete` falla.

## Dudas abiertas
- [NECESITA ACLARACIÓN] P1 (celular compartido) y P2 (uno o dos proyectos de Supabase) en `docs/riesgos-y-preguntas.md`.
