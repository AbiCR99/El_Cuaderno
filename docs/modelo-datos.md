# Modelo de datos (fase 1) y políticas RLS

Este documento es la referencia. Las migraciones reales se escriben en `supabase/migrations/` durante la spec 001/002, a partir de este diseño, y cualquier diferencia se corrige primero acá.

## Cambios respecto del modelo del brief

| Brief | Ahora | Por qué |
|---|---|---|
| `categorias.margen` | `margen_pct numeric(5,2)` + `es_sin_categoria` | Permite decimales (17,5 %) y proteger "Sin categoría" de ser borrada. |
| `productos.margen_override` | Eliminado | El brief no pide margen por producto. El escape para un producto puntual es `precio_fijo`. Si hace falta, vuelve con una spec. |
| `productos.codigo_barras` | Pasa a tabla `codigos_barras` en fase 2 | Un producto puede tener varios códigos (unidad, pack). En fase 1 no se usa. |
| `productos.actualizado_en` | `precio_actualizado_en` y `costo_actualizado_en` | "Hace 3 días" se refiere al precio, no a cualquier cambio. |
| — | `productos.ultima_compra_fecha` | Fecha de factura de la última compra que fijó el costo. Detecta facturas viejas (D12). |
| `alias_producto.texto_proveedor` | `texto_original` + `texto_normalizado` (único por proveedor) | El emparejamiento compara texto normalizado (mayúsculas, sin tildes, espacios colapsados). |
| `proveedores` | + `cuit` (opcional) | Identifica al proveedor aunque el nombre aparezca escrito distinto. Gemini lo lee si figura. |
| `facturas` | + `id uuid` generado en el celular, `origen`, `numero`, `intentos`, `proximo_intento_en`, `ultimo_error`, `extraccion_cruda`, `modelo_ia` | Idempotencia de la cola offline, reintentos, auditoría de lo que leyó Gemini. |
| `factura_items` | + `renglon`, `bonificacion_pct` y `bonificacion_monto`, `importe_renglon`, `proveedor_id` y `fecha` copiados, `categoria_sugerida_id`, `legibilidad`, `precio_venta_aplicado` | Control cruzado de lectura, y lo que pide la fase 3 en cada renglón. |
| `fraccionados` | `fraccionamientos` (una fila por compra a granel) + `productos.modo_venta` y `precio_100g` | El modo de venta es del producto; cada compra a granel deja su registro. |
| — | `perfiles` | Rol de cada usuario de Auth. |

## Tipos

```sql
create type public.rol_usuario    as enum ('admin', 'dev', 'familia');
create type public.modo_venta     as enum ('unidad', 'fraccion', 'peso');
create type public.estado_factura as enum ('en_cola', 'procesando', 'revision', 'confirmada', 'error', 'descartada');
create type public.origen_factura as enum ('camara', 'pdf', 'manual', 'repetir');
create type public.origen_cambio  as enum ('factura', 'manual', 'recalculo_rubro', 'cambio_rubro', 'fraccionado');
```

## Tablas

