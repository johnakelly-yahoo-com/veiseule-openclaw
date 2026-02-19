---
summary: "Ingreso por webhook para activación y ejecuciones de agentes aisladas"
read_when:
  - Al agregar o cambiar endpoints de webhook
  - Al conectar sistemas externos con OpenClaw
title: "Webhooks"
---

# Webhooks

El Gateway puede exponer un pequeño endpoint HTTP de webhook para disparadores externos.

## Habilitar

```json5
{
  hooks: {
    enabled: true,
    token: "shared-secret",
    path: "/hooks",
  },
}
```

Notas:

- `hooks.token` es obligatorio cuando `hooks.enabled=true`.
- `hooks.path` tiene como valor predeterminado `/hooks`.

## Autenticación

Cada solicitud debe incluir el token del hook. Prefiera encabezados:

- `Authorization: Bearer <token>` (recomendado)
- `x-openclaw-token: <token>`
- Se rechazan los tokens en la query string (`?token=...` devuelve `400`).

## Endpoints

### `POST /hooks/wake`

Carga útil:

```json
{ "text": "System line", "mode": "now" }
```

- `text` **obligatorio** (string): La descripción del evento (p. ej., "Nuevo correo recibido").
- `mode` opcional (`now` | `next-heartbeat`): Si se debe activar un heartbeat inmediato (predeterminado `now`) o esperar la siguiente verificación periódica.

Efecto:

- Encola un evento del sistema para la sesión **principal**
- Si `mode=now`, activa un heartbeat inmediato

### `POST /hooks/agent`

Carga útil:

```json
{
  "message": "Run this",
  "name": "Email",
  "sessionKey": "hook:email:msg-123",
  "wakeMode": "now",
  "deliver": true,
  "channel": "last",
  "to": "+15551234567",
  "model": "openai/gpt-5.2-mini",
  "thinking": "low",
  "timeoutSeconds": 120
}
```

- `message` **obligatorio** (string): El prompt o mensaje que el agente debe procesar.
- `name` opcional (string): Nombre legible para humanos del hook (p. ej., "GitHub"), usado como prefijo en los resúmenes de sesión.
- `agentId` opcional (string): Enruta este hook a un agente específico. Los IDs desconocidos vuelven al agente predeterminado. Cuando se establece, el hook se ejecuta usando el workspace y la configuración del agente resuelto.
- `sessionKey` opcional (string): La clave usada para identificar la sesión del agente. Por defecto, este campo se rechaza a menos que `hooks.allowRequestSessionKey=true`.
- `wakeMode` opcional (`now` | `next-heartbeat`): Si se debe activar un heartbeat inmediato (predeterminado `now`) o esperar la siguiente verificación periódica.
- `deliver` opcional (boolean): Si `true`, la respuesta del agente se enviará al canal de mensajería. El valor predeterminado es `true`. Las respuestas que solo son acuses de heartbeat se omiten automáticamente.
- `channel` opcional (string): El canal de mensajería para la entrega. Uno de: `last`, `whatsapp`, `telegram`, `discord`, `slack`, `mattermost` (plugin), `signal`, `imessage`, `msteams`. El valor predeterminado es `last`.
- `to` opcional (string): El identificador del destinatario para el canal (p. ej., número de teléfono para WhatsApp/Signal, ID de chat para Telegram, ID de canal para Discord/Slack/Mattermost (plugin), ID de conversación para MS Teams). Por defecto, el último destinatario en la sesión principal.
- `model` opcional (string): Anulación del modelo (p. ej., `anthropic/claude-3-5-sonnet` o un alias). Debe estar en la lista de modelos permitidos si hay restricciones.
- `thinking` opcional (string): Anulación del nivel de razonamiento (p. ej., `low`, `medium`, `high`).
- `timeoutSeconds` opcional (number): Duración máxima de la ejecución del agente en segundos.

Efecto:

- Ejecuta un turno de agente **aislado** (clave de sesión propia)
- Siempre publica un resumen en la sesión **principal**
- Si `wakeMode=now`, activa un heartbeat inmediato

## Política de clave de sesión (cambio incompatible)

Las sobrescrituras de `sessionKey` en la carga útil de `/hooks/agent` están deshabilitadas por defecto.

- Recomendado: establece un `hooks.defaultSessionKey` fijo y desactiva las anulaciones por solicitud.
- Opcional: permite anulaciones por solicitud solo cuando sea necesario y restringe los prefijos.

Configuración recomendada:

```json5
{
  hooks: {
    enabled: true,
    token: "${OPENCLAW_HOOKS_TOKEN}",
    defaultSessionKey: "hook:ingress",
    allowRequestSessionKey: false,
    allowedSessionKeyPrefixes: ["hook:"],
  },
}
```

Configuración de compatibilidad (comportamiento heredado):

```json5
{
  hooks: {
    enabled: true,
    token: "${OPENCLAW_HOOKS_TOKEN}",
    allowRequestSessionKey: true,
    allowedSessionKeyPrefixes: ["hook:"], // muy recomendable
  },
}
```

