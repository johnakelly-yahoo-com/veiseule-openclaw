---
title: "Telegram"
---

# Telegram (Bot API)

Estado: listo para producción para mensajes directos del bot + grupos mediante grammY. Long-polling por defecto; webhook opcional.

## Configuración rápida (principiante)

1. Cree un bot con **@BotFather** ([enlace directo](https://t.me/BotFather)). Confirme que el identificador sea exactamente `@BotFather`, luego copie el token.
2. Configure el token:
   - Env: `TELEGRAM_BOT_TOKEN=...`
   - O config: `channels.telegram.botToken: "..."`.
   - Si ambos están configurados, la config tiene prioridad (el fallback por env es solo para la cuenta predeterminada).
3. Inicie el Gateway.
4. El acceso a mensajes directos es por emparejamiento de forma predeterminada; apruebe el código de emparejamiento en el primer contacto.

Configuración mínima:

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123:abc",
      dmPolicy: "pairing",
    },
  },
}
```

## Qué es

- Un canal de la Bot API de Telegram propiedad del Gateway.
- Enrutamiento determinista: las respuestas regresan a Telegram; el modelo nunca elige canales.
- Los mensajes directos comparten la sesión principal del agente; los grupos permanecen aislados (`agent:<agentId>:telegram:group:<chatId>`).

## Configuración (ruta rápida)

### 1. Crear un token de bot (BotFather)

1. Abra Telegram y chatee con **@BotFather** ([enlace directo](https://t.me/BotFather)). Confirme que el identificador sea exactamente `@BotFather`.
2. Ejecute `/newbot`, luego siga las indicaciones (nombre + nombre de usuario que termine en `bot`).
3. Copie el token y guárdelo de forma segura.

Configuraciones opcionales en BotFather:

- `/setjoingroups` — permitir/denegar agregar el bot a grupos.
- `/setprivacy` — controlar si el bot ve todos los mensajes del grupo.

### 2. Configurar el token (env o config)

Ejemplo:

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

Opción por env: `TELEGRAM_BOT_TOKEN=...` (funciona para la cuenta predeterminada).
Si se configuran env y config, la config tiene prioridad.

Soporte multi‑cuenta: use `channels.telegram.accounts` con tokens por cuenta y `name` opcional. Consulte [`gateway/configuration`](/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) para el patrón compartido.

3. Inicie el Gateway. Telegram se inicia cuando se resuelve un token (primero config, luego fallback por env).
4. El acceso a mensajes directos es por emparejamiento de forma predeterminada. Apruebe el código cuando el bot sea contactado por primera vez.
5. Para grupos: agregue el bot, decida el comportamiento de privacidad/admin (abajo), luego configure `channels.telegram.groups` para controlar el bloqueo por menciones + listas de permitidos.

## Token + privacidad + permisos (lado de Telegram)

### Creación del token (BotFather)

- `/newbot` crea el bot y devuelve el token (manténgalo en secreto).
- Si un token se filtra, revóquelo/regénere vía @BotFather y actualice su configuración.

### Visibilidad de mensajes en grupos (Modo de privacidad)

Los bots de Telegram usan **Modo de privacidad** por defecto, lo que limita qué mensajes de grupo reciben.
Si su bot debe ver _todos_ los mensajes del grupo, tiene dos opciones:

- Desactivar el modo de privacidad con `/setprivacy` **o**
- Agregar el bot como **admin** del grupo (los bots admin reciben todos los mensajes).

**Nota:** Al alternar el modo de privacidad, Telegram requiere eliminar y volver a agregar el bot
a cada grupo para que el cambio tenga efecto.

### Permisos de grupo (derechos de admin)

El estado de admin se configura dentro del grupo (UI de Telegram). Los bots admin siempre reciben todos
los mensajes del grupo, así que use admin si necesita visibilidad completa.

## Cómo funciona (comportamiento)

- Los mensajes entrantes se normalizan en el sobre de canal compartido con contexto de respuesta y marcadores de medios.
- Las respuestas en grupos requieren una mención por defecto (mención nativa @ o `agents.list[].groupChat.mentionPatterns` / `messages.groupChat.mentionPatterns`).
- Anulación multi‑agente: configure patrones por agente en `agents.list[].groupChat.mentionPatterns`.
- Las respuestas siempre regresan al mismo chat de Telegram.
- El long‑polling usa el runner de grammY con secuenciación por chat; la concurrencia total está limitada por `agents.defaults.maxConcurrent`.
- La Bot API de Telegram no admite confirmaciones de lectura; no existe la opción `sendReadReceipts`.

## Streaming de borradores

OpenClaw puede transmitir respuestas parciales en mensajes directos de Telegram usando `sendMessageDraft`.

Requisitos:

- Modo de hilos habilitado para el bot en @BotFather (modo de temas del foro).
- Solo hilos de chats privados (Telegram incluye `message_thread_id` en mensajes entrantes).
- `channels.telegram.streamMode` no configurado como `"off"` (predeterminado: `"partial"`, `"block"` habilita actualizaciones de borradores por bloques).

El streaming de borradores es solo para mensajes directos; Telegram no lo admite en grupos o canales.

## Formato (HTML de Telegram)

- El texto saliente de Telegram usa `parse_mode: "HTML"` (el subconjunto de etiquetas compatibles de Telegram).
- La entrada tipo Markdown se renderiza en **HTML seguro para Telegram** (negrita/cursiva/tachado/código/enlaces); los elementos de bloque se aplanan a texto con saltos de línea/viñetas.
- El HTML sin procesar de los modelos se escapa para evitar errores de análisis de Telegram.
- Si Telegram rechaza la carga HTML, OpenClaw reintenta el mismo mensaje como texto plano.

## Comandos (nativos + personalizados)

OpenClaw registra comandos nativos (como `/status`, `/reset`, `/model`) en el menú del bot de Telegram al iniciar.
Puede agregar comandos personalizados al menú mediante config:

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

## Solución de problemas de configuración (comandos)

- `setMyCommands failed` en los registros normalmente significa que HTTPS/DNS saliente está bloqueado hacia `api.telegram.org`.
- Si ve fallas `sendMessage` o `sendChatAction`, revise el enrutamiento IPv6 y DNS.

Más ayuda: [Solución de problemas del canal](/channels/troubleshooting).

Notas:

- Los comandos personalizados son **solo entradas de menú**; OpenClaw no los implementa a menos que usted los maneje en otro lugar.
- Some commands can be handled by plugins/skills without being registered in Telegram’s command menu. These still work when typed (they just won't show up in `/commands` / the menu).
- Los nombres de comandos se normalizan (se elimina el prefijo `/`, se convierten a minúsculas) y deben coincidir con `a-z`, `0-9`, `_` (1–32 caracteres).
- Los comandos personalizados **no pueden sobrescribir comandos nativos**. Los conflictos se ignoran y se registran.
- Si `commands.native` está deshabilitado, solo se registran comandos personalizados (o se limpian si no hay ninguno).

### Device pairing commands (`device-pair` plugin)

If the `device-pair` plugin is installed, it adds a Telegram-first flow for pairing a new phone:

1. `/pair` generates a setup code (sent as a separate message for easy copy/paste).
2. Paste the setup code in the iOS app to connect.
3. `/pair approve` approves the latest pending device request.

More details: [Pairing](/channels/pairing#pair-via-telegram-recommended-for-ios).

## Límites

- El texto saliente se divide en `channels.telegram.textChunkLimit` (predeterminado 4000).
- División opcional por saltos de línea: configure `channels.telegram.chunkMode="newline"` para dividir en líneas en blanco (límites de párrafo) antes de dividir por longitud.
- Las descargas/cargas de medios están limitadas por `channels.telegram.mediaMaxMb` (predeterminado 5).
- Las solicitudes a la Bot API de Telegram expiran tras `channels.telegram.timeoutSeconds` (predeterminado 500 vía grammY). Configure un valor menor para evitar bloqueos largos.
- El contexto de historial de grupos usa `channels.telegram.historyLimit` (o `channels.telegram.accounts.*.historyLimit`), con fallback a `messages.groupChat.historyLimit`. Configure `0` para deshabilitarlo (predeterminado 50).
- El historial de mensajes directos puede limitarse con `channels.telegram.dmHistoryLimit` (turnos del usuario). Anulaciones por usuario: `channels.telegram.dms["<user_id>"].historyLimit`.

## Modos de activación de grupos

Por defecto, el bot solo responde a menciones en grupos (`@botname` o patrones en `agents.list[].groupChat.mentionPatterns`). Para cambiar este comportamiento:

### Vía config (recomendado)

```json5
{
  channels: {
    telegram: {
      groups: {
        "-1001234567890": { requireMention: false }, // always respond in this group
      },
    },
  },
}
```

**Importante:** Configurar `channels.telegram.groups` crea una **lista de permitidos**: solo se aceptarán los grupos listados (o `"*"`).
Los temas del foro heredan la configuración del grupo padre (allowFrom, requireMention, skills, prompts) a menos que agregue anulaciones por tema en `channels.telegram.groups.<groupId>.topics.<topicId>`.

Para permitir todos los grupos con respuesta siempre activa:

```json5
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

