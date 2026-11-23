# Spec 001 — Base del proyecto y acceso
Estado: borrador

## Contexto y objetivo
Antes de cualquier funcionalidad hace falta una PWA instalable en Chrome Android, publicada, a la que entren la administradora, la desarrolladora y la familia. Los tres tienen los mismos permisos (D5); lo que cambia es cómo entran: la administradora y la desarrolladora con email y contraseña, y la familia con un PIN compartido (D3). Sin esta base no se puede probar nada en el celular real.

## Usuarios
Administradora, desarrolladora, familia.

## Historias de usuario
- HU-1. Como administradora quiero abrir la app desde un ícono y entrar directo, sin escribir contraseña cada vez.
- HU-2. Como familiar quiero entrar solo con un PIN para calcular y consultar precios.
- HU-3. Como desarrolladora quiero que nadie sin cuenta pueda ver ni modificar datos del almacén.

## Requisitos funcionales
- RF-1: EL SISTEMA se puede instalar desde Chrome Android como app con ícono propio y nombre "El Cuaderno".
- RF-2: CUANDO se abre la app por primera vez en un dispositivo, EL SISTEMA muestra dos opciones: "Soy la administradora" y "Familia".
- RF-3: CUANDO la administradora o la desarrolladora ingresan email y contraseña válidos, EL SISTEMA abre Inicio y mantiene la sesión abierta en ese dispositivo.
- RF-4: CUANDO un familiar ingresa el PIN correcto de 6 dígitos, EL SISTEMA abre Inicio y mantiene la sesión abierta en ese dispositivo.
- RF-5: SI el PIN o la contraseña son incorrectos, ENTONCES EL SISTEMA muestra "PIN incorrecto" / "Email o contraseña incorrectos" y borra solo el PIN.
- RF-6: SI no hay conexión en el primer ingreso, ENTONCES EL SISTEMA muestra "Necesitás internet la primera vez".
- RF-7: EL SISTEMA muestra las mismas pantallas y permite las mismas acciones a los tres roles.
- RF-8: EL SISTEMA rechaza en la base cualquier lectura o escritura hecha sin sesión o con un usuario que no tiene perfil.
- RF-9: CUANDO se publica una versión nueva, EL SISTEMA la aplica sola la próxima vez que se abre la app, sin pasos de la usuaria.
- RF-10: EL SISTEMA registra qué usuario (administradora, desarrolladora o familia) hizo cada carga y cada cambio de precio.

## Requisitos no funcionales
- La pantalla de ingreso carga en menos de 3 s en 4G.
- Lighthouse PWA "instalable" en verde.
- Secretos solo en variables de entorno de Vercel (clave publicable) y de Supabase (todo lo demás).
- Desarrollo con Node 22 LTS y Supabase CLI por `npx`, sin Docker (D16).

## Casos límite
- La sesión expira o se revoca: volver a la pantalla de ingreso sin perder la cola local.
- Un usuario de Auth sin fila en `perfiles`: no ve nada y se muestra "Tu usuario no tiene permisos".
- Cambio del PIN desde el dashboard: los familiares con sesión abierta siguen hasta que la sesión expire (aceptado).

## Fuera de alcance
- Alta de usuarios desde la app (se hacen desde el dashboard de Supabase).
- Recuperar contraseña desde la app.
- Permisos distintos por rol.

## Criterios de finalización
- App instalada en un Android real; ingreso con email y con PIN probado.
- Script de verificación de RLS: sin sesión, toda lectura y escritura falla; con sesión de cada rol, funciona.
- Proyectos `dev` y `prod` creados en la cuenta nueva de Supabase (D6).

## Dudas abiertas
- (ninguna)
