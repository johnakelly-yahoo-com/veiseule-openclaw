---
summary: "Estado del soporte del bot de Telegram, capacidades y configuración"
read_when:
  - Trabajo en funciones de Telegram o webhooks
title: "Telegram"
---

# Telegram (Bot API)

Estado: listo para producción para mensajes directos del bot + grupos mediante grammY. Long-polling por defecto; webhook opcional.

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">Default DM policy para Telegram es emparejamiento.
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">Diagnósticos entre canales y guías de resolución paso a paso.
</Card>
  <Card title="Gateway configuration" icon="settings" href="/gateway/configuration">Patrones y ejemplos completos de configuración de canales.
</Card>
</CardGroup>

## Configuración (ruta rápida)

<Steps>
  <Step title="Create the bot token in BotFather">Abra Telegram y chatee con **@BotFather** ([enlace directo](https://t.me/BotFather)).

    ```
    Ejecute `/newbot`, luego siga las indicaciones (nombre + nombre de usuario que termine en `bot`).
    ```

  
</Step>

  <Step title="Configure token and DM policy">

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123:abc",
      dmPolicy: "pairing",
      groups: { "*": { requireMention: true } },
    },
  },
}
```

    ```
    Opción por env: `TELEGRAM_BOT_TOKEN=...` (funciona para la cuenta predeterminada).
    ```

  
</Step>

  <Step title="Start gateway and approve first DM">

```bash
`openclaw pairing approve telegram <CODE>`
```

    ```
    Los remitentes desconocidos reciben un código de emparejamiento; los mensajes se ignoran hasta aprobarse (los códigos expiran tras 1 hora).
    ```

  
</Step>

  <Step title="Add the bot to a group">Para grupos: agregue el bot, decida el comportamiento de privacidad/admin (abajo), luego configure `channels.telegram.groups` para controlar el bloqueo por menciones + listas de permitidos.
</Step>
</Steps>

<Note>
El orden de resolución de tokens tiene en cuenta la cuenta. En la práctica, los valores de configuración prevalecen sobre la variable de entorno de respaldo, y `TELEGRAM_BOT_TOKEN` solo se aplica a la cuenta predeterminada.
</Note>

## Configuración del lado de Telegram

<AccordionGroup>
  <Accordion title="Privacy mode and group visibility">**El bot no ve mensajes del grupo en absoluto:**

    ```
    **Nota:** Al alternar el modo de privacidad, Telegram requiere eliminar y volver a agregar el bot
    a cada grupo para que el cambio tenga efecto.
    ```

  
</Accordion>

  <Accordion title="Group permissions">El estado de admin se configura dentro del grupo (UI de Telegram).

    ```
    Agregar el bot como **admin** del grupo (los bots admin reciben todos los mensajes).
    ```

  
</Accordion>

  <Accordion title="Helpful BotFather toggles">

    ```
    `/setjoingroups` — permitir/denegar agregar el bot a grupos.
    ```

  
</Accordion>
</AccordionGroup>

## Control de acceso y activación

<Tabs>
  <Tab title="DM policy">
    `channels.telegram.dmPolicy` controla el acceso a mensajes directos:

    ```
    - `pairing` (predeterminado)
    - `allowlist`
    - `open` (requiere que `allowFrom` incluya `"*"`)
    - `disabled`
    
    `channels.telegram.allowFrom` acepta IDs numéricos de usuario de Telegram. Se aceptan y normalizan los prefijos `telegram:` / `tg:`.
    El asistente de incorporación acepta entradas `@username` y las resuelve a IDs numéricos.
    Si actualizaste y tu configuración contiene entradas `@username` en la allowlist, ejecuta `openclaw doctor --fix` para resolverlas (mejor esfuerzo; requiere un token de bot de Telegram).
    
    ### Cómo encontrar tu ID de usuario de Telegram
    
    Más seguro (sin bot de terceros):
    
    1. Envía un DM a tu bot.
    2. Ejecuta `openclaw logs --follow`.
    3. Lee `from.id`.
    
    Método oficial de la Bot API:
    ```

```bash
curl "https://api.telegram.org/bot<bot_token>/getUpdates"
```

    ```
    **Nota de privacidad:** `@userinfobot` es un bot de terceros.
    ```

  
</Tab>

  <Tab title="Group policy and allowlists">Dos controles independientes:

    ```
    {
      channels: {
        telegram: {
          groups: {
            "*": { requireMention: false }, // all groups, always respond
          },
        },
      },
    }
    ```

```json5
{
  channels: {
    telegram: {
      groups: {
        "-1001234567890": {
          groupPolicy: "open",
          requireMention: false,
        },
      },
    },
  },
}
```

  
</Tab>

  <Tab title="Mention behavior">Las respuestas en grupos requieren mención de forma predeterminada.

    ```
    La mención puede provenir de:
    
    - mención nativa `@botusername`, o
    - patrones de mención en:
      - `agents.list[].groupChat.mentionPatterns`
      - `messages.groupChat.mentionPatterns`
    
    Comandos de activación a nivel de sesión:
    
    - `/activation always`
    - `/activation mention`
    
    Estos actualizan solo el estado de la sesión. Usa la configuración para persistencia.
    
    Ejemplo de configuración persistente:
    ```

```json5
{
  channels: {
    telegram: {
      groups: {
        "*": { requireMention: true }, // or omit groups entirely
      },
    },
  },
}
```

    ```
    Reenvíe cualquier mensaje del grupo a `@userinfobot` o `@getidsbot` en Telegram para ver el ID del chat (número negativo como `-1001234567890`).
    ```

  
</Tab>
</Tabs>

## Comportamiento en tiempo de ejecución

- Un canal de la Bot API de Telegram propiedad del Gateway.
- Enrutamiento determinista: las respuestas regresan a Telegram; el modelo nunca elige canales.
- Los mensajes entrantes se normalizan en el sobre de canal compartido con contexto de respuesta y marcadores de medios.
- Las sesiones de grupo están aisladas por ID de grupo. Agrega `:topic:<threadId>` a la clave de sesión del grupo de Telegram para que cada tema quede aislado.
- Los mensajes DM pueden incluir `message_thread_id`; OpenClaw los enruta con claves de sesión sensibles al hilo y preserva el ID del hilo para las respuestas.
- El long polling utiliza el runner de grammY con secuenciación por chat/por hilo. El long‑polling usa el runner de grammY con secuenciación por chat; la concurrencia total está limitada por `agents.defaults.maxConcurrent`.
- La Bot API de Telegram no admite confirmaciones de lectura; no existe la opción `sendReadReceipts`.

## Referencia de funciones

<AccordionGroup>
  <Accordion title="Live stream preview (message edits)">OpenClaw puede transmitir respuestas parciales en mensajes directos de Telegram usando `sendMessageDraft`.

    ```
    Requisito:
    
    - `channels.telegram.streamMode` no es `"off"` (predeterminado: `"partial"`)
    
    Modos:
    
    - `off`: sin vista previa en vivo
    - `partial`: actualizaciones frecuentes de vista previa a partir de texto parcial
    - `block`: actualizaciones de vista previa por bloques usando `channels.telegram.draftChunk`
    
    Valores predeterminados de `draftChunk` para `streamMode: "block"`:
    
    - `minChars: 200`
    - `maxChars: 800`
    - `breakPreference: "paragraph"`
    
    `maxChars` está limitado por `channels.telegram.textChunkLimit`.
    
    Funciona en chats directos y en grupos/temas.
    
    Para respuestas solo de texto, OpenClaw mantiene el mismo mensaje de vista previa y realiza una edición final en el mismo (sin segundo mensaje).
    
    Para respuestas complejas (por ejemplo, cargas útiles de medios), OpenClaw vuelve al envío final normal y luego limpia el mensaje de vista previa.
    
    `streamMode` es independiente del block streaming. Cuando el block streaming está habilitado explícitamente para Telegram, OpenClaw omite la vista previa en streaming para evitar el doble streaming.
    
    Flujo de razonamiento solo para Telegram:
    
    - `/reasoning stream` envía el razonamiento a la vista previa en vivo mientras se genera
    - la respuesta final se envía sin el texto de razonamiento
    ```

  
</Accordion>

  <Accordion title="Formatting and HTML fallback">El texto saliente de Telegram usa `parse_mode: "HTML"` (el subconjunto de etiquetas compatibles de Telegram).

    ```
    - El texto tipo Markdown se renderiza a HTML seguro para Telegram.
    - El HTML crudo del modelo se escapa para reducir errores de análisis de Telegram.
    - Si Telegram rechaza el HTML analizado, OpenClaw reintenta como texto plano.
    
    Las vistas previas de enlaces están habilitadas por defecto y pueden deshabilitarse con `channels.telegram.linkPreview: false`.
    ```

  
</Accordion>

  <Accordion title="Native commands and custom commands">Some commands can be handled by plugins/skills without being registered in Telegram’s command menu.

    ```
    Puede agregar comandos personalizados al menú mediante config:
    ```

```json5
{
  channels: {
    telegram: {
      customCommands: [
        { command: "backup", description: "Git backup" },
        { command: "generate", description: "Create an image" },
      ],
    },
  },
}
```

    ```
    Reglas:
    
    - los nombres se normalizan (eliminar `/` inicial, minúsculas)
    - patrón válido: `a-z`, `0-9`, `_`, longitud `1..32`
    - los comandos personalizados no pueden sobrescribir comandos nativos
    - los conflictos/duplicados se omiten y se registran
    
    Notas:
    
    - los comandos personalizados son solo entradas de menú; no implementan comportamiento automáticamente
    - los comandos de plugin/skill pueden seguir funcionando al escribirse aunque no se muestren en el menú de Telegram
    
    Si los comandos nativos están deshabilitados, los integrados se eliminan. Los comandos personalizados/de plugin aún pueden registrarse si están configurados.
    
    Fallo común de configuración:
    
    - `setMyCommands failed` normalmente significa que el DNS/HTTPS saliente hacia `api.telegram.org` está bloqueado.
    
    ### Comandos de emparejamiento de dispositivo (plugin `device-pair`)
    
    Cuando el plugin `device-pair` está instalado:
    
    1. `/pair` genera un código de configuración
    2. pega el código en la app de iOS
    3. `/pair approve` aprueba la última solicitud pendiente
    
    Más detalles: [Pairing](/channels/pairing#pair-via-telegram-recommended-for-ios).
    ```

  
</Accordion>

  <Accordion title="Inline buttons">Configurar el alcance del teclado inline:

```json5
{
  channels: {
    telegram: {
      capabilities: {
        inlineButtons: "allowlist",
      },
    },
  },
}
```

    ```
    Anulación por cuenta:
    ```

```json5
{
  channels: {
    telegram: {
      accounts: {
        main: {
          capabilities: {
            inlineButtons: "allowlist",
          },
        },
      },
    },
  },
}
```

    ```
    `allowlist` — mensajes directos + grupos, pero solo remitentes permitidos por `allowFrom`/`groupAllowFrom` (mismas reglas que los comandos de control)
    ```

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  message: "Choose an option:",
  buttons: [
    [
      { text: "Yes", callback_data: "yes" },
      { text: "No", callback_data: "no" },
    ],
    [{ text: "Cancel", callback_data: "cancel" }],
  ],
}
```

    ```
    Cuando un usuario hace clic en un botón, los datos de callback se envían de vuelta al agente como un mensaje con el formato:
    `callback_data: value`
    ```

  
</Accordion>

  <Accordion title="Telegram message actions for agents and automation">
    Las acciones de herramientas de Telegram incluyen:

    ```
    - `sendMessage` (`to`, `content`, opcional `mediaUrl`, `replyToMessageId`, `messageThreadId`)
    - `react` (`chatId`, `messageId`, `emoji`)
    - `deleteMessage` (`chatId`, `messageId`)
    - `editMessage` (`chatId`, `messageId`, `content`)
    
    Las acciones de mensajes del canal exponen alias ergonómicos (`send`, `react`, `delete`, `edit`, `sticker`, `sticker-search`).
    
    Controles de habilitación:
    
    - `channels.telegram.actions.sendMessage`
    - `channels.telegram.actions.editMessage`
    - `channels.telegram.actions.deleteMessage`
    - `channels.telegram.actions.reactions`
    - `channels.telegram.actions.sticker` (predeterminado: deshabilitado)
    
    Semántica de eliminación de reacciones: [/tools/reactions](/tools/reactions)
    ```

  
</Accordion>

  <Accordion title="Reply threading tags">Telegram admite respuestas encadenadas opcionales mediante etiquetas:

    ```
    - `[[reply_to_current]]` responde al mensaje que activó la acción
    - `[[reply_to:<id>]]` responde a un ID específico de mensaje de Telegram
    
    `channels.telegram.replyToMode` controla el comportamiento:
    
    - `off` (predeterminado)
    - `first`
    - `all`
    
    Nota: `off` desactiva el encadenamiento implícito de respuestas. Las etiquetas explícitas `[[reply_to_*]]` siguen respetándose.
    ```

  
</Accordion>

  <Accordion title="Forum topics and thread behavior">Temas (supergrupos tipo foro)

    ```
    La configuración específica por tema está disponible en `channels.telegram.groups.<chatId> .topics.<threadId>
    ```

  
</Accordion>

  <Accordion title="Audio, video, and stickers">
    ### Mensajes de audio

    ```
    `[[audio_as_voice]]` — enviar audio como nota de voz en lugar de archivo.
    ```

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/voice.ogg",
  asVoice: true,
}
```

    ```
    Video messages (video vs video note)
    ```

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/video.mp4",
  asVideoNote: true,
}
```

    ```
    Las notas de video no admiten subtítulos; el texto del mensaje proporcionado se envía por separado.
    
    ### Stickers
    
    Manejo de stickers entrantes:
    
    - WEBP estático: descargado y procesado (marcador `<media:sticker>`)
    - TGS animado: omitido
    - WEBM de video: omitido
    
    Campos de contexto del sticker:
    
    - `Sticker.emoji`
    - `Sticker.setName`
    - `Sticker.fileId`
    - `Sticker.fileUniqueId`
    - `Sticker.cachedDescription`
    
    Archivo de caché de stickers:
    
    - `~/.openclaw/telegram/sticker-cache.json`
    
    Los stickers se describen una vez (cuando es posible) y se almacenan en caché para reducir llamadas repetidas de visión.
    
    Habilitar acciones de stickers:
    ```

```json5
{
  channels: {
    telegram: {
      actions: {
        sticker: true,
      },
    },
  },
}
```

    ```
    Envío de stickers
    ```

```json5
{
  action: "sticker",
  channel: "telegram",
  to: "123456789",
  fileId: "CAACAgIAAxkBAAI...",
}
```

    ```
    Caché de stickers
    ```

```json5
{
  action: "sticker-search",
  channel: "telegram",
  query: "cat waving",
  limit: 5,
}
```

  
</Accordion>

  <Accordion title="Reaction notifications">Recibe la actualización `message_reaction` de la API de Telegram

    ```
    Cuando está habilitado, OpenClaw pone en cola eventos del sistema como:
    
    - `Telegram reaction added: 👍 by Alice (@alice) on msg 42`
    
    Configuración:
    
    - `channels.telegram.reactionNotifications`: `off | own | all` (predeterminado: `own`)
    - `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` (predeterminado: `minimal`)
    
    Notas:
    
    - `own` significa reacciones de usuarios a mensajes enviados por el bot únicamente (mejor esfuerzo mediante caché de mensajes enviados).
    - Telegram no proporciona IDs de hilo en actualizaciones de reacciones.
      - los grupos no foro se enrutan a la sesión del chat de grupo
      - los grupos foro se enrutan a la sesión del tema general del grupo (`:topic:1`), no al tema exacto de origen
    
    `allowed_updates` para polling/webhook incluye `message_reaction` automáticamente.
    ```

  
</Accordion>

  <Accordion title="Ack reactions">**Cómo funcionan las reacciones:**
Las reacciones de Telegram llegan como **eventos `message_reaction` separados**, no como propiedades en las cargas de mensajes. Cuando un usuario agrega una reacción, OpenClaw:

    ```
    Orden de resolución:
    
    - `channels.telegram.accounts.<accountId>.ackReaction`
    - `channels.telegram.ackReaction`
    - `messages.ackReaction`
    - emoji de identidad del agente como fallback (`agents.list[].identity.emoji`, si no "👀")
    
    Notas:
    
    - Telegram espera emoji unicode (por ejemplo "👀").
    - Usa `""` para deshabilitar la reacción para un canal o cuenta.
    ```

  
</Accordion>

  <Accordion title="Config writes from Telegram events and commands">
    Las escrituras de configuración del canal están habilitadas por defecto (`configWrites !== false`).

    ```
    Por defecto, Telegram puede escribir actualizaciones de configuración activadas por eventos del canal o `/config set|unset`.
    ```

```json5
{
  channels: { telegram: { configWrites: false } },
}
```

  
</Accordion>

  <Accordion title="Long polling vs webhook">Predeterminado: long polling.

    ```
    Si su URL pública es diferente, use un proxy inverso y apunte `channels.telegram.webhookUrl` al endpoint público.
    ```

  
</Accordion>

  <Accordion title="Limits, retry, and CLI targets">
    El texto saliente se divide en `channels.telegram.textChunkLimit` (predeterminado 4000).
    División opcional por saltos de línea: configure `channels.telegram.chunkMode="newline"` para dividir en líneas en blanco (límites de párrafo) antes de dividir por longitud.
    Las descargas/cargas de medios están limitadas por `channels.telegram.mediaMaxMb` (predeterminado 5).
    - `channels.telegram.timeoutSeconds` sobrescribe el tiempo de espera del cliente de la API de Telegram (si no se establece, se aplica el valor predeterminado de grammY).
    El contexto de historial de grupos usa `channels.telegram.historyLimit` (o `channels.telegram.accounts.*.historyLimit`), con fallback a `messages.groupChat.historyLimit`. Configure `0` para deshabilitarlo (predeterminado 50).
    Anulaciones por usuario: `channels.telegram.dms["<user_id>"].historyLimit`.<user_id>"].historyLimit`
    - los reintentos de la API saliente de Telegram son configurables mediante `channels.telegram.retry`.

    ```
    El destino de envío en la CLI puede ser un ID numérico de chat o un nombre de usuario:
    ```