Para mantener solo menciones en todos los grupos (comportamiento predeterminado):

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

### Vía comando (nivel de sesión)

Envíe en el grupo:

- `/activation always` - responder a todos los mensajes
- `/activation mention` - requerir menciones (predeterminado)

**Nota:** Los comandos actualizan solo el estado de la sesión. Para comportamiento persistente tras reinicios, use config.

### Obtener el ID del chat del grupo

Reenvíe cualquier mensaje del grupo a `@userinfobot` o `@getidsbot` en Telegram para ver el ID del chat (número negativo como `-1001234567890`).

**Consejo:** Para su propio ID de usuario, envíe un mensaje directo al bot y este responderá con su ID de usuario (mensaje de emparejamiento), o use `/whoami` una vez que los comandos estén habilitados.

**Nota de privacidad:** `@userinfobot` es un bot de terceros. Si lo prefiere, agregue el bot al grupo, envíe un mensaje y use `openclaw logs --follow` para leer `chat.id`, o use la Bot API `getUpdates`.

## Escrituras de configuración

Por defecto, Telegram puede escribir actualizaciones de configuración activadas por eventos del canal o `/config set|unset`.

Esto ocurre cuando:

- Un grupo se actualiza a supergrupo y Telegram emite `migrate_to_chat_id` (cambia el ID del chat). OpenClaw puede migrar `channels.telegram.groups` automáticamente.
- Ejecuta `/config set` o `/config unset` en un chat de Telegram (requiere `commands.config: true`).

