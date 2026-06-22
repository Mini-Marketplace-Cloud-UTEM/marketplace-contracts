# Registro de servicios en producción

Tabla central de URLs reales. El BFF (y cualquier consumidor) debe leer
esta tabla para saber a qué dirección llamar a cada servicio — no hay que
hardcodear URLs adivinadas ni asumir `api.marketplace.example.com` (eso
es un placeholder que varios contratos todavía tienen).

**Estado: incompleta a propósito.** Se llena a medida que cada grupo
despliega. Mientras un campo diga `pendiente`, usar el mock (Prism) de
ese grupo si existe (ver `data-dictionary/contratos-mock.md`).

| Grupo | Servicio | URL producción (Render) | URL mock | Estado |
|---|---|---|---|---|
| 1 | BFF | _pendiente_ | — | 🔴 No desplegado todavía |
| 2 | Auth | `https://api-grupo2.onrender.com/api/v1` | — | ✅ Declarada en su propio contrato |
| 3 | Catálogo | https://grupo-3-catalogo.onrender.com/ | `http://127.0.0.1:4010` (Prism local) | 🟡 Solo mock local, falta URL pública |
| 4 | Carro/Checkout/Inventario | _pendiente_ (su contrato declara `api.marketplace.example.com`, que es placeholder) | _pendiente_ | 🔴 |
| 5 | Pedidos | _pendiente_ | _pendiente_ | 🔴 Sin servicio desplegado. Ver contrato temporal en `services/group-5-pedidos/openapi.yaml` |
| 6 | Despacho | _pendiente_ (repo `G6-Shipment-Service` corre local con `uvicorn`/Docker, sin URL pública confirmada) | _pendiente_ | 🔴 |
| 7 | Reportería | _pendiente_ | _pendiente_ | 🔴 Sin servicio desplegado. Ver contrato temporal en `services/group-7-reporteria/openapi.yaml` |
| 8 | Pagos/Notificaciones | _pendiente_ (su contrato declara `api.marketplace.example.com`, que es placeholder) | _pendiente_ | 🔴 |

## Cómo actualizar esta tabla

Cada grupo edita su propia fila vía Pull Request cuando tenga una URL
real (Render, o cualquier hosting). No se necesita aprobación de nadie
más que el propio grupo para esa fila.

## Convención de URL

`https://<algo-identificable>.onrender.com` — no usar IPs, no usar
`localhost` salvo en la columna de desarrollo local de cada contrato.
