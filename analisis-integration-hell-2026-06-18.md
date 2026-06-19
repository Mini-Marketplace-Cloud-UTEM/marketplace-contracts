# Análisis de "Integration Hell" y Plan de Remediación

**Fecha:** 2026-06-18 (actualizado el mismo día tras hacerse público el repo de Grupo 6)
**Preparado por:** Grupo 1 (Frontend / BFF)
**Fuentes revisadas:** los 9 repos públicos de la organización GitHub
(`Mini-Marketplace-Cloud-UTEM`, incluyendo el código fuente real de
`G6-Shipment-Service`) + toda la documentación local en
`OneDrive/.../Arquitectura/` (PDFs, DOCX, YAML, JSON, TEX) de cada grupo.

Este documento reemplaza/extiende a `revision-contratos-2026-06-18.md`
(que solo comparaba contra GitHub). Acá se incluye también la
documentación "fuente" de cada grupo (la que no está en GitHub), y se
encontró que **el problema no es solo que los contratos no calzan entre
grupos — varios grupos tienen contratos que no calzan ni consigo mismos.**

---

## 1. Resumen ejecutivo

No existe una única fuente de verdad. Cada grupo tiene su contrato repartido
entre 2 y 4 lugares distintos (repo de servicio propio, OneDrive en PDF,
OneDrive en DOCX, a veces un YAML/JSON suelto), y **estas copias no están
sincronizadas entre sí**. El repo `marketplace-contracts`, que debía ser el
punto único de verdad, está prácticamente vacío — nadie lo está usando.

Esto generó tres niveles de problema:

1. **Desincronización interna** (un grupo contradice su propio documento):
   Grupo 5 usa 3 formatos de `orderId` distintos en sus propios archivos;
   Grupo 6 cambió de v1.0 a v1.1 con cambios incompatibles sin changelog
   explícito — y peor aún, **su código real implementado no es ni v1.0 ni
   v1.1 completos: es v1.0 en la práctica, mientras su propio README dice
   que el contrato vigente es v1.1** (ver §4.5); Grupo 4 tiene un JSON
   suelto que es una versión vieja (sin `/v1`, moneda USD) distinta de su
   YAML actual (con `/v1`, CLP).
2. **Desincronización entre lo que un grupo "cree" del otro y la realidad**:
   Grupo 6 escribió su propia interpretación de los contratos de G1/G4/G5/G7/G8
   (carpeta `v1.1/*.md`), y esa interpretación ya difiere de los contratos
   reales en varios puntos (ver §4.6).
3. **Desincronización de convenciones globales**: cada grupo definió su
   propio formato de ID, de error y de moneda sin mirar lo que definieron
   los demás (ver §4).

---

## 2. Inventario de fuentes por grupo

