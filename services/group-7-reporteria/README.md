# Grupo 7 — Reportería, Bash y Streaming

**Estado (2026-06-23): construyeron un servicio real completo, pero el
contrato (forma de error/paginación) no se corrigió y no hay despliegue.**

El repo de servicio es
[`Grupo-7-Reporter-a-bash-y-Streaming`](https://github.com/Mini-Marketplace-Cloud-UTEM/Grupo-7-Reporter-a-bash-y-Streaming).
Pasaron de tener solo un `.docx` de arquitectura (antes del 19-jun) a un
servicio FastAPI real: rutas (`app/api/routes/reports.py`, `batch.py`),
capa de servicios, conexión async a Postgres/Supabase, un **worker de
Pub/Sub** (`app/workers/pubsub_consumer.py`), migraciones SQL, Docker y
tests pytest.

## Lo que falta corregir (sin cambios desde el 2026-06-20)

El borrador `openapi.yaml` que dejamos en esta carpeta fue ajustado
parcialmente por Bastian Encina el 20 de junio, pero ese ajuste **nunca
llegó al código real** (`app/schemas/responses.py`):

- `ErrorResponse` sigue con `timestamp`, `status`, `code`, `message`,
  `correlationId` (versión larga) — debía ser `{code, message, details?,
  correlationId?}`.
- `Pagination` sigue con `totalItems`, `totalPages`, `currentPage`,
  `pageSize` — debía ser `{data, pagination: {page, pageSize, total,
  totalPages, hasNext, hasPrev}}`.
- `orderId` acepta ambos formatos (`ORD-1001` y `ORD-YYYYMMDD-NNN`) —
  falta dejar solo el segundo.

## Sin despliegue real

El `openapi.yaml` declara `https://api-reporteria-g7.render.com` con el
comentario explícito "esto no funciona aún porque no tenemos un render".
No hay URL real que probar todavía.

## Mecanismo de testing interesante: `USE_MOCKS` / `X-MOCK-HTTP-STATUS`

Agregaron un middleware global (`app/middleware/mock_status.py`) que,
con `USE_MOCKS=true`, intercepta el header `X-MOCK-HTTP-STATUS` de la
petición entrante y fuerza ese código en la respuesta HTTP saliente
(100-599 válidos, el resto se ignora con warning). Útil para que el BFF
pruebe su propio manejo de errores sin depender de fallas reales de G7.

**Ojo al usarlo:** solo cambia el `status_code`, no el `body` — si el BFF
espera que el body de error tenga `{code, message}` para ese status
forzado, va a seguir recibiendo el body real (200 OK) con un status
distinto. Hay que manejarlo explícitamente en el cliente HTTP, no
asumirlo. No debería estar activo en producción.

**Acción pendiente de Grupo 7:** propagar la corrección de error/paginación
del `openapi.yaml` al código real, y desplegar el servicio.
