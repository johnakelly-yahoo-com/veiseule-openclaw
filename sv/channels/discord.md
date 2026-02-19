---
summary: "Status, funktioner och konfiguration för Discord-botstöd"
read_when:
  - Arbetar med funktioner för Discord-kanalen
title: "Discord"
---

# Discord (Bot API)

Status: redo för DM och textkanaler i guild via den officiella Discord-botgatewayen.

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Discord-DM använder parkopplingsläge som standard.
  
</Card>
  <Card title="Slash commands" icon="terminal" href="/tools/slash-commands">
    Inbyggt kommandobeteende och kommandokatalog.
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    Kanalöverskridande diagnostik och reparationsflöde.
  
</Card>
</CardGroup>

## Snabbstart

<Steps>
  <Step title="Create a Discord bot and enable intents">
    Skapa en applikation i Discord Developer Portal, lägg till en bot och aktivera sedan:


    ```
    - **Message Content Intent**
    - **Server Members Intent** (krävs för rollbaserade tillåtelselistor och rollbaserad dirigering; rekommenderas för namn-till-ID-matchning i tillåtelselistor)
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
    Miljövariabelreserv för standardkontot:
    ```

```bash
DISCORD_BOT_TOKEN=...
```

  
</Step>

  <Step title="Invite the bot and start gateway">
    Bjud in boten till din server med meddelandebehörigheter.

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
    Parningskoder upphör att gälla efter 1 timme.
    ```

  
</Step>
</Steps>

<Note>
Tokenupplösning är kontomedveten. Konfigurerade tokenvärden har företräde framför env‑fallback. `DISCORD_BOT_TOKEN` används endast för standardkontot.
</Note>

## Körningsmodell

- Gateway äger Discord‑anslutningen.
- Svarsdirigering är deterministisk: inkommande från Discord besvaras på Discord.
- Som standard (`session.dmScope=main`) delar direktchattar agentens huvudsession (`agent:main:main`).
- Guild‑kanaler är isolerade sessionsnycklar (`agent:<agentId>:discord:channel:<channelId>`).
- Grupp‑DM ignoreras som standard (`channels.discord.dm.groupEnabled=false`).
- Inbyggda slash‑kommandon körs i isolerade kommandosessioner (`agent:<agentId>:discord:slash:<userId>`), samtidigt som `CommandTargetSessionKey` följer med till den dirigerade konversationssessionen.

## Åtkomstkontroll och dirigering

<Tabs>
  <Tab title="DM policy">
    `channels.discord.dmPolicy` styr åtkomst till DM (äldre: `channels.discord.dm.policy`):

    ```
    - `pairing` (standard)
    - `allowlist`
    - `open` (kräver att `channels.discord.allowFrom` inkluderar `"*"`; äldre: `channels.discord.dm.allowFrom`)
    - `disabled`
    
    Om DM‑policyn inte är open blockeras okända användare (eller uppmanas att para i `pairing`‑läge).
    
    DM‑målformat för leverans:
    
    - `user:<id>`
    - `<@id>`‑omnämnande
    
    Enbart numeriska ID:n är tvetydiga och avvisas om inte en uttrycklig måltyp för användare/kanal anges.
    ```

  
</Tab>

  <Tab title="Guild policy">
    Guild‑hantering styrs av `channels.discord.groupPolicy`:

    ```
    - `open`
    - `allowlist`
    - `disabled`
    
    Säker standard när `channels.discord` finns är `allowlist`.
    
    `allowlist`‑beteende:
    
    - guild måste matcha `channels.discord.guilds` (`id` föredras, slug accepteras)
    - valfria avsändar‑allowlistor: `users` (ID:n eller namn) och `roles` (endast roll‑ID:n); om någon är konfigurerad tillåts avsändare när de matchar `users` ELLER `roles`
    - om en guild har `channels` konfigurerat nekas kanaler som inte är listade
    - om en guild saknar `channels`‑block tillåts alla kanaler i den allowlistade guilden
    
    Exempel:
    ```

```json5
{
  channels: {
    discord: {
      groupPolicy: "allowlist",
      guilds: {
        "123456789012345678": {
          requireMention: true,
          users: ["987654321098765432"],
          roles: ["123456789012345678"],
          channels: {
            general: { allow: true },
            help: { allow: true, requireMention: true },
          },
        },
      },
    },
  },
}
```

    ```
    Om du endast anger `DISCORD_BOT_TOKEN` och inte skapar ett `channels.discord`‑block är runtime‑fallback `groupPolicy="open"` (med en varning i loggarna).
    ```

  
</Tab>

  <Tab title="Mentions and group DMs">
    Guild‑meddelanden kräver omnämnande som standard.

    ```
    Identifiering av omnämnanden inkluderar:
    
    - explicit bot‑omnämnande
    - konfigurerade omnämnandemönster (`agents.list[].groupChat.mentionPatterns`, fallback `messages.groupChat.mentionPatterns`)
    - implicit svar‑till‑bot‑beteende i stödda fall
    
    `requireMention` konfigureras per guild/kanal (`channels.discord.guilds...`).
    
    Grupp‑DM:
    
    - standard: ignoreras (`dm.groupEnabled=false`)
    - valfri allowlist via `dm.groupChannels` (kanal‑ID:n eller sluggar)
    ```

  
</Tab>
</Tabs>

### Rollbaserad agentdirigering

Använd `bindings[].match.roles` för att dirigera Discord‑guildmedlemmar till olika agenter baserat på roll‑ID. Rollbaserade bindningar accepterar endast roll‑ID:n och utvärderas efter peer‑ eller parent‑peer‑bindningar och före guild‑endast‑bindningar. Om en bindning även anger andra matchningsfält (till exempel `peer` + `guildId` + `roles`) måste alla konfigurerade fält matcha.

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

## Inställning i Developer Portal

<AccordionGroup>
  <Accordion title="Create app and bot">

    ```
    1. Discord Developer Portal -> **Applications** -> **New Application**
    2. **Bot** -> **Add Bot**
    3. Kopiera bot‑token
    ```

  
</Accordion>

  <Accordion title="Privileged intents">
    I **Bot -> Privileged Gateway Intents**, aktivera:

    ```
    - Message Content Intent
    - Server Members Intent (rekommenderas)
    
    Presence intent är valfri och krävs endast om du vill ta emot närvarouppdateringar. Att ställa in bot‑närvaro (`setPresence`) kräver inte att närvarouppdateringar för medlemmar är aktiverade.
    ```

  
</Accordion>

  <Accordion title="OAuth scopes and baseline permissions">
    OAuth URL‑generator:

    ```
    - scopes: `bot`, `applications.commands`
    
    Typiska grundläggande behörigheter:
    
    - View Channels
    - Send Messages
    - Read Message History
    - Embed Links
    - Attach Files
    - Add Reactions (valfritt)
    
    Undvik `Administrator` om det inte uttryckligen behövs.
    ```

  
</Accordion>

  <Accordion title="Copy IDs">
    Aktivera Discord Developer Mode och kopiera sedan:

    ```
    - server‑ID
    - kanal‑ID
    - användar‑ID
    
    Föredra numeriska ID:n i OpenClaw‑konfigurationen för tillförlitliga granskningar och prober.
    ```

  
</Accordion>
</AccordionGroup>

## Inbyggda kommandon och kommandoautentisering

- `commands.native` är som standard `"auto"` och är aktiverat för Discord.
- Per-kanal-åsidosättning: `channels.discord.commands.native`.
- `commands.native=false` rensar uttryckligen tidigare registrerade Discord-nativekommandon.
- Native command-auth använder samma Discord allowlists/policys som normal meddelandehantering.
- Kommandon kan fortfarande vara synliga i Discord-gränssnittet för användare som inte är auktoriserade; körning upprätthåller fortfarande OpenClaw-auth och returnerar "not authorized".

Se [Slash commands](/tools/slash-commands) för kommandokatalog och beteende.

## Försökspolicy

<AccordionGroup>
  <Accordion title="Reply tags and native replies">
    Discord stöder reply-taggar i agentens utdata:

    ```
    - `[[reply_to_current]]`
    - `[[reply_to:<id>]]`
    
    Styrs av `channels.discord.replyToMode`:
    
    - `off` (standard)
    - `first`
    - `all`
    
    Obs: `off` inaktiverar implicit svarstrådning. Explicita `[[reply_to_*]]`-taggar respekteras fortfarande.
    
    Meddelande-ID:n exponeras i kontext/historik så att agenter kan rikta in sig på specifika meddelanden.
    ```

  
</Accordion>

  <Accordion title="History, context, and thread behavior">
    Guild-historikkontext:

    ```
    - `channels.discord.historyLimit` standard `20`
    - fallback: `messages.groupChat.historyLimit`
    - `0` inaktiverar
    
    DM-historikkontroller:
    
    - `channels.discord.dmHistoryLimit`
    - `channels.discord.dms["<user_id>"].historyLimit`
    
    Trådbeteende:
    
    - Discord-trådar dirigeras som kanalsessioner
    - överordnad trådmetadata kan användas för länkning till överordnad session
    - trådkonfiguration ärver överordnad kanalkonfiguration om inte en trådspecifik post finns
    
    Kanalämnen injiceras som **otillförlitlig** kontext (inte som systemprompt).
    ```

  
</Accordion>

  <Accordion title="Reaction notifications">
    Per-guild läge för reaktionsnotiser:

    ```
    - `off`
    - `own` (standard)
    - `all`
    - `allowlist` (använder `guilds.<id>.users`)
    
    Reaktionshändelser omvandlas till systemhändelser och bifogas den dirigerade Discord-sessionen.
    ```

  
</Accordion>

  <Accordion title="Ack reactions">`ackReaction` skickar en bekräftelse-emoji medan OpenClaw bearbetar ett inkommande meddelande.

    ```
    Prioritetsordning:
    
    - `channels.discord.accounts.<accountId>.ackReaction`
    - `channels.discord.ackReaction`
    - `messages.ackReaction`
    - fallback till agentidentitetens emoji (`agents.list[].identity.emoji`, annars "👀")
    
    Obs:
    
    - Discord accepterar unicode-emoji eller anpassade emoji-namn.
    - Använd `""` för att inaktivera reaktionen för en kanal eller ett konto.
    ```

  
</Accordion>

  <Accordion title="Config writes">
    Kanalinitierade konfigurationsskrivningar är aktiverade som standard.

    ```
    Detta påverkar flödena `/config set|unset` (när kommandofunktioner är aktiverade).
    
    Inaktivera:
    ```

```json5
{
  channels: {
    discord: {
      configWrites: false,
    },
  },
}
```

  
</Accordion>

  <Accordion title="Gateway proxy">
    Dirigera Discord gateway WebSocket-trafik via en HTTP(S)-proxy med `channels.discord.proxy`.

```json5
{
  channels: {
    discord: {
      proxy: "http://proxy.example:8080",
    },
  },
}
```

    ```
    Per-konto-åsidosättning:
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

  <Accordion title="PluralKit support">
    Aktivera PluralKit-upplösning för att mappa proxade meddelanden till systemmedlemsidentitet:

```json5
{
  channels: {
    discord: {
      pluralkit: {
        enabled: true,
        token: "pk_live_...", // valfritt; krävs för privata system
      },
    },
  },
}
```

    ```
    Obs:
    
    - allowlists kan använda `pk:<memberId>`
    - medlemmars visningsnamn matchas efter namn/slug
    - uppslag använder ursprungligt meddelande-ID och är tidsfönsterbegränsade
    - om uppslag misslyckas behandlas proxade meddelanden som botmeddelanden och släpps om inte `allowBots=true`
    ```

  
</Accordion>

  <Accordion title="Presence configuration">
    Presence-uppdateringar tillämpas endast när du anger ett status- eller aktivitetsfält.

    ```
    Endast status-exempel:
    ```

```json5
{
  channels: {
    discord: {
      status: "idle",
    },
  },
}
```

    ```
    Aktivitetsexempel (anpassad status är standardaktivitetstypen):
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
    Streaming-exempel:
    ```

```json5
{
  channels: {
    discord: {
      activity: "Live coding",
      activityType: 1,
      activityUrl: "https://twitch.tv/openclaw",
    },
  },
}
```

    ```
    Karta över aktivitetstyper:
    
    - 0: Playing
    - 1: Streaming (kräver `activityUrl`)
    - 2: Listening
    - 3: Watching
    - 4: Custom (använder aktivitetstexten som statusläge; emoji är valfri)
    - 5: Competing
    ```

  