| Grupo | Repo GitHub | Contrato en GitHub | Copias locales (OneDrive) | ¿Coinciden entre sí? |
|---|---|---|---|---|
| 1 (nosotros) | `Grupo-1-Front` | `marketplace-bff-api.yaml` (aún sin commitear en `marketplace-contracts`) | — | N/A |
| 2 (Auth) | `Grupo2_IdentidadUsuario` | `ContratoRestGrupo2` (OpenAPI) | `Contrato Rest.docx`, `arquitectura_identity_service_E1.pdf` | ❌ El docx es una versión vieja (camelCase, sin `roles[]`, sin `phone`) |
| 3 (Catálogo) | `grupo-3-CATALOGO` | `contrato` (OpenAPI) | `catalogo-productos-contrato.pdf`, `swagger_catalogo.yaml`, `Evento de Catalogo Schema.json` | ❌ El PDF es conceptual/viejo; el YAML local sí coincide con GitHub; el evento JSON usa `visibleStock` mientras el PDF usa `stockVisible` |
| 4 (Carro/Checkout) | `G4` | `Contrato G4 .YAML` (OpenAPI) | mismo YAML (idéntico ✅), `Contrato G4.JSON`, `E1_Arquitectura_de_Software.pdf` | ❌ El JSON es la v1.0 (sin `/v1`, USD); el YAML es v2.0 (con `/v1`, CLP) |
| 5 (Pedidos) | `Grupo5` | **No existe**, solo README | Arquitectura, Contrato REST y Contrato de eventos (cada uno en .md + .pdf + .docx) | ❌ Ver §4.1 — el propio grupo se contradice en formato de ID, casing de estados y nombre de eventos |
| 6 (Despacho) | `G6-Shipment-Service` (se hizo público recién) | Código FastAPI real (`app/main.py`, `schemas.py`, `models.py`) — implementa el modelo **v1.0** | v1.0 (tex+pdf), v1.1 (pdf) del contrato, `G6_Interno.md` (guía de ingeniería interna que describe v1.1), + 5 `.md` de "interpretación" de otros contratos, README propio | ❌ **4 versiones distintas y ninguna coincide con el código real** — ver §4.5 |
| 7 (Reportería) | `Grupo-7-Reporter...` | Solo un `.docx` de arquitectura (contiene un contrato REST + de eventos embebido, no como OpenAPI real) | — | N/A (es la única fuente) |
| 8 (Pagos/Notif.) | `G8-Pagos-y-Notificaciones` | `G8_Contratos.yaml` (OpenAPI) | `Contratos Grupo 8 ArqSoft.docx` | ❌ El docx usa `code` en el error, el YAML real usa `error`; el docx usa `message`/`read` en notificaciones, el YAML usa `title`/`body`/`status` |

**Conclusión del inventario:** de 8 grupos, **6 tienen al menos dos versiones
del contrato que no coinciden entre sí**. Grupo 5 y Grupo 6 ni siquiera
tienen el contrato en GitHub — viven solo en documentos de Office.

---

## 3. Causas raíz

1. **No hay un repositorio único obligatorio.** `marketplace-contracts`
   existe pero ningún grupo lo usa; cada uno mantiene "su" contrato en su
   propio repo de servicio, y además en documentos de Office que nadie
   vuelve a actualizar cuando el contrato cambia.
2. **Los documentos de Office (PDF/DOCX) se tratan como contrato**, cuando
   en la práctica quedan congelados en el momento en que se exportaron y
   nunca se vuelven a tocar — mientras el YAML/JSON sigue evolucionando.
3. **Cambios de versión sin comunicarlos** (Grupo 6 v1.0→v1.1, Grupo 4
   JSON viejo nunca borrado) — no hay changelog ni aviso a los consumidores.
4. **Cada grupo decidió por su cuenta** el formato de ID, el nombre del
   campo de error y el tipo de dato para dinero, sin un acuerdo previo
   transversal (las "convenciones no negociables" solo existían en el
   `CLAUDE.md` de Grupo 1, no en un documento compartido por todos).
5. **Integraciones basadas en suposiciones, no en el contrato real**:
   Grupo 6 escribió su propia interpretación de los contratos de otros 5
   grupos en vez de referenciar directamente sus OpenAPI — y ya quedó
   desactualizada.

---

## 4. Catálogo consolidado de conflictos

### 4.1 `orderId` — tres formatos distintos en el ecosistema

| Quién | Formato | Fuente |
|---|---|---|
| BFF (nosotros) | UUID | `marketplace-bff-api.yaml` |
| Grupo 8 (Pagos) | `ORD-20260611-001` | YAML real en GitHub |
| Grupo 6 (Despacho) | `ORD-20260611-001` (mismo que G8) | v1.0/v1.1 |
| Grupo 5 (Pedidos) | `ord-20260616-1155` (Contrato REST), `ord_20260616_1155` (Contrato de eventos), `UUID` (su propio modelo de BD) | 3 formatos **dentro del mismo grupo** |

**Recomendación:** estandarizar en el formato que YA usan G6 y G8
(`ORD-YYYYMMDD-NNN`, mayúsculas, con guion), porque son 2 grupos los que ya
construyeron contra ese formato. Grupo 5 debe alinear su contrato (interno
puede seguir usando UUID como PK de base de datos, pero el campo público
`orderId` debe ser ese código). El BFF debe cambiar `Order.id` de
`format: uuid` a `string` con ese patrón.