Deshabilitar con:

```json5
{
  channels: { telegram: { configWrites: false } },
}
```

## Temas (supergrupos tipo foro)

Los temas de foro de Telegram incluyen un `message_thread_id` por mensaje. OpenClaw:

- Agrega `:topic:<threadId>` a la clave de sesión del grupo de Telegram para que cada tema quede aislado.
- Envía indicadores de escritura y respuestas con `message_thread_id` para que las respuestas permanezcan en el tema.
- El tema general (id de hilo `1`) es especial: los envíos de mensajes omiten `message_thread_id` (Telegram lo rechaza), pero los indicadores de escritura aún lo incluyen.
- Expone `MessageThreadId` + `IsForum` en el contexto de plantillas para enrutamiento/plantillas.
- La configuración específica por tema está disponible en `channels.telegram.groups.<chatId>.topics.<threadId>` (skills, listas de permitidos, auto‑respuesta, prompts del sistema, deshabilitar).
- Las configuraciones de tema heredan las del grupo (requireMention, listas de permitidos, skills, prompts, habilitado) a menos que se anulen por tema.

Los chats privados pueden incluir `message_thread_id` en algunos casos límite. OpenClaw mantiene la clave de sesión de mensajes directos sin cambios, pero aun así usa el id de hilo para respuestas/streaming de borradores cuando está presente.

## Botones en línea

Telegram admite teclados en línea con botones de callback.

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

Para configuración por cuenta:

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

Alcancías:

- `off` — botones en línea deshabilitados
- `dm` — solo mensajes directos (destinos de grupo bloqueados)
- `group` — solo grupos (destinos de mensajes directos bloqueados)
- `all` — mensajes directos + grupos
- `allowlist` — mensajes directos + grupos, pero solo remitentes permitidos por `allowFrom`/`groupAllowFrom` (mismas reglas que los comandos de control)

Predeterminado: `allowlist`.
Legado: `capabilities: ["inlineButtons"]` = `inlineButtons: "all"`.

