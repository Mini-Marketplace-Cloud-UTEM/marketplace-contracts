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
│   ├── conventions.md           # reglas obligatorias (IDs, dinero, error, naming, paginación...)
│   ├── canonical-models.md      # entidades compartidas (User, Product, Order, Payment...)
│   ├── contratos-mock.md        # cómo levantar un mock (Prism) mientras no hay servicio real
│   └── estandar-jwt.md          # cómo se valida el JWT entre servicios
├── registro-de-servicios.md     # tabla de URLs reales (Render) por grupo — incompleta a propósito
└── services/
    ├── group-1-bff/openapi.yaml         # Frontend / BFF
    ├── group-2-auth/openapi.yaml        # Auth (login, JWT)
    ├── group-3-catalogo/openapi.yaml    # Catálogo de productos
    ├── group-4-carrito/openapi.yaml     # Carro, checkout, inventario, concurrencia
    ├── group-5-pedidos/openapi.yaml     # Pedidos — BORRADOR redactado por Grupo 1, pendiente que G5 lo adopte/ajuste
    ├── group-6-despacho/                # Despacho — sin openapi.yaml formal todavía (ver README ahí)
    ├── group-7-reporteria/openapi.yaml  # Reportería — BORRADOR redactado por Grupo 1, pendiente que G7 lo adopte/ajuste
    └── group-8-pagos/openapi.yaml       # Pago simulado y notificaciones
```

## Cómo validar un contrato

```bash
npx @redocly/cli lint services/group-1-bff/openapi.yaml
npx @stoplight/spectral-cli lint services/group-1-bff/openapi.yaml
```

## Documentos de análisis (raíz del repo)

- `revision-contratos-2026-06-18.md` — comparación contrato BFF vs cada grupo.
- `guia-organizacion-y-contratos.md` — guía corta para presentar a los
  líderes de grupo en la reunión.
- `matriz-conflictos-contratos.md` — tabla de referencia rápida: qué
  contrato existe hoy y en qué choca cada uno contra los demás.
- `decisiones-ejecutivas-2026-06-19.md` — las 9 decisiones que resuelven
  los conflictos de la matriz, listas para validar satisfacción de cada
  grupo en la reunión.

## Cómo subir tu contrato

1. Tu `openapi.yaml` real (el que tu servicio efectivamente implementa,
   no un borrador) va en `services/group-N-x/openapi.yaml`.
2. Reutiliza los esquemas de `shared/components.yaml` con `$ref` en vez
   de redefinir tu propio `Error`/`Pagination`/`Money`.
3. Respeta las reglas de `data-dictionary/conventions.md`.
4. Todo cambio se sube vía Pull Request, con revisión de al menos un
   grupo consumidor. Si el cambio rompe compatibilidad, sube la versión
   en la URL (`/v1/` → `/v2/`).
