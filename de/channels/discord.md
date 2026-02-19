---
summary: "„Status der Discord-Bot-Unterstützung, Funktionen und Konfiguration“"
read_when:
  - Arbeit an Discord-Kanal-Funktionen
title: "Discord"
---

# Discord (Bot API)

Status: bereit für Direktnachrichten und Guild-Textkanäle über das offizielle Discord-Bot-Gateway.

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Discord-DMs verwenden standardmäßig den Pairing-Modus.
  
</Card>
  <Card title="Slash commands" icon="terminal" href="/tools/slash-commands">
    Natives Befehlsverhalten und Befehlskatalog.
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    Kanalübergreifende Diagnose- und Reparaturabläufe.
  
</Card>
</CardGroup>

## Schnellstart

<Steps>
  <Step title="Create a Discord bot and enable intents">Discord-App + Bot-Benutzer erstellen

    ```
    **Server Members Intent** (empfohlen; erforderlich für einige Mitglieder-/Benutzerabfragen und Allowlist-Abgleiche in Guilds)
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
    Env-Fallback für das Standardkonto:
    ```

```bash
`DISCORD_BOT_TOKEN=...`
```

  
</Step>

  <Step title="Invite the bot and start gateway">Laden Sie den Bot mit den erforderlichen Berechtigungen auf Ihren Server ein, um Nachrichten dort zu lesen/senden, wo Sie ihn verwenden möchten.

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
    Pairing-Codes laufen nach 1 Stunde ab.
    ```

  
</Step>
</Steps>

<Note>
Die Token-Auflösung ist account-spezifisch. Konfigurations-Tokenwerte haben Vorrang vor dem Env-Fallback. `DISCORD_BOT_TOKEN` wird nur für das Standardkonto verwendet.
</Note>

## Laufzeitmodell

- Gateway verwaltet die Discord-Verbindung.
- Deterministisches Routing beibehalten: Antworten gehen immer an den Kanal zurück, auf dem sie eingegangen sind.
- Der Agent kann `discord` mit Aktionen wie folgenden aufrufen:
- Direktchats werden in der Hauptsitzung des Agenten zusammengeführt (Standard: `agent:main:main`); Guild-Kanäle bleiben als `agent:<agentId>:discord:channel:<channelId>` isoliert (Anzeigenamen verwenden `discord:<guildSlug>#<channelSlug>`).
- Gruppen-DMs werden standardmäßig ignoriert; aktivieren Sie sie über `channels.discord.dm.groupEnabled` und beschränken Sie sie optional über `channels.discord.dm.groupChannels`.
- Native Befehle verwenden isolierte Sitzungsschlüssel (`agent:<agentId>:discord:slash:<userId>`) statt der gemeinsamen Sitzung `main`.

## Zugriffskontrolle und Routing

<Tabs>
  <Tab title="DM policy">Um alle DMs zu ignorieren: setzen Sie `channels.discord.dm.enabled=false` oder `channels.discord.dm.policy="disabled"`.

    ```
    Für eine harte Allowlist: setzen Sie `channels.discord.dm.policy="allowlist"` und listen Sie Absender in `channels.discord.dm.allowFrom` auf.
    ```

  
</Tab>

  <Tab title="Guild policy">Das Verhalten wird durch `channels.discord.replyToMode` gesteuert:

    ```
    Optionale Guild-Regeln: setzen Sie `channels.discord.guilds` nach Guild-ID (bevorzugt) oder Slug, mit Regeln pro Kanal.
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
    Konfigurieren Sie OpenClaw mit `channels.discord.token` (oder `DISCORD_BOT_TOKEN` als Fallback).
    ```

  
</Tab>

  <Tab title="Mentions and group DMs">
    Guild-Nachrichten sind standardmäßig durch Erwähnungen (Mentions) geschützt.

    ```
    Warnung: Wenn Sie Antworten an andere Bots erlauben (`channels.discord.allowBots=true`), verhindern Sie Bot-zu-Bot-Schleifen mit `requireMention`, `channels.discord.guilds.*.channels.<id> .users`-Allowlists und/oder klaren Schutzmechanismen in `AGENTS.md` und `SOUL.md`.
    ```

  