### Envío de botones

Use la herramienta de mensajes con el parámetro `buttons`:

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

Cuando un usuario hace clic en un botón, los datos de callback se envían de vuelta al agente como un mensaje con el formato:
`callback_data: value`

### Opciones de configuración

Las capacidades de Telegram pueden configurarse en dos niveles (se muestra el formato de objeto arriba; los arreglos de cadenas heredados aún son compatibles):

- `channels.telegram.capabilities`: configuración global predeterminada de capacidades aplicada a todas las cuentas de Telegram salvo anulación.
- `channels.telegram.accounts.<account>.capabilities`: capacidades por cuenta que anulan los valores globales para esa cuenta específica.

Use la configuración global cuando todos los bots/cuentas de Telegram deban comportarse igual. Use la configuración por cuenta cuando distintos bots necesiten comportamientos diferentes (por ejemplo, una cuenta solo maneja mensajes directos mientras otra está permitida en grupos).

## Control de acceso (mensajes directos + grupos)

### Acceso DM

- Predeterminado: `channels.telegram.dmPolicy = "pairing"`. Los remitentes desconocidos reciben un código de emparejamiento; los mensajes se ignoran hasta aprobarse (los códigos expiran tras 1 hora).
- Aprobar vía:
  - `openclaw pairing list telegram`
  - `openclaw pairing approve telegram <CODE>`
- El emparejamiento es el intercambio de tokens predeterminado usado para mensajes directos de Telegram. Detalles: [Emparejamiento](/channels/pairing)
- `channels.telegram.allowFrom` acepta IDs numéricos de usuario (recomendado) o entradas `@username`. **No** es el nombre de usuario del bot; use el ID del remitente humano. El asistente acepta `@username` y lo resuelve al ID numérico cuando es posible.

#### Encontrar su ID de usuario de Telegram

Más seguro (sin bot de terceros):

1. Inicie el Gateway y envíe un mensaje directo a su bot.
2. Ejecute `openclaw logs --follow` y busque `from.id`.

Alternativa (Bot API oficial):

1. Envíe un mensaje directo a su bot.
2. Obtenga actualizaciones con el token del bot y lea `message.from.id`:

   ```bash
   curl "https://api.telegram.org/bot<bot_token>/getUpdates"
   ```

Terceros (menos privado):

- Envíe un mensaje directo a `@userinfobot` o `@getidsbot` y use el ID de usuario devuelto.

### Acceso a grupos

Dos controles independientes:

**1. Qué grupos están permitidos** (lista de permitidos de grupos vía `channels.telegram.groups`):

- Sin configuración `groups` = todos los grupos permitidos
- Con configuración `groups` = solo se permiten los grupos listados o `"*"`
- Ejemplo: `"groups": { "-1001234567890": {}, "*": {} }` permite todos los grupos

**2. Qué remitentes están permitidos** (filtrado de remitentes vía `channels.telegram.groupPolicy`):

- `"open"` = todos los remitentes en grupos permitidos pueden enviar mensajes
- `"allowlist"` = solo remitentes en `channels.telegram.groupAllowFrom` pueden enviar mensajes
- `"disabled"` = no se aceptan mensajes de grupo en absoluto
  El valor predeterminado es `groupPolicy: "allowlist"` (bloqueado a menos que agregue `groupAllowFrom`).

La mayoría de los usuarios quieren: `groupPolicy: "allowlist"` + `groupAllowFrom` + grupos específicos listados en `channels.telegram.groups`

Para permitir que **cualquier miembro del grupo** hable en un grupo específico (manteniendo los comandos de control restringidos a remitentes autorizados), configure una anulación por grupo:

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

## Long‑polling vs webhook

- Predeterminado: long‑polling (no se requiere URL pública).
- Modo webhook: configure `channels.telegram.webhookUrl` y `channels.telegram.webhookSecret` (opcionalmente `channels.telegram.webhookPath`).
  - El listener local se vincula a `0.0.0.0:8787` y sirve `POST /telegram-webhook` por defecto.
  - Si su URL pública es diferente, use un proxy inverso y apunte `channels.telegram.webhookUrl` al endpoint público.

## Encadenamiento de respuestas

Telegram admite respuestas encadenadas opcionales mediante etiquetas:

- `[[reply_to_current]]` -- responder al mensaje que activó.
- `[[reply_to:<id>]]` -- responder a un ID de mensaje específico.

Controlado por `channels.telegram.replyToMode`:

- `first` (predeterminado), `all`, `off`.

## Mensajes de audio (voz vs archivo)

Telegram distingue **notas de voz** (burbuja redonda) de **archivos de audio** (tarjeta con metadatos).
OpenClaw usa archivos de audio por defecto para compatibilidad hacia atrás.

Para forzar una burbuja de nota de voz en las respuestas del agente, incluya esta etiqueta en cualquier parte de la respuesta:

- `[[audio_as_voice]]` — enviar audio como nota de voz en lugar de archivo.

La etiqueta se elimina del texto entregado. Otros canales ignoran esta etiqueta.

Para envíos con la herramienta de mensajes, configure `asVoice: true` con una URL de `media` compatible con voz
(`message` es opcional cuando hay medios):

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/voice.ogg",
  asVoice: true,
}
```

## Video messages (video vs video note)

Telegram distinguishes **video notes** (round bubble) from **video files** (rectangular).
OpenClaw defaults to video files.

For message tool sends, set `asVideoNote: true` with a video `media` URL:

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/video.mp4",
  asVideoNote: true,
}
```

(Note: Video notes do not support captions. If you provide a message text, it will be sent as a separate message.)

## Stickers

OpenClaw admite recibir y enviar stickers de Telegram con caché inteligente.

### Recepción de stickers

Cuando un usuario envía un sticker, OpenClaw lo maneja según el tipo de sticker:

- **Stickers estáticos (WEBP):** Se descargan y procesan mediante visión. El sticker aparece como un marcador `<media:sticker>` en el contenido del mensaje.
- **Stickers animados (TGS):** Se omiten (formato Lottie no compatible para procesamiento).
- **Stickers de video (WEBM):** Se omiten (formato de video no compatible para procesamiento).

Campo de contexto de plantilla disponible al recibir stickers:

- `Sticker` — objeto con:
  - `emoji` — emoji asociado al sticker
  - `setName` — nombre del conjunto de stickers
  - `fileId` — ID de archivo de Telegram (para enviar el mismo sticker)
  - `fileUniqueId` — ID estable para búsqueda en caché
  - `cachedDescription` — descripción de visión en caché cuando está disponible

### Caché de stickers

Los stickers se procesan mediante las capacidades de visión de la IA para generar descripciones. Dado que los mismos stickers se envían repetidamente, OpenClaw almacena estas descripciones en caché para evitar llamadas redundantes a la API.

**Cómo funciona:**

1. **Primer encuentro:** La imagen del sticker se envía a la IA para análisis de visión. La IA genera una descripción (por ejemplo, "Un gato caricaturesco saludando con entusiasmo").
2. **Almacenamiento en caché:** La descripción se guarda junto con el ID de archivo del sticker, el emoji y el nombre del conjunto.
3. **Encuentros posteriores:** Cuando se vuelve a ver el mismo sticker, se usa directamente la descripción en caché. La imagen no se envía a la IA.

**Ubicación de la caché:** `~/.openclaw/telegram/sticker-cache.json`

**Formato de entrada de caché:**

```json
{
  "fileId": "CAACAgIAAxkBAAI...",
  "fileUniqueId": "AgADBAADb6cxG2Y",
  "emoji": "👋",
  "setName": "CoolCats",
  "description": "A cartoon cat waving enthusiastically",
  "cachedAt": "2026-01-15T10:30:00.000Z"
}
```

**Beneficios:**

- Reduce los costos de API al evitar llamadas de visión repetidas para el mismo sticker
- Respuestas más rápidas para stickers en caché (sin retraso de procesamiento de visión)
- Habilita la funcionalidad de búsqueda de stickers basada en descripciones en caché

La caché se completa automáticamente a medida que se reciben stickers. No se requiere gestión manual de la caché.

### Envío de stickers

El agente puede enviar y buscar stickers usando las acciones `sticker` y `sticker-search`. Estas están deshabilitadas por defecto y deben habilitarse en config:

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

**Enviar un sticker:**

```json5
{
  action: "sticker",
  channel: "telegram",
  to: "123456789",
  fileId: "CAACAgIAAxkBAAI...",
}
```