```bash
Ejemplo: `openclaw message send --channel telegram --target 123456789 --message "hi"`.
```

  
</Accordion>
</AccordionGroup>

## Solución de problemas

<AccordionGroup>
  <Accordion title="Bot does not respond to non mention group messages">

    ```
    Si configuró `channels.telegram.groups.*.requireMention=false`, el **modo de privacidad** de la Bot API de Telegram debe estar desactivado.
    ```

  
</Accordion>

  <Accordion title="Bot not seeing group messages at all">

    ```
    - cuando existe `channels.telegram.groups`, el grupo debe estar listado (o incluir `"*"`)
    - verifica la membresía del bot en el grupo
    - revisa los registros: `openclaw logs --follow` para conocer las razones de omisión
    ```

  
</Accordion>

  <Accordion title="Commands work partially or not at all">

    ```
    `setMyCommands failed` en los registros normalmente significa que HTTPS/DNS saliente está bloqueado hacia `api.telegram.org`.
    ```

  
</Accordion>

  <Accordion title="Polling or network instability">

    ```
    - Node 22+ + fetch/proxy personalizado puede provocar un comportamiento de aborto inmediato si los tipos de AbortSignal no coinciden.
    - Algunos hosts resuelven `api.telegram.org` primero a IPv6; una salida IPv6 defectuosa puede causar fallos intermitentes en la API de Telegram.
    - Valida las respuestas DNS:
    ```

