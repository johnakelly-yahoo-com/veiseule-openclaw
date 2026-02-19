---
summary: "Status van Discord-botondersteuning, mogelijkheden en configuratie"
read_when:
  - Werken aan functies voor het Discord-kanaal
title: "Discord"
---

# Discord (Bot API)

Status: klaar voor DM’s en tekstkanalen in servers via de officiële Discord-botgateway.

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Discord DM's gebruiken standaard de koppelingsmodus.
  
</Card>
  <Card title="Slash commands" icon="terminal" href="/tools/slash-commands">
    Native commandogedrag en commandocatalogus.
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    Cross-channeldiagnostiek en herstelproces.
  
</Card>
</CardGroup>

## Snelle installatie (beginner)

<Steps>
  <Step title="Create a Discord bot and enable intents">Maak een applicatie aan in de Discord Developer Portal, voeg een bot toe en schakel vervolgens het volgende in:

    ```
    Schakel in de instellingen van de Discord-app **Message Content Intent** in (en **Server Members Intent** als je toegestane lijsten of naamopzoekingen wilt gebruiken).
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
    Omgevingsfallback voor het standaardaccount:
    ```

```bash
`DISCORD_BOT_TOKEN=...`
```

  
</Step>

  <Step title="Invite the bot and start gateway">Nodig de bot uit op je server met de rechten om berichten te lezen/verzenden waar je hem wilt gebruiken.

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
    Koppelingscodes verlopen na 1 uur.
    ```

  
</Step>
</Steps>

<Note>
Tokenresolutie is accountbewust. Config-tokenwaarden hebben voorrang op env-fallback. `DISCORD_BOT_TOKEN` wordt alleen gebruikt voor het standaardaccount.
</Note>

## Runtime-model

- Gateway beheert de Discord-verbinding.
- Routering deterministisch houden: antwoorden gaan altijd terug naar het kanaal waarop ze zijn binnengekomen.
- De agent kan `discord` aanroepen met acties zoals:
- Directe chats worden samengevoegd in de hoofdsessie van de agent (standaard `agent:main:main`); serverkanalen blijven geïsoleerd als `agent:<agentId>:discord:channel:<channelId>` (weergavenamen gebruiken `discord:<guildSlug>#<channelSlug>`).
- Groeps-DM’s worden standaard genegeerd; inschakelen via `channels.discord.dm.groupEnabled` en optioneel beperken met `channels.discord.dm.groupChannels`.
- Native opdrachten gebruiken geïsoleerde sessiesleutels (`agent:<agentId>:discord:slash:<userId>`) in plaats van de gedeelde `main`-sessie.

## Toegangscontrole en routering

<Tabs>
  <Tab title="DM policy">Om alle DM’s te negeren: stel `channels.discord.dm.enabled=false` of `channels.discord.dm.policy="disabled"` in.

    ```
    Voor een harde toegestane lijst: stel `channels.discord.dm.policy="allowlist"` in en vermeld afzenders in `channels.discord.dm.allowFrom`.
    ```

  
</Tab>

  <Tab title="Guild policy">
    Guild-afhandeling wordt beheerd door `channels.discord.groupPolicy`:

    ```
    Native opdrachten respecteren dezelfde toegestane lijsten als DM’s/serverberichten (`channels.discord.dm.allowFrom`, `channels.discord.guilds`, per-kanaalregels).
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
    Configureer OpenClaw met `channels.discord.token` (of `DISCORD_BOT_TOKEN` als terugval).
    ```

  
</Tab>

  <Tab title="Mentions and group DMs">
    Guild-berichten zijn standaard mention-gated.

    ```
    Waarschuwing: als je antwoorden op andere bots toestaat (`channels.discord.allowBots=true`), voorkom bot-tot-bot-antwoordlussen met `requireMention`, `channels.discord.guilds.*.channels.<id> .users`-toegestane lijsten en/of door guardrails te wissen in `AGENTS.md` en `SOUL.md`.
    ```

  
</Tab>
</Tabs>

### Rolgebaseerde agentroutering

Gebruik `bindings[].match.roles` om Discord-guildleden op basis van rol-ID naar verschillende agents te routeren. Rolgebaseerde bindings accepteren alleen rol-ID's en worden geëvalueerd na peer- of parent-peer-bindings en vóór guild-only-bindings. Als een binding ook andere match-velden instelt (bijvoorbeeld `peer` + `guildId` + `roles`), moeten alle geconfigureerde velden overeenkomen.

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