Parámetros:

- `fileId` (obligatorio) — el ID de archivo de Telegram del sticker. Obténgalo de `Sticker.fileId` al recibir un sticker, o de un resultado `sticker-search`.
- `replyTo` (opcional) — ID del mensaje al que responder.
- `threadId` (opcional) — ID del hilo del mensaje para temas de foro.

**Buscar stickers:**

El agente puede buscar stickers en caché por descripción, emoji o nombre del conjunto:

```json5
{
  action: "sticker-search",
  channel: "telegram",
  query: "cat waving",
  limit: 5,
}
```

Devuelve stickers coincidentes desde la caché:

```json5
{
  ok: true,
  count: 2,
  stickers: [
    {
      fileId: "CAACAgIAAxkBAAI...",
      emoji: "👋",
      description: "A cartoon cat waving enthusiastically",
      setName: "CoolCats",
    },
  ],
}
```

La búsqueda usa coincidencia difusa en el texto de la descripción, caracteres emoji y nombres de conjuntos.

**Ejemplo con encadenamiento:**

```json5
{
  action: "sticker",
  channel: "telegram",
  to: "-1001234567890",
  fileId: "CAACAgIAAxkBAAI...",
  replyTo: 42,
  threadId: 123,
}
```

## Streaming (borradores)

Telegram puede transmitir **burbujas de borrador** mientras el agente genera una respuesta.
OpenClaw usa la Bot API `sendMessageDraft` (no son mensajes reales) y luego envía la
respuesta final como un mensaje normal.

Requisitos (Bot API de Telegram 9.3+):

- **Chats privados con temas habilitados** (modo de temas del foro para el bot).
- Los mensajes entrantes deben incluir `message_thread_id` (hilo de tema privado).
- El streaming se ignora para grupos/supergrupos/canales.

Config:

- `channels.telegram.streamMode: "off" | "partial" | "block"` (predeterminado: `partial`)
  - `partial`: actualizar la burbuja de borrador con el texto de streaming más reciente.
  - `block`: actualizar la burbuja de borrador en bloques más grandes (por fragmentos).
  - `off`: deshabilitar el streaming de borradores.
- Opcional (solo para `streamMode: "block"`):
  - `channels.telegram.draftChunk: { minChars?, maxChars?, breakPreference? }`
    - valores predeterminados: `minChars: 200`, `maxChars: 800`, `breakPreference: "paragraph"` (limitado a `channels.telegram.textChunkLimit`).

Nota: el streaming de borradores es independiente del **streaming por bloques** (mensajes del canal).
El streaming por bloques está desactivado por defecto y requiere `channels.telegram.blockStreaming: true`
si desea mensajes tempranos de Telegram en lugar de actualizaciones de borrador.

Stream de razonamiento (solo Telegram):

- `/reasoning stream` transmite el razonamiento en la burbuja de borrador mientras se
  genera la respuesta, y luego envía la respuesta final sin razonamiento.
- Si `channels.telegram.streamMode` es `off`, el stream de razonamiento está deshabilitado.
  Más contexto: [Streaming + fragmentación](/concepts/streaming).

## Política de reintentos

Las llamadas salientes a la API de Telegram reintentan ante errores transitorios de red/429 con backoff exponencial y jitter. Configure mediante `channels.telegram.retry`. Consulte [Política de reintentos](/concepts/retry).

## Herramienta del agente (mensajes + reacciones)

- Herramienta: `telegram` con la acción `sendMessage` (`to`, `content`, opcional `mediaUrl`, `replyToMessageId`, `messageThreadId`).
- Herramienta: `telegram` con la acción `react` (`chatId`, `messageId`, `emoji`).
- Herramienta: `telegram` con la acción `deleteMessage` (`chatId`, `messageId`).
- Semántica de eliminación de reacciones: consulte [/tools/reactions](/tools/reactions).
- Control de herramientas: `channels.telegram.actions.reactions`, `channels.telegram.actions.sendMessage`, `channels.telegram.actions.deleteMessage` (predeterminado: habilitado) y `channels.telegram.actions.sticker` (predeterminado: deshabilitado).

## Notificaciones de reacciones

