# Grupo 7 — Reportería, Bash y Streaming

**Estado (2026-06-23): servicio real desplegado en Railway y ya
integrado en el BFF (`/v1/admin/reports/*`).**

URL real: `https://grupo-7-reporter-a-bash-y-streaming-production.up.railway.app`
(`/docs` para el Swagger interactivo). Nota: **Railway, no Render** —
no usar la URL vieja `api-reporteria-g7.render.com`.

## Endpoints confirmados en vivo (2026-06-23)

| Endpoint | Descripción |
|---|---|
| `GET /reports/sales` | resumen financiero (`from`/`to` opcionales) |
| `GET /reports/orders-by-status` | conteo de pedidos por estado |
| `GET /reports/top-products` | ranking paginado (`page`/`pageSize`) |
| `GET /reports/average-ticket` | ticket promedio |
| `GET /reports/peak-hours` | distribución de pedidos por hora (0-23) |
| `GET /reports/delivery-performance` | tiempo promedio de entrega |
| `POST /reports/batch/recalculate` | recálculo batch asíncrono (requiere `Idempotency-Key`) |

Todos (salvo `/health`) exigen `X-Request-Id`/`X-Correlation-Id`/`X-Consumer`
como headers obligatorios. **Ninguno valida JWT/Authorization** — el
control de acceso queda del lado del BFF (implementado como rol `admin`).

## Pendiente de corregir (confirmado vigente, sin cambios desde el 20-jun)

- `Pagination` de `top-products` sigue con `totalItems`, `totalPages`,
  `currentPage`, `pageSize` — no `{page, pageSize, total, totalPages,
  hasNext, hasPrev}`. El BFF ya traduce esto en su capa
  (`Grupo-1-BFF/app/routers/reports.py`).
- No hay forma de error custom documentada — solo el `422` nativo de
  FastAPI para validación. No se confirmó cómo responden otros códigos
  de error (404, 500) en producción.

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
