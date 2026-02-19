---
summary: "Estado de compatibilidad del bot de Discord, capacidades y configuración"
read_when:
  - Al trabajar en funciones del canal de Discord
title: "Discord"
---

# Discord (API de Bot)

Estado: listo para mensajes directos y canales de texto de guild mediante el gateway oficial de bots de Discord.

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Los DM de Discord usan el modo de emparejamiento de forma predeterminada.
  
</Card>
  <Card title="Slash commands" icon="terminal" href="/tools/slash-commands">
    Comportamiento nativo de comandos y catálogo de comandos.
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    Diagnósticos entre canales y flujo de reparación.
  
</Card>
</CardGroup>

## Configuración rápida (principiante)

<Steps>
  <Step title="Create a Discord bot and enable intents">Crear la app de Discord + usuario bot

    ```
    En la configuración de la app de Discord, habilite **Message Content Intent** (y **Server Members Intent** si planea usar listas de permitidos o búsquedas por nombre).
    ```

  
</Step>

  <Step title="Configure token">

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "YOUR_BOT_TOKEN",
    },
  },
}
```

    ```
    Alternativa mediante variable de entorno para la cuenta predeterminada:
    ```

```bash
`DISCORD_BOT_TOKEN=...`
```

  
</Step>

  <Step title="Invite the bot and start gateway">Invite el bot a su servidor con los permisos necesarios para leer/enviar mensajes donde quiera usarlo.

```bash
openclaw gateway
```

  
</Step>

  <Step title="Approve first DM pairing">

```bash
openclaw pairing list discord
openclaw pairing approve discord <CODE>
```

    ```
    Los códigos de emparejamiento caducan después de 1 hora.
    ```

  
</Step>
</Steps>

<Note>
La resolución de tokens tiene en cuenta la cuenta. Los valores de token en la configuración prevalecen sobre la alternativa mediante variable de entorno. `DISCORD_BOT_TOKEN` solo se utiliza para la cuenta predeterminada.
</Note>

## Modelo de ejecución

- Gateway gestiona la conexión con Discord.
- El enrutamiento de respuestas es determinista: las respuestas entrantes de Discord vuelven a Discord.
- Por defecto (`session.dmScope=main`), los chats directos comparten la sesión principal del agente (`agent:main:main`).
- Los chats directos se colapsan en la sesión principal del agente (predeterminado `agent:main:main`); los canales de guild permanecen aislados como `agent:<agentId>:discord:channel:<channelId>` (los nombres visibles usan `discord:<guildSlug>#<channelSlug>`).
- Los mensajes directos grupales se ignoran de forma predeterminada; habilítelos mediante `channels.discord.dm.groupEnabled` y, opcionalmente, restrinja con `channels.discord.dm.groupChannels`.
- Los comandos nativos usan claves de sesión aisladas (`agent:<agentId>:discord:slash:<userId>`) en lugar de la sesión compartida `main`.

## Control de acceso y enrutamiento

<Tabs>
  <Tab title="DM policy">Para mantener el comportamiento antiguo “abierto a cualquiera”: establezca `channels.discord.dm.policy="open"` y `channels.discord.dm.allowFrom=["*"]`.

    ```
    Para una lista de permitidos estricta: establezca `channels.discord.dm.policy="allowlist"` y enumere los remitentes en `channels.discord.dm.allowFrom`.
    ```

  
