# Grupo 2 — Identidad, Usuarios y Sesiones

**Estado (2026-06-23): servicio real desplegado, pero con dos problemas
que afectan directamente al BFF en producción.**

## 1. URL real distinta a la declarada en el contrato — urgente

`services/group-2-auth/openapi.yaml` (y el `.env` de producción del BFF)
apuntan a `https://api-grupo2.onrender.com/api/v1`. Esa URL **responde
404** — no existe. La URL real, confirmada por el propio Grupo 2 y en su
README (`Grupo2_IdentidadUsuario`), es:

```
https://grupo2-identidadusuario.onrender.com
```

(nota: sin el prefijo `/api/v1`).

**Acción pendiente de Grupo 1:** actualizar `AUTH_SERVICE_URL` en el
entorno de producción del BFF (Render) y en `app/config.py` /
`.env.example` de `Grupo-1-BFF`.

**Acción pendiente de Grupo 2:** actualizar el `servers:` de su
`openapi.yaml` para que el contrato refleje la URL real — hoy solo está
avisado por WhatsApp y en el README, no en el contrato versionado.

## 2. `POST /auth/validate` es un mock sin validación real

Probado en vivo el 2026-06-23: tanto `GET /auth/me` como
`POST /auth/validate` responden `200` con un usuario fijo (`mock_user`)
**sin enviar ningún header `Authorization`**. Nunca devuelven `401`.

Esto significa que el estándar de JWT centralizado
(`data-dictionary/estandar-jwt.md` — cada servicio valida contra G2 en
vez de verificar la firma localmente) **todavía no aporta seguridad
real**: cualquier request, con o sin token válido, "pasa". No es
bloqueante para seguir desarrollando el mockup, pero **no se puede
considerar el sistema seguro** hasta que G2 implemente la validación de
verdad.

## 3. Lo que sí está bien

- El contrato (`ContratoRestGrupo2`) es idéntico al que tenemos copiado
  en `services/group-2-auth/openapi.yaml` — sin drift de forma.
- Endpoints presentes y sin cambios: `register`, `login`, `logout`,
  `refresh`, `validate`, `me`, `users`, `identity/roles`.
- La forma de error/éxito coincide con lo esperado por el BFF
  (`Grupo-1-BFF/app/routers/auth.py` ya traduce `access_token`,
  `refresh_token`, `avatar_url` snake_case → camelCase correctamente).

## Pendiente de Grupo 2

- Migrar el contrato público a camelCase (sigue en `access_token`,
  `avatar_url`, etc.) — decisión ejecutiva 2026-06-19, punto 3.
- Implementar validación real de JWT en `/auth/validate` y `/auth/me`.
- Actualizar `servers:` del `openapi.yaml` con la URL real.