</Accordion>

  <Accordion title="Exec approvals in Discord">
    Discord stöder knappbaserade exec-godkännanden i DM och kan valfritt publicera godkännandeförfrågningar i den ursprungliga kanalen.

    ```
    Konfigurationssökväg:
    
    - `channels.discord.execApprovals.enabled`
    - `channels.discord.execApprovals.approvers`
    - `channels.discord.execApprovals.target` (`dm` | `channel` | `both`, standard: `dm`)
    - `agentFilter`, `sessionFilter`, `cleanupAfterResolve`
    
    När `target` är `channel` eller `both` är godkännandeförfrågan synlig i kanalen. Endast konfigurerade godkännare kan använda knapparna; andra användare får ett tillfälligt (ephemeral) avslag. Godkännandeförfrågningar inkluderar kommandotexten, så aktivera endast kanalvisning i betrodda kanaler. Om kanal-ID inte kan härledas från sessionsnyckeln faller OpenClaw tillbaka till DM-leverans.
    
    Om godkännanden misslyckas med okända godkännande-ID:n, verifiera listan över godkännare och att funktionen är aktiverad.
    
    Relaterad dokumentation: [Exec approvals](/tools/exec-approvals)
    ```

  
</Accordion>
</AccordionGroup>

## Verktyg och åtgärdsgrindar

