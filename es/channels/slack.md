---
summary: "Configuración de Slack para modo socket o webhook HTTP"
read_when:
  - Configurar Slack o depurar el modo socket/HTTP de Slack
title: "Slack"
---

# Slack

Estado: listo para producción para DMs + canales mediante integraciones de apps de Slack. Modo HTTP (Events API)

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Los DMs de Slack usan el modo de emparejamiento de forma predeterminada.
</Card>
  <Card title="Slash commands" icon="terminal" href="/tools/slash-commands">    Comportamiento nativo de los comandos y catálogo de comandos.
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">    Diagnósticos entre canales y guías de resolución.
  
</Card>
</CardGroup>

## Configuración rápida (principiante)

<Tabs>
  <Tab title="Socket Mode (default)">
    <Steps>
      <Step title="Create Slack app and tokens">        En la configuración de la app de Slack:

        ```
        **OAuth & Permissions** → instale la app y copie el **Bot User OAuth Token** (`xoxb-...`).
        ```

```json5
{
  channels: {
    slack: {
      enabled: true,
      appToken: "xapp-...",
      botToken: "xoxb-...",
    },
  },
}
```

        ```
            Fallback de entorno (solo cuenta predeterminada):
        ```

```bash
`SLACK_APP_TOKEN=xapp-...`
```

        
</Step>
      
        <Step title="Suscribirse a eventos de la app">
          Suscribe los eventos del bot para:
      
          - `app_mention`
          - `message.channels`, `message.groups`, `message.im`, `message.mpim`
          - `reaction_added`, `reaction_removed`
          - `member_joined_channel`, `member_left_channel`
          - `channel_rename`
          - `pin_added`, `pin_removed`
      
          Habilita también la **pestaña Mensajes** de App Home para los DMs.
        
</Step>
      
        <Step title="Iniciar gateway">

```bash
openclaw gateway
```

        
</Step>
      
</Steps>

  
</Tab>

  <Tab title="HTTP Events API mode">
    <Steps>
      <Step title="Configure Slack app for HTTP">

        ```
        Use el modo de webhook HTTP cuando su Gateway sea accesible por Slack a través de HTTPS (típico en despliegues de servidor). El modo HTTP usa Events API + Interactivity + Slash Commands con una URL de solicitud compartida.
        ```

```json5
{
  channels: {
    slack: {
      enabled: true,
      mode: "http",
      botToken: "xoxb-...",
      signingSecret: "your-signing-secret",
      webhookPath: "/slack/events",
    },
  },
}
```

      Modo HTTP multi‑cuenta: configure `channels.slack.accounts.<id> .mode = "http"` y proporcione un
      `webhookPath` único por cuenta para que cada app de Slack apunte a su propia URL.

  
</Tab>
</Tabs>

## Modelo de tokens

- `botToken` + `appToken` son obligatorios para el modo Socket.
- El modo HTTP requiere `botToken` + `signingSecret`.
- Los tokens de configuración anulan el fallback de entorno.
- El fallback de entorno `SLACK_BOT_TOKEN` / `SLACK_APP_TOKEN` se aplica solo a la cuenta predeterminada.
- Ejemplo con userTokenReadOnly configurado explícitamente (permitir escrituras con token de usuario):
- Opcional: añade `chat:write.customize` si quieres que los mensajes salientes usen la identidad del agente activo (`username` e icono personalizados). `icon_emoji` usa la sintaxis `:emoji_name:`.

<Tip>
Las operaciones de escritura (enviar/editar/eliminar mensajes, agregar/quitar reacciones, fijar/desfijar,
  cargas de archivos) usan el token del bot de forma predeterminada. Si `userTokenReadOnly: false` y
  no hay token de bot disponible, OpenClaw recurre al token de usuario.
</Tip>

## Control de acceso y enrutamiento

