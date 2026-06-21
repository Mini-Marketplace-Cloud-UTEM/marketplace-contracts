# Matriz de contratos y conflictos

**Fecha:** 2026-06-18
**Objetivo:** referencia rápida — qué contrato tiene cada grupo hoy, y en
qué choca contra los demás. Versión en tabla, para consulta rápida en la
reunión.

---

## 1. Inventario: qué contrato existe hoy

| Grupo | Servicio | Fuente real | Estado |
|---|---|---|---|
| 1 | Frontend/BFF | `services/group-1-bff/openapi.yaml` | ✅ Completo, propio |
| 2 | Auth | `services/group-2-auth/openapi.yaml` | ✅ Completo (repo `Grupo2_IdentidadUsuario`) |
| 3 | Catálogo | `services/group-3-catalogo/openapi.yaml` | ✅ Completo (repo `grupo-3-CATALOGO`) |
| 4 | Carro/Checkout/Inventario | `services/group-4-carrito/openapi.yaml` | ✅ Completo (repo `G4`) |
| 5 | Pedidos | — | 🆕 **No existe.** Solo docs locales que se contradicen entre sí. |
| 6 | Despacho | — | ✅ Código real (`G6-Shipment-Service`) implementa **v1.2** (multi-paquete, cotización, camelCase, outbox de eventos) desde el 2026-06-20. Tienen `openapi.yaml` auto-generado en su repo; pendiente subirlo a `marketplace-contracts`. |
| 7 | Reportería | — | 🆕 Solo un `.docx` de arquitectura con contrato embebido, sin OpenAPI real. |
| 8 | Pagos/Notificaciones | `services/group-8-pagos/openapi.yaml` | ✅ Completo (repo `G8-Pagos-y-Notificaciones`) |

---

## 2. Matriz por dimensión

### 2.1 Formato de `errorResponse`

| Grupo | Forma real | Campo de error | ¿Coincide con la convención (`code`)? |
|---|---|---|---|
| 1 (BFF) | `{code, message, details?}` | `code` | ✅ (es la convención propuesta) |
| 2 | `{code, message}` | `code` | ✅ (le falta `details`, no es grave) |
| 3 | `{timestamp, status, code, message, correlationId}` | `code` | ✅ (campos extra, no rompe) |
| 4 | `{error, message, details?}` | **`error`** | ❌ |
| 5 | `{error_code, message}` | **`error_code`** | ❌ (un tercer nombre distinto) |
| 6 | `{code, message, details?, correlationId?}` (v1.2, código real) | `code` | ✅ ya es exactamente la forma corta acordada |
| 7 | `{timestamp, status, code, message, correlationId}` | `code` | ✅ |
| 8 | `{error, message, details?}` | **`error`** | ❌ |

**Choque:** 3 nombres de campo distintos para lo mismo (`code` /
`error` / `error_code`). 5 de 8 ya usan `code` — pedir a G4, G5 y G8 que
se alineen.

### 2.2 Dinero (tipo y moneda)

| Grupo | Tipo | Moneda default | ¿Cumple "entero + CLP"? |
|---|---|---|---|
| 1 (BFF) | `number/float` | CLP (implícito) | ❌ **Nuestro propio contrato no cumple su propia regla** — `ProductSummary.price`, `Cart.totalPrice`, `Order.total`, etc. están declarados `float`, no `integer`. |
| 2 | N/A (no maneja dinero) | — | — |
| 3 | `number` (sin restricción) | CLP (implícito) | ⚠️ Ambiguo, no fuerza integer |
| 4 | `number/double` | **USD** | ❌ |
| 5 | sin tipo declarado (ejemplos vienen enteros) | CLP (implícito) | ⚠️ A confirmar cuando exista contrato real |
| 6 | `integer` (`shippingCost`, calculado por motor de tarifas, v1.2) | CLP | ✅ implementado en código, no solo en el papel |
| 7 | `integer` (`totalAmount`, `amountPaid`) | CLP (implícito) | ✅ |
| 8 | `number/double` | CLP (enum cerrado) | ❌ |

