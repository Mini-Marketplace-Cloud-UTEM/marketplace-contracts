# Modelos canónicos compartidos

Define el nombre y forma que cada entidad de negocio debería tener en el
**contrato público** (no en la base de datos interna de cada grupo),
para que dos grupos que hablan de "un pedido" se refieran exactamente a
los mismos campos.

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
| `avatarUrl` | string (uri), nullable | ⚠️ G2 expone `avatar_url` (snake_case) — debe migrar a camelCase (decisión 2026-06-19, ver `conventions.md`); mientras tanto el BFF traduce. |
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
| `categoryId` / `categoryName` | uuid / string | ⚠️ G3 expone snake_case — debe migrar a camelCase (decisión 2026-06-19); mientras tanto el BFF traduce. |
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
| `totalAmount` | `Money` | ⚠️ G4 usa `number/double` + `USD` default — debe ser `integer`, y el enum de moneda debe quedar solo `[CLP]` (decidido 2026-06-19, sin `USD`). |
| `createdAt`/`updatedAt` | date-time | ✅ |

`CartItem`: `itemId, cartId, productId, name, quantity, unitPrice
(Money), subTotal (Money)` — mismos ajustes de Money pendientes.

## Order / OrderItem (dueño: Grupo 5 — Pedidos) 🆕

**Sin contrato real publicado.** Campos a partir de la documentación
local (inconsistente entre sus propios archivos — ver §2.3 de
`matriz-conflictos-contratos.md`):

| Campo canónico | Tipo | Estado real |
|---|---|---|
| `orderId` | string `ORD-YYYYMMDD-NNN` | ⚠️ G5 usa 3 formatos distintos en sus propios documentos; debe converger al formato ya usado por G6/G8. |
| `userId` | uuid | ✅ (asumido) |
| `status` | enum (ver tabla de mapeo abajo) | ⚠️ Mezcla mayúsculas (doc de arquitectura) y minúsculas (ejemplos REST/eventos) dentro del mismo grupo. |
| `items[]` | array de `OrderItem` | ✅ (asumido) |
| `totalAmount` | `Money` | ✅ (ya viene entero en sus ejemplos) |
| `shippingAddress` | objeto (street/city/region/country) | A confirmar contra G5 — hoy el dato de dirección viaja en el payload de creación, no como objeto estructurado. |
| `createdAt`/`updatedAt` | date-time | ✅ (asumido) |

### Mapeo de vocabularios de estado — decidido

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

**Decisión ejecutiva de Grupo 1 (2026-06-19):** se adopta esta tabla como
la propuesta oficial a presentar en la reunión — se lleva como decisión
tomada, no como pregunta abierta, y se ajusta solo si Grupo 5, 6 u 8
levantan una objeción concreta.

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

**Actualizado 2026-06-20:** G6 liberó v1.2, que implementa el modelo
multi-paquete (un pedido puede generar N shipments, uno por cada
`PackageInput` enviado) y ya cumple casi todas nuestras convenciones por
su cuenta. Detalle completo en `services/group-6-despacho/README.md`.

| Campo canónico | Tipo | Estado real (código v1.2) |
|---|---|---|
| `shipmentId` | string `SHP-...` | ✅ |
| `orderId` | string | ✅ (ya **no** es UNIQUE — 1 pedido puede tener N shipments) |
| `customerName`, `address`, `city` | string | ✅ camelCase (vía `alias_generator` de Pydantic) |
| `originCd` | `NORTE/CENTRO/SUR` | 🆕 nuevo en v1.2, centro de distribución de origen del paquete |
| `weightKg`, `volumetricWeight` | number | ✅ |
| `shippingCost` | `Money` (integer, CLP) | ✅ calculado por el motor de tarifas de G6 |
| `status` | `PENDING/IN_TRANSIT/DELIVERED/CANCELLED/FAILED/RETURNED` | ✅ (sin cambios respecto a v1.0) |
| `estimatedDelivery` | date-time, nullable | ✅ |

**Gap puntual (no de convención, sino un bug):** `GET /shipments?orderId=`
devuelve los campos en snake_case (`shipment_id`, `order_id`...) en vez de
camelCase, porque ese endpoint específico construye la respuesta como un
dict plano sin pasar por el modelo Pydantic que usa el resto de los
endpoints. Reportado a G6 — no bloquea, pero el BFF debe normalizar esa
respuesta puntual mientras lo corrigen.

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