### 4.2 Dinero: float vs entero, CLP vs USD

| Grupo | Tipo | Moneda default |
|---|---|---|
| BFF (regla) | integer | CLP |
| Grupo 3 | `number` (sin restricción) | CLP (implícito) |
| Grupo 4 | `number/double` | **USD** |
| Grupo 6 (cotización envío, v1.1) | integer | CLP ✅ |
| Grupo 8 | `number/double` | CLP (enum cerrado) |

**Recomendación:** todos los montos deben ser `integer` y la moneda
default debe ser `CLP` en todo el sistema. Grupo 4 es el más alejado de la
convención (USD float) y debe corregir ambos campos.

### 4.3 Forma del error: 3 variantes

| Variante | Quién la usa |
|---|---|
| `{code, message, details?}` | BFF (G1), Grupo 2 (real), Grupo 3 |
| `{error, message, details?}` | Grupo 4, Grupo 8 (real) |
| `{timestamp, status, code, message, correlationId}` | Grupo 3 (coincide en `code`), Grupo 5, Grupo 6, Grupo 7 |

**Recomendación:** 5 de 8 grupos ya usan el campo `code` (no `error`).
Pedir a **Grupo 4 y Grupo 8** que renombren `error` → `code` para
homogeneizar. Adoptar como forma estándar del proyecto:
`{code, message, details?, correlationId?}` (timestamp/status quedan
como opcionales, ya que no todos los necesitan).

### 4.4 Naming (camelCase vs snake_case)

Todos los servicios backend (G2, G3, G4, G5, G6, G8) usan `snake_case` en
sus payloads reales — es esperado y correcto, **siempre que el BFF
traduzca**. Esto no es un conflicto a resolver entre grupos, es
exactamente el trabajo que le corresponde al BFF. Único caso a vigilar:
Grupo 7 mezcla camelCase (eventos propios) con snake_case (evento
`ShipmentDelivered` reenviado de G6 sin normalizar) dentro del mismo
documento — confirmar que cada consumidor normalice ese evento
específico.

### 4.5 Grupo 6: cuatro versiones del contrato, y el código real implementa la más vieja

G6 es el caso más grave de desincronización interna del proyecto. Existen
**cuatro fuentes distintas**, y no coinciden entre sí:

1. **v1.0** (tex/pdf): 1 pedido = 1 envío. `POST /shipments` devuelve **un
   objeto**. `order_id` es `UNIQUE` (relación 1:1).
2. **v1.1** (tex/pdf): 1 pedido = N cajas (multi-origen). `POST /shipments`
   devuelve **un arreglo** de IDs. Se **elimina el constraint UNIQUE**.
   `GET /shipments/{id}` y `GET /shipments?order_id=` pasan de devolver un
   objeto a devolver un arreglo — ruptura de contrato sin cambiar la URL
   (`/v1/` se mantiene igual en ambas versiones). Agrega "Estado Global"
   del pedido (`PARTIALLY_DELIVERED`, `HAS_ISSUES`) que delega al
   consumidor (G1/BFF) calcularlo a partir del arreglo de cajas.
3. **`G6_Interno.md`** (guía de ingeniería interna del equipo): describe
   el modelo multi-origen de v1.1 con más detalle todavía (peso
   volumétrico, zonas de tarifa, patrón Outbox, `package_index`/
   `total_packages` calculado con `ROW_NUMBER()`), o sea, asume que v1.1
   es lo que hay que construir.
