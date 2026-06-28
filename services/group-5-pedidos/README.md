# Grupo 5 — Pedidos (Order Management)

**Estado (2026-06-21): contrato real publicado por Grupo 5.**

El `openapi.yaml` y `events.md` en esta carpeta son una copia directa de
lo que Grupo 5 publicó en su propio repo
([`Grupo5-Pedidos`](https://github.com/Mini-Marketplace-Cloud-UTEM/Grupo5-Pedidos),
carpeta `contrato/`). Reemplaza por completo el borrador `0.1.0-draft`
que Grupo 1 había dejado acá como punto de partida.

## Qué cambió respecto al borrador

Grupo 5 adoptó el borrador casi sin cambios y queda **100% alineado** a
las convenciones del proyecto (ver
`data-dictionary/guia-y-lineamiento-de-desarrollo.md`):

- `orderId` con formato `ORD-YYYYMMDD-NNN` (mismo patrón que G6/G8).
- camelCase en todo el contrato público.
- Error corto vía `$ref` a `shared/components.yaml#/Error`.
- Paginación estándar (`{data, pagination: {...}}`) vía `$ref` a
  `shared/components.yaml#/Pagination` en `GET /users/{userId}/orders`.
- Dinero `integer`, moneda `[CLP]`.
- Confirma explícitamente que **no orquesta checkout** — `POST /orders`
  es invocado por Grupo 4 después de reservar stock, no por el frontend
  ni por el BFF directamente.
- JWT validado vía `POST /auth/validate` de Grupo 2 (no verificación
  local de firma).

## Actualización 2026-06-22 — sin cambios funcionales, pero ojo con esto

Agregaron `example:` extensos a cada respuesta (el archivo creció de ~3KB
a ~17KB) — esquemas, paths y reglas de negocio quedan idénticos a la
versión del 21 de junio. Lo único relevante: **crearon su propia copia de
`shared/components.yaml` dentro de su repo** (`Grupo5-Pedidos/shared/`),
y sus `$ref` ahora apuntan a esa copia local en vez de al repo central.
Hoy es una copia textual fiel (mismo `Error`, `Pagination`, `Money`,
`EventEnvelope`) — **pero esto crea dos fuentes del mismo archivo**, que
pueden divergir en silencio si editamos el original en
`marketplace-contracts/shared/components.yaml` y no avisamos a G5. Viola
el principio de "fuente única de verdad" de `guia-y-lineamiento-de-desarrollo.md`.

**Acción pendiente de Grupo 5:** idealmente borrar su copia local y que
su `openapi.yaml` resuelva contra el repo central, o documentar
explícitamente que la sincronizan a mano.

## Pendiente

- `OrderStatus` mantiene `STOCK_RESERVED` "por compatibilidad" aunque en
  la práctica el stock ya lo reserva G4 antes de crear el pedido — no es
  un bloqueante, solo una nota de diseño que dejaron ellos mismos en el
  contrato.

## Actualización 2026-06-28 — el servicio real en vivo es otro, más chico

La URL `api-grupo5-pedidos.onrender.com` (la del contrato copiado arriba)
ya no es la que está en producción. El servicio real verificado hoy es:

`https://grupo5-pedidos-mock-kdyl.onrender.com` (`/docs` para Swagger)

Es explícitamente un **mock** ("Mock oficial para desbloquear a G4 y
G1", versión 1.2.0) y solo expone **un endpoint real**: `POST /orders`.
No hay `GET /orders` ni `GET /orders/{id}` todavía — el BFF no puede
listar ni consultar pedidos contra este servicio hoy, solo crearlos.

Lo que sí se confirmó en vivo y coincide con lo documentado arriba:
- `orderId` con formato `ORD-YYYYMMDD-NNN` ✅ (probado: `ORD-20260626-001`)
- Dinero como `integer`, sin decimales ✅
- `status: "CREATED"` ✅ (coincide con la tabla de equivalencia de
  `guia-y-lineamiento-de-desarrollo.md`)
- Naming 100% camelCase ✅
- `Idempotency-Key` obligatorio en la creación ✅

**Acción pendiente de Grupo 5:** publicar `GET /orders` y
`GET /orders/{orderId}` — sin esos dos el BFF solo puede crear pedidos,
no mostrarlos al usuario.