</Tab>

  <Tab title="Guild policy">El comportamiento se controla mediante `channels.discord.replyToMode`:

    ```
    Los comandos nativos respetan las mismas listas de permitidos que los mensajes directos/mensajes de guild (`channels.discord.dm.allowFrom`, `channels.discord.guilds`, reglas por canal).
    ```

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "abc.123",
      groupPolicy: "allowlist",
      guilds: {
        "*": {
          channels: {
            general: { allow: true },
          },
        },
      },
      mediaMaxMb: 8,
      actions: {
        reactions: true,
        stickers: true,
        emojiUploads: true,
        stickerUploads: true,
        polls: true,
        permissions: true,
        messages: true,
        threads: true,
        pins: true,
        search: true,
        memberInfo: true,
        roleInfo: true,
        roles: false,
        channelInfo: true,
        channels: true,
        voiceStatus: true,
        events: true,
        moderation: false,
        presence: false,
      },
      replyToMode: "off",
      dm: {
        enabled: true,
        policy: "pairing", // pairing | allowlist | open | disabled
        allowFrom: ["123456789012345678", "steipete"],
        groupEnabled: false,
        groupChannels: ["openclaw-dm"],
      },
      guilds: {
        "*": { requireMention: true },
        "123456789012345678": {
          slug: "friends-of-openclaw",
          requireMention: false,
          reactionNotifications: "own",
          users: ["987654321098765432", "steipete"],
          channels: {
            general: { allow: true },
            help: {
              allow: true,
              requireMention: true,
              users: ["987654321098765432"],
              skills: ["search", "docs"],
              systemPrompt: "Keep answers short.",
            },
          },
        },
      },
    },
  },
}
```

    ```
    Si solo configura `DISCORD_BOT_TOKEN` y nunca crea una sección `channels.discord`, el runtime
        establece por defecto `groupPolicy` como `open`.
    ```

  
</Tab>

  <Tab title="Mentions and group DMs">
    Los mensajes de servidores (guild) están restringidos por mención de forma predeterminada.

    ```
    Advertencia: si permite respuestas a otros bots (`channels.discord.allowBots=true`), evite bucles bot‑a‑bot con `requireMention`, listas de permitidos `channels.discord.guilds.*.channels.<id> .users` y/o barreras claras en `AGENTS.md` y `SOUL.md`.
    ```

  
</Tab>
</Tabs>

### Enrutamiento de agentes basado en roles

Usa `bindings[].match.roles` para enrutar miembros de un servidor de Discord a distintos agentes según el ID de rol. Las vinculaciones basadas en roles solo aceptan IDs de rol y se evalúan después de las vinculaciones de peer o parent-peer y antes de las vinculaciones solo de guild. Si una vinculación también establece otros campos de coincidencia (por ejemplo, `peer` + `guildId` + `roles`), todos los campos configurados deben coincidir.

```json5
{
  bindings: [
    {
      agentId: "opus",
      match: {
        channel: "discord",
        guildId: "123456789012345678",
        roles: ["111111111111111111"],
      },
    },
    {
      agentId: "sonnet",
      match: {
        channel: "discord",
        guildId: "123456789012345678",
      },
    },
  ],
}
```

## Configuración del Developer Portal

<AccordionGroup>
  <Accordion title="Create app and bot">

    ```
    Cree un bot de Discord y copie el token del bot.
    ```

  
</Accordion>

  <Accordion title="Privileged intents">En **Bot** → **Privileged Gateway Intents**, habilite:

    ```
    - Message Content Intent
    - Server Members Intent (recomendado)
    
    Presence intent es opcional y solo es necesario si deseas recibir actualizaciones de presencia. Establecer la presencia del bot (`setPresence`) no requiere habilitar actualizaciones de presencia para los miembros.
    ```

  
</Accordion>

  <Accordion title="OAuth scopes and baseline permissions">En su app: **OAuth2** → **URL Generator**

    ```
    - scopes: `bot`, `applications.commands`
    
    Permisos básicos habituales:
    
    - View Channels
    - Send Messages
    - Read Message History
    - Embed Links
    - Attach Files
    - Add Reactions (opcional)
    
    Evita `Administrator` salvo que sea estrictamente necesario.
    ```

  
</Accordion>

  <Accordion title="Copy IDs">
    Activa el Modo Desarrollador de Discord y luego copia:

    ```
    - ID del servidor
    - ID del canal
    - ID del usuario
    
    Prefiere IDs numéricos en la configuración de OpenClaw para auditorías y pruebas más fiables.
    ```

  
</Accordion>
</AccordionGroup>

## Comandos nativos y autorización de comandos

- Comandos nativos opcionales: `commands.native` tiene como valor predeterminado `"auto"` (activado para Discord/Telegram, desactivado para Slack).
- `channels.discord.execApprovals.enabled: true` en su configuración.
- Anule con `channels.discord.commands.native: true|false|"auto"`; `false` borra los comandos registrados previamente.
- La autorización de comandos nativos utiliza las mismas allowlists/policies de Discord que el manejo normal de mensajes.
- Los comandos slash pueden seguir siendo visibles en la interfaz de Discord para usuarios no permitidos; OpenClaw aplica las listas de permitidos en la ejecución y responde “no autorizado”.

Consulte [Exec approvals](/tools/exec-approvals) y [Slash commands](/tools/slash-commands) para el flujo general de aprobaciones y comandos.

## Detalles de la funcionalidad

<AccordionGroup>
  <Accordion title="Reply tags and native replies">
    Discord admite etiquetas de respuesta en la salida del agente:

    ```
    - `[[reply_to_current]]`
    - `[[reply_to:<id>]]`
    
    Controlado por `channels.discord.replyToMode`:
    
    - `off` (predeterminado)
    - `first`
    - `all`
    
    Nota: `off` desactiva el encadenamiento implícito de respuestas. Las etiquetas explícitas `[[reply_to_*]]` siguen respetándose.
    
    Los IDs de mensaje se incluyen en el contexto/historial para que los agentes puedan dirigirse a mensajes específicos.
    ```

  
</Accordion>

  <Accordion title="History, context, and thread behavior">
    Contexto del historial del servidor (guild):

    ```
    - `channels.discord.historyLimit` predeterminado `20`
    - alternativa: `messages.groupChat.historyLimit`
    - `0` lo desactiva
    
    Controles del historial en DM:
    
    - `channels.discord.dmHistoryLimit`
    - `channels.discord.dms["<user_id>"].historyLimit`
    
    Comportamiento de hilos:
    
    - Los hilos de Discord se enrutan como sesiones de canal
    - los metadatos del hilo padre pueden usarse para la vinculación con la sesión padre
    - la configuración del hilo hereda la del canal padre salvo que exista una entrada específica para el hilo
    
    Los temas del canal se inyectan como contexto **no confiable** (no como system prompt).
    ```

  
</Accordion>

  <Accordion title="Reaction notifications">.reactionNotifications`:

    ```
    `guilds.<id> .reactionNotifications`: modo de eventos del sistema de reacciones (`off`, `own`, `all`, `allowlist`).
    ```

  
