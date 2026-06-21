# Convenciones obligatorias

Estas reglas aplican al **contrato público** que cada grupo expone hacia
los demás (vía REST o eventos). Internamente, cada servicio puede usar lo
que quiera en su propia base de datos.

Origen: detectadas como necesarias durante la revisión de contratos de
los 8 grupos, donde se encontró que cada grupo había definido su propio
formato sin acuerdo previo.

**Estado:** las reglas de este documento son la **decisión ejecutiva de
Grupo 1** tomada el 2026-06-19 (ver `decisiones-ejecutivas-2026-06-19.md`
para el detalle y el razonamiento de cada una). Se presentan en la
próxima reunión para confirmar satisfacción de los demás grupos — no son
un debate abierto, son una propuesta concreta a validar o ajustar.

## IDs

- **Decidido:** IDs de negocio expuestos en el contrato (`orderId`,
  `paymentId`, `shipmentId`, etc.) usan código legible
  `PREFIJO-YYYYMMDD-NNN` (ej. `ORD-20260618-001`), con fecha incluida.
  Ya implementado así por Grupo 6 y Grupo 8 — quedan como están.
- El ID interno de fila en base de datos (PK) puede ser UUID; no tiene
  que coincidir con el ID de negocio expuesto.
- **Acción pendiente de cada grupo:** Grupo 5 debe migrar a este formato
  — hoy usa 3 variantes distintas en sus propios documentos
  (`ord-20260616-1155`, `ord_20260616_1155`, UUID). Grupo 7 debe migrar
  de su secuencial simple (`ORD-1001`) al formato con fecha.

## Dinero

- **Decidido:** siempre `integer`. Nunca `float`/`double`.
- **Decidido:** moneda siempre `CLP` — el enum es `[CLP]`, sin `USD`
  (eliminado también de nuestro propio contrato BFF, que todavía lo
  tenía como opción de flexibilidad futura no necesaria para este
  proyecto). Ver esquema `Money` en `shared/components.yaml`.
- Grupo 4 hoy usa `number/double` y `USD` por defecto — debe corregir.

## Forma de error

- **Decidido:** `{ code, message, details?, correlationId? }` (la
  versión corta, sin `timestamp`/`status`) — ver esquema `Error` en
  `shared/components.yaml`.
- El campo se llama **`code`**, nunca `error`. Grupo 4 y Grupo 8 (en su
  contrato OpenAPI real) usan `error` — deben renombrar.
- Grupo 3, Grupo 6 y Grupo 7 usan la versión larga
  (`timestamp, status, code, message, correlationId`) — deben quitar
  `timestamp`/`status` para alinearse a la versión corta.
- **Decidido:** `details` es un **arreglo de `{field, message}`**, no un
  objeto libre. Grupo 4 y Grupo 8 lo definen como objeto libre — deben
  ajustarlo. Para errores que no son de validación de formulario (ej.
  `INSUFFICIENT_STOCK`), usar una entrada con el campo relevante como
  `field` y el detalle en `message` (ej. `{field: "quantity", message:
  "Disponible: 1, solicitado: 3"}`).

## Naming

- **Decidido:** el contrato público (lo que un grupo expone a otros
  servicios) va en **camelCase**, sin excepción — se exige a todos los
  backends, no solo se delega la traducción al BFF.
- Grupo 4 y Grupo 8 ya cumplen esto en su contrato real. **Grupo 2,
  Grupo 3, Grupo 5 y Grupo 6 deben migrar** su contrato público de
  `snake_case` a `camelCase` (su base de datos interna puede seguir como
  esté, esto es solo sobre lo que exponen hacia afuera).

## Enums

- `UPPER_SNAKE_CASE` (`PENDING`, `IN_TRANSIT`, `DELIVERED`, etc.).

## Paginación

- **Decidido:** `{ data: [...], pagination: { page, pageSize, total,
  totalPages, hasNext, hasPrev } }` — ver esquema `Pagination` en
  `shared/components.yaml`.
