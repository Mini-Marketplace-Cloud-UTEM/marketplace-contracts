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

## Pendiente

- Confirmar URL real en Render y actualizar
  `marketplace-contracts/registro-de-servicios.md` (hoy declara
  `https://api-grupo5-pedidos.onrender.com/v1` como placeholder a
  confirmar).
- `OrderStatus` mantiene `STOCK_RESERVED` "por compatibilidad" aunque en
  la práctica el stock ya lo reserva G4 antes de crear el pedido — no es
  un bloqueante, solo una nota de diseño que dejaron ellos mismos en el
  contrato.