<Tabs>
  <Tab title="DM policy">DMs ignorados: el remitente no está aprobado cuando `channels.slack.dm.policy="pairing"`.

    ```
    Para permitir a cualquiera: configure `channels.slack.dm.policy="open"` y `channels.slack.dm.allowFrom=["*"]`.
    ```

  
</Tab>

  <Tab title="Channel policy">`channels.slack.groupPolicy` controla el manejo de canales (`open|disabled|allowlist`).

    ```
    `allowlist` requiere que los canales estén listados en `channels.slack.channels`.
    ```

  
</Tab>

  <Tab title="Mentions and channel users">    Los mensajes en canales están restringidos por mención de forma predeterminada.

    ```
    El control por mención se gestiona mediante `channels.slack.channels` (configure `requireMention` en `true`); `agents.list[].groupChat.mentionPatterns` (o `messages.groupChat.mentionPatterns`) también cuentan como menciones.
    ```

  
</Tab>
</Tabs>

## Comandos y comportamiento de slash

- El registro de comandos nativos usa `commands.native` (predeterminado global `"auto"` → Slack desactivado) y puede anularse por espacio de trabajo con `channels.slack.commands.native`.
- {
  channels: {
  slack: {
  enabled: true,
  appToken: "xapp-...",
  botToken: "xoxb-...",
  userToken: "xoxp-...",
  userTokenReadOnly: false,
  },
  },
  }
- Cuando los comandos nativos están habilitados, registra los comandos slash correspondientes en Slack (nombres `/<command>`).
- Si los comandos nativos no están habilitados, puedes ejecutar un único comando slash configurado mediante `channels.slack.slashCommand`.

Configuración predeterminada del comando slash:

- `enabled: false`
- `name: "openclaw"`
- `sessionPrefix: "slack:slash"`
- `ephemeral: true`

Las sesiones de slash usan claves aisladas:

- Los slash commands usan sesiones `agent:<agentId>:slack:slash:<userId>` (prefijo configurable mediante `channels.slack.slashCommand.sessionPrefix`).

y aun así enrutan la ejecución del comando contra la sesión de conversación de destino (`CommandTargetSessionKey`).

## Hilos, sesiones y etiquetas de respuesta

- Los DMs se enrutan como `direct`; los canales como `channel`; los MPIM como `group`.
- Con el valor predeterminado `session.dmScope=main`, los DMs de Slack se consolidan en la sesión principal del agente.
- Los canales se asignan a sesiones `agent:<agentId>:slack:channel:<channelId>`.
- Las respuestas en hilos pueden crear sufijos de sesión de hilo (`:thread:<threadTs>`) cuando corresponda.
- El valor predeterminado de `channels.slack.thread.historyScope` es `thread`; el de `thread.inheritParent` es `false`.
- `channels.slack.historyLimit` (o `channels.slack.accounts.*.historyLimit`) controla cuántos mensajes recientes del canal/grupo se incluyen en el prompt.

Controles de respuesta en hilos:

- {
  channels: {
  slack: {
  replyToMode: "off",
  replyToModeByChatType: { group: "first" },
  },
  },
  }
- {
  channels: {
  slack: {
  replyToMode: "first",
  replyToModeByChatType: { direct: "off", group: "off" },
  },
  },
  }
- alternativa heredada para chats directos: `channels.slack.dm.replyToMode`

Se admiten etiquetas de respuesta manuales:

- `[[reply_to_current]]`
- `[[reply_to:<id>]]` — responder a un id de mensaje específico.

Nota: `replyToMode="off"` desactiva el encadenamiento implícito de respuestas. Las etiquetas explícitas `[[reply_to_*]]` siguen respetándose.

## Medios, fragmentación y entrega

<AccordionGroup>
  <Accordion title="Inbound attachments">
    Los archivos adjuntos de Slack se descargan desde URLs privadas alojadas en Slack (flujo de solicitud autenticado por token) y se escriben en el almacén de medios cuando la obtención se realiza correctamente y los límites de tamaño lo permiten.

    ```
    Las cargas de medios están limitadas por `channels.slack.mediaMaxMb` (predeterminado 20).
    ```

  
