---
title: "Emparejamiento"
---

# Emparejamiento

“El emparejamiento” es el paso explícito de **aprobación del propietario** de OpenClaw.
Se utiliza en dos lugares:

1. **Emparejamiento de mensajes directos (DM)** (quién tiene permitido hablar con el bot)
2. **Emparejamiento de nodos** (qué dispositivos/nodos tienen permitido unirse a la red del Gateway)

Contexto de seguridad: [Security](/gateway/security)

## 1. Emparejamiento de mensajes directos (acceso entrante al chat)

Cuando un canal se configura con la política de mensajes directos `pairing`, los remitentes desconocidos reciben un código corto y su mensaje **no se procesa** hasta que usted apruebe.

Las políticas predeterminadas de mensajes directos están documentadas en: [Security](/gateway/security)

Códigos de emparejamiento:

- 8 caracteres, en mayúsculas, sin caracteres ambiguos (`0O1I`).
- **Expiran después de 1 hora**. El bot solo envía el mensaje de emparejamiento cuando se crea una nueva solicitud (aproximadamente una vez por hora por remitente).
- Las solicitudes pendientes de emparejamiento de mensajes directos están limitadas a **3 por canal** de forma predeterminada; las solicitudes adicionales se ignoran hasta que una expire o sea aprobada.

### Aprobar un remitente

```bash
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

Canales compatibles: `telegram`, `whatsapp`, `signal`, `imessage`, `discord`, `slack`.

### Dónde vive el estado

Almacenado bajo `~/.openclaw/credentials/`:

- Solicitudes pendientes: `<channel>-pairing.json`
- Almacén de lista de permitidos aprobados: `<channel>-allowFrom.json`

Trate estos elementos como sensibles (controlan el acceso a su asistente).

## 2. Emparejamiento de dispositivos de nodo (nodos iOS/Android/macOS/headless)

Los nodos se conectan al Gateway como **dispositivos** con `role: node`. El Gateway
crea una solicitud de emparejamiento de dispositivo que debe ser aprobada.

### Emparejar vía Telegram (recomendado para iOS)

Si usas el plugin `device-pair`, puedes realizar el emparejamiento inicial del dispositivo completamente desde Telegram:

1. En Telegram, envía un mensaje a tu bot: `/pair`
2. El bot responde con dos mensajes: un mensaje de instrucciones y un **código de configuración** separado (fácil de copiar/pegar en Telegram).
3. En tu teléfono, abre la app de OpenClaw en iOS → Settings → Gateway.
4. Pega el código de configuración y conéctate.
5. De vuelta en Telegram: `/pair approve`

El código de configuración es una carga útil JSON codificada en base64 que contiene:

- `url`: the Gateway WebSocket URL (`ws://...` or `wss://...`)
- `token`: a short-lived pairing token

Treat the setup code like a password while it is valid.

### Aprobar un dispositivo de nodo

```bash
openclaw devices list
openclaw devices approve <requestId>
openclaw devices reject <requestId>
```

### Almacenamiento del estado de emparejamiento de nodos

Almacenado bajo `~/.openclaw/devices/`:

- `pending.json` (de corta duración; las solicitudes pendientes expiran)
- `paired.json` (dispositivos emparejados + tokens)

### Notas

- La API heredada `node.pair.*` (CLI: `openclaw nodes pending/approve`) es un
  almacén de emparejamiento independiente propiedad del Gateway. Los nodos WS aún requieren emparejamiento de dispositivos.

## Documentación relacionada

- Modelo de seguridad + inyección de prompts: [Security](/gateway/security)
- Actualización segura (ejecutar doctor): [Updating](/install/updating)
- Configuraciones de canales:
  - Telegram: [Telegram](/channels/telegram)
  - WhatsApp: [WhatsApp](/channels/whatsapp)
  - Signal: [Signal](/channels/signal)
  - BlueBubbles (iMessage): [BlueBubbles](/channels/bluebubbles)
  - iMessage (heredado): [iMessage](/channels/imessage)
  - Discord: [Discord](/channels/discord)
  - Slack: [Slack](/channels/slack)

