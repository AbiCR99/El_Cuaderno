---
name: sdd
description: Úsala siempre que trabajes con Spec-Driven Development en El Cuaderno (docs/constitution.md o cualquier archivo dentro de specs/) - redactar, revisar o cambiar specs, planes y tareas, o implementar y validar tareas de una spec.
---

# Spec-Driven Development (SDD)

## Flujo
Constitución → Spec → Clarificación → Plan → Tareas → Implementación → Validación → Cambio.
- Nunca pases a la fase siguiente sin aprobación explícita de la desarrolladora.
- La spec manda: si algo no está en la spec, no se implementa. Si falta una decisión, pará y preguntá.
- Fuentes de requisitos válidas: `docs/brief-original.md` y decisiones registradas en `MEMORY.md` o `docs/riesgos-y-preguntas.md`. Cualquier otra cosa va como [NECESITA ACLARACIÓN].
- Un cambio de requisitos se hace primero en la spec, después en el plan y las tareas, y por último en el código.
- Cada spec vive en `specs/NNN-nombre/` con `spec.md`, `plan.md` y `tasks.md`. Su estado se refleja en `specs/README.md`.
- Al cerrar cada fase, actualizá `MEMORY.md`.

## Plantilla de spec
```
# Spec NNN — <Nombre>
Estado: borrador | aprobada | implementada
## Contexto y objetivo
## Usuarios
## Definiciones (solo si hay términos ambiguos)
## Historias de usuario
- HU-1. Como <rol> quiero <acción> para <beneficio>.
## Requisitos funcionales
## Requisitos no funcionales
## Casos límite
## Fuera de alcance
## Criterios de finalización
## Dudas abiertas
- [NECESITA ACLARACIÓN] <duda>
```
La spec describe el QUÉ y el POR QUÉ. Nada de stack, arquitectura ni nombres de archivos.

## Requisitos en EARS
- RF-x: CUANDO <evento>, EL SISTEMA <respuesta>.
- RF-x: SI <condición no deseada>, ENTONCES EL SISTEMA <respuesta>.
- RF-x: MIENTRAS <estado>, EL SISTEMA <respuesta>.
- RF-x: EL SISTEMA <comportamiento permanente>.
Cada RF debe ser verificable: nada de "rápido" o "simple" sin un criterio medible.

## Plan (plan.md)
Archivos y responsabilidades · funciones puras de `src/dominio/` · migraciones y RLS · interfaz y sus estados · decisiones con alternativa descartada · tests con Vitest y verificación con Chrome DevTools · qué RF cubre cada parte · toques del camino feliz si hay pantalla.

## Tareas (tasks.md)
```
- [ ] **Tn. <Descripción>.** RF-x, RF-y
  - Hecho cuando: <comprobación verificable>.
```
20–40 min por tarea, en orden de dependencia, máximo 10.

## Implementación
Una tarea por vez: tests primero (en rojo), código, `npm run test && npm run typecheck && npm run lint` en verde, marcar la tarea y parar.
