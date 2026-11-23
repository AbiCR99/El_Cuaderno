# Spec 004 — Carga por foto y PDF con Gemini, con cola y reintentos
Estado: borrador

## Contexto y objetivo
Sacar una foto o subir un PDF tiene que reemplazar el tipeo de renglones. La lectura nunca puede bloquear a la usuaria: si no hay conexión o la API está saturada, la factura espera en cola y se lee sola.

## Usuarios
Administradora.

## Historias de usuario
- HU-1. Como administradora quiero sacar una foto a la factura y recibir los renglones listos para revisar.
- HU-2. Como administradora quiero subir el PDF que me manda el proveedor por WhatsApp o mail.
- HU-3. Como administradora quiero sacar la foto aunque no haya señal y olvidarme.

## Requisitos funcionales
- RF-1: CUANDO la administradora toca Cámara, EL SISTEMA abre la cámara trasera del sistema.
- RF-2: CUANDO se toma la foto, EL SISTEMA la comprime en el celular a un máximo de 1600 px de lado y alrededor de 200–300 KB (WebP; JPEG si no hay WebP) antes de subirla.
- RF-3: CUANDO se toma la foto o se elige el PDF, EL SISTEMA la guarda primero en el dispositivo y después intenta subirla.
- RF-4: CUANDO se sube un PDF con texto, EL SISTEMA extrae el texto en el celular y envía el texto (no la imagen) para estructurar.
- RF-5: SI el PDF no tiene texto útil (escaneado), ENTONCES EL SISTEMA convierte cada página en imagen y sigue como una foto.
- RF-6: EL SISTEMA llama a Gemini solo desde la Edge Function, con la clave guardada en secretos de Supabase.
- RF-7: EL SISTEMA pide a Gemini una respuesta con el esquema de `docs/gemini-extraccion.md` y la lista de rubros vigente como única opción de rubro.
- RF-8: CUANDO Gemini responde bien, EL SISTEMA guarda la respuesta completa, crea los renglones, empareja proveedor y alias, y deja la factura lista para revisar.
- RF-9: SI no hay conexión, ENTONCES EL SISTEMA deja la factura en la cola local, muestra "Guardada. La leo apenas haya conexión." y la sube sola al volver la conexión, aunque la app esté cerrada si el navegador lo permite.
- RF-10: SI Gemini responde 429, error 5xx o no responde en 60 s, ENTONCES EL SISTEMA reintenta solo con esperas de 1, 2, 5, 15 y 60 minutos, sin intervención de la usuaria.
- RF-11: SI tras 8 intentos sigue fallando, o la imagen no es una factura, ENTONCES EL SISTEMA la marca con error y ofrece "Sacar otra" y "Cargar a mano".
- RF-12: MIENTRAS se lee la factura, EL SISTEMA muestra "Leyendo la factura…" y, pasados 20 s, permite volver a Inicio sin cancelar la lectura.
- RF-13: EL SISTEMA ofrece "Cargar a mano" en todo momento como alternativa.
- RF-14: CUANDO la administradora toca "Agregar otra hoja" antes de enviar, EL SISTEMA agrupa varias fotos en una sola factura. [NECESITA ACLARACIÓN P7]

## Requisitos no funcionales
- La foto nunca se borra del dispositivo hasta que la base confirma que el archivo y la fila existen.
- Subir la misma factura dos veces por reintento no crea dos facturas (id generado en el celular).
- El modelo se cambia con una variable, sin desplegar código.

## Casos límite
- Foto movida u oscura (Gemini marca dudosa).
- Factura con 40 renglones.
- PDF con varias facturas o protegido con contraseña.
- Se cierra la app en medio de la subida.
- Dos facturas leyéndose a la vez.

## Fuera de alcance
- Recorte o enderezado de la foto en la app.
- Detectar facturas repetidas (P8, se decide en la spec 003).

## Criterios de finalización
- Set de prueba de 15–20 facturas reales: comparación de modelos documentada en `docs/gemini-extraccion.md` y modelo elegido.
- Demo en Android con modo avión: foto → cola → conexión → revisión disponible.
- Test de la función con una respuesta 429 simulada.

## Dudas abiertas
- [NECESITA ACLARACIÓN] P7 (varias hojas).