</Accordion>

  <Accordion title="Outbound text and files">
    - los fragmentos de texto usan `channels.slack.textChunkLimit` (predeterminado 4000)
    - `channels.slack.chunkMode="newline"` habilita la división priorizando párrafos
    - los envíos de archivos usan las API de carga de Slack y pueden incluir respuestas en hilos (`thread_ts`)
    - el límite de medios salientes sigue `channels.slack.mediaMaxMb` cuando está configurado; de lo contrario, los envíos del canal usan los valores predeterminados por tipo MIME del pipeline de medios
  
</Accordion>

  <Accordion title="Delivery targets">    Destinos explícitos preferidos:

    ```
    - `user:<id>` para MD
    - `channel:<id>` para canales
    
    Las MD de Slack se abren mediante las API de conversaciones de Slack al enviar a destinos de usuario.
    ```

  
</Accordion>
</AccordionGroup>

## Acciones y controles

Las acciones de herramientas de Slack pueden controlarse con `channels.slack.actions.*`:

Grupos de acciones disponibles en la herramienta actual de Slack:

| Política de grupos | Predeterminado |
| ------------------ | -------------- |
| messages           | habilitado     |
| reactions          | habilitado     |
| pins               | habilitado     |
| memberInfo         | habilitado     |
| emojiList          | habilitado     |

## Eventos y comportamiento operativo

- Las ediciones/eliminaciones de mensajes y las transmisiones en hilos se asignan a eventos del sistema.
- Los eventos de añadir/eliminar reacciones se asignan a eventos del sistema.
- Los eventos de entrada/salida de miembros, creación/renombrado de canales y añadir/eliminar fijaciones se asignan a eventos del sistema.
- `channel_id_changed` puede migrar claves de configuración del canal cuando `configWrites` está habilitado.
- Los metadatos de tema/propósito del canal se tratan como contexto no confiable y pueden inyectarse en el contexto de enrutamiento.

## Reaccionar + listar reacciones

`ackReaction` envía un emoji de confirmación mientras OpenClaw está procesando un mensaje entrante.

Orden de resolución:

- `o`channels.slack.channels.<name> .ackReaction\`
- `channels.slack.ackReaction`
- `replyToMode`
- emoji de identidad del agente como alternativa (`agents.list[].identity.emoji`, en caso contrario "👀")

Notas

- Slack espera códigos cortos (por ejemplo, `"eyes"`).
- Usa `""` para desactivar la reacción en un canal o cuenta.

## Lista de verificación de manifest y scopes

<AccordionGroup>
  <Accordion title="Slack app manifest example">

```json
{
  "display_information": {
    "name": "OpenClaw",
    "description": "Slack connector for OpenClaw"
  },
  "features": {
    "bot_user": {
      "display_name": "OpenClaw",
      "always_online": false
    },
    "app_home": {
      "messages_tab_enabled": true,
      "messages_tab_read_only_enabled": false
    },
    "slash_commands": [
      {
        "command": "/openclaw",
        "description": "Send a message to OpenClaw",
        "should_escape": false
      }
    ]
  },
  "oauth_config": {
    "scopes": {
      "bot": [
        "chat:write",
        "channels:history",
        "channels:read",
        "groups:history",
        "groups:read",
        "groups:write",
        "im:history",
        "im:read",
        "im:write",
        "mpim:history",
        "mpim:read",
        "mpim:write",
        "users:read",
        "app_mentions:read",
        "reactions:read",
        "reactions:write",
        "pins:read",
        "pins:write",
        "emoji:read",
        "commands",
        "files:read",
        "files:write"
      ],
      "user": [
        "channels:history",
        "channels:read",
        "groups:history",
        "groups:read",
        "im:history",
        "im:read",
        "mpim:history",
        "mpim:read",
        "users:read",
        "reactions:read",
        "pins:read",
        "emoji:read",
        "search:read"
      ]
    }
  },
  "settings": {
    "socket_mode_enabled": true,
    "event_subscriptions": {
      "bot_events": [
        "app_mention",
        "message.channels",
        "message.groups",
        "message.im",
        "message.mpim",
        "reaction_added",
        "reaction_removed",
        "member_joined_channel",
        "member_left_channel",
        "channel_rename",
        "pin_added",
        "pin_removed"
      ]
    }
  }
}
```

  
</Accordion>

  <Accordion title="Optional user-token scopes (read operations)">
    Si configuras `channels.slack.userToken`, los scopes de lectura habituales son:

    ```
    `channels:history`, `groups:history`, `im:history`, `mpim:history`
    [https://docs.slack.dev/reference/methods/conversations.history](https://docs.slack.dev/reference/methods/conversations.history)
    ```

  
</Accordion>
</AccordionGroup>

## Solución de problemas

<AccordionGroup>
  <Accordion title="No replies in channels">
    Comprueba, en orden:

    ```
    `users`: lista de permitidos de usuarios opcional por canal.
    ```

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

  
</Accordion>

  <Accordion title="DM messages ignored">
    Comprueba:

    ```
    Para permitir **ningún canal**, configure `channels.slack.groupPolicy: "disabled"` (o mantenga una lista de permitidos vacía).
    ```

```bash
openclaw pairing list slack
```

  
</Accordion>

  <Accordion title="Socket mode not connecting">
    Valida los tokens de bot + app y la habilitación de Socket Mode en la configuración de la app de Slack.
  
</Accordion>

  <Accordion title="HTTP mode not receiving events">
    Valida:

    ```
    - signing secret
    - ruta del webhook
    - URLs de solicitud de Slack (Events + Interactivity + Slash Commands)
    - `webhookPath` único por cuenta HTTP
    ```

  
</Accordion>

  <Accordion title="Native/slash commands not firing">
    Verifica si tu intención era:

    ```
    Slash Commands → cree `/openclaw` si usa `channels.slack.slashCommand`. Si habilita comandos nativos, agregue un comando slash por cada comando integrado (los mismos nombres que `/help`). De forma predeterminada, los nativos están desactivados para Slack a menos que configure `channels.slack.commands.native: true` (el valor global `commands.native` es `"auto"`, lo que deja Slack desactivado).
    ```

  
</Accordion>
</AccordionGroup>

## Punteros de referencia de configuración

Referencia principal:

- `users:read` (búsqueda de usuarios)
  [https://docs.slack.dev/reference/methods/users.info](https://docs.slack.dev/reference/methods/users.info)

  Campos de Slack más relevantes:

  - modo/autenticación: `mode`, `botToken`, `appToken`, `signingSecret`, `webhookPath`, `accounts.*`
  - Acceso a MD: `dm.enabled`, `dmPolicy`, `allowFrom` (heredado: `dm.policy`, `dm.allowFrom`), `dm.groupEnabled`, `dm.groupChannels`
  - `allow`: permitir/denegar el canal cuando `groupPolicy="allowlist"`.
  - hilos/historial: `replyToMode`, `replyToModeByChatType`, `thread.*`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`
  - entrega: `textChunkLimit`, `chunkMode`, `mediaMaxMb`
  - operaciones/funcionalidades: `configWrites`, `commands.native`, `slashCommand.*`, `actions.*`, `userToken`, `userTokenReadOnly`

## Relacionado

- [Pairing](/channels/pairing)
- `channel`: canales estándar (públicos/privados)
- Para el flujo de triaje: [/channels/troubleshooting](/channels/troubleshooting).
- Configuración (modo HTTP)
- Lista completa de comandos + configuración: [Slash commands](/tools/slash-commands)