</Tab>
</Tabs>

### Rollenbasiertes Agent-Routing

Verwende `bindings[].match.roles`, um Discord-Guild-Mitglieder anhand ihrer Rollen-ID an verschiedene Agenten weiterzuleiten. Rollenbasierte Bindings akzeptieren nur Rollen-IDs und werden nach Peer- oder Parent-Peer-Bindings und vor Guild-only-Bindings ausgewertet. Wenn ein Binding zusätzlich weitere Match-Felder setzt (z. B. `peer` + `guildId` + `roles`), müssen alle konfigurierten Felder übereinstimmen.

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

## Einrichtung im Developer Portal

<AccordionGroup>
  <Accordion title="Create app and bot">

    ```
    Erstellen Sie einen Discord-Bot und kopieren Sie das Bot-Token.
    ```

  
</Accordion>

  <Accordion title="Privileged intents">Unter **Bot** → **Privileged Gateway Intents** aktivieren Sie:

    ```
    - Message Content Intent
    - Server Members Intent (empfohlen)
    
    Presence Intent ist optional und nur erforderlich, wenn du Presence-Updates empfangen möchtest. Das Setzen der Bot-Presence (`setPresence`) erfordert nicht, dass Presence-Updates für Mitglieder aktiviert sind.
    ```

  
</Accordion>

  <Accordion title="OAuth scopes and baseline permissions">Einladungs-URL erzeugen (OAuth2 URL Generator)

    ```
    - scopes: `bot`, `applications.commands`
    
    Typische Basisberechtigungen:
    
    - Kanäle anzeigen
    - Nachrichten senden
    - Nachrichtenverlauf lesen
    - Links einbetten
    - Dateien anhängen
    - Reaktionen hinzufügen (optional)
    
    Vermeide `Administrator`, sofern nicht ausdrücklich erforderlich.
    ```

  
</Accordion>

  <Accordion title="Copy IDs">
    Aktiviere den Discord-Entwicklermodus und kopiere dann:

    ```
    - Server-ID
    - Kanal-ID
    - Benutzer-ID
    
    Bevorzuge numerische IDs in der OpenClaw-Konfiguration für zuverlässige Audits und Prüfungen.
    ```

  
</Accordion>
</AccordionGroup>

## Native Befehle und Befehlsautorisierung

- Optionale native Befehle: `commands.native` ist standardmäßig `"auto"` (an für Discord/Telegram, aus für Slack).
- Oder Konfiguration: `channels.discord.token: "..."`.
- Überschreiben Sie mit `channels.discord.commands.native: true|false|"auto"`; `false` löscht zuvor registrierte Befehle.
- Die Autorisierung nativer Befehle verwendet dieselben Discord-Allowlisten/-Richtlinien wie die normale Nachrichtenverarbeitung.
- Slash-Befehle können in der Discord-UI für nicht allowlistete Benutzer sichtbar sein; OpenClaw erzwingt die Allowlists bei der Ausführung und antwortet mit „not authorized“.

Siehe [Slash commands](/tools/slash-commands) für Befehlsübersicht und Verhalten.

## Funktionsdetails

<AccordionGroup>
  <Accordion title="Reply tags and native replies">
    Discord unterstützt Reply-Tags in der Agent-Ausgabe:

    ```
    Native Antwort-Threading ist **standardmäßig aus**; aktivieren Sie es mit `channels.discord.replyToMode` und Antwort-Tags.
    ```

  
