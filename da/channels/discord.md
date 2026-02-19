---
summary: "Status for Discord-botunderstøttelse, funktioner og konfiguration"
read_when:
  - Arbejder med Discord-kanalfunktioner
title: "Discord"
---

# Discord (Bot API)

Status: klar til DM’er og guild-tekstkanaler via den officielle Discord bot-gateway.

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Discord DMs bruger som standard parringstilstand.
  
</Card>
  <Card title="Slash commands" icon="terminal" href="/tools/slash-commands">
    Indbygget kommandoadfærd og kommandokatalog.
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    Diagnostik og reparationsflow på tværs af kanaler.
  
</Card>
</CardGroup>

## Hurtig opsætning

<Steps>
  <Step title="Create a Discord bot and enable intents">
    Opret en applikation i Discord Developer Portal, tilføj en bot, og aktivér derefter:

    ```
    - **Message Content Intent**
    - **Server Members Intent** (påkrævet for rolle-allowlister og rollebaseret routing; anbefales til allowliste-matchning fra navn til ID)
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
    Miljø-tilbagefald for standardkontoen:
    ```

```bash
DISCORD_BOT_TOKEN=...
```

  
</Step>

  <Step title="Invite the bot and start gateway">
    Inviter botten til din server med beskedtilladelser.

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
    Parringskoder udløber efter 1 time.
    ```

  
</Step>
</Steps>

<Note>
Tokenopløsning er kontobevidst. Config-tokenværdier har forrang over env fallback. `DISCORD_BOT_TOKEN` bruges kun til standardkontoen.
</Note>

## Runtime-model

- Gateway ejer Discord-forbindelsen.
- Svarruting er deterministisk: Discord-indgående svarer tilbage til Discord.
- Som standard (`session.dmScope=main`) deler direkte chats agentens hovedsession (`agent:main:main`).
- Guild-kanaler er isolerede sessionsnøgler (`agent:<agentId>:discord:channel:<channelId>`).
- Gruppe-DM'er ignoreres som standard (`channels.discord.dm.groupEnabled=false`).
- Native slash-kommandoer kører i isolerede kommandosessioner (`agent:<agentId>:discord:slash:<userId>`), mens de stadig medfører `CommandTargetSessionKey` til den routede samtalesession.

## Adgangskontrol og ruting

<Tabs>
  <Tab title="DM policy">
    `channels.discord.dmPolicy` styrer DM-adgang (legacy: `channels.discord.dm.policy`):

    ```
    - `pairing` (standard)
    - `allowlist`
    - `open` (kræver at `channels.discord.allowFrom` inkluderer `"*"`; legacy: `channels.discord.dm.allowFrom`)
    - `disabled`
    
    Hvis DM-policy ikke er open, blokeres ukendte brugere (eller bliver bedt om at parre i `pairing`-tilstand).
    
    DM-målformat til levering:
    
    - `user:<id>`
    - `<@id>` mention
    
    Bare numeriske ID'er er tvetydige og afvises, medmindre en eksplicit bruger-/kanalmåltype er angivet.
    ```

  
</Tab>

  <Tab title="Guild policy">
    Håndtering af guilds styres af `channels.discord.groupPolicy`:

    ```
    - `open`
    - `allowlist`
    - `disabled`
    
    Sikker standard, når `channels.discord` findes, er `allowlist`.
    
    `allowlist`-adfærd:
    
    - guild skal matche `channels.discord.guilds` (`id` foretrækkes, slug accepteres)
    - valgfri afsender-allowlists: `users` (ID'er eller navne) og `roles` (kun rolle-ID'er); hvis en af dem er konfigureret, tillades afsendere, når de matcher `users` ELLER `roles`
    - hvis en guild har `channels` konfigureret, afvises ikke-listede kanaler
    - hvis en guild ikke har en `channels`-blok, er alle kanaler i den allowlistede guild tilladt
    
    Eksempel:
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
    Hvis du kun angiver `DISCORD_BOT_TOKEN` og ikke opretter en `channels.discord`-blok, er runtime fallback `groupPolicy="open"` (med en advarsel i logs).
    ```

  
</Tab>

  <Tab title="Mentions and group DMs">
    Guild-beskeder er som standard mention-gated.

    ```
    Mention-detektion inkluderer:
    
    - eksplicit bot-mention
    - konfigurerede mention-mønstre (`agents.list[].groupChat.mentionPatterns`, fallback `messages.groupChat.mentionPatterns`)
    - implicit svar-til-bot-adfærd i understøttede tilfælde
    
    `requireMention` konfigureres pr. guild/kanal (`channels.discord.guilds...`).
    
    Gruppe-DM'er:
    
    - standard: ignoreret (`dm.groupEnabled=false`)
    - valgfri allowlist via `dm.groupChannels` (kanal-ID'er eller slugs)
    ```

  
</Tab>
</Tabs>

### Rollebaseret agent-ruting

Brug `bindings[].match.roles` til at route Discord-guildmedlemmer til forskellige agenter efter rolle-ID. Rollebaserede bindings accepterer kun rolle-ID'er og evalueres efter peer- eller parent-peer-bindings og før guild-only-bindings. Hvis en binding også angiver andre match-felter (for eksempel `peer` + `guildId` + `roles`), skal alle konfigurerede felter matche.

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

## Opsætning af Developer Portal

<AccordionGroup>
  <Accordion title="Create app and bot">

    ```
    1. Discord Developer Portal -> **Applications** -> **New Application**
    2. **Bot** -> **Add Bot**
    3. Kopiér bot-tokenet
    ```

  
</Accordion>

  <Accordion title="Privileged intents">
    I **Bot -> Privileged Gateway Intents**, aktivér:

    ```
    - Message Content Intent
    - Server Members Intent (anbefalet)
    
    Presence intent er valgfri og kun påkrævet, hvis du vil modtage presence-opdateringer. At sætte bot presence (`setPresence`) kræver ikke aktivering af presence-opdateringer for medlemmer.
    ```

  
</Accordion>

  <Accordion title="OAuth scopes and baseline permissions">
    OAuth URL-generator:

    ```
    - scopes: `bot`, `applications.commands`
    
    Typiske grundlæggende tilladelser:
    
    - View Channels
    - Send Messages
    - Read Message History
    - Embed Links
    - Attach Files
    - Add Reactions (valgfri)
    
    Undgå `Administrator`, medmindre det udtrykkeligt er nødvendigt.
    ```

  
</Accordion>

  <Accordion title="Copy IDs">
    Aktivér Discord Developer Mode, og kopiér derefter:

    ```
    - server-ID
    - kanal-ID
    - bruger-ID
    
    Foretræk numeriske ID'er i OpenClaw-config for pålidelige audits og probes.
    ```

  
</Accordion>
</AccordionGroup>

## Native kommandoer og kommando-godkendelse

- `commands.native` er som standard sat til `"auto"` og er aktiveret for Discord.
- Per-kanal tilsidesættelse: `channels.discord.commands.native`.
- `commands.native=false` rydder eksplicit tidligere registrerede Discord-native kommandoer.
- Native kommando-godkendelse bruger de samme Discord allowlists/politikker som normal beskedhåndtering.
- Kommandoer kan stadig være synlige i Discord UI for brugere, der ikke er autoriserede; udførelse håndhæver stadig OpenClaw-godkendelse og returnerer "not authorized".

Se [Slash commands](/tools/slash-commands) for kommandokatalog og adfærd.

## Retry-politik

<AccordionGroup>
  <Accordion title="Reply tags and native replies">
    Discord understøtter svar-tags i agentoutput:
- `[[reply_to_current]]`
- `[[reply_to:<id>]]`

Styres af `channels.discord.replyToMode`:

- `off` (standard)
- `first`
- `all`

Bemærk: `off` deaktiverer implicit svartrådning. Eksplicitte `[[reply_to_*]]`-tags respekteres stadig.

Besked-ID’er vises i kontekst/historik, så agenter kan målrette specifikke beskeder.

    ```
    
        Guild-historikkontekst:
    ```

  
</Accordion>

  <Accordion title="History, context, and thread behavior">- `channels.discord.historyLimit` standard `20`
- fallback: `messages.groupChat.historyLimit`
- `0` deaktiverer

DM-historikstyring:

- `channels.discord.dmHistoryLimit`
- `channels.discord.dms["<user_id>"].historyLimit`

Trådadfærd:

- Discord-tråde routes som kanalsessioner
- overordnet trådmetadata kan bruges til kobling til overordnet session
- trådkonfiguration arver fra overordnet kanalkonfiguration, medmindre der findes en trådspecifik indgang

Kanaltopics indsættes som **utroværdig** kontekst (ikke som system prompt).

    ```
    
        Per-guild reaktionsnotifikationstilstand:
    ```

  
</Accordion>

  <Accordion title="Reaction notifications">- `off`
- `own` (standard)
- `all`
- `allowlist` (bruger `guilds.<id>.users`)

Reaktionshændelser omdannes til systemhændelser og vedhæftes den routede Discord-session.

    ```
    
        `ackReaction` sender en bekræftelses-emoji, mens OpenClaw behandler en indgående besked.
    ```

  
</Accordion>

  <Accordion title="Ack reactions">Løsningsrækkefølge:

- `channels.discord.accounts.<accountId>.ackReaction`
- `channels.discord.ackReaction`
- `messages.ackReaction`
- agentidentitets-emoji fallback (`agents.list[].identity.emoji`, ellers "👀")

Bemærk:

- Discord accepterer unicode-emoji eller navne på brugerdefinerede emoji.
- Brug `""` for at deaktivere reaktionen for en kanal eller konto.

    ```
    Resolution order:
    
    - `channels.discord.accounts.<accountId>.ackReaction`
    - `channels.discord.ackReaction`
    - `messages.ackReaction`
    - agent identity emoji fallback (`agents.list[].identity.emoji`, else "👀")
    
    Bemærkninger:
    
    - Discord accepterer unicode-emoji eller brugerdefinerede emoji-navne.
    - Brug `""` for at deaktivere reaktionen for en kanal eller konto.
    ```

  
</Accordion>

  <Accordion title="Config writes">Dette påvirker `/config set|unset`-flows (når kommandofunktioner er aktiveret).

Deaktiver:

    ```
    {
      channels: {
        discord: {
          configWrites: false,
        },
      },
    }
    ```

```json5

    Route Discord gateway WebSocket-trafik gennem en HTTP(S)-proxy med `channels.discord.proxy`.
```

  
</Accordion>

  <Accordion title="Gateway proxy">{
  channels: {
    discord: {
      proxy: "http://proxy.example:8080",
    },
  },
}

```json5
Per-konto tilsidesættelse:
```

    ```
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

```json5

    Aktivér PluralKit-opslag for at mappe proxy-beskeder til systemmedlemsidentitet:
```

  
</Accordion>

  <Accordion title="PluralKit support">{
  channels: {
    discord: {
      pluralkit: {
        enabled: true,
        token: "pk_live_...", // valgfri; nødvendig for private systemer
      },
    },
  },
}

```json5
Bemærk:

- allowlists kan bruge `pk:<memberId>`
- medlemsvisningsnavne matches efter navn/slug
- opslag bruger oprindeligt besked-ID og er tidsvinduesbegrænsede
- hvis opslag mislykkes, behandles proxy-beskeder som botbeskeder og droppes, medmindre `allowBots=true`
```

    ```
    
        Presence-opdateringer anvendes kun, når du angiver et status- eller aktivitetsfelt.
    ```

  
</Accordion>

  <Accordion title="Presence configuration">Kun status-eksempel:

    ```
    {
      channels: {
        discord: {
          status: "idle",
        },
      },
    }
    ```

```json5
Aktivitetseksempel (brugerdefineret status er standard aktivitetstype):
```

    ```
    {
      channels: {
        discord: {
          activity: "Focus time",
          activityType: 4,
        },
      },
    }
    ```

```json5
Streaming-eksempel:
```

    ```
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

```json5
Kort over aktivitetstyper:

- 0: Playing
- 1: Streaming (kræver `activityUrl`)
- 2: Listening
- 3: Watching
- 4: Custom (bruger aktivitetsteksten som status; emoji er valgfri)
- 5: Competing
```

    ```
    
        Discord understøtter knapbaserede exec-godkendelser i DM’er og kan valgfrit sende godkendelsesprompter i den oprindelige kanal.
    ```

  
</Accordion>

  <Accordion title="Exec approvals in Discord">Konfigurationssti:

- `channels.discord.execApprovals.enabled`
- `channels.discord.execApprovals.approvers`
- `channels.discord.execApprovals.target` (`dm` | `channel` | `both`, standard: `dm`)
- `agentFilter`, `sessionFilter`, `cleanupAfterResolve`

Når `target` er `channel` eller `both`, er godkendelsesprompten synlig i kanalen. Kun konfigurerede godkendere kan bruge knapperne; andre brugere modtager et midlertidigt afslag. Godkendelsesprompter indeholder kommandoteksten, så aktiver kun kanallevering i betroede kanaler. Hvis kanal-ID ikke kan udledes fra sessionsnøglen, falder OpenClaw tilbage til DM-levering.

Hvis godkendelser fejler med ukendte godkendelses-ID’er, skal du kontrollere godkenderlisten og at funktionen er aktiveret.

Relateret dokumentation: [Exec approvals](/tools/exec-approvals)

    ```
      
</Accordion>
    ```

  
</Accordion>
</AccordionGroup>

## Discord-beskedhandlinger omfatter beskeder, kanaladministration, moderering, presence og metadatahandlinger.

Kerneeksempler:

messaging: `sendMessage`, `readMessages`, `editMessage`, `deleteMessage`, `threadReply`

- messaging: `sendMessage`, `readMessages`, `editMessage`, `deleteMessage`, `threadReply`
- reaktioner: `react`, `reactions`, `emojiList`
- moderation: `timeout`, `kick`, `ban`
- tilstedeværelse: `setPresence`

Handlings‑gates findes under `channels.discord.actions.*`.

Standard gate-adfærd:

| Handlingsgruppe                                                                                                                                                                              | Standard |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| reaktioner, beskeder, tråde, fastgørelser, afstemninger, søgning, memberInfo, roleInfo, channelInfo, kanaler, voiceStatus, begivenheder, stickers, emojiUploads, stickerUploads, tilladelser | enabled  |
| roles                                                                                                                                                                                        | disabled |
| moderation                                                                                                                                                                                   | disabled |
| presence                                                                                                                                                                                     | disabled |

## Components v2 UI

OpenClaw bruger Discord components v2 til exec-godkendelser og markører på tværs af kontekster. Discord-beskedhandlinger kan også acceptere `components` til brugerdefineret UI (avanceret; kræver Carbon component-instanser), mens ældre `embeds` fortsat er tilgængelige, men ikke anbefales.

- `channels.discord.ui.components.accentColor` angiver accentfarven, der bruges af Discord component-containere (hex).
- Angiv pr. konto med `channels.discord.accounts.<id> .ui.components.accentColor`.
- `embeds` ignoreres, når components v2 er til stede.

Eksempel:

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

## Talebeskeder

Discord-talebeskeder viser en waveform-forhåndsvisning og kræver OGG/Opus-lyd samt metadata. OpenClaw genererer automatisk waveformen, men det kræver, at `ffmpeg` og `ffprobe` er tilgængelige på gateway-værten for at inspicere og konvertere lydfiler.

Krav og begrænsninger:

- Angiv en **lokal filsti** (URL’er afvises).
- Udelad tekstindhold (Discord tillader ikke tekst + talebesked i samme payload).
- Alle lydformater accepteres; OpenClaw konverterer til OGG/Opus efter behov.

Eksempel:

```bash
message(action="send", channel="discord", target="channel:123", path="/path/to/audio.mp3", asVoice=true)
```

## Fejlfinding

<AccordionGroup>
  <Accordion title="Used disallowed intents or bot sees no guild messages">

    ```
    - aktivér Message Content Intent
    - aktivér Server Members Intent, når du er afhængig af bruger-/medlemsopslag
    - genstart gateway efter ændring af intents
    ```

  
</Accordion>

  <Accordion title="Guild messages blocked unexpectedly">

    ```
    
        Almindelige årsager:
    ```

```bash
openclaw doctor
openclaw channels status --probe
openclaw logs --follow
```

  
</Accordion>

  <Accordion title="Require mention false but still blocked">
    Almindelige årsager:

    ```
    - `groupPolicy="allowlist"` uden matchende guild-/kanal-allowlist
    - `requireMention` konfigureret det forkerte sted (skal være under `channels.discord.guilds` eller kanalindgangen)
    - afsender blokeret af guild-/kanal-`users`-allowlist
    ```

  
</Accordion>

  <Accordion title="Permissions audit mismatches">
    `channels status --probe`-tilladelsestjek fungerer kun for numeriske kanal-ID’er.

    ```
    Hvis du bruger slug-nøgler, kan runtime-matchning stadig fungere, men probe kan ikke fuldt ud verificere tilladelser.
    ```

  
</Accordion>

  <Accordion title="DM and pairing issues">

    ```
    - DM deaktiveret: `channels.discord.dm.enabled=false`
    - DM-politik deaktiveret: `channels.discord.dmPolicy="disabled"` (ældre: `channels.discord.dm.policy`)
    - afventer parringsgodkendelse i `pairing`-tilstand
    ```

  
</Accordion>

  <Accordion title="Bot to bot loops">
    Som standard ignoreres beskeder oprettet af bots.

    ```
    Hvis du sætter `channels.discord.allowBots=true`, skal du bruge strenge mention- og allowlist-regler for at undgå loop-adfærd.
    ```

  
</Accordion>
</AccordionGroup>

## Konfigurationsreferencehenvisninger

Primær reference:

- [Configuration reference - Discord](/gateway/configuration-reference#discord)

Vigtige Discord-felter:

- opstart/autentificering: `enabled`, `token`, `accounts.*`, `allowBots`
- policy: `groupPolicy`, `dm.*`, `guilds.*`, `guilds.*.channels.*`
- kommando: `commands.native`, `commands.useAccessGroups`, `configWrites`
- svar/historik: `replyToMode`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`
- levering: `textChunkLimit`, `chunkMode`, `maxLinesPerMessage`
- medier/genforsøg: `mediaMaxMb`, `retry`
- handlinger: `actions.*`
- tilstedeværelse: `activity`, `status`, `activityType`, `activityUrl`
- UI: `ui.components.accentColor`
- funktioner: `pluralkit`, `execApprovals`, `intents`, `agentComponents`, `heartbeat`, `responsePrefix`

## Sikkerhed og drift

- Behandl bot-tokens som hemmeligheder (`DISCORD_BOT_TOKEN` foretrækkes i superviserede miljøer).
- Tildel mindst mulige Discord-tilladelser.
- Hvis kommandoimplementering/tilstand er forældet, genstart gatewayen og kontrollér igen med `openclaw channels status --probe`.

## Relateret

- [Parring](/channels/pairing)
- [Kanalsrouting](/channels/channel-routing)
- [Fejlfinding](/channels/troubleshooting)
- [Slash-kommandoer](/tools/slash-commands)