```sql
create table public.perfiles (
  user_id   uuid primary key references auth.users on delete cascade,
  rol       public.rol_usuario not null,
  nombre    text not null,
  creado_en timestamptz not null default now()
);

create table public.categorias (
  id               bigint generated always as identity primary key,
  nombre           text not null,
  margen_pct       numeric(5,2) check (margen_pct >= 0),   -- 30.00 = 30 %
  precio_fijo      boolean not null default false,
  es_sin_categoria boolean not null default false,          -- no se puede borrar ni renombrar
  orden            smallint not null default 0,
  creado_en        timestamptz not null default now(),
  check (precio_fijo or margen_pct is not null)
);
create unique index categorias_nombre_unico on public.categorias (lower(nombre));
create unique index categorias_una_sin_categoria on public.categorias (es_sin_categoria) where es_sin_categoria;

create table public.proveedores (
  id           bigint generated always as identity primary key,
  nombre       text not null,
  cuit         text,                                -- 11 dígitos sin guiones
  dias_pedido  smallint[] not null default '{}',    -- ISO: 1 = lunes … 7 = domingo
  dias_entrega smallint[] not null default '{}',
  creado_en    timestamptz not null default now(),
  check (dias_pedido  <@ array[1,2,3,4,5,6,7]::smallint[]),
  check (dias_entrega <@ array[1,2,3,4,5,6,7]::smallint[])
);
create unique index proveedores_cuit_unico on public.proveedores (cuit) where cuit is not null;

create table public.productos (
  id                    bigint generated always as identity primary key,
  nombre                text not null,
  categoria_id          bigint not null references public.categorias on delete restrict,
  precio_fijo           boolean not null default false,  -- además del precio fijo del rubro
  modo_venta            public.modo_venta not null default 'unidad',
  costo_unitario        numeric(14,2),     -- último costo por unidad de venta, IVA incluido, neto de bonificaciones
  precio_venta          integer,           -- pesos enteros; por unidad, por bolsita o por kg según modo_venta
  precio_100g           integer,           -- solo modo 'peso'
  costo_actualizado_en  timestamptz,
  precio_actualizado_en timestamptz,
  ultima_compra_fecha   date,              -- fecha de la factura más nueva que fijó el costo (D12)
  ultimo_proveedor_id   bigint references public.proveedores,
  activo                boolean not null default true,
  creado_en             timestamptz not null default now(),
  actualizado_en        timestamptz not null default now(),  -- trigger; lo usa la sincronización
  check (precio_venta is null or precio_venta >= 0),
  check (modo_venta = 'peso' or precio_100g is null)
);
create index productos_categoria on public.productos (categoria_id);
create index productos_actualizado on public.productos (actualizado_en);

create table public.alias_producto (
  id                bigint generated always as identity primary key,
  proveedor_id      bigint not null references public.proveedores on delete cascade,
  texto_original    text not null,
  texto_normalizado text not null,
  producto_id       bigint not null references public.productos on delete cascade,
  unidades_por_pack numeric(10,3),          -- la última usada, para precargar
  creado_en         timestamptz not null default now(),
  unique (proveedor_id, texto_normalizado)
);
create index alias_producto_producto on public.alias_producto (producto_id);

create table public.facturas (
  id                 uuid primary key,                 -- lo genera el celular (crypto.randomUUID) para que la cola sea idempotente
  proveedor_id       bigint references public.proveedores,
  proveedor_leido    text,                             -- como lo leyó Gemini, hasta emparejar
  cuit_leido         text,
  fecha              date,
  tipo               text check (tipo in ('A','B','C','X','remito','presupuesto','otro')),
  numero             text,
  con_iva            boolean,                          -- true: los precios ya traen IVA; null: sin confirmar
  origen             public.origen_factura not null,
  archivo_path       text,                             -- ruta en Storage (bucket facturas)
  texto_pdf          text,                             -- texto extraído con pdf.js, si es PDF digital
  estado             public.estado_factura not null default 'en_cola',
  intentos           smallint not null default 0,
  proximo_intento_en timestamptz,
  ultimo_error       text,
  extraccion_cruda   jsonb,                            -- respuesta completa de Gemini
  modelo_ia          text,
  creada_por         uuid references auth.users default auth.uid(),
  creada_en          timestamptz not null default now(),
  confirmada_en      timestamptz
);
create index facturas_estado on public.facturas (estado, proximo_intento_en);
create index facturas_proveedor_numero on public.facturas (proveedor_id, numero) where numero is not null;  -- aviso de repetida (D12), no único
create index facturas_proveedor_fecha on public.facturas (proveedor_id, fecha desc);

create table public.factura_items (
  id                       bigint generated always as identity primary key,
  factura_id               uuid not null references public.facturas on delete cascade,
  renglon                  smallint not null,
  descripcion_original     text not null,
  producto_id              bigint references public.productos,     -- null hasta emparejar
  categoria_sugerida_id    bigint references public.categorias,
  cantidad_packs           numeric(10,3) not null check (cantidad_packs > 0),
  unidades_por_pack        numeric(10,3) not null default 1 check (unidades_por_pack > 0),
  precio_unitario          numeric(14,4) not null check (precio_unitario >= 0),  -- por pack, como figura
  bonificacion_pct         numeric(5,2) not null default 0,
  bonificacion_monto       numeric(14,2) not null default 0,       -- descuento total del renglón en $
  alicuota_iva             numeric(5,2),                           -- null = 21
  importe_renglon          numeric(14,2),                          -- leído, para control cruzado
  legibilidad              text check (legibilidad in ('clara','dudosa')),
  costo_unitario_calculado numeric(14,2),                          -- al confirmar
  precio_venta_aplicado    integer,                                -- al confirmar
  proveedor_id             bigint references public.proveedores,   -- copia al confirmar (fase 3)
  fecha                    date,                                   -- copia al confirmar (fase 3)
  unique (factura_id, renglon)
);
create index factura_items_producto_fecha on public.factura_items (producto_id, fecha desc);

create table public.fraccionamientos (
  id                  bigint generated always as identity primary key,
  producto_id         bigint not null references public.productos,
  factura_id          uuid references public.facturas,   -- la carga manual que lo originó
  precio_compra       numeric(14,2) not null,             -- total pagado por el granel, IVA según la factura
  peso_compra_g       integer not null check (peso_compra_g > 0),
  modo_venta          public.modo_venta not null check (modo_venta in ('fraccion','peso')),
  cantidad_fracciones integer check (cantidad_fracciones > 0),  -- modo 'fraccion'
  peso_fraccion_g     integer check (peso_fraccion_g > 0),      -- modo 'fraccion'
  registrado_en       timestamptz not null default now()
);

create table public.historial_precios (
  id             bigint generated always as identity primary key,
  producto_id    bigint not null references public.productos on delete cascade,
  costo_unitario numeric(14,2),
  precio_venta   integer,
  precio_100g    integer,
  margen_pct     numeric(5,2),          -- el que se usó; null si precio fijo o manual
  origen         public.origen_cambio not null,
  factura_id     uuid references public.facturas,
  usuario_id     uuid references auth.users default auth.uid(),
  registrado_en  timestamptz not null default now()
);
create index historial_producto_fecha on public.historial_precios (producto_id, registrado_en desc);
```