## Instellen van de Developer Portal

<AccordionGroup>
  <Accordion title="Create app and bot">

    ```
    Discord Developer Portal → **Applications** → **New Application**
    ```

  
</Accordion>

  <Accordion title="Privileged intents">In **Bot** → **Privileged Gateway Intents**, schakel in:

    ```
    - Message Content Intent
    - Server Members Intent (aanbevolen)
    
    Presence intent is optioneel en alleen vereist als je presence-updates wilt ontvangen. Het instellen van bot presence (`setPresence`) vereist niet dat presence-updates voor leden zijn ingeschakeld.
    ```

  
</Accordion>

  <Accordion title="OAuth scopes and baseline permissions">Genereer een uitnodigings-URL (OAuth2 URL Generator)

    ```
    - scopes: `bot`, `applications.commands`
    
    Typische basisrechten:
    
    - View Channels
    - Send Messages
    - Read Message History
    - Embed Links
    - Attach Files
    - Add Reactions (optioneel)
    
    Vermijd `Administrator` tenzij dit expliciet nodig is.
    ```

  
</Accordion>

  <Accordion title="Copy IDs">
    Schakel Discord Developer Mode in en kopieer vervolgens:

    ```
    - server-ID
    - channel-ID
    - user-ID
    
    Gebruik bij voorkeur numerieke ID's in de OpenClaw-configuratie voor betrouwbare audits en probes.
    ```

  
</Accordion>
</AccordionGroup>

## Native commands en commando-authenticatie

- Optionele native opdrachten: `commands.native` staat standaard op `"auto"` (aan voor Discord/Telegram, uit voor Slack).
- Gedrag wordt geregeld door `channels.discord.replyToMode`:
- Overschrijf met `channels.discord.commands.native: true|false|"auto"`; `false` wist eerder geregistreerde opdrachten.
- Native command-authenticatie gebruikt dezelfde Discord-allowlists/-policies als normale berichtafhandeling.
- Slash-commando’s kunnen in de Discord-UI zichtbaar blijven voor gebruikers die niet op de toegestane lijst staan; OpenClaw handhaaft de toegestane lijsten bij uitvoering en antwoordt met “niet geautoriseerd”.

Zie [Exec approvals](/tools/exec-approvals) en [Slash commands](/tools/slash-commands) voor de bredere goedkeurings- en opdrachtflow.

## Functiedetails

<AccordionGroup>
  <Accordion title="Reply tags and native replies">
    Discord ondersteunt reply-tags in agentuitvoer:

    ```
    Native antwoord-threading staat **standaard uit**; schakel in met `channels.discord.replyToMode` en reply-tags.
    ```

  
</Accordion>

  <Accordion title="History, context, and thread behavior">
    Guild-geschiedeniscontext:

    ```
    - `channels.discord.historyLimit` standaard `20`
    - fallback: `messages.groupChat.historyLimit`
    - `0` schakelt uit
    
    DM-geschiedenisinstellingen:
    
    - `channels.discord.dmHistoryLimit`
    - `channels.discord.dms["<user_id>"].historyLimit`
    
    Thread-gedrag:
    
    - Discord-threads worden gerouteerd als kanaalsessies
    - parent-threadmetadata kan worden gebruikt voor parent-session-koppeling
    - threadconfiguratie erft de parent-kanaalconfiguratie tenzij er een thread-specifieke entry bestaat
    
    Channel topics worden geïnjecteerd als **niet-vertrouwde** context (niet als system prompt).
    ```

  
</Accordion>

  <Accordion title="Reaction notifications">.reactionNotifications`:

    ```
    - `off`
    - `own` (standaard)
    - `all`
    - `allowlist` (gebruikt `guilds.<id>.users`)
    
    Reaction-events worden omgezet in system-events en gekoppeld aan de gerouteerde Discord-sessie.
    ```

  
</Accordion>

  <Accordion title="Ack reactions">
    `ackReaction` verstuurt een bevestigingsemoji terwijl OpenClaw een inkomend bericht verwerkt.

    ```
    Resolutievolgorde:
    
    - `channels.discord.accounts.<accountId>.ackReaction`
    - `channels.discord.ackReaction`
    - `messages.ackReaction`
    - agent identity emoji fallback (`agents.list[].identity.emoji`, anders "👀")
    
    Opmerkingen:
    
    - Discord accepteert unicode-emoji of aangepaste emoji-namen.
    - Gebruik `""` om de reactie voor een kanaal of account uit te schakelen.
    ```

  
</Accordion>

  <Accordion title="Config writes">.channels` aanwezig is, worden niet-vermelde kanalen standaard geweigerd.

    ```
    Dit beïnvloedt `/config set|unset`-flows (wanneer commandofuncties zijn ingeschakeld).
    
    Uitschakelen:
    ```

