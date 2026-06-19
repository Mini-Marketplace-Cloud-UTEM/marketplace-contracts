# Grupo 6 — Despacho (Shipment Service)

**Estado: sin `openapi.yaml` formal — hay código real, pero no coincide
con la documentación de contrato.**

El repo de servicio es
[`G6-Shipment-Service`](https://github.com/Mini-Marketplace-Cloud-UTEM/G6-Shipment-Service).
Ahí existen **4 versiones distintas** del contrato (v1.0, v1.1, una guía
de ingeniería interna, y el código FastAPI real), y **el código real
implementa el modelo v1.0** (envío único, `order_id` único, sin
cotización, sin eventos publicados de verdad), mientras que el README de
ese repo y la documentación interna describen el modelo v1.1
(multi-paquete, cotización, eventos Kafka) como si ya estuviera vigente.

Ver el detalle línea por línea (con referencias a `main.py`, `schemas.py`,
`models.py`) en `analisis-integration-hell-2026-06-18.md` (sección 4.5) en
la raíz de este repo.

**Acción pendiente:** Grupo 6 debe decidir si se queda en v1.0 o termina
de implementar v1.1, corregir su README para que describa lo que el
código realmente hace, y subir el `openapi.yaml` resultante a esta
carpeta (`services/group-6-despacho/openapi.yaml`).