</Accordion>

  <Accordion title="Ack reactions">
    `ackReaction` envía un emoji de confirmación mientras OpenClaw procesa un mensaje entrante.

    ```
    Orden de resolución:
    
    - `channels.discord.accounts.<accountId>.ackReaction`
    - `channels.discord.ackReaction`
    - `messages.ackReaction`
    - emoji de identidad del agente como alternativa (`agents.list[].identity.emoji`, o "👀" si no existe)
    
    Notas:
    
    - Discord acepta emoji unicode o nombres de emoji personalizados.
    - Usa `""` para desactivar la reacción en un canal o cuenta.
    ```

  
</Accordion>

  <Accordion title="Config writes">
    Las escrituras de configuración iniciadas desde el canal están habilitadas de forma predeterminada.

    ```
    Esto afecta a los flujos `/config set|unset` (cuando las funciones de comandos están habilitadas).
    
    Desactivar:
    ```

```json5
{
  channels: { discord: { configWrites: false } },
}
```

  
</Accordion>

  <Accordion title="Gateway proxy">
    Enruta el tráfico WebSocket del gateway de Discord a través de un proxy HTTP(S) con `channels.discord.proxy`.

```json5
O config: `channels.discord.token: "..."`.
```

    ```
    Anulación por cuenta:
    ```

```json5
{
  channels: {
    discord: {
      accounts: {
        primary: {
          proxy: "http://proxy.example:8080",
        },
      },
    },
  },
}
```

  
</Accordion>

  <Accordion title="PluralKit support">`pluralkit`: resolver mensajes proxied de PluralKit para que los miembros del sistema aparezcan como remitentes distintos.