**Cómo funcionan las reacciones:**
Las reacciones de Telegram llegan como **eventos `message_reaction` separados**, no como propiedades en las cargas de mensajes. Cuando un usuario agrega una reacción, OpenClaw:

1. Recibe la actualización `message_reaction` de la API de Telegram
2. La convierte en un **evento del sistema** con el formato: `"Telegram reaction added: {emoji} by {user} on msg {id}"`
3. Encola el evento del sistema usando la **misma clave de sesión** que los mensajes normales
4. Cuando llega el siguiente mensaje en esa conversación, los eventos del sistema se drenan y se anteponen al contexto del agente

El agente ve las reacciones como **notificaciones del sistema** en el historial de la conversación, no como metadatos del mensaje.

**Configuración:**

- `channels.telegram.reactionNotifications`: controla qué reacciones disparan notificaciones
  - `"off"` — ignorar todas las reacciones
  - `"own"` — notificar cuando los usuarios reaccionan a mensajes del bot (mejor esfuerzo; en memoria) (predeterminado)
  - `"all"` — notificar todas las reacciones

- `channels.telegram.reactionLevel`: controla la capacidad de reacción del agente
  - `"off"` — el agente no puede reaccionar a mensajes
  - `"ack"` — el bot envía reacciones de acuse (👀 mientras procesa) (predeterminado)
  - `"minimal"` — el agente puede reaccionar con moderación (guía: 1 por cada 5–10 intercambios)
  - `"extensive"` — el agente puede reaccionar libremente cuando sea apropiado

**Grupos de foro:** Las reacciones en grupos de foro incluyen `message_thread_id` y usan claves de sesión como `agent:main:telegram:group:{chatId}:topic:{threadId}`. Esto asegura que reacciones y mensajes en el mismo tema permanezcan juntos.

**Ejemplo de configuración:**

```json5
{
  channels: {
    telegram: {
      reactionNotifications: "all", // See all reactions
      reactionLevel: "minimal", // Agent can react sparingly
    },
  },
}
```

**Requisitos:**

- Los bots de Telegram deben solicitar explícitamente `message_reaction` en `allowed_updates` (configurado automáticamente por OpenClaw)
- En modo webhook, las reacciones se incluyen en el webhook `allowed_updates`
- En modo polling, las reacciones se incluyen en el `getUpdates` `allowed_updates`

## Destinos de entrega (CLI/cron)

- Use un ID de chat (`123456789`) o un nombre de usuario (`@name`) como destino.
- Ejemplo: `openclaw message send --channel telegram --target 123456789 --message "hi"`.

## Solución de problemas

**El bot no responde a mensajes sin mención en un grupo:**

- Si configuró `channels.telegram.groups.*.requireMention=false`, el **modo de privacidad** de la Bot API de Telegram debe estar desactivado.
  - BotFather: `/setprivacy` → **Disable** (luego elimine y vuelva a agregar el bot al grupo)
- `openclaw channels status` muestra una advertencia cuando la config espera mensajes de grupo sin mención.
- `openclaw channels status --probe` puede verificar adicionalmente la membresía para IDs numéricos explícitos de grupos (no puede auditar reglas comodín `"*"`).
- Prueba rápida: `/activation always` (solo sesión; use config para persistencia)

**El bot no ve mensajes del grupo en absoluto:**

- Si `channels.telegram.groups` está configurado, el grupo debe estar listado o usar `"*"`
- Revise Privacidad en @BotFather → "Group Privacy" debe estar **OFF**
- Verifique que el bot sea realmente miembro (no solo admin sin acceso de lectura)
- Revise los registros del Gateway: `openclaw logs --follow` (busque "skipping group message")

**El bot responde a menciones pero no a `/activation always`:**

- El comando `/activation` actualiza el estado de la sesión pero no persiste en la config
- Para comportamiento persistente, agregue el grupo a `channels.telegram.groups` con `requireMention: false`

**Comandos como `/status` no funcionan:**

- Asegúrese de que su ID de usuario de Telegram esté autorizado (vía emparejamiento o `channels.telegram.allowFrom`)
- Los comandos requieren autorización incluso en grupos con `groupPolicy: "open"`

**El long‑polling se aborta inmediatamente en Node 22+ (a menudo con proxies/fetch personalizado):**

