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

## Gaps que quedan (reportados a G6, no bloqueantes)

1. **Bug de naming puntual:** `GET /shipments?orderId=` devuelve los
   campos en snake_case (`shipment_id`, `order_id`, etc.), a diferencia
   de todos los demás endpoints. Causa: ese caso construye la respuesta
   como un dict plano sin pasar por el modelo `ShipmentResponse`
   (`CamelModel`), así que no se le aplica el alias de camelCase.
2. **El `openapi.yaml` auto-generado está incompleto:** como `ErrorResponse`
   y `Pagination`/`ShipmentListResponse` no se declaran con
   `response_model=` en sus endpoints, FastAPI no los incluye en el
   spec generado. El comportamiento real (verificado leyendo
   `app/main.py` y `app/schemas.py`) sí cumple la convención — pero no
   se puede confiar 100% en este `openapi.yaml` como documentación
   completa todavía.
3. Su doc `docs/contratos/v1.1/G1_Frontend.md` describe un agregado de
   "estado global" (`PARTIALLY_DELIVERED`, `HAS_ISSUES`) a partir de un
   arreglo de cajas por pedido — es una propuesta para cuando se quiera
   formalizar esa lógica en el BFF, todavía no es parte del contrato
   real ni una obligación.

## Cómo lo consumimos (G1/BFF) hoy

- `GET /api/v1/shipments?orderId={orderId}` → arreglo de shipments del
  pedido (normalizar a camelCase en el BFF mientras dure el bug del
  punto 1).
- `GET /api/v1/shipments/{shipmentId}` → detalle de un shipment.
- `POST /api/v1/shipments/quotes` → cotización de envío (sin headers de
  trazabilidad obligatorios), pensado para que G4 lo use en checkout.

**Acción pendiente de G6:** confirmar este `openapi.yaml` como su
contrato oficial (o regenerarlo después de corregir el punto 1 y 2), y
mantenerlo actualizado en esta carpeta vía PR.