```json5
{
  channels: {
    discord: {
      pluralkit: {
        enabled: true,
        token: "pk_live_...", // optional; required for private systems
      },
    },
  },
}
```

    ```
    Notas:
    
    - las allowlists pueden usar `pk:<memberId>`
    - los nombres visibles de los miembros se comparan por nombre/slug
    - las búsquedas usan el ID original del mensaje y están limitadas por una ventana de tiempo
    - si la búsqueda falla, los mensajes proxy se tratan como mensajes de bot y se descartan salvo que `allowBots=true`
    ```

  
</Accordion>

  <Accordion title="Presence configuration">
    Las actualizaciones de presencia solo se aplican cuando estableces un campo de estado o actividad.

    ```
    Ejemplo solo de estado:
    ```

```json5
`[[reply_to_current]]` — responder al mensaje de Discord que activó.
```

    ```
    Ejemplo de actividad (el estado personalizado es el tipo de actividad predeterminado):
    ```

```json5
{
  channels: {
    discord: {
      activity: "Focus time",
      activityType: 4,
    },
  },
}
```

    ```
    Ejemplo de streaming:
    ```

```json5
Ejecute el Gateway; inicia automáticamente el canal de Discord cuando hay un token disponible (primero config, luego env como respaldo) y `channels.discord.enabled` no es `false`.
```

    ```
    Mapa de tipos de actividad:
    
    - 0: Playing
    - 1: Streaming (requiere `activityUrl`)
    - 2: Listening
    - 3: Watching
    - 4: Custom (usa el texto de la actividad como estado; el emoji es opcional)
    - 5: Competing
    ```

  
</Accordion>

  <Accordion title="Exec approvals in Discord">
    Discord admite aprobaciones de ejecución mediante botones en DMs y puede publicar opcionalmente solicitudes de aprobación en el canal de origen.

    ```
    `execApprovals`: aprobaciones de exec solo para Discord por DM (UI de botones). Admite `enabled`, `approvers`, `agentFilter`, `sessionFilter`.
    ```

  
</Accordion>
</AccordionGroup>

## Acciones de herramientas

Las acciones de mensajes de Discord incluyen mensajería, administración de canales, moderación, presencia y acciones de metadatos.

Ejemplos principales:

- `readMessages`, `sendMessage`, `editMessage`, `deleteMessage`
- `react` / `reactions` (agregar o listar reacciones)
- `timeout`, `kick`, `ban`
- presence: `setPresence`

Las puertas de acción se encuentran en `channels.discord.actions.*`.

Comportamiento predeterminado de las puertas:

| Grupo de acciones                                                                                             | Predeterminado |
| ------------------------------------------------------------------------------------------------------------- | -------------- |
| `stickers`, `emojiUploads`, `stickerUploads`, `polls`, `permissions`, `messages`, `threads`, `pins`, `search` | enabled        |
| roles                                                                                                         | deshabilitado  |
| moderación                                                                                                    | deshabilitado  |
| presencia                                                                                                     | deshabilitado  |

## Interfaz de usuario de Components v2

OpenClaw utiliza Discord components v2 para aprobaciones de ejecución y marcadores entre contextos. Las acciones de mensajes de Discord también pueden aceptar `components` para una interfaz de usuario personalizada (avanzado; requiere instancias de componentes Carbon), mientras que los `embeds` heredados siguen disponibles pero no se recomiendan.

- `channels.discord.ui.components.accentColor` establece el color de acento utilizado por los contenedores de componentes de Discord (hex).
- Configúralo por cuenta con `channels.discord.accounts.<id> .ui.components.accentColor`.
- Los `embeds` se ignoran cuando hay components v2 presentes.

Ejemplo:

```json5
{
  channels: {
    discord: {
      enabled: true,
      dm: { enabled: false },
      guilds: {
        YOUR_GUILD_ID: {
          users: ["YOUR_USER_ID"],
          requireMention: true,
          channels: {
            help: { allow: true, requireMention: true },
          },
        },
      },
      retry: {
        attempts: 3,
        minDelayMs: 500,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
    },
  },
}
```

## messages

Los mensajes de voz de Discord muestran una vista previa de la forma de onda y requieren audio OGG/Opus más metadatos. OpenClaw genera la forma de onda automáticamente, pero necesita que `ffmpeg` y `ffprobe` estén disponibles en el host del gateway para inspeccionar y convertir archivos de audio.

