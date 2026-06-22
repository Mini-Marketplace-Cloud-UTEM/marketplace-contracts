# Contrato de Eventos — Grupo 5: Pedidos

Usa el sobre estándar `EventEnvelope` definido en
`marketplace-contracts/shared/components.yaml`:

```yaml
EventEnvelope:
  eventId: uuid
  eventType: string        # UPPER_SNAKE_CASE
  version: string          # "1.0"
  occurredAt: date-time    # UTC
  producer: string         # "group-5-pedidos"
  correlationId: uuid?     # trazabilidad
  payload: object          # camelCase
```

Tecnología del curso: "Contratos JSON Pub/Sub" (Upstash Kafka Free /
Cloudflare Queues / Supabase Realtime — a confirmar cuál usa el equipo).

---

## 1. Eventos que PUBLICA Grupo 5

### `ORDER_CREATED`

Se publica inmediatamente después de `POST /orders` exitoso (201).
Consumido por **Grupo 7 (Reportería)** y **Grupo 8 (Notificaciones)** — ya
está confirmado en sus propios contratos
(`services/group-7-reporteria/openapi.yaml`, sección `x-events.consumed`).

```json
{
  "eventId": "evt-3f2a1b00-0000-4000-8000-000000000001",
  "eventType": "ORDER_CREATED",
  "version": "1.0",
  "occurredAt": "2026-06-20T10:00:00Z",
  "producer": "group-5-pedidos",
  "correlationId": "99999999-8888-4777-9666-555555555555",
  "payload": {
    "orderId": "ORD-20260620-001",
    "userId": "e9d8c7b6-a543-2109-8765-fedcba098765",
    "status": "CREATED",
    "totalAmount": 1599980,
    "currency": "CLP",
    "items": [
      {
        "productId": "f0e9d8c7-b6a5-4321-0987-fedcba098765",
        "name": "Notebook Lenovo IdeaPad",
        "quantity": 2,
        "unitPrice": 799990,
        "subtotal": 1599980
      }
    ]
  }
}
```

### `ORDER_STATUS_CHANGED`

Se publica en cada transición de estado posterior a la creación (PAID,
READY_TO_SHIP, SHIPPED, DELIVERED). Consumido por Grupo 8 (Notificaciones)
para avisar al usuario, y por Grupo 7 (Reportería) para reconciliar el
dashboard en tiempo casi real.

```json
{
  "eventId": "evt-3f2a1b00-0000-4000-8000-000000000002",
  "eventType": "ORDER_STATUS_CHANGED",
  "version": "1.0",
  "occurredAt": "2026-06-20T10:05:00Z",
  "producer": "group-5-pedidos",
  "correlationId": "99999999-8888-4777-9666-555555555555",
  "payload": {
    "orderId": "ORD-20260620-001",
    "userId": "e9d8c7b6-a543-2109-8765-fedcba098765",
    "previousStatus": "PAYMENT_PENDING",
    "status": "PAID"
  }
}
```

### `ORDER_CANCELLED`

Caso especial de `ORDER_STATUS_CHANGED` con su propio `eventType` (igual que
en el borrador original de Grupo 1), porque normalmente dispara un flujo
distinto en Notificaciones (cancelación ≠ avance normal).

```json
{
  "eventId": "evt-3f2a1b00-0000-4000-8000-000000000003",
  "eventType": "ORDER_CANCELLED",
  "version": "1.0",
  "occurredAt": "2026-06-20T10:10:00Z",
  "producer": "group-5-pedidos",
  "correlationId": "99999999-8888-4777-9666-555555555555",
  "payload": {
    "orderId": "ORD-20260620-001",
    "userId": "e9d8c7b6-a543-2109-8765-fedcba098765",
    "status": "CANCELLED",
    "reason": "PAYMENT_REJECTED"
  }
}
```

---

## 2. Eventos que CONSUME Grupo 5

### `PAYMENT_APPROVED` / `PAYMENT_REJECTED` / `PAYMENT_PENDING` (de Grupo 8)

Confirmado contra el contrato real de G8
(`services/group-8-pagos/openapi.yaml`). Forma del payload:

```json
{
  "eventId": "evt-550e8400-e29b-41d4-a716-446655440001",
  "eventType": "PAYMENT_APPROVED",
  "version": "1.0",
  "occurredAt": "2026-06-20T10:04:00Z",
  "producer": "g8-pagos",
  "correlationId": "99999999-8888-4777-9666-555555555555",
  "payload": {
    "paymentId": "PAY-a1b2c3d4-e5f6-7890-1234-56789abcdef0",
    "orderId": "ORD-20260620-001",
    "userId": "e9d8c7b6-a543-2109-8765-fedcba098765",
    "amount": 1599980,
    "currency": "CLP"
  }
}
```

**Lógica de G5 al consumir:**
- `PAYMENT_APPROVED` → `PATCH /orders/{orderId}/status` interno a `PAID`,
  luego dispara la llamada a G6 para crear el shipment.
- `PAYMENT_REJECTED` → transición a `FAILED`, publica `ORDER_CANCELLED` con
  `reason: PAYMENT_REJECTED`.
- `PAYMENT_PENDING` → no cambia de estado (el pedido ya nace en
  `PAYMENT_PENDING` o se queda esperando), solo se loguea.

**Idempotencia obligatoria:** si el mismo `eventId` llega dos veces (reintento
del bus), G5 debe ignorar la segunda entrega sin re-disparar la transición ni
la llamada a G6. Se recomienda un `UNIQUE` sobre `eventId` procesado, igual
patrón que usa G8 para sus propias notificaciones.

### `SHIPMENT_DELIVERED` / `SHIPMENT_FAILED` (de Grupo 6) — ⚠️ NO disponible aún

El enunciado del curso y el borrador original asumen que Despacho publica
estos eventos. **Hoy no es así:** G6 implementó el patrón Outbox
(`outbox_events`) pero **no tiene un worker que despache a un broker real**
(confirmado en `services/group-6-despacho/README.md` del repo de contratos).

**Mitigación temporal (a usar hasta que G6 tenga el worker):**
G5 hace polling periódico (ej. cada 60s para pedidos en `READY_TO_SHIP` o
`SHIPPED`) a:

```
GET /api/v1/shipments?orderId={orderId}   (Grupo 6)
```

y actualiza su propio estado según el `status` del shipment devuelto.
Cuando G6 active el worker del Outbox, este polling se reemplaza por
consumo real de `SHIPMENT_DELIVERED`/`SHIPMENT_FAILED`. Documentar este
cambio como versión `1.1` de este contrato cuando ocurra.

---

## 3. Tabla resumen

| Evento | Dirección | Productor real | Estado |
|---|---|---|---|
| `ORDER_CREATED` | Publica | G5 | ✅ Listo para implementar |
| `ORDER_STATUS_CHANGED` | Publica | G5 | ✅ Listo para implementar |
| `ORDER_CANCELLED` | Publica | G5 | ✅ Listo para implementar |
| `PAYMENT_APPROVED` | Consume | G8 | ✅ Contrato confirmado |
| `PAYMENT_REJECTED` | Consume | G8 | ✅ Contrato confirmado |
| `PAYMENT_PENDING` | Consume | G8 | ✅ Contrato confirmado |
| `SHIPMENT_DELIVERED` | Consume | G6 | 🔴 Bloqueado — usar polling temporal |
| `SHIPMENT_FAILED` | Consume | G6 | 🔴 Bloqueado — usar polling temporal |
