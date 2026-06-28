# Grupo 4 — Carro, Checkout e Inventario

**Estado (2026-06-28): primer servicio real desplegado.**

URL real: `https://g4-carrito-checkout-inventario-y.onrender.com`
(`/docs` para Swagger). El `openapi.yaml` de esta carpeta sigue siendo
el contrato viejo/placeholder — falta que Grupo 4 suba su contrato real
acá (el de Render es lo único confirmado en vivo por ahora).

## Verificado en vivo hoy

- Arquitectura conceptual correcta: `POST /v1/cart` → `POST
  /v1/cart/{id}/items` (valida contra G3, según su descripción) →
  `POST /v1/checkout` → reserva/libera stock. Coincide con el flujo
  esperado.
- Usan `Idempotency-Key` en checkout y `X-Correlation-Id` en todos los
  endpoints — buena práctica de trazabilidad.
- `POST /v1/stock/reservations` funciona limpio: devuelve
  `{reservation_id, status: "RESERVED", productId, quantity}`.

## Bloqueante encontrado: no reconoce productos reales de G3

Se probó `POST /v1/cart/{id}/items` con el producto real "Taladro
Eléctrico 700W" (`550e8400-e29b-41d4-a716-446655440000`, confirmado que
existe en el catálogo real de G3) y devolvió
`{"detail":"PRODUCTO_NO_ENCONTRADO_EN_CATALOGO"}` — el mismo error que
con un UUID inventado. Dicen en la documentación del endpoint que
"valida precio y existencia con Grupo 3", pero esa validación no está
funcionando contra el catálogo real hoy. Sin esto, no se puede agregar
nada al carrito.

## Dinero sigue sin resolver

`Cart.total_price` y `CartItem.unitPrice` son `number` (float), no
entero — confirmado en producción real (`total_price: 0.0`). Además
`currency` es un string libre sin validar (nada impide mandar `"USD"`).
Con la actualización del 2026-06-28 (ver
`data-dictionary/guia-y-lineamiento-de-desarrollo.md` sección 2), la
regla pasó a exigir entero de 64 bits — G4 sigue sin cumplir ni la regla
vieja ni la nueva.

## Inconsistencias de forma

- Naming mezclado dentro del mismo objeto: `CartItem.item_id` (snake)
  junto a `productId`/`unitPrice` (camel). `GET /v1/inventory/{id}`
  mezcla `stockVisible` (camel) con `reservas_activas` (snake, en
  español).
- Errores no usan la forma estándar `{code, message, details?}` — son el
  `{"detail": "..."}` nativo de FastAPI, con códigos a veces en español
  (`CARRITO_NO_ENCONTRADO`, `CARRITO_VACIO`) y a veces en inglés.
- `GET /v1/inventory/{id}` no valida que el producto exista: con un
  UUID inventado devolvió `200` con `stockVisible: 100` igual.
- 5 de 8 endpoints no documentan la forma de su respuesta en el Swagger
  (`schema: {}}`) — hubo que probarlos en vivo a ciegas para saber qué
  devuelven.

## Pendiente de Grupo 4

1. Arreglar la validación contra G3 (bloqueante — sin esto no se puede
   probar nada más).
2. Dinero a entero de 64 bits + moneda restringida a `CLP`.
3. Unificar naming y forma de error en todo el contrato.
4. Subir su `openapi.yaml` real a esta carpeta (hoy sigue el viejo).