Requisitos y restricciones:

- Proporciona una **ruta de archivo local** (las URL se rechazan).
- Omite el contenido de texto (Discord no permite texto + mensaje de voz en la misma carga útil).
- Se acepta cualquier formato de audio; OpenClaw convierte a OGG/Opus cuando es necesario.

Ejemplo:

```bash
message(action="send", channel="discord", target="channel:123", path="/path/to/audio.mp3", asVoice=true)
```

## Solución de problemas

<AccordionGroup>
  <Accordion title="Used disallowed intents or bot sees no guild messages">

    ```
    - habilita Message Content Intent
    - habilita Server Members Intent cuando dependas de la resolución de usuario/miembro
    - reinicia el gateway después de cambiar los intents
    ```

  
</Accordion>

  <Accordion title="Guild messages blocked unexpectedly">

    ```
    `groupPolicy`: controla el manejo de canales de guild (`open|disabled|allowlist`); `allowlist` requiere listas de permitidos de canales.
    ```

```bash
openclaw doctor
openclaw channels status --probe
openclaw logs --follow
```

  
</Accordion>

  <Accordion title="Require mention false but still blocked">
    Causas comunes:

    ```
    Para permitir **ningún canal**, configure `channels.discord.groupPolicy: "disabled"` (o mantenga una lista de permitidos vacía).
    ```

  
</Accordion>

  <Accordion title="Permissions audit mismatches">**Auditorías de permisos** (`channels status --probe`) solo verifican IDs numéricos de canales.

    ```
    Si utilizas claves slug, la coincidencia en tiempo de ejecución puede seguir funcionando, pero la sonda no puede verificar completamente los permisos.
    ```

  
</Accordion>

  <Accordion title="DM and pairing issues">

    ```
    **Los mensajes directos no funcionan**: `channels.discord.dm.enabled=false`, `channels.discord.dm.policy="disabled"`, o aún no ha sido aprobado (`channels.discord.dm.policy="pairing"`).
    ```

  
</Accordion>

  <Accordion title="Bot to bot loops">Los mensajes escritos por el bot se ignoran por defecto; configure `channels.discord.allowBots=true` para permitirlos (los mensajes propios siguen filtrados).

    ```
    Si estableces `channels.discord.allowBots=true`, utiliza reglas estrictas de menciones y listas de permitidos para evitar comportamientos en bucle.
    ```

  
</Accordion>
</AccordionGroup>

## Puntos de referencia de configuración

Referencia principal:

- [Configuration reference - Discord](/gateway/configuration-reference#discord)

Campos de Discord más relevantes:

- `presence` (estado/actividad del bot, predeterminado `false`)
- `guilds.<id> .channels.<channel> .allow`: permitir/denegar el canal cuando `groupPolicy="allowlist"`.
- Use `commands.useAccessGroups: false` para omitir comprobaciones de grupos de acceso para comandos.
- `dmHistoryLimit`: límite de historial de mensajes directos en turnos de usuario. Anulaciones por usuario: `dms["<user_id>"].historyLimit`.
- `maxLinesPerMessage`: máximo suave de líneas por mensaje.
- media/retry: `mediaMaxMb`, `retry`
- actions: `actions.*`
- presence: `activity`, `status`, `activityType`, `activityUrl`
- UI: `ui.components.accentColor`
- features: `pluralkit`, `execApprovals`, `intents`, `agentComponents`, `heartbeat`, `responsePrefix`

## Seguridad y operaciones

- Trata los tokens de bot como secretos (`DISCORD_BOT_TOKEN` es preferible en entornos supervisados).
- Discord bloquea los “privileged intents” a menos que los habilite explícitamente.
- Si el despliegue/estado de comandos está desactualizado, reinicia el gateway y vuelve a comprobar con `openclaw channels status --probe`.

## Relacionado

- [Emparejamiento](/channels/pairing)
- [Enrutamiento de canales](/channels/channel-routing)
- [Resolución de problemas](/channels/troubleshooting)
- Lista completa de comandos + configuración: [Slash commands](/tools/slash-commands)
