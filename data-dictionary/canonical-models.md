# Modelos canónicos compartidos

Define el nombre y forma que cada entidad de negocio debería tener en el
**contrato público** (no en la base de datos interna de cada grupo),
para que dos grupos que hablan de "un pedido" se refieran exactamente a
los mismos campos. Basado en lo encontrado en
`analisis-integration-hell-2026-06-18.md`.

Convención de esta tabla: ✅ = ya implementado así por el grupo dueño,
⚠️ = implementado distinto hoy (columna "Estado real" explica el gap),
🆕 = todavía no existe contrato real.

## User (dueño: Grupo 2 — Auth)

| Campo canónico | Tipo | Estado real |
|---|---|---|
| `id` | uuid | ✅ |
| `name` | string | ⚠️ Nuestro contrato BFF pedía `firstName`/`lastName` separados; G2 solo tiene `name` único. El BFF se adapta a `name` único. |
| `email` | string (email) | ✅ |
| `phone` | string | ✅ (G2 lo tiene, falta agregarlo a nuestro `UserProfile` del BFF) |
| `avatarUrl` | string (uri), nullable | ✅ (G2: `avatar_url`, BFF traduce) |
| `roles` | array de `guest/customer/seller/admin` | ✅ (falta agregarlo a nuestro `UserProfile` del BFF si el frontend necesita diferenciar vistas) |
| `active` | boolean | ✅ |
| `createdAt`/`updatedAt` | date-time | ✅ |

## Product (dueño: Grupo 3 — Catálogo)

| Campo canónico | Tipo | Estado real |
|---|---|---|
| `id` | uuid | ✅ |
| `name` | string | ✅ |
| `description` | string | ✅ |
| `price` | `Money` (integer + CLP) | ⚠️ G3 usa `number` sin restricción — pedir que se declare `integer`. |
| `stock` | integer | ⚠️ G3 lo llama `stock_visible`. |
| `categoryId` / `categoryName` | uuid / string | ✅ (snake_case en G3, BFF traduce) |
| `sku` | string | ✅ |
| `status` | `ACTIVE/INACTIVE/DELETED` | ✅ |
| `images` | array de uri | ✅ |

**Gap conocido:** G3 no expone endpoint `/categories` con jerarquía
(`parentId`), que el contrato BFF sí espera. Tampoco soporta
`minPrice`/`maxPrice`/`sortBy`.

## Category (dueño: Grupo 3 — Catálogo)

🆕 No existe en el contrato real de G3 (solo `category_id` +
`category_name` planos dentro de `Product`). El BFF lo define
unilateralmente (`id, name, slug, parentId, children`) sin contrato real
detrás todavía.

## Cart / CartItem (dueño: Grupo 4 — Carro/Checkout)

| Campo canónico | Tipo | Estado real |
|---|---|---|
| `cartId` | string de negocio | ⚠️ G4 usa uuid puro como `cartId`, y requiere conocerlo explícitamente (no existe "mi carro activo" sin id). |
| `userId` | uuid | ✅ |
| `status` | `ACTIVE/COMPLETED/CLOSED` | ✅ |
| `items[]` | array de `CartItem` | ✅ |
| `totalAmount` | `Money` | ⚠️ G4 usa `number/double` + `USD` default — debe ser integer + CLP. |
| `createdAt`/`updatedAt` | date-time | ✅ |

`CartItem`: `itemId, cartId, productId, name, quantity, unitPrice
(Money), subTotal (Money)` — mismos ajustes de Money pendientes.

## Order / OrderItem (dueño: Grupo 5 — Pedidos) 🆕

**Sin contrato real publicado.** Campos a partir de la documentación
local (inconsistente entre sus propios archivos — ver
`analisis-integration-hell-2026-06-18.md` §4.1):

| Campo canónico | Tipo | Estado real |
|---|---|---|
| `orderId` | string `ORD-YYYYMMDD-NNN` | ⚠️ G5 usa 3 formatos distintos en sus propios documentos; debe converger al formato ya usado por G6/G8. |
| `userId` | uuid | ✅ (asumido) |
| `status` | enum (ver tabla de mapeo abajo) | ⚠️ Mezcla mayúsculas (doc de arquitectura) y minúsculas (ejemplos REST/eventos) dentro del mismo grupo. |
| `items[]` | array de `OrderItem` | ✅ (asumido) |
| `totalAmount` | `Money` | ✅ (ya viene entero en sus ejemplos) |
| `shippingAddress` | objeto (street/city/region/country) | A confirmar contra G5 — hoy el dato de dirección viaja en el payload de creación, no como objeto estructurado. |
| `createdAt`/`updatedAt` | date-time | ✅ (asumido) |