Discord-meddelandeåtgärder inkluderar meddelandehantering, kanaladministration, moderering, presence och metadataåtgärder.

Kärnexempel:

- messaging: `sendMessage`, `readMessages`, `editMessage`, `deleteMessage`, `threadReply`
- reactions: `react`, `reactions`, `emojiList`
- moderation: `timeout`, `kick`, `ban`
- presence: `setPresence`

Åtgärdsgrindar finns under `channels.discord.actions.*`.

Standardbeteende för grindar:

| Åtgärdsgrupp                                                                                                                                                             | Standard |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------- |
| reactions, messages, threads, pins, polls, search, memberInfo, roleInfo, channelInfo, channels, voiceStatus, events, stickers, emojiUploads, stickerUploads, permissions | enabled  |
| roles                                                                                                                                                                    | disabled |
| moderation                                                                                                                                                               | disabled |
| presence                                                                                                                                                                 | disabled |

## Components v2 UI

OpenClaw använder Discord components v2 för exec-godkännanden och markörer över kontextgränser. Discord-meddelandeåtgärder kan också ta emot `components` för anpassat UI (avancerat; kräver Carbon component-instanser), medan äldre `embeds` fortfarande är tillgängliga men inte rekommenderas.

- `channels.discord.ui.components.accentColor` anger accentfärgen som används av Discords komponentbehållare (hex).
- Ställ in per konto med `channels.discord.accounts.<id>`.ui.components.accentColor\`.
- `embeds` ignoreras när components v2 finns.

Exempel:

```json5
{
  channels: {
    discord: {
      ui: {
        components: {
          accentColor: "#5865F2",
        },
      },
    },
  },
}
```

## Röstmeddelanden

Discord-röstmeddelanden visar en vågformsförhandsvisning och kräver OGG/Opus-ljud samt metadata. OpenClaw genererar vågformen automatiskt, men det kräver att `ffmpeg` och `ffprobe` finns tillgängliga på gateway-värden för att inspektera och konvertera ljudfiler.

Krav och begränsningar:

- Ange en **lokal filsökväg** (URL:er avvisas).
- Utelämna textinnehåll (Discord tillåter inte text + röstmeddelande i samma payload).
- Alla ljudformat accepteras; OpenClaw konverterar till OGG/Opus vid behov.

Exempel:

```bash
message(action="send", channel="discord", target="channel:123", path="/path/to/audio.mp3", asVoice=true)
```

## Felsökning

<AccordionGroup>
  <Accordion title="Used disallowed intents or bot sees no guild messages">

    ```
    - aktivera Message Content Intent
    - aktivera Server Members Intent när du är beroende av användar-/medlemsupplösning
    - starta om gateway efter att ha ändrat intents
    ```

  
</Accordion>

  <Accordion title="Guild messages blocked unexpectedly">

    ```
    - verifiera `groupPolicy`
    - verifiera guild allowlist under `channels.discord.guilds`
    - om guildens `channels`-mapp finns är endast listade kanaler tillåtna
    - verifiera beteendet för `requireMention` och omnämningsmönster
    
    Användbara kontroller:
    ```

```bash
openclaw doctor
openclaw channels status --probe
openclaw logs --follow
```

  
</Accordion>

  <Accordion title="Require mention false but still blocked">
    Vanliga orsaker:

    ```
    - `groupPolicy="allowlist"` utan matchande guild-/kanal-allowlist
    - `requireMention` konfigurerad på fel plats (måste ligga under `channels.discord.guilds` eller kanalposten)
    - avsändaren blockeras av guildens/kanalens `users`-allowlist
    ```

  
</Accordion>

  <Accordion title="Permissions audit mismatches">
    Behörighetskontroller med `channels status --probe` fungerar endast för numeriska kanal-ID:n.

    ```
    Om du använder slug-nycklar kan matchning vid körning fortfarande fungera, men probe kan inte fullständigt verifiera behörigheter.
    ```

  
</Accordion>

  <Accordion title="DM and pairing issues">

    ```
    - DM inaktiverat: `channels.discord.dm.enabled=false`
    - DM-policy inaktiverad: `channels.discord.dmPolicy="disabled"` (äldre: `channels.discord.dm.policy`)
    - inväntar godkännande för parkoppling i `pairing`-läge
    ```

  
</Accordion>

  <Accordion title="Bot to bot loops">
    Som standard ignoreras meddelanden som skrivits av botar.

    ```
    Om du sätter `channels.discord.allowBots=true`, använd strikta regler för omnämnanden och allowlist för att undvika loopbeteende.
    ```

  
</Accordion>
</AccordionGroup>

## Konfigurationsreferens – hänvisningar

Primär referens:

- [Konfigurationsreferens – Discord](/gateway/configuration-reference#discord)

Viktiga Discord-fält:

- uppstart/autentisering: `enabled`, `token`, `accounts.*`, `allowBots`
- policy: `groupPolicy`, `dm.*`, `guilds.*`, `guilds.*.channels.*`
- kommando: `commands.native`, `commands.useAccessGroups`, `configWrites`
- reply/history: `replyToMode`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`
- leverans: `textChunkLimit`, `chunkMode`, `maxLinesPerMessage`
- media/återförsök: `mediaMaxMb`, `retry`
- åtgärder: `actions.*`
- närvaro: `activity`, `status`, `activityType`, `activityUrl`
- UI: `ui.components.accentColor`
- funktioner: `pluralkit`, `execApprovals`, `intents`, `agentComponents`, `heartbeat`, `responsePrefix`

## Säkerhet och drift

- Behandla bot-token som hemligheter (`DISCORD_BOT_TOKEN` föredras i övervakade miljöer).
- Bevilja minsta möjliga Discord-behörigheter.
- Om kommandodistribution/status är inaktuell, starta om gateway och kontrollera igen med `openclaw channels status --probe`.

## Relaterat

- [Parning](/channels/pairing)
- [Kanalroutning](/channels/channel-routing)
- [Felsökning](/channels/troubleshooting)
- [Snedstreckskommandon](/tools/slash-commands)

