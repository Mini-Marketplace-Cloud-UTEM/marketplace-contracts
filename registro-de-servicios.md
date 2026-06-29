# Registro de servicios en producción

Tabla central de URLs reales. El BFF (y cualquier consumidor) debe leer
esta tabla para saber a qué dirección llamar a cada servicio — no hay que
hardcodear URLs adivinadas ni asumir `api.marketplace.example.com` (eso
es un placeholder que varios contratos todavía tienen).

**Estado: incompleta a propósito.** Se llena a medida que cada grupo
despliega. Mientras un campo diga `pendiente`, usar el mock (Prism) de
ese grupo si existe (ver `data-dictionary/guia-y-lineamiento-de-desarrollo.md`, sección 14).

| Grupo | Servicio | URL producción (Render) | URL mock | Estado |
|---|---|---|---|---|
| 1 | BFF | `https://grupo-1-bff.onrender.com` | — | ✅ Desplegado y probado en vivo (2026-06-22) — `/v1/products*` y `/v1/auth/*` funcionando |
| 2 | Auth | `https://grupo2-identidadusuario.onrender.com` (sin `/api/v1`) | — | 🟡 **Corregido 2026-06-23** — la URL de `services/group-2-auth/openapi.yaml` (`api-grupo2.onrender.com/api/v1`) está **muerta (404)**. Ver `services/group-2-auth/README.md`: además es un mock que no rechaza tokens inválidos. |
| 3 | Catálogo | `https://grupo-3-catalogo.onrender.com` | `http://127.0.0.1:4010` (Prism local) | ✅ Confirmada en vivo (2026-06-22) — ver `services/group-3-catalogo/README.md` (gaps y bug de encoding detectados) |
| 4 | Carro/Checkout/Inventario | `https://g4-carrito-checkout-inventario-y.onrender.com` (`/docs` para Swagger) | — | 🟡 Confirmado en vivo (2026-06-28) — endpoints existen y responden, pero `add_item_to_cart` no reconoce productos reales de G3 (probado con un producto que sí existe) y el dinero sigue en `number`/`float`. Ver `services/group-4-carrito/README.md`. |
| 5 | Pedidos | `https://grupo5-pedidos-mock-kdyl.onrender.com` (`/docs` para Swagger) | — | 🟡 Confirmado en vivo (2026-06-28) — reemplaza la URL anterior (`api-grupo5-pedidos.onrender.com`, ya no vigente). Es un mock declarado: solo `POST /orders` existe, falta `GET /orders` y `GET /orders/{id}`. Ver `services/group-5-pedidos/README.md`. |
| 6 | Despacho | `https://g6-despacho-oficial.onrender.com` | `https://g6-despacho.onrender.com` | ✅ Desplegado y probado en vivo (E3). El mock E2 sigue activo en su URL respectiva. |
| 7 | Reportería | `https://grupo-7-reporter-a-bash-y-streaming-production.up.railway.app` (Railway, no Render) | — | ✅ Desplegado y confirmado en vivo (2026-06-23) — ver `services/group-7-reporteria/README.md`. Ya integrado en el BFF bajo `/v1/admin/reports/*`. |
| 8 | Pagos/Notificaciones | `https://g8-pagos-y-notificaciones.onrender.com` (`/docs` para Swagger) | — | 🔴 Desplegado, pero 3 endpoints de pago fallan en producción por un problema de autorización de tokens JWT en Render (funcionan bien corridos en terminal/local) |

## Cómo actualizar esta tabla

Cada grupo edita su propia fila vía Pull Request cuando tenga una URL
real (Render, o cualquier hosting). No se necesita aprobación de nadie
más que el propio grupo para esa fila.

## Convención de URL

`https://<algo-identificable>.onrender.com` — no usar IPs, no usar
`localhost` salvo en la columna de desarrollo local de cada contrato.
