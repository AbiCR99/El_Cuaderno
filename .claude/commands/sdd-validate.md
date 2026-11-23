---
description: SDD · Valida una spec RF por RF (tests + Chrome DevTools + RLS)
argument-hint: "[NNN]"
---

Usá el subagente `reviewer` para recorrer `specs/$0*/spec.md` requisito por requisito: qué test cubre cada RF y el resultado de correrlo. Los RF de interfaz, verificados con Chrome DevTools en 360 px (y en modo offline si corresponde). Si la spec toca la base, que corra las comprobaciones de RLS sin sesión (todo falla) y con sesión (todo funciona).

Si algún RF no está cubierto o falla, que lo diga claramente. NO arregles nada todavía.

Al final, criterios de finalización y veredicto: ¿la spec está cumplida?
