# Contratos-mock: cómo no esperar a que todos terminen

**Problema:** un grupo no puede empezar a integrar contra el contrato de
otro si ese otro grupo todavía no tiene su servicio real implementado.

**Solución:** el contrato (`openapi.yaml`) debe declarar **ejemplos
completos (`example:`) en cada respuesta**, y cada grupo debe levantar un
**mock server público** que sirva esos ejemplos como si la
infraestructura real ya existiera — sin necesidad de programar lógica de
negocio ni conectar base de datos todavía.

## Cómo hacerlo (Prism)

Grupo 3 ya usa este patrón (`http://127.0.0.1:4010 — Mock local con
Prism` en su contrato). Es la herramienta sugerida para todos:

```bash
npx @stoplight/prism-cli mock services/group-N-x/openapi.yaml
```

Esto levanta un servidor que responde automáticamente con los `example:`
definidos en el propio YAML — cero código adicional.

## Qué se pide a cada grupo

1. Completar `example:` en cada respuesta de su `openapi.yaml` (datos de
   ejemplo realistas, no placeholders vacíos).
2. Desplegar ese mock en una URL pública (mismo Render/Docker que usarían
   para el servicio real).
3. Publicar esa URL como `servers:` temporal en su propio contrato, o en
   un futuro registro de servicios.
4. Cuando el servicio real esté listo, **se reemplaza solo la URL** —
   el consumidor no debería tener que cambiar nada de su código, porque
   la forma de la respuesta (el contrato) es la misma.

## Por qué esto importa ahora

- Permite a Grupo 1 (BFF) desarrollar contra Grupo 5 (Pedidos) o Grupo 6
  (Despacho v1.1) sin esperar a que esos grupos terminen su
  implementación real.
- Permite que el catálogo embebido de Grupo 3 (ver delegación de UI de
  catálogo, acordada con Grupo 1) funcione con datos de ejemplo antes de
  tener la base de datos real conectada.
- Aplica igual para cualquier par consumidor/proveedor del proyecto.