```json5
{
  channels: { discord: { configWrites: false } },
}
```

  
</Accordion>

  <Accordion title="Gateway proxy">
    Routeer Discord gateway WebSocket-verkeer via een HTTP(S)-proxy met `channels.discord.proxy`.

```json5
Of config: `channels.discord.token: "..."`.
```

    ```
    Per-account-override:
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
    Schakel PluralKit-resolutie in om geproxiede berichten toe te wijzen aan de identiteit van het systeemlid:

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
    Opmerkingen:
    
    - allowlists kunnen `pk:<memberId>` gebruiken
    - weergavenamen van leden worden gematcht op naam/slug
    - lookups gebruiken de originele bericht-ID en zijn beperkt tot een tijdsvenster
    - als lookup mislukt, worden geproxiede berichten behandeld als botberichten en verwijderd tenzij `allowBots=true`
    ```

  
</Accordion>

  <Accordion title="Presence configuration">
    Presence-updates worden alleen toegepast wanneer je een status- of activiteitsveld instelt.

    ```
    Alleen status-voorbeeld:
    ```

```json5
`channelInfo`, `channelList`, `voiceStatus`, `eventList`, `eventCreate`
```

    ```
    Activiteitsvoorbeeld (custom status is het standaardactiviteitstype):
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
    Streaming-voorbeeld:
    ```

```json5
Om het oude “open voor iedereen”-gedrag te behouden: stel `channels.discord.dm.policy="open"` en `channels.discord.dm.allowFrom=["*"]` in.
```

    ```
    Activity type-overzicht:
    
    - 0: Playing
    - 1: Streaming (vereist `activityUrl`)
    - 2: Listening
    - 3: Watching
    - 4: Custom (gebruikt de activiteitstekst als statusstatus; emoji is optioneel)
    - 5: Competing
    ```

  
</Accordion>

  <Accordion title="Exec approvals in Discord">Discord ondersteunt knopgebaseerde exec-goedkeuringen in DM's en kan optioneel goedkeuringsprompts plaatsen in het oorspronkelijke kanaal.

    ```
    `channels.discord.execApprovals.enabled: true` in je config.
    ```

  
</Accordion>
</AccordionGroup>

## Toolacties

Discord-berichtacties omvatten berichten, kanaalbeheer, moderatie, presence en metadata-acties.

Kernvoorbeelden:

- `readMessages`, `sendMessage`, `editMessage`, `deleteMessage`
- reactions: `react`, `reactions`, `emojiList`
- `timeout`, `kick`, `ban`
- presence: `setPresence`

Actiegates bevinden zich onder `channels.discord.actions.*`.

Standaard gate-gedrag:

| Actiegroep                                                                                                    | Standaard     |
| ------------------------------------------------------------------------------------------------------------- | ------------- |
| `stickers`, `emojiUploads`, `stickerUploads`, `polls`, `permissions`, `messages`, `threads`, `pins`, `search` | ingeschakeld  |
| roleInfo                                                                                                      | uitgeschakeld |
| moderatie                                                                                                     | uitgeschakeld |
| aanwezigheid                                                                                                  | uitgeschakeld |

## Components v2 UI

OpenClaw gebruikt Discord components v2 voor exec-goedkeuringen en cross-contextmarkeringen. Discord-berichtacties kunnen ook `components` accepteren voor aangepaste UI (geavanceerd; vereist Carbon component-instanties), terwijl legacy `embeds` beschikbaar blijven maar niet worden aanbevolen.

