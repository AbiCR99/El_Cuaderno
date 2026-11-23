# Arquitectura y estructura del repositorio

## Frontend: React + Vite + TypeScript

**Por qué React:** es lo que enseña The Odin Project al final del camino de JavaScript, así que lo que escriba el agente se puede leer y revisar con lo que estás estudiando. Tiene el ecosistema más grande para PWA, IndexedDB y Supabase, y Claude Code lo conoce muy bien.

**Por qué Vite y no Next.js:** la app es 100 % del lado del cliente (tiene que funcionar offline) y el backend es Supabase. Next.js suma servidor, rutas de API y conceptos (RSC, SSR) que no se necesitan y complican el modo offline. Vite da un sitio estático que Vercel sirve sin configuración.

**Por qué TypeScript:** `supabase gen types` genera los tipos de la base, y el compilador atrapa errores del agente (un campo mal escrito, un `null` olvidado en un precio) antes de que lleguen a la usuaria. Es JavaScript con anotaciones: se lee igual.

**Alternativas descartadas:** JavaScript puro (menos red de seguridad), Next.js (servidor innecesario), Svelte o Vue (no los cubre TOP), React Native (no hace falta tienda; la PWA instalable alcanza en Chrome Android).

## Dependencias previstas (cada una se aprueba al agregarla)

| Paquete | Para qué | Spec |
|---|---|---|
| `react`, `react-dom`, `react-router` | UI y navegación | 001 |
| `@supabase/supabase-js` | Auth, base, Storage, Edge Functions | 001 |
| `vite-plugin-pwa` | Service worker (Workbox) y manifest instalable | 001 |
| `dexie` | IndexedDB: lista de precios offline y cola de facturas | 004, 006 |
| `minisearch` | Búsqueda tolerante (prefijo + difusa) en el dispositivo | 006 |
| `pdfjs-dist` | Extraer texto de PDF digitales y rasterizar escaneados | 004 |
| `vitest` | Tests de `src/dominio` | 002 |
| `eslint`, `typescript` | Calidad y tipos | 001 |

Sin librería de estilos ni de componentes: CSS Modules con variables. Sin librería de fechas: `Intl.RelativeTimeFormat('es-AR')` para "hace 3 días". La compresión de imágenes se hace con `<canvas>` (unas 30 líneas), sin dependencia.

## Backend: Supabase

- **Postgres con RLS** en todas las tablas. Roles: `admin` (la dueña), `dev` (vos), `familia` (cuenta compartida con PIN), todos con los mismos permisos (D5); el rol sirve para saber quién hizo cada cambio. Sin sesión, nada. Ver `docs/modelo-datos.md`.
- **RPC atómicas** para escribir: `confirmar_factura`, `aplicar_precios`, `eliminar_rubro`. El cliente calcula los precios con `src/dominio/precios.ts` y la RPC los guarda junto con el historial, todo o nada.
- **RPC de sincronización**: `lista_precios(desde)` devuelve los productos cambiados (con alias) para la base local del celular.
- **Storage**: bucket privado `facturas` (fotos y PDF). Solo `admin`/`dev`.
- **Edge Function `extraer-factura`** (Deno): recibe el id de la factura, lee el archivo o el texto, llama a Gemini con structured output y guarda los renglones. La key está en `supabase secrets`.
- **pg_cron + pg_net**: cada 2 minutos reintenta facturas en cola (429, errores transitorios). Se adelanta de la fase 2 porque el brief pide que la cola "se reintente sola" aunque la app esté cerrada.
- **Proyectos** (D6): cuenta nueva de Supabase en plan gratis, con dos proyectos, `dev` y `prod`, para no probar migraciones sobre los datos reales. El MCP de Supabase apunta solo a `dev`. El plan gratis pausa un proyecto tras una semana sin uso y no hace backups: `prod` se usa a diario y se hace un `npx supabase db dump` semanal.

## Entorno de desarrollo (D16)

Mac mini con macOS Monterey: **Node 22 LTS** (Node 24 ya no tiene binarios para Monterey). Supabase CLI con `npx supabase`, sin instalación global ni Docker; no se levanta una base local, se trabaja contra `dev`.

## Gemini

Modelo configurable por variable `GEMINI_MODEL`. Punto de partida: `gemini-3.1-flash-lite` (GA, el más económico de la línea 3.x). Se compara contra `gemini-2.5-flash-lite` (más barato, generación anterior) con el set de facturas reales antes de fijarlo. Detalle en `docs/gemini-extraccion.md`.

## Estructura del repositorio

```
el_cuaderno/
├── CLAUDE.md                 # Entrada de Claude Code: importa AGENTS.md y MEMORY.md
├── AGENTS.md                 # Reglas del proyecto (≤ 40 líneas)
├── MEMORY.md                 # Estado y decisiones entre sesiones (≤ 50 líneas)
├── README.md
├── .mcp.json                 # Supabase (dev), Context7, Chrome DevTools
├── .claude/
│   ├── settings.json         # Permisos compartidos
│   ├── agents/               # planner, implementer, reviewer
│   ├── commands/             # /sdd-*, /feature
│   └── skills/               # sdd, reglas-precios, supabase-rls, gemini-facturas, pwa-offline, ux-almacen
├── docs/
│   ├── brief-original.md
│   ├── constitution.md
│   ├── arquitectura.md
│   ├── modelo-datos.md
│   ├── gemini-extraccion.md
│   ├── riesgos-y-preguntas.md
│   ├── fase-1/  pantallas-y-flujos.md · backlog.md
│   ├── fase-2-plan.md
│   └── fase-3-plan.md
├── specs/
│   ├── README.md             # Índice y estado de cada spec
│   └── 001-… a 007-…/        # spec.md (ahora) · plan.md y tasks.md (con /sdd-plan y /sdd-tasks)
│
│   ── A partir de la spec 001 (todavía no existe) ──
├── index.html · vite.config.ts · tsconfig.json · package.json
├── public/                   # íconos PWA, manifest
├── src/
│   ├── dominio/              # Lógica pura + *.test.ts: precios, iva, bultos, redondeo, fraccionados, normalizar, busqueda
│   ├── datos/                # Cliente Supabase, tipos generados, Dexie (db local), sincronización, cola
│   ├── pantallas/            # Inicio, SubirProductos, Revision, Manual, Porcentajes, Precios, Ingreso
│   ├── componentes/          # BotonGrande, Renglon, Buscador, Aviso, EstadoVacio…
│   └── estilos/
└── supabase/
    ├── config.toml
    ├── migrations/           # Esquema, RLS, RPC, semilla de rubros
    ├── functions/extraer-factura/
    └── tests/                # Scripts SQL de verificación de RLS
```

`src/dominio/` es la única carpeta que conoce las reglas de negocio. Las pantallas llaman a funciones de dominio y a `src/datos/`; nunca hacen cuentas por su cuenta.