**Choque:** ni nosotros mismos cumplimos la regla que le exigimos a los
demás. Hay que corregir el contrato BFF antes de pedirle a G4/G8 que
corrijan el suyo — si no, perdemos autoridad para exigirlo.

### 2.3 Formato de IDs de negocio (`orderId` y similares)

| Grupo | Formato usado | Ejemplo |
|---|---|---|
| 1 (BFF) | uuid (a cambiar, ver `conventions.md`) | `b3d2a1c0-1234-...` |
| 5 | **3 formatos distintos dentro del mismo grupo** | `ord-20260616-1155` / `ord_20260616_1155` / uuid (en su modelo de BD) |
| 6 | string libre, sin formato impuesto por el servicio; en ejemplos usa el mismo patrón que G8 | `ORD-20260611-001` |
| 7 | código secuencial sin fecha | `ORD-1001` |
| 8 | código con fecha | `ORD-20260611-001` |

**Choque:** ni los grupos que "ya usan código legible" (6, 7, 8) están de
acuerdo en el patrón exacto — G7 usa un secuencial simple (`ORD-1001`),
G6 y G8 usan fecha+secuencial (`ORD-20260611-001`). Hay que fijar **un**
patrón único antes de que más grupos construyan validaciones contra el
formato equivocado.

### 2.4 Naming (camelCase vs snake_case) — no es solo "el BFF traduce"

| Grupo | Casing real en su contrato público |
|---|---|
| 1 (BFF) | camelCase |
| 2 | snake_case |
| 3 | snake_case |
| 4 | **camelCase** |
| 5 | snake_case |
| 6 | **camelCase** (v1.2, vía `alias_generator` de Pydantic) — con una excepción puntual: `GET /shipments?orderId=` todavía devuelve snake_case porque ese endpoint construye la respuesta como dict plano sin pasar por el modelo. Reportado a G6. |
| 7 | camelCase en el envelope de eventos, **snake_case en el evento `ShipmentDelivered`** (inconsistente dentro del mismo documento) |
| 8 | **camelCase** |

**Choque (no detectado en el análisis anterior):** la suposición de que
"todos los backends usan snake_case y el BFF traduce" es **falsa** — G4,
G6 y G8 ya exponen camelCase en su contrato público por su cuenta. Esto
es bueno (cumplen la convención) pero significa que el BFF no puede
aplicar una regla de traducción uniforme a todos los servicios; tiene
que saber, servicio por servicio, si necesita traducir o no.

### 2.5 Paginación — 4 formas distintas en 8 grupos

| Grupo | Forma |
|---|---|
| 1 (BFF) | `{page, pageSize, totalItems, totalPages, hasNextPage, hasPrevPage}` |
| 3 | `{data, pagination: {page, size, total, totalPages, hasNext, hasPrev}}` |
| 4 | N/A (no expone listados paginados) |
| 5 | **Sin paginación** — `GET /users/{user_id}/orders` devuelve un arreglo plano |
| 6 | `{data, pagination: {page, pageSize, total, totalPages, hasNext, hasPrev}}` (v1.2) — ✅ ya coincide exactamente con la convención acordada |
| 8 | `{data, pagination: {page, size, total}}` (sin `totalPages`/`hasNext`/`hasPrev`) |

**Choque adicional:** el `shared/components.yaml` que ya publicamos
define `Pagination` como `{page, pageSize, total, totalPages, hasNext,
hasPrev}` — que **tampoco coincide exactamente** con el `Pagination` que
ya está en nuestro propio `services/group-1-bff/openapi.yaml`
(`totalItems`/`hasNextPage`/`hasPrevPage`). Hay que reconciliar esto
antes de pedirle a los demás que adopten el esquema compartido.

### 2.6 Vocabulario de estado de pedido/pago/envío

| Concepto | Vocabulario | Grupo |
|---|---|---|
| Estado de pedido (BFF) | `PENDING, CONFIRMED, PROCESSING, SHIPPED, DELIVERED, CANCELLED` | 1 |
| Estado de pedido (doc arquitectura) | `CREATED, PAYMENT_PENDING, PAID, STOCK_RESERVED, READY_TO_SHIP, SHIPPED, DELIVERED, CANCELLED, FAILED` | 5 |
| Estado de pedido (ejemplos REST/eventos, **minúsculas**) | `created, paid, cancelled, ...` | 5 (contradice su propio doc de arquitectura) |
| Estado de pago | `PENDING, APPROVED, REJECTED, CANCELLED` | 8 |
| Estado de envío | `PENDING, IN_TRANSIT, DELIVERED, CANCELLED, FAILED, RETURNED` | 6 |

