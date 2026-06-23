# Grupo 3 — Catálogo

**Estado (2026-06-22): API real en producción confirmada y consumida en
vivo por el BFF.** `https://grupo-3-catalogo.onrender.com`.

## El contrato real va adelantado a `openapi.yaml`

El `openapi.yaml` de esta carpeta describe la API en **snake_case**
(`stock_visible`, `category_id`, paginación con `size`) y un error con
`timestamp`/`status`. El servicio **real** desplegado, verificado en vivo
el 2026-06-22, ya responde distinto — y mejor:

| Campo | `openapi.yaml` (comiteado) | Servicio real (verificado) |
|---|---|---|
| Stock | `stock_visible` | `stockVisible` (camelCase) |
| Categoría | `category_id`/`category_name` | `categoryId`/`categoryName` |
| Fechas | `created_at`/`updated_at` | `createdAt`/`updatedAt` |
| Paginación (respuesta) | `{page, size, total, ...}` | `{page, pageSize, total, totalPages, hasNext, hasPrev}` |
| Paginación (query param de entrada) | `size` | `size` (sin cambios — solo la respuesta es camelCase) |
| Error | `{timestamp, status, code, message, correlationId}` | `{code, message, correlationId}` (plano, sin `details`) |

**Acción pendiente de Grupo 3:** actualizar su `openapi.yaml` para que
describa lo que el servicio real ya hace — ya cumplen camelCase y forma
de error corta, solo falta que el documento lo refleje.

## Bug detectado: encoding de caracteres con tilde

El 2026-06-22 se observó al menos una respuesta con caracteres corruptos
(`Eléctrico` → `El�ctrico`, bytes inválidos como UTF-8 confirmados con
`httpx`/Python). En pruebas posteriores el mismo producto respondió bien.
**Parece intermitente** — posiblemente una instancia/réplica con un
encoding de base de datos distinto. Reportar a Grupo 3 para que revisen
la configuración de encoding de su base de datos/conexión.

## Gaps conocidos (no implementados en el servicio real)

- `GET /categories` no existe — el BFF no puede listar categorías por
  separado, solo lo que viene embebido en cada producto (`categoryId`/`categoryName`).
- `minPrice`/`maxPrice`/`sortBy` no están soportados ni en `/products` ni
  en `/products/search` — el BFF los recibe del frontend pero no puede
  aplicarlos todavía.
- `/products/search` requiere `q` obligatorio; solo soporta filtrar por
  `category_id` (no por nombre) y `status`.

## Cómo lo consume el BFF

Implementado en `Grupo-1-BFF/app/routers/catalog.py` — traduce la
respuesta camelCase de G3 a los nombres de campo del contrato BFF
(`stockVisible`→`inStock`/`stock`, genera `slug` porque G3 no lo tiene,
etc.). Header `X-Consumer: grupo1-bff` enviado en cada request (opcional
según el contrato de G3, pero se manda igual por trazabilidad).
