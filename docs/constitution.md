# Constitución — El Cuaderno

Principios innegociables. Toda spec, plan y tarea debe cumplirlos.

1. **Más rápido que hoy**: ninguna función le suma pasos a la usuaria. Si algo agrega un toque, se justifica en la spec o no entra.
2. **La spec manda**: nada se implementa si no está en la spec activa. Si falta una decisión, se para y se pregunta. No se inventan requisitos.
3. **Una sola lógica de dominio**: costos, IVA, bultos, márgenes y redondeo viven en `src/dominio/` como funciones puras (sin React ni Supabase), con tests en Vitest.
4. **Tests como puerta**: prohibido avanzar con `npm run test` o `npm run typecheck` en rojo.
5. **Los datos de la usuaria son sagrados**: nunca se pierde una factura (cola local + cola en servidor), todo cambio de costo o precio deja historial, y las migraciones no borran datos.
6. **Seguridad**: RLS activa en todas las tablas; solo usuarios con perfil acceden (sin sesión, nada); ningún secreto llega al cliente.
7. **Idioma**: interfaz y documentación en español rioplatense; identificadores de dominio en español sin tildes ni ñ.