```bash
dig +short api.telegram.org A
dig +short api.telegram.org AAAA
```

  
</Accordion>
</AccordionGroup>

Más ayuda: [Solución de problemas del canal](/channels/troubleshooting).

## Referencia de configuración (Telegram)

Referencia principal:

- `channels.telegram.enabled`: habilitar/deshabilitar el inicio del canal.

- `channels.telegram.botToken`: token del bot (BotFather).

- `channels.telegram.tokenFile`: leer el token desde una ruta de archivo.

- `channels.telegram.dmPolicy`: `pairing | allowlist | open | disabled` (predeterminado: emparejamiento).

- `channels.telegram.allowFrom`: lista de permitidos de mensajes directos (ids/nombres de usuario). `open` requiere `"*"`. `openclaw doctor --fix` puede resolver entradas heredadas `@username` a IDs.

- `channels.telegram.groupPolicy`: `open | allowlist | disabled` (predeterminado: lista de permitidos).

- `channels.telegram.groupAllowFrom`: lista de permitidos de remitentes de grupo (ids/nombres de usuario). `openclaw doctor --fix` puede resolver entradas heredadas `@username` a IDs.

- `channels.telegram.groups`: valores predeterminados por grupo + lista de permitidos (use `"*"` para valores globales).
  - `channels.telegram.groups.<id>.groupPolicy`: anulación por grupo de groupPolicy (`open | allowlist | disabled`).
  - `channels.telegram.groups.<id>.requireMention`: bloqueo por mención predeterminado.
  - `channels.telegram.groups.<id>.skills`: filtro de skills (omitir = todas las skills, vacío = ninguna).
  - `channels.telegram.groups.<id>.allowFrom`: anulación por grupo de lista de permitidos de remitentes.
  - `channels.telegram.groups.<id>.systemPrompt`: prompt adicional del sistema para el grupo.
  - `channels.telegram.groups.<id>.enabled`: deshabilitar el grupo cuando `false`.
  - .topics.<threadId>`channels.telegram.groups.<id>.*`: anulaciones por tema (mismos campos que el grupo).
  - `channels.telegram.groups.<id>.topics.<threadId>.groupPolicy`: anulación por tema de groupPolicy (`open | allowlist | disabled`).
  - .topics.<threadId>`channels.telegram.groups.<id>.requireMention`: anulación por tema del bloqueo por mención.

