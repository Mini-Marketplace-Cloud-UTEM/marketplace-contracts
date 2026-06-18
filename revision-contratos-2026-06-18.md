# Revisión de compatibilidad de contratos — Grupo 1 (Frontend/BFF)

**Fecha:** 2026-06-18
**Preparado por:** Grupo 1 (Frontend / BFF)
**Objetivo:** Identificar qué hay que cambiar, acordar, quitar o agregar en los
contratos de cada grupo para que el desarrollo pueda avanzar con un contrato
API claro entre todos los equipos.

**Contratos revisados:**

| Grupo | Servicio | Repo | Estado del contrato |
|---|---|---|---|
| 1 | Frontend/BFF (nosotros) | `Grupo-1-Front` | Completo |
| 2 | Auth | `Grupo2_IdentidadUsuario` | Completo |
| 3 | Catálogo | `grupo-3-CATALOGO` | Completo |
| 4 | Carro/Checkout/Inventario | `G4` | Completo |
| 5 | Pedidos | `Grupo5` | **No existe** (solo README) |
| 6 | Despacho | — | **Repo no visible aún** |
| 7 | Reportería | — | **Repo no visible aún** |
| 8 | Pagos y Notificaciones | `G8-Pagos-y-Notificaciones` | Completo |

---

## 🔴 Bloqueantes (impiden avanzar con el flujo de checkout)

### 1. ¿Quién orquesta el checkout: el BFF o el Grupo 4?

- El contrato del BFF describe que **el BFF orquesta** la compra: valida
  stock con G3 → crea el pedido en G5 → limpia el carro en G4.
- El contrato de G4 ya incluye `POST /v1/checkout`, que internamente valida
  stock, reserva inventario por 15 min y **se integra directamente con Order
  Service (Grupo 5)**.
- Hoy ambos contratos asumen que son los dueños de esa orquestación.
- **Decisión necesaria:** ¿el BFF llama a `/v1/checkout` de G4 (y G4 habla
  con G5), o el BFF orquesta directo contra G3/G5/G4 y G4 solo expone
  operaciones atómicas (cart, inventory)? Esto define todo el flujo de compra
  y afecta a G4 y G5 directamente.
- **Con quién:** Grupo 4 y Grupo 5.

### 2. Grupo 5 (Pedidos) no tiene contrato

- Bloquea poder cerrar el punto anterior y validar el formato de `orderId`
  (ver punto 3) y el vocabulario de estados de pedido.
- **Con quién:** Grupo 5. Pedirles que prioricen subir el contrato antes de
  la próxima reunión.

### 3. Formato de `orderId`: ¿UUID o código legible?

- Nuestro contrato BFF define `Order.id` como `format: uuid`.
- G8 (Pagos) ya usa `orderId` como código humano: `"ORD-20260611-001"`.
- Como el ID nace en Grupo 5 (que aún no tiene contrato), hay que fijar este
  formato **ahora**, antes de que cada grupo lo defina por su cuenta.
- **Con quién:** Grupo 5 (dueño del ID), Grupo 8 (ya lo consume).

### 4. Modelo de carrito incompatible (cartId explícito vs carro implícito)

- G4 exige crear el carro explícitamente (`POST /v1/cart` → `cartId`) y todo
  se referencia por ese ID.
- **No existe en G4 un endpoint para "dame mi carro activo"** sin conocer el
  `cartId` de antemano.
- Nuestro `GET /cart` asume que el carro es implícito a partir del JWT (sin
  que el frontend conozca ningún `cartId`).
- **Opciones a proponer:** (a) G4 agrega `GET /v1/users/{userId}/cart` o
  equivalente, o (b) el BFF guarda el `cartId` por usuario en su propia
  sesión/DB y lo gestiona de forma transparente para el frontend.
- **Con quién:** Grupo 4.

---

## 🟡 Convenciones a corregir (violan las reglas no negociables del proyecto)

### 5. Dinero como float en vez de entero (CLP)

- Afecta a **G4** (`totalAmount`, `unitPrice`, `subTotal` como
  `number/double`) y **G8** (`amount` como `number/double`).
- Nuestra convención: dinero siempre entero, CLP no tiene decimales.
- **Acción:** pedir a ambos grupos que cambien esos campos a `integer`, o
  el BFF redondea/castea al traducir (parche, no resuelve el problema de
  fondo si ellos hacen cálculos con decimales internamente).
- **Con quién:** Grupo 4, Grupo 8.

### 6. Moneda por defecto `USD` en G4

- El checkout de G4 usa `currency: "USD"` por defecto. Todo el marketplace
  opera en CLP.
- **Con quién:** Grupo 4.

### 7. Campo de error inconsistente: `error` vs `code`

- Nuestra convención: `{ code, message, details? }`, y el frontend nunca
  debe hacer lógica sobre `message`.
- **G2** ya usa `{ code, message }` ✅ — compatible.
- **G3** usa `{ timestamp, status, code, message, correlationId }` ✅ —
  compatible (campos extra no son problema).
- **G4 y G8** usan `{ error, message, details? }` ❌ — el campo se llama
  `error`, no `code`.
- **Acción:** pedir a G4 y G8 que renombren `error` → `code` para
  homogeneizar, o documentar explícitamente que el BFF debe traducirlo
  (menos ideal: significa lógica de traducción duplicada en cada llamada).
