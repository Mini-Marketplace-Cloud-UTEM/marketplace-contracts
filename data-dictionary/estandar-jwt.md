# Estándar de validación de JWT entre servicios

**Decisión ejecutiva — Grupo 1, 2026-06-19, confirmada.** Se presenta en
la próxima reunión para validar satisfacción de los demás grupos, pero se
adopta como estándar de trabajo desde ya — no es un debate abierto.

## Problema

Cada servicio recibe `Authorization: Bearer <token>` (eso ya está
acordado en todos los contratos), pero **no había acuerdo sobre quién
verifica ese token, ni cómo**. Sin esto, cualquier servicio podría estar
confiando ciegamente en el header sin comprobar que sea válido.

## Decisión: validación centralizada contra Grupo 2

Grupo 2 ya construyó (según su propio documento de arquitectura)
`POST /auth/validate` pensado exactamente para esto — "validación
server-to-server, no solo el frontend". Se adopta ese mecanismo como
estándar en vez de pedirle a cada servicio que implemente verificación
de firma JWT por su cuenta:

1. Cada servicio (G3, G4, G5, G6, G7, G8) recibe el request con
   `Authorization: Bearer <token>`.
2. Antes de procesar la lógica de negocio, llama a
   `POST /auth/validate` de Grupo 2 reenviando ese mismo token.
3. Si Grupo 2 responde `200 { valid: true, user }`, el servicio continúa
   y puede usar `user.id`/`user.roles` para autorización fina.
4. Si responde `401` o `valid: false`, el servicio rechaza con `401`
   usando el `Error` estándar (`{code: "UNAUTHORIZED", message}`).

```text
Frontend → BFF → Servicio (G3/G4/G5/G6/G7/G8)
                      │
                      ▼
              POST /auth/validate (Grupo 2)
                      │
              200 { valid, user } ó 401
```

## Por qué esta opción y no verificación local de firma

| Opción | Pros | Contras |
|---|---|---|
| **Validación centralizada (elegida)** | Cero trabajo criptográfico en cada servicio; ya está construido en G2; un solo lugar para revocar sesiones | Un request HTTP extra por llamada; depende de que G2 esté disponible |
| Verificar firma localmente (HS256, secreto compartido) | Sin llamada de red extra | Hay que distribuir un secreto a 7 equipos sin commitearlo a ningún repo — riesgo real de que alguien lo suba por error |
| Verificar firma localmente (RS256, llave pública) | Más seguro, sin secreto compartido | Más trabajo de setup (JWKS o llave pública distribuida) para un proyecto académico — no se justifica todavía |

Para este proyecto, dado el tamaño y plazos, la opción centralizada es la
más simple de adoptar por todos sin trabajo adicional de criptografía.

## Optimización opcional (no obligatoria)

Cada servicio puede cachear el resultado de `validate` por un TTL corto
(ej. 30-60 segundos) usando el token como clave, para no llamar a G2 en
cada request. No es un requisito para esta fase del proyecto.

## Qué se le pide a Grupo 2

- Mantener `POST /auth/validate` siempre disponible — todos los demás
  servicios dependerán de él en cada request autenticado.
- Confirmar el código de error si el token está vencido vs. si es
  inválido (hoy ambos casos devuelven genéricamente 401 — no es
  bloqueante, pero ayuda para debugging).

## Pendiente de acordar en la reunión

- ¿Qué pasa si Grupo 2 no está disponible? (¿el servicio rechaza todo, o
  hay un modo degradado?) — para esta fase del proyecto, se acepta que
  si G2 está caído, todo el sistema rechaza requests autenticados.