- `channels.telegram.capabilities.inlineButtons`: `off | dm | group | all | allowlist` (predeterminado: lista de permitidos).

- `channels.telegram.accounts.<account>.capabilities.inlineButtons`: anulación por cuenta.

- `channels.telegram.replyToMode`: `off | first | all` (predeterminado: `first`).

- `channels.telegram.textChunkLimit`: tamaño de fragmento saliente (caracteres).

- `channels.telegram.chunkMode`: `length` (predeterminado) o `newline` para dividir en líneas en blanco (límites de párrafo) antes de fragmentar por longitud.

- `channels.telegram.linkPreview`: alternar previsualizaciones de enlaces para mensajes salientes (predeterminado: true).

- `channels.telegram.streamMode`: `off | partial | block` (streaming de borradores).

- `channels.telegram.mediaMaxMb`: límite de medios entrantes/salientes (MB).

- `channels.telegram.retry`: política de reintentos para llamadas salientes a la API de Telegram (intentos, minDelayMs, maxDelayMs, jitter).

- `channels.telegram.network.autoSelectFamily`: anular autoSelectFamily de Node (true=habilitar, false=deshabilitar). Predeterminado deshabilitado en Node 22 para evitar tiempos de espera de Happy Eyeballs.

- `channels.telegram.proxy`: URL de proxy para llamadas a la Bot API (SOCKS/HTTP).