**Choque:** 4 vocabularios distintos para describir el ciclo de vida de
"lo mismo" (un pedido, desde que se crea hasta que llega). Sin una tabla
de equivalencia explícita (propuesta en
`data-dictionary/canonical-models.md`), cada integración va a inventar su
propio mapeo.

---

## 3. Choques específicos entre pares de grupos (no solo vs. convención)

| Choque | Entre | Detalle |
|---|---|---|
| ¿Quién orquesta el checkout? | G4 ↔ G5 ↔ BFF | G4 ya llama a G5 al confirmar checkout; nuestro contrato BFF asume que el BFF orquesta todo — hay que alinear. |
| Documentación vs. código real | G6 (consigo mismo) | Persiste, pero ya no en contra nuestra: hoy (2026-06-20) el código v1.2 implementa más de lo que su propio doc `Documentacion_API_G6_v1.2.md` describe (forma de error y paginación reales son más completas que la prosa). Ver `services/group-6-despacho/README.md`. |
| Bug de naming en un endpoint | G6 (consigo mismo) | `GET /shipments?orderId=` devuelve snake_case mientras el resto de los endpoints de G6 devuelve camelCase — reportado a G6, ver `services/group-6-despacho/README.md`. |
| Documento de diseño vs. contrato real | G2, G3 (parcial), G4 (JSON viejo), G8 | Cada uno tiene un PDF/DOCX/JSON que no coincide con su propio OpenAPI real — riesgo de que alguien implemente mirando el documento viejo. |
| `UserProfile.name` único vs. separado | G2 ↔ BFF | G2 expone `name` único; nuestro contrato pedía `firstName`/`lastName`. |
| Moneda CLP vs. USD en el mismo total | G4 ↔ G6 (cotización envío) | G6 cotiza en CLP entero; G4 suma todo en USD float — el total del carrito quedaría mezclando ambas. |

---

## 4. Decisiones tomadas (2026-06-19) — a validar con los demás grupos

Todo lo que antes era "a decidir en la reunión" ya quedó resuelto como
decisión ejecutiva de Grupo 1. Detalle completo y razonamiento en
`decisiones-ejecutivas-2026-06-19.md`. Resumen:

1. **Error → `code`** (no `error`), forma corta `{code, message, details?,
   correlationId?}`, `details` como arreglo `{field, message}`. G4, G5,
   G8 deben cambiar el campo; G3/G6/G7 deben quitar `timestamp`/`status`.
2. **Dinero `integer`, moneda solo `[CLP]`** (sin `USD`) — ya corregido en
   nuestro propio contrato BFF.
3. **`orderId` = `ORD-YYYYMMDD-NNN`** (con fecha) — G5 y G7 deben migrar.
4. **Paginación `{page, pageSize, total, totalPages, hasNext, hasPrev}`**
   — ya reconciliado entre `shared/components.yaml` y el contrato BFF;
   G3, G6 y G8 deben migrar el suyo.
5. **Tabla de equivalencia de estados** de `canonical-models.md` aprobada
   como propuesta oficial.
6. **G4 orquesta el checkout** — el BFF consume `POST /v1/checkout` de
   G4 en vez de orquestar él mismo.
7. **Naming: camelCase obligatorio** en el contrato público de todos los
   backends (no solo traducción en el BFF) — G2, G3, G5, G6 deben migrar.
8. **G6 integra contra v1.2** (actualizado 2026-06-20 — su v1.0/v1.1
   anteriores quedaron superadas por una release real que ya implementa
   el modelo multi-paquete y cumple naming/error/paginación).
9. **JWT: validación centralizada** vía `POST /auth/validate` de G2,
   confirmado (ver `data-dictionary/estandar-jwt.md`).