### Mapeo de vocabularios de estado (pendiente de acordar)

Cada grupo definió su propio vocabulario de estado para "lo que le pasa a
un pedido". Hace falta una tabla de equivalencia explícita:

| BFF `OrderStatus` | G5 (doc arquitectura) | G8 `Payment.status` | G6 `ShipmentStatus` |
|---|---|---|---|
| `PENDING` | `CREATED` | `PENDING` | — |
| `CONFIRMED` | `PAYMENT_PENDING` → `PAID` | `APPROVED` | — |
| `PROCESSING` | `STOCK_RESERVED` → `READY_TO_SHIP` | — | `PENDING` (despacho) |
| `SHIPPED` | `SHIPPED` | — | `IN_TRANSIT` |
| `DELIVERED` | `DELIVERED` | — | `DELIVERED` |
| `CANCELLED` | `CANCELLED` / `FAILED` | `REJECTED`/`CANCELLED` | `CANCELLED`/`FAILED`/`RETURNED` |

Esta tabla es una propuesta de Grupo 1, no un acuerdo confirmado — debe
validarse con Grupo 5, 6 y 8 en la reunión.

## Payment (dueño: Grupo 8 — Pagos)

| Campo canónico | Tipo | Estado real |
|---|---|---|
| `paymentId` | string `PAY-...` | ✅ |
| `orderId` | string `ORD-YYYYMMDD-NNN` | ✅ |
| `userId` | uuid | ✅ |
| `amount` | `Money` | ⚠️ G8 usa `number/double` — debe ser integer. |
| `method` | `CARD/TRANSFER` | ✅ |
| `status` | `PENDING/APPROVED/REJECTED/CANCELLED` | ✅ (confirmar si `CANCELLED` es válido — el docx interno de G8 lo incluye, el OpenAPI real listado en el análisis previo no lo confirmaba explícitamente) |
| `idempotencyKey` | uuid | ✅ |

## Shipment (dueño: Grupo 6 — Despacho)

**Atención:** el contrato real (código) implementa el modelo simple de
abajo (v1.0). La documentación de v1.1 describe un modelo multi-paquete
que **no está implementado en el código todavía** — ver
`analisis-integration-hell-2026-06-18.md` §4.5.

| Campo canónico | Tipo | Estado real (código v1.0) |
|---|---|---|
| `shipmentId` | string `SHP-...` | ✅ |
| `orderId` | string `ORD-YYYYMMDD-NNN` | ✅ (y hoy es UNIQUE — 1 pedido = 1 envío) |
| `customerName`, `address`, `city` | string | ✅ (snake_case en el JSON real, BFF traduce) |
| `weightKg` | number | ✅ |
| `status` | `PENDING/IN_TRANSIT/DELIVERED/CANCELLED/FAILED/RETURNED` | ✅ |
| `estimatedDelivery` | date-time, nullable | ✅ |

## Notification (dueño: Grupo 8 — Notificaciones)

| Campo canónico | Tipo | Estado real |
|---|---|---|
| `notificationId` | string `NTF-...` | ✅ |
| `userId` | uuid | ✅ |
| `eventId` | string | ✅ |
| `eventType` | `ORDER_CREATED/PAYMENT_APPROVED/PAYMENT_REJECTED/PAYMENT_PENDING/SHIPMENT_DELIVERED/STOCK_REJECTED` | ✅ |
| `title`, `body` | string | ✅ |
| `channel` | `EMAIL/DASHBOARD/LOG` | ✅ |
| `status` | `PENDING/DELIVERED/READ/FAILED` | ✅ |

---

Este documento se actualiza a medida que cada grupo publique su
`openapi.yaml` real en `services/group-N-x/`. Mientras un campo diga
🆕 o ⚠️, **no asumir que es definitivo**.