4. **El código real** (`app/main.py`, `schemas.py`, `models.py` en el repo
   `G6-Shipment-Service`, recién hecho público): **implementa el modelo
   v1.0**, no v1.1. Evidencia concreta:
   - `models.py`: `order_id = Column(String, unique=True, ...)` — la
     constraint UNIQUE que v1.1 dice haber eliminado **sigue activa**.
   - No existe el endpoint `POST /api/v1/shipments/quotes` (cotización) en
     `main.py`, aunque el propio `README.md` del repo lo lista en la tabla
     "Resumen del Contrato de la API" como si ya existiera.
   - `ShipmentCreate` solo acepta `order_id, customer_name, address, city,
     weight_kg` — no existe `packages[]`, `origin_cd` ni
     `volumetric_weight`.
   - `GET /api/v1/shipments/{id}` y `GET /api/v1/shipments?order_id=`
     devuelven **un objeto único**, no un arreglo.
   - No hay tabla `outbox_events` ni `shipment_status_history` en
     `models.py`, ni código que publique eventos a ningún broker — pese a
     que el README dice "publica actualizaciones en el tópico Kafka
     `shipment-events`". **Hoy no se emite ningún evento real.**
   - La paginación de `GET /api/v1/shipments` usa `limit`/`offset`, no
     `page`/`pageSize` como pide la convención del proyecto.
   - El prefijo real de todas las rutas es `/api/v1/shipments`, no
     `/v1/shipments` como asumían nuestros documentos anteriores.

**Conclusión:** el propio README de G6 dice "el servicio implementa los
estándares definidos en el contrato oficial" y describe ahí mismo
endpoints de v1.1 (quotes, multi-origen, eventos Kafka) — pero el código
que está en el mismo repo, dos carpetas más abajo, implementa v1.0.
**Para integrar contra G6 hoy, hay que usar v1.0** (objeto único,
`order_id` único, sin cotización, sin eventos reales), no v1.1, sin
importar lo que diga el README o los documentos de diseño.

**Recomendación:** G6 debe (a) corregir su README para que refleje lo que
el código realmente hace hoy, (b) decidir si construye v1.1 o se queda en
v1.0 antes de la próxima reunión, y (c) si construye v1.1, versionar la
URL (`/v1/` → `/v2/`) porque el cambio de forma de respuesta (objeto →
arreglo) rompe a cualquiera que integre hoy contra v1.0.

### 4.6 Grupo 6 cree cosas sobre otros grupos que no son ciertas

La carpeta `v1.1/*.md` de G6 es su propia interpretación de los contratos
de G1, G4, G5, G7, G8. Comparado contra los contratos reales:

- **Sobre G1 (nosotros):** G6 asume que el **frontend llama directo** a su
  API de despacho y calcula ahí el "estado global" del pedido. Esto viola
  la arquitectura BFF (el frontend solo habla con el BFF) y agrega lógica
  de negocio a nuestro lado que nunca se acordó.
- **Sobre G4:** G6 asume que la cotización de envío (`POST
  /shipments/quotes`) responde en CLP entero — correcto y compatible —
  pero G4 real sigue en USD float, así que el total del carrito quedaría
  con dos monedas/tipos mezclados.
- **Sobre G8:** G6 asume comunicación 100% asíncrona y confía en que G5 ya
  validó el pago antes de pedir el despacho — supuesto no verificado con
  G8 ni G5.
- **Sobre G5:** G6 documenta un endpoint de cancelación
  `PATCH /shipments/{id}/cancel` en su interpretación de G5, que
  **contradice su propio contrato general**, donde la cancelación es
  `PATCH /shipments/{id}` con `{"status": "CANCELLED"}` — G6 se contradice
  a sí mismo.

### 4.7 ¿Quién orquesta el checkout? — Ya hay evidencia para decidir

Este era el bloqueante #1 de la revisión anterior. Con la documentación de
G5 ahora disponible, el panorama es más claro:

- **G4** ya expone `POST /v1/checkout`, que valida stock, reserva
  inventario, y según su propio PDF de arquitectura, **llama a
  `POST /v1/orders` de G5** para crear el pedido.
- **G5**, en su matriz de dependencias y diagrama de integración, confirma
  que recibe el pedido **desde Checkout (G4)**, y desde ahí **G5 mismo**
  llama de forma síncrona a G4 (confirmar reserva), G8 (validar pago) y G6
  (generar despacho), y emite eventos async a G7/G8 (notificaciones).
- **Nuestro contrato BFF actual** describe una tercera orquestación
  (BFF llama a G3, luego a G5, luego limpia carro en G4) que **no coincide
  con lo que G4 y G5 ya construyeron entre ellos**.

