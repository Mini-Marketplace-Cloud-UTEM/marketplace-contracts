# Guía de uso de JSON Web Tokens (JWT)

Cómo se transporta, emite y valida el token de sesión entre todos los
servicios del proyecto.

## Cómo se transporta el token

Todo endpoint protegido recibe el token en el header estándar:

```
Authorization: Bearer <token>
```

## Quién emite el token

El servicio de **Auth** emite el token al iniciar sesión
(`POST /auth/login`) y lo renueva vía `POST /auth/refresh`. Ningún otro
servicio emite ni firma tokens.

## Validación centralizada

Ningún servicio verifica la firma del JWT localmente. En su lugar, antes
de procesar la lógica de negocio de un request autenticado, cada
servicio llama al endpoint de validación del servicio de Auth
reenviando el mismo token:

1. El servicio recibe el request con `Authorization: Bearer <token>`.
2. Llama a `POST /auth/validate` del servicio de Auth, reenviando ese
   mismo token.
3. Si Auth responde `200 { valid: true, user }`, el servicio continúa y
   puede usar `user.id`/`user.roles` para autorización fina.
4. Si responde `401` o `valid: false`, el servicio rechaza el request
   con `401` usando el `Error` estándar (`{code: "UNAUTHORIZED",
   message}`).

```text
Frontend → BFF → Servicio (Catálogo/Carro/Pedidos/Despacho/Reportería/Pagos)
                      │
                      ▼
              POST /auth/validate (Auth)
                      │
              200 { valid, user } ó 401
```

## Por qué validación centralizada y no verificación local de firma

| Opción | Pros | Contras |
|---|---|---|
| **Validación centralizada (la usada)** | Cero trabajo criptográfico en cada servicio; un solo lugar para revocar sesiones | Un request HTTP extra por llamada; depende de la disponibilidad del servicio de Auth |
| Verificar firma localmente (HS256, secreto compartido) | Sin llamada de red extra | Hay que distribuir un secreto a todos los equipos sin commitearlo a ningún repo — riesgo real de subirlo por error |
| Verificar firma localmente (RS256, llave pública) | Más seguro, sin secreto compartido | Más trabajo de setup (JWKS o llave pública distribuida) |

Para un proyecto de este tamaño y plazos, la opción centralizada es la
más simple de adoptar por todos sin trabajo adicional de criptografía.

## Cacheo opcional

Cada servicio puede cachear el resultado de `validate` por un TTL corto
(ej. 30-60 segundos) usando el token como clave, para no llamar al
servicio de Auth en cada request. No es un requisito obligatorio.

## Qué debe garantizar el servicio de Auth

- Mantener `POST /auth/validate` siempre disponible — todos los demás
  servicios dependen de él en cada request autenticado.
- Diferenciar el código de error cuando el token está vencido vs. cuando
  es inválido (ayuda al debugging del lado del consumidor).

## Comportamiento si el servicio de Auth no está disponible

Si el servicio de Auth no responde, todo el sistema rechaza los requests
autenticados — no existe un modo degradado que permita continuar sin
validación.

## Autorización para mutaciones administrativas

Aplica a **cualquier endpoint que cree, edite o elimine algo en nombre de
un administrador**: usuarios, contraseñas, productos, inventario, etc.
No alcanza con validar que el token sea válido — hay que confirmar que
quien lo presenta tiene rol `admin` antes de ejecutar la acción.

1. El request llega con `Authorization: Bearer <token>` (igual que
   cualquier endpoint autenticado — ver arriba). El servicio que recibe
   la mutación llama a `POST /auth/validate` con ese mismo token.
2. Si la respuesta no incluye `admin` en `user.roles`, el servicio
   rechaza con `403` y el `Error` estándar:
   `{code: "FORBIDDEN", message: "Se requiere rol admin."}`.
3. Solo si el rol viene confirmado por Auth, el servicio ejecuta la
   mutación.

**Regla dura: el campo de identidad nunca es un dato del body.** Un
`userId`/`requestedBy` suelto en el JSON del request no prueba nada — lo
puede escribir cualquiera, incluso quien no es admin. La única prueba de
identidad válida es el JWT, porque lo firmó Auth y se valida contra
Auth. Si un endpoint de mutación hoy solo recibe "un id y qué hacer con
él" sin pasar por este flujo, no cumple el contrato de seguridad y debe
corregirse.
