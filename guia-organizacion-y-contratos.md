# Guía rápida — Organización, Repos y Contratos

**Para:** líderes de los 8 grupos
**De:** Grupo 1 (Frontend / BFF)
**Fecha:** 2026-06-18
**Objetivo:** que todos entendamos cómo está armada la organización, cómo
nos conectamos entre servicios, y dónde tiene que vivir cada contrato para
dejar de tener versiones desincronizadas.

---

## 1. Cómo está organizada la org de GitHub

- Organización: `Mini-Marketplace-Cloud-UTEM`.
- Cada grupo tiene **su propio repo de servicio** (su código vive ahí, eso
  no cambia).
- Existe además **un repo separado y compartido por todos**:
  **`marketplace-contracts`**. Este repo no tiene código — solo contratos
  (OpenAPI/JSON Schema) y las convenciones que todos debemos respetar.
- Repos detectados hoy en la org:
  - `Grupo-1-Front` (nosotros — Frontend/BFF)
  - `Grupo2_IdentidadUsuario` (Auth)
  - `grupo-3-CATALOGO` (Catálogo)
  - `G4` (Carro/Checkout/Inventario)
  - `Grupo5` (Pedidos — **sin contrato subido todavía**)
  - `G6-Shipment-Service` (Despacho — **el código real implementa v1.0,
    pero su propia documentación/README dicen que ya está en v1.1**, hay
    que aclarar cuál es la versión vigente)
  - `Grupo-7-Reporter-a-bash-y-Streaming` (Reportería)
  - `G8-Pagos-y-Notificaciones` (Pagos/Notificaciones)
  - `marketplace-contracts` (el repo de contratos compartido)

## 2. Cómo nos conectamos entre servicios (arquitectura)

- El **frontend (React) habla únicamente con el BFF** (Grupo 1). Nunca le
  habla directo a los servicios de otros grupos.
- El **BFF (Grupo 1) sí habla con todos los demás servicios**: traduce y
  orquesta las llamadas entre el frontend y cada grupo (2 al 8).
- Esto significa que **el contrato Frontend↔BFF lo controla 100% Grupo 1**,
  pero **el contrato BFF↔Servicio lo controla cada grupo dueño del
  servicio**, y el BFF se adapta a esos contratos (traduce nombres de
  campos, formatos, etc.) — no al revés.
- Aparte de las llamadas síncronas (REST), hay un canal **asíncrono vía
  eventos** (Pub/Sub en JSON, según la guía de tecnologías del curso) para
  cosas como notificaciones y reportería (ej. cuando se crea un pedido o
  se aprueba un pago).

## 3. DÓNDE tiene que vivir el contrato de cada grupo

Esto es lo más importante a resolver en la reunión: **hoy cada grupo tiene
el contrato repartido en 2 a 4 lugares distintos (su propio repo, un PDF,
un DOCX, a veces un YAML suelto) y esas copias no coinciden entre sí.**

A partir de ahora:

- ✅ **El único contrato válido y vinculante es el que está en
  `marketplace-contracts/services/group-N/openapi.yaml`** (o el nombre de
  archivo equivalente, ya estandarizado por carpeta de grupo).
- ❌ Los PDF, DOCX, o YAML/JSON sueltos en Drive/OneDrive **no son
  contrato** — son solo documentación de diseño interna del grupo, pueden
  quedar desactualizados y nadie más debe asumir que reflejan la API real.
- ❌ Tener el contrato solo en el repo de servicio propio (sin subirlo
  también a `marketplace-contracts`) **tampoco sirve** — los demás grupos
  no deberían tener que entrar a buscar en 7 repos distintos para saber
  contra qué integrar.
- 🔁 **Todo cambio al contrato se sube como Pull Request a
  `marketplace-contracts`**, con al menos un grupo consumidor como
  revisor. Si el cambio rompe compatibilidad (ej. una respuesta que antes
  era un objeto y ahora es un arreglo), se debe **subir la versión en la
  URL** (`/v1/` → `/v2/`) y avisar a los grupos que dependen de ese
  endpoint.

## 4. Reglas no negociables (resumen para todos los grupos)

Estas reglas aplican al contrato que cada grupo expone hacia los demás
(internamente, cada servicio puede usar lo que quiera en su propia base de
datos):

| Regla | Detalle |
|---|---|
| IDs de negocio | Código legible `PREFIJO-YYYYMMDD-NNN` (ej. `ORD-20260618-001`) para entidades como pedido, pago, envío. El ID interno de fila en la base de datos puede ser UUID, pero el campo expuesto en el contrato debe ser el código legible. |
| Dinero | Siempre `integer`, moneda `CLP`. Nunca `float`/`double`, nunca `USD` como default. |
| Errores | Forma estándar `{ code, message, details? }`. El campo se llama `code`, no `error`. |
| Naming | El contrato público (lo que un grupo expone a otros) va en `camelCase`. Internamente cada grupo puede usar `snake_case` en su propia base de datos. |
| Enums | `UPPER_SNAKE_CASE` (`PENDING`, `IN_TRANSIT`, etc.) |
| Paginación | `{ data: [...], pagination: { page, pageSize, total, totalPages, hasNext, hasPrev } }` |
| Fechas | ISO 8601 / UTC (`2026-06-18T14:30:00Z`) |

## 5. Qué traer a la reunión / qué necesitamos decidir juntos

- **Grupo 5 (Pedidos):** subir su contrato real a `marketplace-contracts`
  antes de la reunión si es posible — hoy es el único grupo sin ningún
  contrato versionado en GitHub, y bloquea a varios equipos.
- **Grupo 6 (Despacho):** su repo (`G6-Shipment-Service`) ya es público,
  pero tiene 4 versiones del contrato que no coinciden (v1.0, v1.1, una
  guía interna, y el código real) — y **el código real solo implementa
  v1.0** (sin cotización, sin eventos publicados de verdad) aunque el
  README diga que ya está en v1.1. Necesitan decidir y comunicar cuál es
  la versión vigente, y subirla a `marketplace-contracts`.
- **Decidir quién orquesta el flujo de checkout** (Grupo 4 vs Grupo 5 vs
  el BFF) — hay una propuesta concreta de Grupo 1 basada en lo que G4 y G5
  ya construyeron entre ellos, se presenta en la reunión.
- **Confirmar el formato único de `orderId`** en todo el sistema.
- **Grupo 4 y Grupo 8:** confirmar el cambio de `error` → `code` en sus
  respuestas de error, y de moneda/montos a CLP entero.
- Cada líder de grupo debería llegar sabiendo **dónde está su propio
  contrato real hoy** (repo + archivo) para subirlo a
  `marketplace-contracts` ese mismo día si todavía no está ahí.

---

📄 Documentos de respaldo con el detalle completo de hallazgos:
- `revision-contratos-2026-06-18.md` — comparación contrato BFF vs cada grupo.
- `analisis-integration-hell-2026-06-18.md` — análisis completo de
  desincronización y plan de remediación.