Triggers: `actualizado_en` en `productos` y `categorias`. Ninguno calcula precios.

Semilla (migración, no `seed.sql`, porque también va a producción): los 11 rubros del brief, con `Cigarrillos` `precio_fijo = true, margen_pct = null` y `Sin categoría` `es_sin_categoria = true, margen_pct = 30`.

## Funciones auxiliares de acceso

Los tres roles tienen **los mismos permisos** (D5). El rol se guarda igual, para saber quién hizo cada cambio y por si en el futuro hace falta distinguirlos.

```sql
create function public.rol_actual() returns public.rol_usuario
language sql stable security definer set search_path = '' as $$
  select rol from public.perfiles where user_id = (select auth.uid())
$$;

create function public.tiene_acceso() returns boolean
language sql stable security definer set search_path = '' as $$
  select public.rol_actual() is not null
$$;

revoke execute on function public.rol_actual(), public.tiene_acceso() from anon, public;
grant  execute on function public.rol_actual(), public.tiene_acceso() to authenticated;
```

## Políticas RLS

Regla general: RLS activada en **todas** las tablas. `anon` no tiene ninguna política. Un usuario de Auth sin fila en `perfiles` no ve nada.

```sql
alter table public.<tabla> enable row level security;   -- para cada tabla

-- Tablas de negocio: categorias, proveedores, productos, alias_producto, facturas,
-- factura_items, fraccionamientos. Cualquier usuario con perfil: lectura y escritura.
create policy con_perfil_todo on public.<tabla>
  for all to authenticated
  using ((select public.tiene_acceso()))
  with check ((select public.tiene_acceso()));

-- historial_precios: se agrega y se lee, nunca se edita ni se borra.
create policy con_perfil_lee on public.historial_precios
  for select to authenticated using ((select public.tiene_acceso()));
create policy con_perfil_agrega on public.historial_precios
  for insert to authenticated with check ((select public.tiene_acceso()));

-- perfiles: lectura para usuarios con perfil (la app necesita saber quién es). Altas solo desde el dashboard.
create policy con_perfil_lee on public.perfiles
  for select to authenticated using ((select public.tiene_acceso()));
```

| Tabla / función | admin · dev · familia | anon o sin perfil |
|---|---|---|
| categorias, proveedores, productos, alias_producto, facturas, factura_items, fraccionamientos | leer/escribir | — |
| historial_precios | leer/agregar (nunca editar ni borrar) | — |
| perfiles | leer | — |
| `lista_precios()`, `confirmar_factura()`, `aplicar_precios()`, `eliminar_rubro()` | sí | — |
| Storage `facturas` | subir/leer | — |

