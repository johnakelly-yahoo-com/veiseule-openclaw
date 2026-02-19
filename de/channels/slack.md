---
summary: "Erstellen Sie eine Slack-App und **deaktivieren Sie den Socket-Modus** (optional, wenn Sie nur HTTP verwenden)."
read_when:
  - „Einrichtung von Slack oder Debugging des Slack-Socket-/HTTP-Modus“
title: "Slack"
---

# Slack

Status: produktionsbereit für DMs + Kanäle über Slack-App-Integrationen. HTTP-Modus (Events API)

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Slack-DMs verwenden standardmäßig den Pairing-Modus.
  
</Card>
  <Card title="Slash commands" icon="terminal" href="/tools/slash-commands">
    Natives Befehlsverhalten und Befehlskatalog.
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    Kanalübergreifende Diagnose- und Reparaturleitfäden.
  
</Card>
</CardGroup>

## Schnellstart

<Tabs>
  <Tab title="Socket Mode (default)">
    <Steps>
      <Step title="Create Slack app and tokens">
        In den Slack-App-Einstellungen:


        ```
        **OAuth & Permissions** → installieren Sie die App und kopieren Sie das **Bot User OAuth Token** (`xoxb-...`).
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
            Env-Fallback (nur Standardkonto):
        ```

```bash
`SLACK_APP_TOKEN=xapp-...`
```

        
</Step>
      
        <Step title="App-Ereignisse abonnieren">
          Abonnieren Sie Bot-Ereignisse für:
      
          - `app_mention`
          - `message.channels`, `message.groups`, `message.im`, `message.mpim`
          - `reaction_added`, `reaction_removed`
          - `member_joined_channel`, `member_left_channel`
          - `channel_rename`
          - `pin_added`, `pin_removed`
      
          Aktivieren Sie außerdem im App Home den **Nachrichten-Tab** für DMs.
        
</Step>
      
        <Step title="Gateway starten">

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
        Verwenden Sie den HTTP-Webhook-Modus, wenn Ihr Gateway für Slack über HTTPS erreichbar ist (typisch für Server-Deployments). Der HTTP-Modus verwendet die Events API + Interactivity + Slash Commands mit einer gemeinsamen Request-URL.
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

        
</Step>
      
        <Step title="Eindeutige Webhook-Pfade für Multi-Account-HTTP verwenden">
          HTTP-Modus pro Konto wird unterstützt.
      
          Weisen Sie jedem Konto einen eigenen `webhookPath` zu, damit sich Registrierungen nicht überschneiden.
        
</Step>
      
</Steps>

  
</Tab>
</Tabs>

## Token-Modell

- `botToken` + `appToken` sind für den Socket-Modus erforderlich.
- Der HTTP-Modus erfordert `botToken` + `signingSecret`.
- Konfigurations-Tokens überschreiben das Env-Fallback.
- Das Env-Fallback `SLACK_BOT_TOKEN` / `SLACK_APP_TOKEN` gilt nur für das Standardkonto.
- Beispiel mit explizit gesetztem userTokenReadOnly (User-Token-Schreibzugriffe erlauben):
- Optional: Fügen Sie `chat:write.customize` hinzu, wenn ausgehende Nachrichten die aktive Agentenidentität (benutzerdefinierter `username` und Icon) verwenden sollen. `icon_emoji` verwendet die Syntax `:emoji_name:`.

<Tip>
Für Aktionen/Verzeichnisabfragen kann bei entsprechender Konfiguration ein User-Token bevorzugt werden. Selbst mit `userTokenReadOnly: false` bleibt das Bot-Token
für Schreibzugriffe bevorzugt, wenn es verfügbar ist.
</Tip>

## Zugriffskontrolle und Routing

<Tabs>
  <Tab title="DM policy">DMs werden ignoriert: Absender nicht freigegeben bei `channels.slack.dm.policy="pairing"`.

    ```
    Um alle zuzulassen: Setzen Sie `channels.slack.dm.policy="open"` und `channels.slack.dm.allowFrom=["*"]`.
    ```

  
</Tab>

  <Tab title="Channel policy">`channels.slack.groupPolicy` steuert die Kanalbehandlung (`open|disabled|allowlist`).

    ```
    Um **keine Kanäle** zuzulassen, setzen Sie `channels.slack.groupPolicy: "disabled"` (oder behalten Sie eine leere Allowlist).
    ```

  