**Recomendación concreta:** el BFF **no debe reimplementar la
orquestación**. Debe llamar a `POST /v1/checkout` de G4 (que ya
orquesta todo el flujo con G5/G6/G8) y exponer al frontend un wrapper
sobre `CheckoutIntent`, en vez de tener su propio `POST /orders`
independiente. Esto implica reescribir la sección "Orders" de nuestro
contrato BFF.

### 4.8 Grupo 7 — el único contrato "nuevo" sin desincronización previa, pero con su propio set de desvíos

Grupo 7 no tiene versiones contradictorias (es la única fuente), pero su
contrato no sigue las convenciones: usa el formato de error de 5 campos
(`timestamp, status, code, message, correlationId`), IDs no-UUID
(`ORD-1001`, `PAY-123`, `P-100`) y reenvía el evento `ShipmentDelivered` de
G6 en snake_case sin normalizar. Expone endpoints útiles para nosotros:
`GET /reports/sales`, `GET /reports/orders-by-status`,
`GET /reports/top-products` — candidatos a agregar a un futuro dashboard
admin en el BFF.

---

## 5. Riesgos priorizados

| # | Riesgo | Bloquea a | Urgencia |
|---|---|---|---|
| 1 | Sin contrato de G5 en GitHub, y el que existe en OneDrive se contradice a sí mismo | Todo el flujo de checkout/pedidos | 🔴 Crítica |
| 2 | Ambigüedad de orquestación del checkout (BFF vs G4 vs G5) | Implementación del BFF y de G1 | 🔴 Crítica |
| 3 | G6 tiene 4 versiones de contrato distintas y el código real solo implementa v1.0 (sin cotización, sin eventos reales) mientras su README afirma que ya está en v1.1 | Integración de despacho desde G1/G4/G5, y cualquier cálculo de costo de envío en el checkout | 🔴 Crítica |
| 4 | 3 formatos de `orderId` circulando | G1, G5, G6, G8 | 🟠 Alta |
| 5 | `marketplace-contracts` vacío — nadie lo usa como fuente única | Todos los grupos | 🟠 Alta |
| 6 | Documentos de Office desactualizados tratados como contrato (G2, G3, G4 JSON viejo, G8) | Cualquiera que implemente mirando el doc en vez del YAML | 🟡 Media |
| 7 | Forma de error en 3 variantes | Manejo de errores transversal en el BFF | 🟡 Media |
| 8 | Dinero float / USD en G4 | Cálculo de totales en checkout | 🟡 Media |

---

## 6. Plan de remediación

### 6.1 Gobernanza (acordar en la reunión)

1. **`marketplace-contracts` pasa a ser la única fuente de verdad
   obligatoria.** Cada grupo debe subir su OpenAPI real a
   `services/group-N/openapi.yaml` en ese repo (no en su propio repo de
   servicio, no en OneDrive). Los documentos de Office quedan solo como
   notas de diseño, **nunca como contrato vinculante**.
2. **Todo cambio de contrato va por PR** a `marketplace-contracts`, con al
   menos un reviewer de un grupo consumidor. Se prohíbe cambiar la forma
   de una respuesta (ej. objeto → arreglo) sin subir la versión de la URL.
3. **Checklist mínimo de convenciones** (a publicar en
   `data-dictionary/conventions.md`, hoy inexistente):
   - IDs de negocio (`orderId`, `paymentId`, `shipmentId`): código humano
     `PREFIJO-YYYYMMDD-NNN`. IDs internos de fila (PK): UUID.
   - Dinero: `integer`, moneda `CLP` siempre.
   - Error: `{code, message, details?, correlationId?}`.
   - JSON: `camelCase` en los contratos públicos (los servicios internos
     pueden usar `snake_case` en su propia DB, pero el contrato REST que
     expone hacia otros grupos debe ser camelCase, o el BFF traduce si el
     servicio no puede cambiarlo a tiempo).
   - Paginación: `{data, pagination: {page, pageSize, total, totalPages,
     hasNext, hasPrev}}`.
