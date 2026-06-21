# Grupo 5 — Pedidos (Order Management)

**Estado: sin contrato publicado aquí todavía.**

Este grupo no tiene repo de servicio en la organización GitHub ni un
`openapi.yaml` publicado. Su única documentación hoy vive en archivos
locales de Office (arquitectura, contrato REST y contrato de eventos en
`.md`/`.pdf`/`.docx`), y esos documentos se contradicen entre sí en varios
puntos (formato de `orderId`, casing de `OrderStatus`, nombre de eventos).

**Actualización 2026-06-19:** Grupo 1 redactó un `openapi.yaml`
**borrador** en esta misma carpeta, como punto de partida (no como
contrato impuesto) — fija el formato de `orderId` (`ORD-YYYYMMDD-NNN`,
reemplazando los 3 formatos distintos que había), corrige el casing de
`OrderStatus` a `UPPER_SNAKE_CASE` consistente, y agrega paginación al
listado de pedidos por usuario.

**Acción pendiente:** Grupo 5 debe revisar ese borrador, ajustarlo a su
realidad de implementación, y subir la versión final (o una distinta, si
no les acomoda) a `services/group-5-pedidos/openapi.yaml`. También falta
el contrato de eventos formal si difiere de lo descrito en `x-events` del
borrador.
