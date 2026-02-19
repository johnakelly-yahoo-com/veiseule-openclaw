---
summary: "Soporte del canal WhatsApp, controles de acceso, comportamiento de entrega y operaciones"
read_when:
  - Al trabajar en el comportamiento del canal WhatsApp/web o el enrutamiento de la bandeja de entrada
title: "WhatsApp"
---

# WhatsApp (canal web)

Estado: WhatsApp Web vía Baileys únicamente. El Gateway es propietario de la(s) sesión(es).

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">La política de DM predeterminada es **emparejamiento**, por lo que los remitentes desconocidos solo reciben un código de emparejamiento y su mensaje **no se procesa**.
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    Diagnósticos entre canales y guías de reparación.
  
</Card>
  <Card title="Gateway configuration" icon="settings" href="/gateway/configuration">
    Patrones y ejemplos completos de configuración de canales.
  
</Card>
</CardGroup>

## Configuración rápida (principiante)

<Steps>
  <Step title="Configure WhatsApp access policy">

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551234567"],
    },
  },
}
```

  
</Step>

  <Step title="Link WhatsApp (QR)">

```bash
openclaw channels login --channel whatsapp
```

    ```
    Para una cuenta específica:
    ```

```bash
openclaw channels login --channel whatsapp --account work
```

  
</Step>

  <Step title="Start the gateway">

```bash
openclaw gateway
```

  
</Step>

  <Step title="Approve first pairing request (if using pairing mode)">

```bash
Apruebe con: `openclaw pairing approve whatsapp <code>` (liste con `openclaw pairing list whatsapp`).
```

    ```
    Los códigos expiran después de 1 hora; las solicitudes pendientes se limitan a 3 por canal.
    ```

  
</Step>
</Steps>

<Note>
Ideal para mantener su WhatsApp personal separado: instale WhatsApp Business y registre allí el número de OpenClaw. (Los metadatos del canal y el flujo de incorporación están optimizados para esa configuración, pero también se admiten configuraciones con número personal.)
</Note>

## Patrones de despliegue

<AccordionGroup>
  <Accordion title="Dedicated number (recommended)">
    Este es el modo operativo más limpio:


    ```
    {
      channels: {
        whatsapp: {
          dmPolicy: "allowlist",
          allowFrom: ["+15551234567"],
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="Personal-number fallback">
    La incorporación admite el modo con número personal y escribe una configuración base compatible con autochat:


    ```
    {
      "whatsapp": {
        "selfChatMode": true,
        "dmPolicy": "allowlist",
        "allowFrom": ["+15551234567"]
      }
    }
    ```

  
</Accordion>

  <Accordion title="WhatsApp Web-only channel scope">El canal de la plataforma de mensajería está basado en WhatsApp Web (`Baileys`) en la arquitectura actual de canales de OpenClaw.

    ```
    No existe un canal de mensajería WhatsApp separado de Twilio en el registro integrado de canales de chat.
    ```

  
</Accordion>
</AccordionGroup>

## Modelo de ejecución

- Gateway es responsable del socket de WhatsApp y del bucle de reconexión.
- Los envíos salientes requieren un listener activo de WhatsApp para la cuenta de destino.
- Los chats de estado y difusión se ignoran (`@status`, `@broadcast`).
- Los chats directos usan las reglas de sesión DM (`session.dmScope`; por defecto `main` consolida los DMs en la sesión principal del agente).
- Los grupos se asignan a sesiones `agent:<agentId>:whatsapp:group:<jid>`.

## Control de acceso y activación

<Tabs>
  <Tab title="DM policy">**Política de DM**: `channels.whatsapp.dmPolicy` controla el acceso a chats directos (predeterminado: `pairing`).

    ```
    Abierto: requiere que `channels.whatsapp.allowFrom` incluya `"*"`.
    ```

  
</Tab>

  <Tab title="Group policy + allowlists">
    El acceso a grupos tiene dos capas:

    ```
    `channels.whatsapp.groups` (lista de permitidos de grupos + valores predeterminados de control por mención; use `"*"` para permitir todo)
    ```

  
</Tab>

  <Tab title="Mentions + /activation">
    Las respuestas en grupos requieren mención por defecto.

    ```
    La detección de menciones incluye:
    
    - menciones explícitas en WhatsApp de la identidad del bot
    - patrones regex de mención configurados (`agents.list[].groupChat.mentionPatterns`, alternativa `messages.groupChat.mentionPatterns`)
    - detección implícita de respuesta al bot (el remitente de la respuesta coincide con la identidad del bot)
    
    Comando de activación a nivel de sesión:
    
    - `/activation mention`
    - `/activation always`
    
    `activation` actualiza el estado de la sesión (no la configuración global). Está restringido al propietario.
    ```

  
</Tab>
</Tabs>

## Comportamiento del número personal y del chat consigo mismo

Cuando el número propio vinculado también está presente en `allowFrom`, se activan las protecciones de WhatsApp para el chat propio:

- omitir confirmaciones de lectura en los turnos de chat propio
- ignorar el comportamiento de activación automática por mención-JID que de otro modo te enviaría un ping a ti mismo
- Las respuestas de autochat usan por defecto `[{identity.name}]` cuando se establece (de lo contrario `[openclaw]`)
  si `messages.responsePrefix` no está configurado.

## Normalización de mensajes y contexto

<AccordionGroup>
  <Accordion title="Inbound envelope + reply context">
    Los mensajes entrantes de WhatsApp se envuelven en el sobre de entrada compartido.

    ````
    Si existe una respuesta citada, el contexto se añade con este formato:
    
    ```text
    [Replying to <sender> id:<stanzaId>]
    <quoted body or media placeholder>
    [/Replying]
    ```
    
    Los campos de metadatos de respuesta también se completan cuando están disponibles (`ReplyToId`, `ReplyToBody`, `ReplyToSender`, sender JID/E.164).
    ````

  
</Accordion>

  <Accordion title="Media placeholders and location/contact extraction">Los mensajes entrantes solo de medios usan marcadores de posición:

    ```
    - `<media:image>`
    - `<media:video>`
    - `<media:audio>`
    - `<media:document>`
    - `<media:sticker>`
    
    Las cargas de ubicación y contacto se normalizan en contexto textual antes del enrutamiento.
    ```

  
</Accordion>

  <Accordion title="Pending group history injection">
    En los grupos, los mensajes no procesados pueden almacenarse en búfer e inyectarse como contexto cuando finalmente se activa el bot.

    ```
    Mensajes recientes _no procesados_ (predeterminado 50) insertados bajo:
    `[Chat messages since your last reply - for context]` (los mensajes ya en la sesión no se reinyectan)
    ```

  
</Accordion>

  <Accordion title="Read receipts">De forma predeterminada, el gateway marca los mensajes entrantes de WhatsApp como leídos (doble check azul) una vez aceptados.

    ```
    {
      channels: {
        whatsapp: {
          accounts: {
            personal: { sendReadReceipts: false },
          },
        },
      },
    }
    ```

  
</Accordion>
</AccordionGroup>

## Entrega, fragmentación y contenido multimedia

<AccordionGroup>
  <Accordion title="Text chunking">Fragmentación opcional por nueva línea: configure `channels.whatsapp.chunkMode="newline"` para dividir en líneas en blanco (límites de párrafo) antes de fragmentar por longitud.
</Accordion>

  <Accordion title="Outbound media behavior">
    - admite cargas de imagen, video, audio (nota de voz PTT) y documentos
    - `audio/ogg` se reescribe como `audio/ogg; codecs=opus` para compatibilidad con notas de voz
    - la reproducción de GIF animados es compatible mediante `gifPlayback: true` en envíos de video
    - las leyendas se aplican al primer elemento multimedia al enviar respuestas con múltiples elementos
    - la fuente del contenido multimedia puede ser HTTP(S), `file://` o rutas locales
  
</Accordion>

  <Accordion title="Media size limits and fallback behavior">
    - límite de guardado de contenido multimedia entrante: `channels.whatsapp.mediaMaxMb` (predeterminado `50`)
    - límite de contenido multimedia saliente para respuestas automáticas: `agents.defaults.mediaMaxMb` (predeterminado `5MB`)
    - las imágenes se optimizan automáticamente (redimensionado/ajuste de calidad) para ajustarse a los límites
    - si falla el envío de contenido multimedia, el primer elemento recurre a enviar una advertencia de texto en lugar de descartar la respuesta silenciosamente
  
</Accordion>
</AccordionGroup>

## Reacciones de confirmación

`channels.whatsapp.ackReaction` (auto-reacción al recibir mensajes: `{emoji, direct, group}`).

```json5
{
  "whatsapp": {
    "ackReaction": {
      "emoji": "👀",
      "direct": true,
      "group": "mentions"
    }
  }
}
```

Notas:

- enviadas inmediatamente después de aceptar la entrada (antes de la respuesta)
- los fallos se registran, pero no bloquean la entrega normal de la respuesta
- el modo de grupo `mentions` reacciona en los turnos activados por mención; la activación de grupo `always` actúa como omisión de esta comprobación
- WhatsApp ignora `messages.ackReaction`; use `channels.whatsapp.ackReaction` en su lugar.

## Multicuenta y credenciales

<AccordionGroup>
  <Accordion title="Account selection and defaults">`channels.whatsapp.accounts.<accountId> .*` (configuración por cuenta + `authDir` opcional).
</Accordion>

  <Accordion title="Credential paths and legacy compatibility">Configure WhatsApp en `~/.openclaw/openclaw.json`.<accountId>Credenciales almacenadas en `~/.openclaw/credentials/whatsapp/<accountId>/creds.json`.
</Accordion>

  <Accordion title="Logout behavior">Inicio de sesión multicuenta: `openclaw channels login --account <id>` (`<id>` = `accountId`).<id>]` borra el estado de autenticación de WhatsApp para esa cuenta.

    ```
    En los directorios de autenticación heredados, `oauth.json` se conserva mientras se eliminan los archivos de autenticación de Baileys.
    ```

  
</Accordion>
</AccordionGroup>

## Herramientas, acciones y escrituras de configuración

- Herramienta: `whatsapp` con acción `react` (`chatJid`, `messageId`, `emoji`, `remove` opcional).
- Controles de acción:
  - `channels.whatsapp.actions.reactions` (control de reacciones de herramientas de WhatsApp).
  - \`channels.whatsapp.accounts.<accountId>
- {
  channels: { whatsapp: { configWrites: false } },
  }

## Solución de problemas (rápida)

<AccordionGroup>
  <Accordion title="Not linked (QR required)">Síntoma: `channels status` muestra `linked: false` o advierte “No vinculado”.

    ```
    Cerrar sesión: `openclaw channels logout` (o `--account <id>`) elimina el estado de autenticación de WhatsApp (pero conserva el `oauth.json` compartido).
    ```

  
</Accordion>

  <Accordion title="Linked but disconnected / reconnect loop">
    Síntoma: cuenta vinculada con desconexiones repetidas o intentos de reconexión.

    ```
    Solución: `openclaw doctor` (o reinicie el gateway). Si persiste, vuelva a vincular vía `channels login` e inspeccione `openclaw logs --follow`.
    ```

  
</Accordion>

  <Accordion title="No active listener when sending">
    Los envíos salientes fallan rápidamente cuando no existe un listener activo del gateway para la cuenta de destino.

    ```
    Asegúrate de que el gateway esté en ejecución y que la cuenta esté vinculada.
    ```

  
</Accordion>

  <Accordion title="Group messages unexpectedly ignored">
    Comprueba en este orden:

    ```
    Política de grupos: `channels.whatsapp.groupPolicy = open|disabled|allowlist` (predeterminado `allowlist`).
    ```

  
</Accordion>

  <Accordion title="Bun runtime warning">
    El entorno de ejecución del gateway de WhatsApp debe usar Node. Bun está marcado como incompatible para un funcionamiento estable del gateway de WhatsApp/Telegram.
  
</Accordion>
</AccordionGroup>

## Punteros de referencia de configuración

Referencia principal:

- WhatsApp Web envía mensajes estándar (sin hilos de respuesta citada en el gateway actual).

Campos de WhatsApp de alta relevancia:

- access: `dmPolicy`, `allowFrom`, `groupPolicy`, `groupAllowFrom`, `groups`
- delivery: `textChunkLimit`, `chunkMode`, `mediaMaxMb`, `sendReadReceipts`, `ackReaction`
- multi-account: `accounts.<id>`.enabled`, `accounts.<id>`.authDir`, anulaciones a nivel de cuenta
- operaciones: `configWrites`, `debounceMs`, `web.enabled`, `web.heartbeatSeconds`, `web.reconnect.*`
- comportamiento de la sesión: `session.dmScope`, `historyLimit`, `dmHistoryLimit`, `dms.<id>``messages.groupChat.historyLimit`

## Relacionado

- [Pairing](/channels/pairing)
- [Channel routing](/channels/channel-routing)
- Guía de solución de problemas: [Solución de problemas del Gateway](/gateway/troubleshooting).