- `channels.telegram.webhookUrl`: habilitar modo webhook (requiere `channels.telegram.webhookSecret`).

- `channels.telegram.webhookSecret`: secreto del webhook (requerido cuando se establece webhookUrl).

- `channels.telegram.webhookPath`: ruta local del webhook (predeterminado `/telegram-webhook`).

- El listener local se vincula a `0.0.0.0:8787` y sirve `POST /telegram-webhook` por defecto.

- `channels.telegram.actions.reactions`: controlar reacciones de herramientas de Telegram.

- `channels.telegram.actions.sendMessage`: controlar envíos de mensajes de herramientas de Telegram.

- `channels.telegram.actions.deleteMessage`: controlar eliminaciones de mensajes de herramientas de Telegram.

- `channels.telegram.actions.sticker`: controlar acciones de stickers de Telegram — enviar y buscar (predeterminado: false).

- `channels.telegram.reactionNotifications`: `off | own | all` — controlar qué reacciones disparan eventos del sistema (predeterminado: `own` cuando no se configura).

- `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` — controlar la capacidad de reacción del agente (predeterminado: `minimal` cuando no se configura).

- Configuración completa: [Configuración](/gateway/configuration)

Campos específicos de Telegram de alta relevancia:

- inicio/autenticación: `enabled`, `botToken`, `tokenFile`, `accounts.*`
- control de acceso: `dmPolicy`, `allowFrom`, `groupPolicy`, `groupAllowFrom`, `groups`, `groups.*.topics.*`
- comandos/menú: `commands.native`, `customCommands`
- hilos/respuestas: `replyToMode`
- Opcional (solo para `streamMode: "block"`):
- formato/entrega: `textChunkLimit`, `chunkMode`, `linkPreview`, `responsePrefix`
- multimedia/red: `mediaMaxMb`, `timeoutSeconds`, `retry`, `network.autoSelectFamily`, `proxy`
- Modo webhook: configure `channels.telegram.webhookUrl` y `channels.telegram.webhookSecret` (opcionalmente `channels.telegram.webhookPath`).
- acciones/capacidades: `capabilities.inlineButtons`, `actions.sendMessage|editMessage|deleteMessage|reactions|sticker`
- Notificaciones de reacciones
- escrituras/historial: `configWrites`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`

## Relacionado

- Detalles: [Emparejamiento](/channels/pairing)
- [Enrutamiento de canales](/channels/channel-routing)
- Solución de problemas de configuración (comandos)