### RPC de sincronización

`lista_precios(desde timestamptz default null)` (`security invoker`, aplica RLS) devuelve los productos cambiados desde `desde`, con sus alias agregados en un `text[]`, para la base local del celular: `id, nombre, categoria_id, modo_venta, precio_fijo, costo_unitario, precio_venta, precio_100g, costo_actualizado_en, precio_actualizado_en, ultima_compra_fecha, ultimo_proveedor_id, alias, activo, actualizado_en`. Los rubros y proveedores se leen directo de sus tablas (son pocos). `revoke ... from anon, public; grant ... to authenticated`.

### RPC de escritura (`security invoker`: aplica RLS)

- `confirmar_factura(p_factura uuid, p_cabecera jsonb, p_renglones jsonb)`: en una transacción crea el proveedor si es nuevo (solo nombre y CUIT), crea los productos nuevos, guarda o actualiza los alias, guarda todos los renglones con proveedor y fecha copiados y pasa la factura a `confirmada`. Para cada producto: si `ultima_compra_fecha` es posterior a la fecha de la factura (factura vieja, D12), **no** toca costo ni precio; si no, escribe costo y precio calculados en el cliente, actualiza `ultima_compra_fecha` e inserta historial. Si la factura ya está confirmada, no hace nada (idempotente). La regla de factura vieja se repite en la base a propósito: protege aunque dos personas confirmen a la vez.
- `aplicar_precios(p_cambios jsonb, p_origen public.origen_cambio)`: recibe `[{producto_id, precio_venta, precio_100g, costo_unitario, margen_pct}]`, actualiza e inserta historial. Se usa para recalcular un rubro, cambiar de rubro y editar a mano.
- `eliminar_rubro(p_rubro bigint, p_destino bigint)`: mueve los productos al destino (o a "Sin categoría") y borra el rubro. No recalcula: si hay que recalcular, el cliente llama después a `aplicar_precios`.

### Storage

Bucket privado `facturas`. Ruta: `{anio}/{mes}/{factura_id}.{webp|jpg|pdf}`. Políticas sobre `storage.objects` con `bucket_id = 'facturas' and (select public.tiene_acceso())` para `insert` y `select`. Sin `update` ni `delete` desde el cliente.

### Edge Function y cron

- `extraer-factura` verifica el JWT y que el usuario tenga perfil, o un header `x-cron-secret` cuando la llama pg_cron. Usa `service_role` solo dentro de la función para leer el archivo y escribir el resultado.
- `pg_cron` cada 2 minutos: toma facturas `en_cola` con `proximo_intento_en <= now()` (máximo 5 por vuelta) y llama a la función con `pg_net`. Backoff: 1, 2, 5, 15, 60 minutos; después de 8 intentos pasa a `error` y aparece en la app con la opción "Cargar a mano".

## Cuentas

| Cuenta | Rol | Ingreso |
|---|---|---|
| La dueña | `admin` | Email + contraseña una sola vez; la sesión queda abierta en su celular. |
| Vos | `dev` | Email + contraseña. |
| Familia | `familia` | Una cuenta compartida (`familia@…`) cuya contraseña es el PIN de 6 dígitos. La pantalla de ingreso solo pide el PIN. Mismos permisos que `admin` (D5). |

## Fase 2 (solo previsto, no se crea ahora)

- `codigos_barras (codigo text pk, producto_id, unidades)`: un producto puede tener varios códigos; `unidades` permite que el código de un pack descuente 6.
- `ventas (id uuid, numero bigint correlativo, total, medio_pago null, usuario_id, creada_en)`. Con PIN compartido, `usuario_id` dice "familia", no qué familiar.
- `venta_items (venta_id, producto_id null, descripcion, cantidad, precio_unitario, importe_suelto bool)`: "importe suelto" no descuenta stock.
- `movimientos_stock (producto_id, cantidad ±, tipo: compra|venta|ajuste|conteo, referencia, fecha)`: el stock es la suma; las compras confirmadas de fase 1 se migran como movimientos tipo `compra` posteriores al conteo inicial.
- `conteos (id, fecha, tipo: inicial|control)` + `conteo_items (conteo_id, producto_id, cantidad)`.
- `productos.stock_minimo numeric null`.
- Las tablas nuevas usan las mismas políticas `tiene_acceso()`.