- Grupo 3 usa `size` en vez de `pageSize` — debe renombrar.
- Grupo 6 usa `limit`/`offset` en vez de `page`/`pageSize`, y no envuelve
  en `{data, pagination}` — debe migrar.
- Grupo 8 solo tiene `{page, size, total}` (le faltan `pageSize`,
  `totalPages`, `hasNext`, `hasPrev`) — debe completar.
- Grupo 5 no pagina `GET /users/{userId}/orders` — debe agregarlo (ya
  reflejado en el contrato borrador que dejamos en
  `services/group-5-pedidos/openapi.yaml`).

## Fechas

- ISO 8601 / UTC (`2026-06-18T14:30:00Z`).

## Eventos (Pub/Sub)

- Tecnología del curso: "Contratos JSON Pub/Sub".
- Sobre estándar: ver esquema `EventEnvelope` en `shared/components.yaml`
  (`eventId, eventType, version, occurredAt, producer, correlationId,
  payload`).
- El `payload` va en camelCase, igual que el resto del contrato público.

## Despliegue

- Todo microservicio se empaqueta y despliega con **Docker** (ya es
  tecnología obligatoria del curso, no es una regla nueva — se formaliza
  aquí porque no todos los grupos tenían un `Dockerfile` todavía).
- Motivo: con 8 equipos en distintos stacks, Docker es el único
  denominador común para que Render despliegue a todos de la misma
  forma, sin depender de que detecte automáticamente cómo correr cada
  lenguaje.
- Referencia/plantilla: el `Dockerfile` de Grupo 6
  (`G6-Shipment-Service`) es el único confirmado hoy — puede usarse como
  punto de partida para FastAPI; cada grupo adapta según su stack
  (Node/Express, etc.).
- No aplica todavía a grupos sin código (Grupo 5, Grupo 7) — se exige
  desde el momento en que empiecen a implementar su servicio, no antes.

## Versionado de contratos

- Todo cambio que altere la forma de una respuesta (ej. de objeto a
  arreglo) o quite un campo requerido **debe** subir la versión en la URL
  (`/v1/` → `/v2/`), nunca cambiar el contrato en silencio bajo la misma
  versión.
- Todo cambio se sube como Pull Request a este repo
  (`marketplace-contracts`), no se anuncia solo de palabra ni se deja
  solo en un documento de Office.

## Fuente de verdad

- El único contrato válido es el `openapi.yaml` (o `events.yaml`) dentro
  de `services/group-N-x/` en este repo.
- PDF, DOCX, o copias en el repo de servicio propio **no son contrato**
  — son documentación de diseño interna, pueden quedar desactualizadas.

## Orquestación del checkout

- **Decidido:** **Grupo 4 orquesta** el checkout. El BFF no tiene su
  propio `POST /orders` independiente — llama a `POST /v1/checkout` de
  G4, que valida stock, reserva inventario, y (según su propio diseño)
  llama a G5 para crear el pedido. Implica reescribir la sección
  "Orders" del contrato BFF.

## Versión vigente de Grupo 6 (Despacho)

- **Actualizado 2026-06-20:** G6 liberó código que implementa **v1.2**
  (multi-paquete, cotización, camelCase, paginación y forma de error ya
  alineadas a este documento, outbox de eventos). Se integra contra esa
  versión — superó la decisión anterior de quedarse en v1.0, no hace
  falta esperar a una fase 2.
- Pendiente de parte de G6: subir su `openapi.yaml` (ya lo generan
  automáticamente desde FastAPI) a
  `services/group-6-despacho/openapi.yaml`, y corregir el endpoint
  `GET /shipments?orderId=`, que todavía devuelve snake_case en vez de
  camelCase (ver `canonical-models.md`).

## Decisiones confirmadas — ver detalle completo

Todas las decisiones de este documento (más el resto: naming, error,
paginación, moneda, mapeo de estados, JWT) están consolidadas con su
razonamiento en `decisiones-ejecutivas-2026-06-19.md`, listas para
presentar en la reunión y validar satisfacción con los demás grupos.
