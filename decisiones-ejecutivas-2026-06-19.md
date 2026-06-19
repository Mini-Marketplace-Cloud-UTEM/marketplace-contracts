# Decisiones ejecutivas — 2026-06-19

**Tomadas por:** Grupo 1 (Frontend/BFF), en rol de coordinación, sobre
todos los conflictos identificados en `analisis-integration-hell-2026-06-18.md`
y `matriz-conflictos-contratos.md`.

**Cómo usar este documento en la reunión:** cada punto es una decisión
ya tomada, no una pregunta abierta. El objetivo de la reunión es validar
**satisfacción** de cada grupo con lo decidido — si alguien tiene una
objeción concreta y justificada, se ajusta puntualmente, pero el punto de
partida es "esto ya quedó resuelto", no "discutámoslo desde cero".

---

## 1. Orquestación del checkout → **G4 orquesta**

El BFF deja de tener su propio `POST /orders` independiente. En su lugar
llama a `POST /v1/checkout` de G4, que ya valida stock, reserva
inventario y (según su propio diseño) llama a G5 para crear el pedido.

**Por qué:** es lo que G4 y G5 ya construyeron entre ellos sin que
nosotros lo supiéramos — alinearnos ahí es menos trabajo que pedirles a
ambos que deshagan su integración para que el BFF orqueste en su lugar.

**Acción:** Grupo 1 reescribe la sección "Orders" de su contrato.

## 2. Versión vigente de Grupo 6 → **v1.0 ahora, v1.1 fase 2**

Se integra contra v1.0 (envío único, sin cotización, sin eventos reales)
porque es lo único que el código de G6 implementa hoy. v1.1 (multi-
paquete, cotización, eventos) queda como mejora futura sin fecha
comprometida.

**Por qué:** v1.0 ya funciona de punta a punta; v1.1 no tiene una sola
línea de código todavía (motor de tarifas, modelo multi-bodega, patrón
Outbox para eventos — todo por construir). Exigir v1.1 ahora bloquearía
a G1/G4/G5 sin necesidad real.

**Acción:** Grupo 6 corrige su README para reflejar que hoy implementa
v1.0, no v1.1.

## 3. Naming → **camelCase obligatorio en todos los backends**

No se delega solo en que el BFF traduzca. Cada grupo expone su contrato
público en camelCase.

**Por qué:** G4 y G8 ya lo hacen por su cuenta — es alcanzable, y evita
que el BFF tenga que mantener una traducción distinta por cada servicio
para siempre.

**Acción:** Grupo 2, Grupo 3, Grupo 5 y Grupo 6 migran su contrato
público de `snake_case` a `camelCase` (su base de datos interna no
cambia).

## 4. Formato de `orderId` → **`ORD-YYYYMMDD-NNN`** (con fecha)

**Por qué:** Grupo 6 y Grupo 8 ya construyeron contra este formato — es
menos trabajo pedirle a Grupo 5 y Grupo 7 que se alineen que pedirle a
dos grupos que ya funcionan que cambien.

**Acción:** Grupo 5 migra desde sus 3 formatos actuales
(`ord-20260616-1155` / `ord_20260616_1155` / uuid interno). Grupo 7
migra desde su secuencial simple (`ORD-1001`).

## 5. Forma del error → versión corta, campo `code`, `details` como arreglo

```json
{ "code": "RESOURCE_NOT_FOUND", "message": "...", "details": [{"field": "...", "message": "..."}], "correlationId": "..." }
```

**Por qué:** ya es lo que usan BFF y G2; quitar `timestamp`/`status` (que
casi nunca consume el cliente) es menos trabajo que agregarlos a todos
los demás. El campo `code` ya lo usan 5 de 8 grupos. El arreglo
`{field, message}` es más útil para el frontend que un objeto libre sin
estructura.

**Acción:**
- Grupo 4 y Grupo 8: renombrar `error` → `code`, y convertir `details` de
  objeto libre a arreglo `{field, message}`.
- Grupo 3, Grupo 6 y Grupo 7: quitar `timestamp` y `status` de su
  `ErrorResponse`.

## 6. Paginación → `{page, pageSize, total, totalPages, hasNext, hasPrev}`

**Por qué:** ya reconciliado entre `shared/components.yaml` y el
contrato BFF (que antes tampoco coincidían entre sí).

**Acción:**
- Grupo 3: renombrar `size` → `pageSize`.
- Grupo 6: migrar de `limit`/`offset` a `page`/`pageSize`, y envolver la
  respuesta en `{data, pagination}`.
- Grupo 8: agregar `pageSize`, `totalPages`, `hasNext`, `hasPrev` (hoy
  solo tiene `page`, `size`, `total`).
- Grupo 5: agregar paginación a `GET /users/{userId}/orders` (hoy
  devuelve un arreglo plano).

## 7. Moneda → solo `CLP`, sin `USD`

El enum de moneda queda `[CLP]` en todos los esquemas, incluido nuestro
propio contrato BFF (que todavía tenía `[CLP, USD]` "por flexibilidad
futura").

**Por qué:** es un marketplace de un solo país/moneda — no hay necesidad
real de soportar USD, y mantenerlo como opción solo agrega ambigüedad.

**Acción:** Grupo 4 corrige su `currency` default (hoy `USD`).

## 8. Mapeo de estados de pedido/pago/envío → aprobado

| BFF `OrderStatus` | G5 (estado interno) | G8 `Payment.status` | G6 `ShipmentStatus` |
|---|---|---|---|
| `PENDING` | `CREATED` | `PENDING` | — |
| `CONFIRMED` | `PAYMENT_PENDING` → `PAID` | `APPROVED` | — |
| `PROCESSING` | `STOCK_RESERVED` → `READY_TO_SHIP` | — | `PENDING` (despacho) |
| `SHIPPED` | `SHIPPED` | — | `IN_TRANSIT` |
| `DELIVERED` | `DELIVERED` | — | `DELIVERED` |
| `CANCELLED` | `CANCELLED` / `FAILED` | `REJECTED`/`CANCELLED` | `CANCELLED`/`FAILED`/`RETURNED` |

**Por qué:** sin esta tabla, cada integración inventa su propio mapeo
ad-hoc. Se lleva como propuesta concreta a la reunión en vez de un
espacio en blanco.

**Acción:** Grupo 5, Grupo 6 y Grupo 8 confirman o corrigen puntualmente.

## 9. Validación de JWT → centralizada vía G2

Cada servicio llama a `POST /auth/validate` de Grupo 2 antes de procesar
un request autenticado, en vez de verificar la firma localmente.

**Por qué:** G2 ya construyó ese endpoint pensando exactamente en esto;
evita distribuir un secreto compartido entre 7 equipos (riesgo real de
que alguien lo suba a un repo por error).

**Acción:** ninguna nueva — ya está documentado en
`data-dictionary/estandar-jwt.md`.

---

## Despliegue: Docker obligatorio para todo microservicio

Ya estaba en la lista de tecnologías del curso; se formaliza como regla
en `data-dictionary/conventions.md` porque no todos los grupos tenían un
`Dockerfile` todavía. No aplica a Grupo 5 ni Grupo 7 hasta que tengan
código (ver esa sección para el detalle).

---

## Qué preguntar a cada grupo en la reunión

No "¿qué les parece?" en general — preguntar puntualmente si cada punto
les genera algún bloqueo real de implementación. Si no, se considera
aceptado y cada grupo aplica su acción pendiente con plazo a definir.