</Accordion>

  <Accordion title="History, context, and thread behavior">
    Guild-Verlaufskontext:

    ```
    - `channels.discord.historyLimit` Standard `20`
    - Fallback: `messages.groupChat.historyLimit`
    - `0` deaktiviert
    
    DM-Verlaufssteuerung:
    
    - `channels.discord.dmHistoryLimit`
    - `channels.discord.dms["<user_id>"].historyLimit`
    
    Thread-Verhalten:
    
    - Discord-Threads werden als Kanalsitzungen weitergeleitet
    - Metadaten des Parent-Threads können für die Verknüpfung mit der Parent-Sitzung verwendet werden
    - Thread-Konfiguration erbt die Konfiguration des Parent-Kanals, sofern kein threadspezifischer Eintrag existiert
    
    Kanalthemen werden als **nicht vertrauenswürdiger** Kontext eingefügt (nicht als System-Prompt).
    ```

  
</Accordion>

  <Accordion title="Reaction notifications">.reactionNotifications`:

    ```
    `allowlist`: Reaktionen von `guilds.<id> .users` auf allen Nachrichten (leere Liste deaktiviert).
    ```

  
</Accordion>

  <Accordion title="Ack reactions">
    `ackReaction` sendet ein Bestätigungs-Emoji, während OpenClaw eine eingehende Nachricht verarbeitet.

    ```
    Auflösungsreihenfolge:
    
    - `channels.discord.accounts.<accountId>.ackReaction`
    - `channels.discord.ackReaction`
    - `messages.ackReaction`
    - Fallback auf Agent-Identitäts-Emoji (`agents.list[].identity.emoji`, sonst "👀")
    
    Hinweise:
    
    - Discord akzeptiert Unicode-Emojis oder benutzerdefinierte Emoji-Namen.
    - Verwende `""`, um die Reaktion für einen Kanal oder ein Konto zu deaktivieren.
    ```

  
</Accordion>

  <Accordion title="Config writes">
    Kanalinitiierte Konfigurationsschreibvorgänge sind standardmäßig aktiviert.

    ```
    Dies betrifft `/config set|unset`-Abläufe (wenn Befehlsfunktionen aktiviert sind).
    
    Deaktivieren:
    ```

```json5
{
  channels: { discord: { configWrites: false } },
}
```

  
</Accordion>

  <Accordion title="Gateway proxy">
    Leite Discord-Gateway-WebSocket-Datenverkehr über einen HTTP(S)-Proxy mit `channels.discord.proxy`.

```json5
Um das frühere „für alle offen“-Verhalten beizubehalten: setzen Sie `channels.discord.dm.policy="open"` und `channels.discord.dm.allowFrom=["*"]`.
```

    ```
    Pro-Konto-Überschreibung:
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
    Aktiviere PluralKit-Auflösung, um weitergeleitete Nachrichten der Systemmitglied-Identität zuzuordnen:

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
    Hinweise:
    
    - Allowlisten können `pk:<memberId>` verwenden
    - Anzeigenamen von Mitgliedern werden nach Name/Slug abgeglichen
    - Abfragen verwenden die ursprüngliche Nachrichten-ID und sind zeitlich begrenzt
    - Wenn die Abfrage fehlschlägt, werden weitergeleitete Nachrichten als Bot-Nachrichten behandelt und verworfen, sofern nicht `allowBots=true` gesetzt ist
    ```

  
</Accordion>

  <Accordion title="Presence configuration">
    Presence-Updates werden nur angewendet, wenn du ein Status- oder Aktivitätsfeld setzt.

    ```
    Beispiel nur Status:
    ```

```json5
`channelInfo`, `channelList`, `voiceStatus`, `eventList`, `eventCreate`
```

    ```
    Aktivitätsbeispiel (Custom-Status ist der Standard-Aktivitätstyp):
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
    Streaming-Beispiel:
    ```

```json5
Mit OpenClaw über Discord-DMs oder Guild-Kanäle kommunizieren.
```

    ```
    Aktivitätstyp-Zuordnung:
    
    - 0: Playing
    - 1: Streaming (erfordert `activityUrl`)
    - 2: Listening
    - 3: Watching
    - 4: Custom (verwendet den Aktivitätstext als Status; Emoji ist optional)
    - 5: Competing
    ```

  
</Accordion>

  <Accordion title="Exec approvals in Discord">    Discord unterstützt button-basierte Exec-Genehmigungen in DMs und kann Genehmigungsanfragen optional im ursprünglichen Kanal veröffentlichen.

    ```
    Config-Pfad:
    
    - `channels.discord.execApprovals.enabled`
    - `channels.discord.execApprovals.approvers`
    - `channels.discord.execApprovals.target` (`dm` | `channel` | `both`, Standard: `dm`)
    - `agentFilter`, `sessionFilter`, `cleanupAfterResolve`
    
    Wenn `target` auf `channel` oder `both` gesetzt ist, ist die Genehmigungsanfrage im Kanal sichtbar. Nur konfigurierte Genehmiger können die Buttons verwenden; andere Nutzer erhalten eine ephemere Ablehnung. Genehmigungsanfragen enthalten den Befehlstext, daher sollte die Zustellung im Kanal nur in vertrauenswürdigen Kanälen aktiviert werden. Wenn die Kanal-ID nicht aus dem Session-Key abgeleitet werden kann, greift OpenClaw auf die Zustellung per DM zurück.
    
    Wenn Genehmigungen mit unbekannten Approval-IDs fehlschlagen, überprüfen Sie die Genehmigerliste und ob die Funktion aktiviert ist.
    
    Verwandte Dokumentation: [Exec approvals](/tools/exec-approvals)
    ```

  
</Accordion>
</AccordionGroup>

## Tool-Aktionen

Discord-Nachrichtenaktionen umfassen Messaging-, Kanalverwaltungs-, Moderations-, Präsenz- und Metadatenaktionen.

Kernbeispiele:

- `readMessages`, `sendMessage`, `editMessage`, `deleteMessage`
- Reagieren + Reaktionen auflisten + emojiList
- `timeout`, `kick`, `ban`
- presence: `setPresence`

Action-Gates befinden sich unter `channels.discord.actions.*`.

Standardverhalten der Gates:

| Aktionsgruppe                                                                                                 | Standard    |
| ------------------------------------------------------------------------------------------------------------- | ----------- |
| `stickers`, `emojiUploads`, `stickerUploads`, `polls`, `permissions`, `messages`, `threads`, `pins`, `search` | aktiviert   |
| Rollen                                                                                                        | deaktiviert |
| Moderation                                                                                                    | deaktiviert |
| presence                                                                                                      | deaktiviert |

## Components v2 UI

OpenClaw verwendet Discord Components v2 für Exec-Genehmigungen und kontextübergreifende Marker. Discord-Nachrichtenaktionen können auch `components` für eine benutzerdefinierte UI akzeptieren (fortgeschritten; erfordert Carbon-Component-Instanzen), während die älteren `embeds` weiterhin verfügbar, jedoch nicht empfohlen sind.

- `channels.discord.ui.components.accentColor` legt die Akzentfarbe fest, die von Discord-Component-Containern verwendet wird (Hex).
- Konfigurieren Sie dies über `channels.discord.retry`..ui.components.accentColor\`.
- `embeds` werden ignoriert, wenn Components v2 vorhanden sind.

Beispiel:

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

Discord-Sprachnachrichten zeigen eine Wellenform-Vorschau und erfordern OGG/Opus-Audio sowie Metadaten. OpenClaw generiert die Wellenform automatisch, benötigt dafür jedoch `ffmpeg` und `ffprobe`, die auf dem Gateway-Host verfügbar sein müssen, um Audiodateien zu analysieren und zu konvertieren.

Anforderungen und Einschränkungen:

- Geben Sie einen **lokalen Dateipfad** an (URLs werden abgelehnt).
- Textinhalt weglassen (Discord erlaubt nicht Text + Sprachnachricht im selben Payload).
- Jedes Audioformat wird akzeptiert; OpenClaw konvertiert bei Bedarf in OGG/Opus.

Beispiel:

```bash
message(action="send", channel="discord", target="channel:123", path="/path/to/audio.mp3", asVoice=true)
```

## Fehlerbehebung

<AccordionGroup>
  <Accordion title="Used disallowed intents or bot sees no guild messages">

    ```
    **„Used disallowed intents“**: Aktivieren Sie **Message Content Intent** (und vermutlich **Server Members Intent**) im Developer Portal und starten Sie das Gateway neu.
    ```

  
</Accordion>

  <Accordion title="Guild messages blocked unexpectedly">

    ```
    `groupPolicy`: steuert die Behandlung von Guild-Kanälen (`open|disabled|allowlist`); `allowlist` erfordert Kanal-Allowlists.
    ```

```bash
openclaw doctor
openclaw channels status --probe
openclaw logs --follow
```

  
</Accordion>

  <Accordion title="Require mention false but still blocked">    Häufige Ursachen:

    ```
    Um **keine Kanäle** zu erlauben, setzen Sie `channels.discord.groupPolicy: "disabled"` (oder lassen Sie die Allowlist leer).
    ```

  
</Accordion>

  <Accordion title="Permissions audit mismatches">**Berechtigungsprüfungen** (`channels status --probe`) prüfen nur numerische Kanal-IDs.

    ```
    Wenn Sie Slug-Keys verwenden, kann das Runtime-Matching weiterhin funktionieren, aber probe kann die Berechtigungen nicht vollständig überprüfen.
    ```

  
</Accordion>

  <Accordion title="DM and pairing issues">

    ```
    **DMs funktionieren nicht**: `channels.discord.dm.enabled=false`, `channels.discord.dm.policy="disabled"` oder Sie wurden noch nicht genehmigt (`channels.discord.dm.policy="pairing"`).
    ```

  
</Accordion>

  <Accordion title="Bot to bot loops">`own`: Reaktionen auf eigene Bot-Nachrichten (Standard).

    ```
    Vom Bot verfasste Nachrichten werden standardmäßig ignoriert; setzen Sie `channels.discord.allowBots=true`, um sie zuzulassen (eigene Nachrichten bleiben gefiltert).
    ```

  
</Accordion>
</AccordionGroup>

## Konfigurationsreferenz – Verweise

Primäre Referenz:

- [Configuration reference - Discord](/gateway/configuration-reference#discord)

Wichtige Discord-Felder:

- `presence` (Bot-Status/Aktivität, Standard `false`)
- `guilds.<id> .channels.<channel> .allow`: Kanal zulassen/ablehnen, wenn `groupPolicy="allowlist"`.
- Verwenden Sie `commands.useAccessGroups: false`, um Zugriffskontrollen für Befehle zu umgehen.
- reply/history: `replyToMode`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`
- delivery: `textChunkLimit`, `chunkMode`, `maxLinesPerMessage`
- media/retry: `mediaMaxMb`, `retry`
- actions: `actions.*`
- presence: `activity`, `status`, `activityType`, `activityUrl`
- UI: `ui.components.accentColor`
- features: `pluralkit`, `execApprovals`, `intents`, `agentComponents`, `heartbeat`, `responsePrefix`

## Sicherheit & Betrieb

- Behandeln Sie Bot-Tokens als Geheimnisse (`DISCORD_BOT_TOKEN` wird in überwachten Umgebungen bevorzugt).
- Gewähren Sie Discord-Berechtigungen nach dem Prinzip der geringsten Privilegien.
- Wenn der Befehl deploy/state veraltet ist, starte das Gateway neu und prüfe erneut mit `openclaw channels status --probe`.

## Verwandt

- [Pairing](/channels/pairing)
- Kanal (z. B.
- [Troubleshooting](/channels/troubleshooting)
- Vollständige Befehlsliste + Konfiguration: [Slash commands](/tools/slash-commands)

