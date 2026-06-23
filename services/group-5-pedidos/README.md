# Grupo 5 — Pedidos (Order Management)

**Estado (2026-06-21): contrato real publicado por Grupo 5.**

El `openapi.yaml` y `events.md` en esta carpeta son una copia directa de
lo que Grupo 5 publicó en su propio repo
([`Grupo5-Pedidos`](https://github.com/Mini-Marketplace-Cloud-UTEM/Grupo5-Pedidos),
carpeta `contrato/`). Reemplaza por completo el borrador `0.1.0-draft`
que Grupo 1 había dejado acá como punto de partida.

## Qué cambió respecto al borrador

Grupo 5 adoptó el borrador casi sin cambios y queda **100% alineado** a
`decisiones-ejecutivas-2026-06-19.md`:

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
el principio de "fuente única de verdad" de `conventions.md`.

**Acción pendiente de Grupo 5:** idealmente borrar su copia local y que
su `openapi.yaml` resuelva contra el repo central, o documentar
explícitamente que la sincronizan a mano.

## Pendiente

- Confirmar URL real en Render: `https://api-grupo5-pedidos.onrender.com/v1`
  responde (no está caída/DNS muerto), pero solo devuelve 404 en la raíz
  y en `/orders` — no se pudo confirmar si el contrato real ya está
  desplegado o si es solo la página default de Render. Falta probar un
  endpoint válido con autenticación.
- `OrderStatus` mantiene `STOCK_RESERVED` "por compatibilidad" aunque en
  la práctica el stock ya lo reserva G4 antes de crear el pedido — no es
  un bloqueante, solo una nota de diseño que dejaron ellos mismos en el
  contrato.