- `channels.discord.ui.components.accentColor` stelt de accentkleur in die wordt gebruikt door Discord componentcontainers (hex).
- Configureer via `channels.discord.retry`..ui.components.accentColor\`.
- `embeds` worden genegeerd wanneer components v2 aanwezig zijn.

Voorbeeld:

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

Discord-spraakberichten tonen een waveform-preview en vereisen OGG/Opus-audio plus metadata. OpenClaw genereert de waveform automatisch, maar hiervoor moeten `ffmpeg` en `ffprobe` beschikbaar zijn op de gateway-host om audiobestanden te inspecteren en te converteren.

Mogelijkheden & beperkingen

- Geef een **lokaal bestandspad** op (URL's worden geweigerd).
- Laat tekstinhoud weg (Discord staat geen tekst + spraakbericht toe in dezelfde payload).
- Elk audioformaat wordt geaccepteerd; OpenClaw converteert indien nodig naar OGG/Opus.

Voorbeeld:

```bash
message(action="send", channel="discord", target="channel:123", path="/path/to/audio.mp3", asVoice=true)
```

## Problemen oplossen

<AccordionGroup>
  <Accordion title="Used disallowed intents or bot sees no guild messages">

    ```
    **“Used disallowed intents”**: schakel **Message Content Intent** (en waarschijnlijk **Server Members Intent**) in het Developer Portal in en start daarna de Gateway opnieuw.
    ```

  
</Accordion>

  <Accordion title="Guild messages blocked unexpectedly">

    ```
    - verifieer `groupPolicy`
    - verifieer guild allowlist onder `channels.discord.guilds`
    - als de guild-`channels` map bestaat, zijn alleen de vermelde kanalen toegestaan
    - verifieer `requireMention`-gedrag en mentionpatronen
    
    Nuttige controles:
    ```

```bash
openclaw doctor
openclaw channels status --probe
openclaw logs --follow
```

  
</Accordion>

  <Accordion title="Require mention false but still blocked">Algemene oorzaken:

    ```
    `groupPolicy`: regelt de afhandeling van serverkanalen (`open|disabled|allowlist`); `allowlist` vereist kanaaltoegestane lijsten.
    ```

  
</Accordion>

  <Accordion title="Permissions audit mismatches">**Rechtenaudits** (`channels status --probe`) controleren alleen numerieke kanaal-id’s.

    ```
    Als je slug-sleutels gebruikt, kan runtime-matching nog steeds werken, maar probe kan de permissies niet volledig verifiëren.
    ```

  
</Accordion>

  <Accordion title="DM and pairing issues">

    ```
    **DM’s werken niet**: `channels.discord.dm.enabled=false`, `channels.discord.dm.policy="disabled"`, of je bent nog niet goedgekeurd (`channels.discord.dm.policy="pairing"`).
    ```

  
</Accordion>

  <Accordion title="Bot to bot loops">Door de bot geschreven berichten worden standaard genegeerd; stel `channels.discord.allowBots=true` in om ze toe te staan (eigen berichten blijven gefilterd).

    ```
    Als je `channels.discord.allowBots=true` instelt, gebruik dan strikte mention- en allowlistregels om loopgedrag te voorkomen.
    ```

  
</Accordion>
</AccordionGroup>

## Config-wegschrijvingen

Primaire referentie:

- [Configuration reference - Discord](/gateway/configuration-reference#discord)

Belangrijke Discord-velden:

- `presence` (botstatus/-activiteit, standaard `false`)
- `guilds.<id> .channels.<channel> .allow`: sta het kanaal toe/weiger wanneer `groupPolicy="allowlist"`.
- Gebruik `commands.useAccessGroups: false` om toegangscontrole voor opdrachten te omzeilen.
- `dmHistoryLimit`: DM-geschiedenislimeit in gebruikersbeurten. Per-gebruiker-overschrijvingen: `dms["<user_id>"].historyLimit`.
- delivery: `textChunkLimit`, `chunkMode`, `maxLinesPerMessage`
- media/retry: `mediaMaxMb`, `retry`
- actions: `actions.*`
- presence: `activity`, `status`, `activityType`, `activityUrl`
- UI: `ui.components.accentColor`
- features: `pluralkit`, `execApprovals`, `intents`, `agentComponents`, `heartbeat`, `responsePrefix`

## Veiligheid & beheer

- Behandel bot-tokens als geheimen (`DISCORD_BOT_TOKEN` heeft de voorkeur in supervised omgevingen).
- Verleen Discord-permissies volgens het least-privilege-principe.
- Als de command deploy/status verouderd is, herstart dan de gateway en controleer opnieuw met `openclaw channels status --probe`.

## Gerelateerd

- [Koppeling](/channels/pairing)
- [Kanaalroutering](/channels/channel-routing)
- [Probleemoplossing](/channels/troubleshooting)
- Volledige opdrachtenlijst + config: [Slash commands](/tools/slash-commands)