4. **Decidir explícitamente quién orquesta el checkout** (recomendación
   en §4.7: G4 orquesta, BFF consume `POST /v1/checkout`).
5. Pedir a **Grupo 5 y Grupo 6** que prioricen subir su contrato a
   `marketplace-contracts` antes de seguir desarrollando — son los dos
   grupos sin contrato versionado en GitHub.

### 6.2 Cambios técnicos concretos por grupo

| Grupo | Acción |
|---|---|
| 1 (nosotros) | Reescribir sección Orders del contrato BFF para consumir `POST /v1/checkout` de G4 en vez de orquestar nosotros mismos. Cambiar `Order.id`/`orderId` de UUID a `ORD-YYYYMMDD-NNN`. Commitear nuestro YAML a `marketplace-contracts` (hoy sigue sin commitear). |
| 2 | Borrar/marcar como obsoleto `Contrato Rest.docx`; el OpenAPI de GitHub es la fuente real. |
| 3 | Subir el `swagger_catalogo.yaml` local (más nuevo que el del repo) o confirmar cuál es la versión correcta. Unificar `stockVisible` vs `visibleStock` en el evento. Agregar `/categories`, `minPrice`/`maxPrice`, `sortBy`. |
| 4 | Borrar `Contrato G4.JSON` (es la v1.0 obsoleta) para evitar que alguien lo use por error. Cambiar `currency` default a CLP y montos a integer. |
| 5 | Subir contrato a GitHub. Unificar formato de `orderId` (adoptar `ORD-YYYYMMDD-NNN`). Unificar casing de `OrderStatus` (hoy mezcla mayúsculas en el doc de arquitectura con minúsculas en los ejemplos REST/eventos). Agregar paginación a `GET /users/{user_id}/orders`. |
| 6 | **Decidir si el código se queda en v1.0 o se actualiza a v1.1**, y corregir el README para que describa lo que el código realmente hace (hoy promete v1.1 y entrega v1.0). Si avanza a v1.1, versionar la URL (`/v2/`). Implementar el endpoint de eventos (hoy no se publica ninguno pese a estar documentado). Cambiar paginación de `limit/offset` a `page/pageSize`. Revisar y corregir las 5 interpretaciones de otros contratos en `v1.1/*.md` contra los reales (en particular: G1/BFF no consume directo la API de despacho, eso violaría la arquitectura BFF). |
| 7 | Publicar el contrato como OpenAPI real en su repo (hoy solo existe embebido en un .docx). |
| 8 | Renombrar `error` → `code` en `ErrorResponse`. Confirmar si `CANCELLED` es válido en `Payment.status`. |

### 6.3 Próximos pasos inmediatos (esta semana)

1. Compartir este documento con los 8 grupos antes de la reunión.
2. En la reunión: validar/ajustar la recomendación de orquestación de
   checkout (§4.7) y el formato de `orderId` (§4.1) — son las dos
   decisiones que más bloquean a otros equipos.
3. Acordar fecha límite para que G5 y G6 suban su contrato real a
   `marketplace-contracts`.
4. Crear `data-dictionary/conventions.md` con las reglas de §6.1.3 y
   pedir que cada grupo lo firme/acepte explícitamente.

---

## 7. Agenda sugerida para la reunión

1. Quién orquesta el checkout (G4 vs BFF vs G5) — **decisión, no debate**.
2. Formato único de `orderId` en todo el sistema.
3. Fecha límite para que G5 y G6 publiquen su contrato real en GitHub.
4. Adopción de `marketplace-contracts` como única fuente de verdad —
   dejar de usar PDFs/DOCX como contrato.
5. Checklist de convenciones (dinero, error, naming, paginación) — pedir
   confirmación expresa de G4 y G8 (error `error`→`code`, dinero a
   integer/CLP).
6. G6: comunicar el breaking change v1.0→v1.1 y acordar versionado de URL
   a futuro.
7. Repaso rápido de los gaps de catálogo (G3) y notificaciones (G8) ya
   levantados en `revision-contratos-2026-06-18.md`.
