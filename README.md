# marketplace-contracts

Fuente única de verdad de los contratos del proyecto Marketplace Cloud
(8 grupos). Ningún PDF, DOCX, o copia en el repo de servicio de cada
grupo es contrato vinculante — solo lo que está aquí.

## Estructura

```text
marketplace-contracts/
├── shared/
│   └── components.yaml          # Error, Pagination, Money, EventEnvelope (reutilizar con $ref)
├── data-dictionary/
│   ├── guia-y-lineamiento-de-desarrollo.md  # reglas obligatorias + modelos canónicos + mocks
│   └── guia-de-uso-de-jwt.md                # cómo se transporta y valida el JWT entre servicios
├── registro-de-servicios.md     # tabla de URLs reales (Render) por grupo — incompleta a propósito
└── services/
    ├── group-1-bff/openapi.yaml         # Frontend / BFF
    ├── group-2-auth/openapi.yaml        # Auth (login, JWT)
    ├── group-3-catalogo/openapi.yaml    # Catálogo de productos
    ├── group-4-carrito/openapi.yaml     # Carro, checkout, inventario, concurrencia
    ├── group-5-pedidos/openapi.yaml     # Pedidos — contrato real de Grupo 5
    ├── group-6-despacho/openapi.yaml    # Despacho — copia del contrato real (auto-generado) de Grupo 6
    ├── group-7-reporteria/openapi.yaml  # Reportería — BORRADOR de Grupo 1, ajustado parcialmente por Grupo 7
    └── group-8-pagos/openapi.yaml       # Pago simulado y notificaciones
```

## Cómo validar un contrato

```bash
npx @redocly/cli lint services/group-1-bff/openapi.yaml
npx @stoplight/spectral-cli lint services/group-1-bff/openapi.yaml
```

## Documentos de análisis (raíz del repo)

- `matriz-conflictos-contratos.md` — tabla de referencia rápida: qué
  contrato existe hoy, en qué choca cada uno contra los demás, y un
  checklist de qué grupo ya corrigió qué (sección 5).
- `decisiones-ejecutivas-2026-06-19.md` — las 9 decisiones que resuelven
  los conflictos de la matriz, listas para validar satisfacción de cada
  grupo en la reunión.

## Cómo subir tu contrato

1. Tu `openapi.yaml` real (el que tu servicio efectivamente implementa,
   no un borrador) va en `services/group-N-x/openapi.yaml`.
2. Reutiliza los esquemas de `shared/components.yaml` con `$ref` en vez
   de redefinir tu propio `Error`/`Pagination`/`Money`.
3. Respeta las reglas de `data-dictionary/guia-y-lineamiento-de-desarrollo.md`.
4. Todo cambio se sube vía Pull Request, con revisión de al menos un
   grupo consumidor. Si el cambio rompe compatibilidad, sube la versión
   en la URL (`/v1/` → `/v2/`).
