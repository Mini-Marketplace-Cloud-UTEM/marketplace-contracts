# Grupo 8 — Pagos y Notificaciones

**Estado (2026-06-23): el contrato sigue 100% conforme, pero un PR
reciente introdujo una regresión puntual en el manejo de errores de
Pagos.**

El `openapi.yaml` de esta carpeta (`docs/G8_Contratos.yaml` en su repo)
no cambió desde el 22 de junio (confirmado con `diff` byte a byte). Los
dos PRs recientes (23-jun) tocaron solo código, no el documento de
contrato.

## Notificaciones nuevas (E2) — consistentes, sin problemas

PR "feat: endpoints mock de Notificaciones (E2)" agrega
`GET /v1/notifications` y `POST /v1/notifications/test`. Sigue exactamente
la misma convención que Pagos: camelCase, `{code, message}` en errores,
paginación completa `{page, pageSize, total, totalPages, hasNext,
hasPrev}`. Nada que corregir.

## Regresión en Pagos — a reportarle a G8

PR "Arreglos pequeños en endpoint CreatePaymentRequest y PaymentResponse"
**quitó** la validación manual de `Idempotency-Key` que devolvía
`400 {code: "MISSING_IDEMPOTENCY_KEY", message}` (documentado en el
contrato), reemplazándola por `Header(..., alias=...)` nativo de FastAPI.

**Efecto:** un request sin ese header ahora devuelve un `422` genérico de
FastAPI/Pydantic (`{"detail": [...]}`), no la forma `{code, message}`
exigida por nuestras convenciones. El resto del endpoint (`amount`
integer, `currency` CLP) no cambió — es un problema puntual de ese caso
de validación, no una regresión general.

**Acción pendiente de Grupo 8:** revertir ese detalle o agregar un
exception handler que traduzca el 422 nativo de FastAPI a la forma de
error estándar del proyecto.

## Impacto para el BFF

Bajo por ahora — `Grupo-1-BFF` todavía no integra contra G8 (el flujo de
pago no está implementado). Pero al implementarlo, el cliente HTTP debe
manejar también un posible `422` sin `code`/`message` para el caso
específico de `Idempotency-Key` faltante, hasta que G8 lo corrija.

## Sin despliegue real confirmado

El README de G8 menciona Render pero sin URL; el `servers:` del
`openapi.yaml` sigue en `api.marketplace.example.com` (placeholder). No
se pudo probar ningún endpoint en vivo.