</Tab>

  <Tab title="Mentions and channel users">
    Kanalnachrichten sind standardmäßig durch Erwähnungen (Mentions) eingeschränkt.


    ```
    Erwähnungs-Gating wird über `channels.slack.channels` gesteuert (setzen Sie `requireMention` auf `true`); `agents.list[].groupChat.mentionPatterns` (oder `messages.groupChat.mentionPatterns`) zählen ebenfalls als Erwähnungen.
    ```

  
</Tab>
</Tabs>

## Befehle und Slash-Verhalten

- Slack-Slash-Commands werden in der Slack-App verwaltet und nicht automatisch entfernt.
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
- Wenn native Befehle aktiviert sind, registrieren Sie entsprechende Slash-Befehle in Slack (Namen im Format `/<command>`).
- Wenn Sie native Commands aktivieren, fügen Sie einen `slash_commands`-Eintrag pro Command hinzu, den Sie bereitstellen möchten (entsprechend der `/help`-Liste). Überschreiben Sie dies mit `channels.slack.commands.native`.

Standard-Slash-Befehlseinstellungen:

- `enabled: false`
- `name: "openclaw"`
- `sessionPrefix: "slack:slash"`
- `ephemeral: true`

Slash-Sitzungen verwenden isolierte Schlüssel:

- Slash Commands verwenden `agent:<agentId>:slack:slash:<userId>`-Sitzungen (Präfix konfigurierbar über `channels.slack.slashCommand.sessionPrefix`).

und leiten die Befehlsausführung weiterhin an die Ziel-Konversationssitzung (`CommandTargetSessionKey`) weiter.

## Threads, Sitzungen und Antwort-Tags

- Gruppen-DMs threaden, Kanäle im Root belassen:
- Mit der Standardeinstellung `session.dmScope=main` werden Slack-DMs der Hauptsitzung des Agenten zugeordnet.
- Kanäle werden auf `agent:<agentId>:slack:channel:<channelId>`-Sitzungen abgebildet.
- Thread-Antworten können, sofern zutreffend, Thread-Sitzungssuffixe (`:thread:<threadTs>`) erstellen.
- `channels.slack.thread.historyScope` ist standardmäßig `thread`; `thread.inheritParent` ist standardmäßig `false`.
- `channels.slack.historyLimit` (oder `channels.slack.accounts.*.historyLimit`) steuert, wie viele aktuelle Kanal-/Gruppennachrichten in den Prompt eingebettet werden.

Antwort-Threading

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
- Legacy `channels.slack.dm.replyToMode` wird weiterhin als Fallback für `direct` akzeptiert, wenn kein Chat-Typ-Override gesetzt ist.

Manuelle Antwort-Tags werden unterstützt:

- `[[reply_to_current]]`
- `[[reply_to:<id>]]` — Antwort auf eine bestimmte Nachrichten-ID.

Hinweis: `replyToMode="off"` deaktiviert implizites Antwort-Threading. Explizite `[[reply_to_*]]`-Tags werden weiterhin berücksichtigt.

## Medien, Chunking und Zustellung

<AccordionGroup>
  <Accordion title="Inbound attachments">    Slack-Dateianhänge werden von Slack-gehosteten privaten URLs (token-authentifizierter Anfragefluss) heruntergeladen und im Media-Store gespeichert, wenn der Abruf erfolgreich ist und Größenlimits dies erlauben.

    ```
    Medien-Uploads sind durch `channels.slack.mediaMaxMb` begrenzt (Standard: 20).
    ```

  
</Accordion>

  <Accordion title="Outbound text and files">Ausgehender Text wird in Blöcke von `channels.slack.textChunkLimit` aufgeteilt (Standard: 4000).
</Accordion>

  <Accordion title="Delivery targets">    Bevorzugte explizite Ziele:

    ```
    - `user:<id>` für DMs
    - `channel:<id>` für Channels
    
    Slack-DMs werden über die Slack-Conversation-APIs geöffnet, wenn an Benutzerziele gesendet wird.
    ```

  
</Accordion>
</AccordionGroup>

## Aktionen und Gates

Slack-Tool-Aktionen können mit `channels.slack.actions.*` eingeschränkt werden:

Verfügbare Aktionsgruppen in der aktuellen Slack-Tooling-Umgebung:

| Aktionsgruppe | Standard  |
| ------------- | --------- |
| messages      | aktiviert |
| reactions     | aktiviert |
| pins          | aktiviert |
| memberInfo    | aktiviert |
| emojiList     | aktiviert |

