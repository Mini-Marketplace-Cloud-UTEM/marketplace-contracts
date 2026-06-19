# Convenciones obligatorias

Estas reglas aplican al **contrato público** que cada grupo expone hacia
los demás (vía REST o eventos). Internamente, cada servicio puede usar lo
que quiera en su propia base de datos.

Origen: detectadas como necesarias durante el análisis de
`analisis-integration-hell-2026-06-18.md`, donde se encontró que cada
grupo había definido su propio formato sin acuerdo previo. Items
marcados como **(pendiente de acordar)** todavía no tienen consenso de
todos los grupos — se resuelven en la reunión.

## IDs

- **IDs de negocio** expuestos en el contrato (`orderId`, `paymentId`,
  `shipmentId`, etc.): código legible `PREFIJO-YYYYMMDD-NNN`
  (ej. `ORD-20260618-001`). Ya implementado así por Grupo 6 y Grupo 8.
- El ID interno de fila en base de datos (PK) puede ser UUID; no tiene
  que coincidir con el ID de negocio expuesto.
- **(pendiente de acordar)** Grupo 5 debe migrar a este formato — hoy usa
  3 variantes distintas en sus propios documentos (`ord-20260616-1155`,
  `ord_20260616_1155`, UUID).

## Dinero

- Siempre `integer`. Nunca `float`/`double`.
- Moneda siempre `CLP` (ver esquema `Money` en `shared/components.yaml`).
- Grupo 4 hoy usa `number/double` y `USD` por defecto — debe corregir.

## Forma de error

- `{ code, message, details?, correlationId? }` — ver esquema `Error` en
  `shared/components.yaml`.
- El campo se llama **`code`**, nunca `error`.
- **(pendiente de acordar)** Grupo 4 y Grupo 8 (en su contrato OpenAPI
  real) usan `error` en vez de `code` — deben renombrar.

## Naming

- El contrato público (lo que un grupo expone a otros servicios) va en
  **camelCase**.
- Internamente cada grupo puede usar `snake_case` en su propia base de
  datos — es trabajo del BFF traducir si el servicio expone snake_case
  hacia afuera y no puede cambiarlo a tiempo.

## Enums

- `UPPER_SNAKE_CASE` (`PENDING`, `IN_TRANSIT`, `DELIVERED`, etc.).

## Paginación

- `{ data: [...], pagination: { page, pageSize, total, totalPages,
  hasNext, hasPrev } }` — ver esquema `Pagination` en
  `shared/components.yaml`.
- Grupo 6 usa `limit`/`offset` en vez de `page`/`pageSize` — debe migrar.

## Fechas

- ISO 8601 / UTC (`2026-06-18T14:30:00Z`).

## Eventos (Pub/Sub)

- Tecnología del curso: "Contratos JSON Pub/Sub".
- Sobre estándar: ver esquema `EventEnvelope` en `shared/components.yaml`
  (`eventId, eventType, version, occurredAt, producer, correlationId,
  payload`).
- El `payload` va en camelCase, igual que el resto del contrato público.

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

## Decisiones abiertas para la próxima reunión

1. ¿Quién orquesta el checkout? (Grupo 4 vs Grupo 5 vs BFF) — ver
   `analisis-integration-hell-2026-06-18.md` §4.7.
2. Formato único de `orderId` (recomendado: adoptarlo como ya está
   definido arriba).
3. Grupo 6: ¿se queda en v1.0 o termina v1.1 del contrato de despacho?