- Node 22+ es más estricto con instancias `AbortSignal`; señales externas pueden abortar llamadas `fetch` de inmediato.
- Actualice a una compilación de OpenClaw que normalice señales de aborto, o ejecute el Gateway en Node 20 hasta que pueda actualizar.

**El bot inicia y luego deja de responder silenciosamente (o registra `HttpError: Network request ... failed`):**

- Algunos hosts resuelven `api.telegram.org` a IPv6 primero. Si su servidor no tiene salida IPv6 funcional, grammY puede quedar bloqueado en solicitudes solo IPv6.
- Solución: habilite salida IPv6 **o** fuerce resolución IPv4 para `api.telegram.org` (por ejemplo, agregue una entrada `/etc/hosts` usando el registro A IPv4, o prefiera IPv4 en la pila DNS del SO), luego reinicie el Gateway.
- Verificación rápida: `dig +short api.telegram.org A` y `dig +short api.telegram.org AAAA` para confirmar qué devuelve DNS.

## Referencia de configuración (Telegram)

Configuración completa: [Configuración](/gateway/configuration)

Opciones del proveedor:

- `channels.telegram.enabled`: habilitar/deshabilitar el inicio del canal.
- `channels.telegram.botToken`: token del bot (BotFather).
- `channels.telegram.tokenFile`: leer el token desde una ruta de archivo.
- `channels.telegram.dmPolicy`: `pairing | allowlist | open | disabled` (predeterminado: emparejamiento).
- `channels.telegram.allowFrom`: lista de permitidos de mensajes directos (ids/nombres de usuario). `open` requiere `"*"`.
- `channels.telegram.groupPolicy`: `open | allowlist | disabled` (predeterminado: lista de permitidos).
- `channels.telegram.groupAllowFrom`: lista de permitidos de remitentes de grupo (ids/nombres de usuario).
- `channels.telegram.groups`: valores predeterminados por grupo + lista de permitidos (use `"*"` para valores globales).
  - `channels.telegram.groups.<id>.groupPolicy`: anulación por grupo de groupPolicy (`open | allowlist | disabled`).
  - `channels.telegram.groups.<id>.requireMention`: bloqueo por mención predeterminado.
  - `channels.telegram.groups.<id>.skills`: filtro de skills (omitir = todas las skills, vacío = ninguna).
  - `channels.telegram.groups.<id>.allowFrom`: anulación por grupo de lista de permitidos de remitentes.
  - `channels.telegram.groups.<id>.systemPrompt`: prompt adicional del sistema para el grupo.
  - `channels.telegram.groups.<id>.enabled`: deshabilitar el grupo cuando `false`.
  - `channels.telegram.groups.<id>.topics.<threadId>.*`: anulaciones por tema (mismos campos que el grupo).
  - `channels.telegram.groups.<id>.topics.<threadId>.groupPolicy`: anulación por tema de groupPolicy (`open | allowlist | disabled`).
  - `channels.telegram.groups.<id>.topics.<threadId>.requireMention`: anulación por tema del bloqueo por mención.
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
- `channels.telegram.actions.reactions`: controlar reacciones de herramientas de Telegram.
- `channels.telegram.actions.sendMessage`: controlar envíos de mensajes de herramientas de Telegram.
- `channels.telegram.actions.deleteMessage`: controlar eliminaciones de mensajes de herramientas de Telegram.
- `channels.telegram.actions.sticker`: controlar acciones de stickers de Telegram — enviar y buscar (predeterminado: false).
- `channels.telegram.reactionNotifications`: `off | own | all` — controlar qué reacciones disparan eventos del sistema (predeterminado: `own` cuando no se configura).
- `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` — controlar la capacidad de reacción del agente (predeterminado: `minimal` cuando no se configura).

Opciones globales relacionadas:

- `agents.list[].groupChat.mentionPatterns` (patrones de bloqueo por mención).
- `messages.groupChat.mentionPatterns` (fallback global).
- `commands.native` (predeterminado `"auto"` → activado para Telegram/Discord, desactivado para Slack), `commands.text`, `commands.useAccessGroups` (comportamiento de comandos). Anule con `channels.telegram.commands.native`.
- `messages.responsePrefix`, `messages.ackReaction`, `messages.ackReactionScope`, `messages.removeAckAfterReply`.