## Ereignisse und operatives Verhalten

- Nachrichtenbearbeitungen/-löschungen/Thread-Broadcasts werden in Systemereignisse abgebildet.
- Reaction-Add/Remove-Ereignisse werden in Systemereignisse abgebildet.
- Mitglied beigetreten/verlassen, Channel erstellt/umbenannt sowie Pin hinzugefügt/entfernt werden in Systemereignisse abgebildet.
- `channel_id_changed` kann Channel-Konfigurationsschlüssel migrieren, wenn `configWrites` aktiviert ist.
- Channel-Topic/Purpose-Metadaten werden als nicht vertrauenswürdiger Kontext behandelt und können in den Routing-Kontext eingespeist werden.

## Reagieren + Reaktionen auflisten

`ackReaction` sendet ein Bestätigungs-Emoji, während OpenClaw eine eingehende Nachricht verarbeitet.

Auflösungsreihenfolge:

- Für
  Multi-Account setzen Sie `channels.slack.accounts.<id>.ackReaction`
- `channels.slack.ackReaction`
- `replyToMode`
- Agenten-Identitäts-Emoji-Fallback (`agents.list[].identity.emoji`, sonst "👀")

Hinweise

- Slack erwartet Shortcodes (zum Beispiel `"eyes"`).
- Verwenden Sie `""`, um die Reaktion für einen Channel oder Account zu deaktivieren.

## Manifest- und Scope-Checkliste

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

  <Accordion title="Optional user-token scopes (read operations)">    Wenn Sie `channels.slack.userToken` konfigurieren, sind typische Lese-Scopes:

    ```
    `channels:history`, `groups:history`, `im:history`, `mpim:history`
    [https://docs.slack.dev/reference/methods/conversations.history](https://docs.slack.dev/reference/methods/conversations.history)
    ```

  
</Accordion>
</AccordionGroup>

## Fehlerbehebung

<AccordionGroup>
  <Accordion title="No replies in channels">    Prüfen Sie der Reihe nach:

    ```
    `users`: optionale kanalweise Benutzer-Allowlist.
    ```

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

  
</Accordion>

  <Accordion title="DM messages ignored">    Prüfen:

    ```
    `allowlist` erfordert, dass Kanäle in `channels.slack.channels` aufgeführt sind.
    ```

```bash
openclaw pairing list slack
```

  
</Accordion>

  <Accordion title="Socket mode not connecting">Erstellen Sie eine Slack-App und aktivieren Sie den **Socket-Modus**.
</Accordion>

  <Accordion title="HTTP mode not receiving events">    Validieren:

    ```
    - Signing Secret
    - Webhook-Pfad
    - Slack-Request-URLs (Events + Interactivity + Slash Commands)
    - eindeutiger `webhookPath` pro HTTP-Account
    ```

  
</Accordion>

  <Accordion title="Native/slash commands not firing">    Überprüfen Sie, ob Sie beabsichtigt haben:

    ```
    Verbunden, aber keine Kanalantworten: Kanal durch `groupPolicy` blockiert oder nicht in der `channels.slack.channels`-Allowlist.
    ```

  
</Accordion>
</AccordionGroup>

## Konfigurationsreferenz-Hinweise

Priorität:

- [Configuration reference - Slack](/gateway/configuration-reference#slack)

  Slack-Felder mit hoher Relevanz:

  - mode/auth: `mode`, `botToken`, `appToken`, `signingSecret`, `webhookPath`, `accounts.*`
  - DM-Zugriff: `dm.enabled`, `dmPolicy`, `allowFrom` (Legacy: `dm.policy`, `dm.allowFrom`), `dm.groupEnabled`, `dm.groupChannels`
  - `allow`: Kanal erlauben/verbieten, wenn `groupPolicy="allowlist"`.
  - Fällt zurück auf `messages.groupChat.historyLimit`.
  - Zustellung: `textChunkLimit`, `chunkMode`, `mediaMaxMb`
  - Betrieb/Features: `configWrites`, `commands.native`, `slashCommand.*`, `actions.*`, `userToken`, `userTokenReadOnly`

## Verwandt

- [Pairing](/channels/pairing)
- `channel`: Standardkanäle (öffentlich/privat)
- Für den Triage-Ablauf: [/channels/troubleshooting](/channels/troubleshooting).
- [Configuration](/gateway/configuration)
- Vollständige Command-Liste + Konfiguration: [Slash commands](/tools/slash-commands)