### `POST /hooks/<name>` (mapped)

Los nombres de hooks personalizados se resuelven mediante `hooks.mappings` (consulte la configuración). Un mapeo puede
convertir payloads arbitrarios en acciones `wake` o `agent`, con plantillas opcionales o
transformaciones de código.

Opciones de mapeo (resumen):

- `hooks.presets: ["gmail"]` habilita el mapeo integrado de Gmail.
- `hooks.mappings` le permite definir `match`, `action` y plantillas en la configuración.
- `hooks.transformsDir` + `transform.module` carga un módulo JS/TS para lógica personalizada.
  - `hooks.transformsDir` (si se configura) debe permanecer dentro de la raíz de transforms bajo tu directorio de configuración de OpenClaw (normalmente `~/.openclaw/hooks/transforms`).
  - `transform.module` debe resolverse dentro del directorio efectivo de transforms (las rutas de traversión/escape se rechazan).
- Use `match.source` para mantener un endpoint de ingesta genérico (enrutamiento impulsado por el payload).
- Las transformaciones TS requieren un cargador TS (p. ej., `bun` o `tsx`) o `.js` precompilado en tiempo de ejecución.
- Configure `deliver: true` + `channel`/`to` en los mapeos para enrutar respuestas a una superficie de chat
  (`channel` tiene como valor predeterminado `last` y recurre a WhatsApp).
- `agentId` dirige el hook a un agente específico; los ID desconocidos recurren al agente predeterminado.
- `hooks.allowedAgentIds` restringe el enrutamiento explícito mediante `agentId`. Omítelo (o incluye `*`) para permitir cualquier agente. Establece `[]` para denegar el enrutamiento explícito mediante `agentId`.
- `hooks.defaultSessionKey` establece la sesión predeterminada para las ejecuciones del agente del hook cuando no se proporciona una clave explícita.
- `hooks.allowRequestSessionKey` controla si las cargas útiles de `/hooks/agent` pueden establecer `sessionKey` (predeterminado: `false`).
- `hooks.allowedSessionKeyPrefixes` restringe opcionalmente los valores explícitos de `sessionKey` provenientes de cargas útiles de solicitud y mapeos.
- `allowUnsafeExternalContent: true` deshabilita el envoltorio externo de seguridad de contenido para ese hook
  (peligroso; solo para fuentes internas de confianza).
- `openclaw webhooks gmail setup` escribe la configuración `hooks.gmail` para `openclaw webhooks gmail run`.
  Consulte [Gmail Pub/Sub](/automation/gmail-pubsub) para el flujo completo de vigilancia de Gmail.

## Responses

- `200` para `/hooks/wake`
- `202` para `/hooks/agent` (ejecución asíncrona iniciada)
- `401` en falla de autenticación
- `429` después de fallos de autenticación repetidos desde el mismo cliente (consulta `Retry-After`)
- `400` en payload inválido
- `413` en payloads de tamaño excesivo

## Examples

```bash
curl -X POST http://127.0.0.1:18789/hooks/wake \
  -H 'Authorization: Bearer SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"text":"New email received","mode":"now"}'
```

```bash
curl -X POST http://127.0.0.1:18789/hooks/agent \
  -H 'x-openclaw-token: SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"message":"Summarize inbox","name":"Email","wakeMode":"next-heartbeat"}'
```

### Use a different model

Agregue `model` al payload del agente (o al mapeo) para anular el modelo en esa ejecución:

```bash
curl -X POST http://127.0.0.1:18789/hooks/agent \
  -H 'x-openclaw-token: SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"message":"Summarize inbox","name":"Email","model":"openai/gpt-5.2-mini"}'
```

Si aplica `agents.defaults.models`, asegúrese de que el modelo de anulación esté incluido allí.

```bash
curl -X POST http://127.0.0.1:18789/hooks/gmail \
  -H 'Authorization: Bearer SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"source":"gmail","messages":[{"from":"Ada","subject":"Hello","snippet":"Hi"}]}'
```

## Security

- Mantenga los endpoints de hook detrás de loopback, tailnet o un proxy inverso de confianza.
- Use un token de hook dedicado; no reutilice tokens de autenticación del gateway.
- Los fallos de autenticación repetidos se limitan por tasa según la dirección del cliente para ralentizar intentos de fuerza bruta.
- Si utilizas enrutamiento multiagente, establece `hooks.allowedAgentIds` para limitar la selección explícita de `agentId`.
- Mantén `hooks.allowRequestSessionKey=false` a menos que necesites sesiones seleccionadas por el solicitante.
- Si habilitas `sessionKey` en la solicitud, restringe `hooks.allowedSessionKeyPrefixes` (por ejemplo, `["hook:"]`).
- Evite incluir payloads sin procesar sensibles en los registros de webhooks.
- Los payloads de hook se tratan como no confiables y, por defecto, se envuelven con límites de seguridad.
  Si debe deshabilitar esto para un hook específico, configure `allowUnsafeExternalContent: true`
  en el mapeo de ese hook (peligroso).
