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
| 4 | Carro/Checkout/Inventario | _pendiente_ (su contrato declara `api.marketplace.example.com`, que es placeholder) | _pendiente_ | 🔴 |
| 5 | Pedidos | `https://api-grupo5-pedidos.onrender.com/v1` (declarada en su contrato, despliegue sin confirmar) | _pendiente_ | 🟡 Contrato real ya publicado, falta confirmar que esté desplegada |
| 6 | Despacho | `https://g6-despacho.onrender.com` | — | ✅ Desplegado y probado en vivo. Actualizado 2026-06-23: corrigieron el bug de naming y ahora exigen `X-Request-Id`/`X-Correlation-Id`/`X-Consumer` como obligatorios en todos los endpoints. |
| 7 | Reportería | _pendiente_ (`api-reporteria-g7.render.com` declarado en su `openapi.yaml` no funciona todavía) | _pendiente_ | 🟡 Tienen un servicio real construido (FastAPI + Postgres + worker Pub/Sub + tests), pero sin desplegar. Ver `services/group-7-reporteria/README.md`. |
| 8 | Pagos/Notificaciones | _pendiente_ (su contrato declara `api.marketplace.example.com`, que es placeholder) | _pendiente_ | 🔴 |

## Cómo actualizar esta tabla

Cada grupo edita su propia fila vía Pull Request cuando tenga una URL
real (Render, o cualquier hosting). No se necesita aprobación de nadie
más que el propio grupo para esa fila.

## Convención de URL

`https://<algo-identificable>.onrender.com` — no usar IPs, no usar
`localhost` salvo en la columna de desarrollo local de cada contrato.