- **Con quién:** Grupo 4, Grupo 8.

---

## 🟢 Gaps funcionales (cosas que falta agregar a algún contrato)

### 8. Falta endpoint de categorías con jerarquía en G3

- Nuestro BFF expone `GET /categories` con árbol (`parentId`, `children`).
- G3 solo tiene `category_id` + `category_name` planos dentro de cada
  producto, sin endpoint de categorías ni jerarquía.
- **Con quién:** Grupo 3.

### 9. Falta soporte de filtros de precio y orden en G3

- Nuestro `GET /products` expone `minPrice`, `maxPrice`, `sortBy`.
- G3 no los soporta en `/products` ni en `/products/search`.
- **Con quién:** Grupo 3.

### 10. Búsqueda dividida en dos endpoints en G3

- G3 separa `GET /products` (listado) de `GET /products/search` (búsqueda
  por texto, `q` con `minLength: 2`), sin poder combinar búsqueda + filtros
  de categoría/precio en una sola llamada.
- Nuestro `GET /products` combina búsqueda + filtros + paginación en un solo
  endpoint. El BFF tendría que orquestar dos llamadas a G3 y fusionar
  resultados, lo cual es frágil con paginación.
- **Con quién:** Grupo 3 — proponer unificar en un solo endpoint con todos
  los filtros opcionales.

### 11. Nuestro BFF no tiene endpoints de pago ni notificaciones

- G8 expone notificaciones explícitamente para el frontend ("usado
  principalmente por el Frontend (Grupo 1)"), pero nuestro contrato BFF no
  tiene ningún endpoint de pagos ni notificaciones.
- **Acción:** agregar a nuestro contrato BFF endpoints que proxyen a G8
  (ej. `GET /notifications`, y el paso de pago dentro del flujo de
  checkout/orders).
- **Con quién:** interno (Grupo 1), una vez resuelto el punto 1.

### 12. Vocabulario de estados distinto entre pago y pedido

- `Payment.status` (G8): `PENDING / APPROVED / REJECTED / CANCELLED`.
- `OrderStatus` (BFF): `PENDING / CONFIRMED / PROCESSING / SHIPPED /
  DELIVERED / CANCELLED`.
- Falta un mapeo explícito y documentado (ej. `APPROVED` → `CONFIRMED`).
- **Con quién:** Grupo 5 (una vez tenga contrato) y Grupo 8.

### 13. `UserProfile.name` único vs `firstName`/`lastName` separados

- G2 modela el nombre del usuario como un solo campo `name`.
- Nuestro contrato BFF espera `firstName` y `lastName` separados.
- Partir un nombre completo de forma confiable no es trivial (nombres
  compuestos, apellidos múltiples, etc.).
- **Acción:** negociar con G2 si dividen el campo, o el BFF/frontend se
  adapta a un solo campo `name`.
- **Con quién:** Grupo 2.

### 14. Falta campo de rol/permisos en `UserProfile` del BFF

- G2 expone `roles: [guest, customer, seller, admin]` por usuario.
- Nuestro `UserProfile` no contempla rol alguno.
- Si el frontend necesita diferenciar vistas (ej. funciones de vendedor o
  admin), falta este campo en nuestro contrato.
- **Con quién:** interno (Grupo 1), a definir según alcance del frontend.

### 15. `refreshToken` viaja en el body en G2, no en cookie

- Nuestro contrato asume que el frontend nunca ve el refresh token (debe ir
  en cookie HttpOnly).
- G2 retorna `refresh_token` en el body de `LoginResponse` y también lo
  exige en el body de `RefreshRequest`.
- No es un conflicto bloqueante: el BFF puede recibir el token de G2 en el
  body, guardarlo en una cookie HttpOnly propia, y reenviarlo en el body al
  llamar a `/auth/refresh` de G2. Se documenta para que quede claro que es
  responsabilidad del BFF, no del frontend.
- **Con quién:** informativo, sin acción requerida de Grupo 2.

---

## Pendientes de visibilidad

- **Grupo 6 (Despacho)** y **Grupo 7 (Reportería)**: sus repos no están
  visibles en la organización GitHub. Pedir que los hagan públicos (o
  compartan acceso) antes de la próxima reunión para poder revisar sus
  contratos.

---

## Resumen para la reunión (orden sugerido de discusión)

1. Dueño de la orquestación del checkout — **Grupo 4 + Grupo 5** (bloqueante).
2. Contrato de Grupo 5 — prioridad máxima, todo lo demás depende de él.
3. Formato de `orderId` (UUID vs código legible) — Grupo 5 + Grupo 8.
4. Modelo de carrito (`cartId` explícito vs carro implícito) — Grupo 4.
5. Dinero como entero (CLP) y moneda por defecto — Grupo 4 + Grupo 8.
6. Unificar campo de error (`code` vs `error`) — Grupo 4 + Grupo 8.
7. Gaps de catálogo (categorías, filtros de precio, orden, búsqueda
   combinada) — Grupo 3.
8. Endpoints de notificaciones/pago en el contrato BFF — interno, post
   punto 1.
9. Campo `name` único vs `firstName`/`lastName`, y rol de usuario — Grupo 2.
10. Visibilidad de los repos de Grupo 6 y Grupo 7.
