# Grupo 6 — Despacho (Shipment Service)

**Estado (2026-06-20): `openapi.yaml` copiado desde su repo — auto-generado
por FastAPI, está incompleto en 2 puntos (ver abajo), pero el código real
ya cumple casi todas las convenciones del proyecto.**

El repo de servicio es
[`G6-Shipment-Service`](https://github.com/Mini-Marketplace-Cloud-UTEM/G6-Shipment-Service).
El `openapi.yaml` en esta carpeta es una copia directa del que el propio
servicio genera (`/docs` de FastAPI) — no es contrato impuesto por
Grupo 1, es lo que su código real expone hoy.

## Qué cambió desde la revisión anterior (2026-06-18)

En ese momento G6 tenía 4 fuentes de contrato que no coincidían entre sí
(v1.0 implementado en código, v1.1 solo documentado, una guía interna, y
un README que afirmaba que v1.1 ya estaba vigente). El 2026-06-20 G6
publicó una serie de commits que **implementan realmente el modelo
multi-paquete** y resuelven casi todo lo que se había señalado:

| Punto | Antes (v1.0, 2026-06-18) | Ahora (v1.2, 2026-06-20) |
| --- | --- | --- |
| Modelo de datos | 1 pedido = 1 envío (`order_id` único) | 1 pedido = N envíos (un `Shipment` por paquete) |
| Cotización | No existía | `POST /shipments/quotes` implementado, con motor de tarifas por zona/peso volumétrico |
| Naming | snake_case | **camelCase**, vía `alias_generator` de Pydantic |
| Forma de error | `{timestamp, status, code, message, correlationId}` | `{code, message, details?, correlationId?}` — coincide exactamente con la convención del proyecto |
| Paginación | `{total, limit, offset, shipments}` | `{data, pagination: {page, pageSize, total, totalPages, hasNext, hasPrev}}` — coincide exactamente |
| Eventos | Documentados, no implementados | Patrón Transactional Outbox real (tabla `outbox_events`); falta el worker que los despache a un broker, pero la estructura ya existe |
| Historial de estado | No existía | `GET /shipments/{id}/history` nuevo |

## Actualización 2026-06-23 — los 2 gaps de código quedaron resueltos

Commits del 2026-06-23 (`fix: corregir snake_case en orderId, añadir
response models faltantes para swagger...`):

1. ✅ **Bug de naming corregido:** `GET /shipments?orderId=` ya devuelve
   camelCase, confirmado en vivo (`{"code":"NOT_FOUND",...}` en un 404 de
   prueba).
2. ✅ **`openapi.yaml` ya completo:** `ErrorResponse` y
   `Pagination`/`ShipmentListResponse` ahora están declarados con
   `response_model=` en todos los endpoints — el spec generado ya
   coincide con el comportamiento real.
3. ✅ Agregaron una suite de tests pytest real (`tests/test_api.py`).

Su doc `docs/contratos/v1.1/G1_Frontend.md` (la propuesta de "estado
global" multi-caja) sigue sin ser parte del contrato real — sin cambios,
no es una obligación.

## Nuevo requisito (2026-06-23, no documentado antes)

Los 3 headers de trazabilidad — `X-Request-Id`, `X-Correlation-Id`,
`X-Consumer` — ahora son **obligatorios** en todos los endpoints (antes
no se exigían). El BFF ya los envía en cada request
(`Grupo-1-BFF/app/routers` — confirmar que el router de shipments, cuando
se implemente, también los incluya).

## Cómo lo consumimos (G1/BFF) hoy

- `GET /api/v1/shipments?orderId={orderId}` → arreglo de shipments del
  pedido, ya en camelCase.
- `GET /api/v1/shipments/{shipmentId}` → detalle de un shipment.
- `POST /api/v1/shipments/quotes` → cotización de envío (sin headers de
  trazabilidad obligatorios — este endpoint específico queda exento),
  pensado para que G4 lo use en checkout.

**Pendiente:** confirmar este `openapi.yaml` como su contrato oficial vía
PR a este repo (hoy seguimos copiándolo manualmente).
