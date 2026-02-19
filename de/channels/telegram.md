---
summary: "Status, Funktionen und Konfiguration der Telegram-Bot-Unterstützung"
read_when:
  - Arbeiten an Telegram-Funktionen oder Webhooks
title: "Telegram"
---

# Telegram (Bot API)

Status: produktionsreif für Bot-DMs + Gruppen über grammY. Long-Polling standardmäßig; Webhook optional.

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Die Standard-DM-Richtlinie für Telegram ist Pairing.
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    Kanalübergreifende Diagnose- und Reparaturleitfäden.
  
</Card>
  <Card title="Gateway configuration" icon="settings" href="/gateway/configuration">
    Vollständige Kanal-Konfigurationsmuster und Beispiele.
  
</Card>
</CardGroup>

## Schnelleinrichtung

<Steps>
  <Step title="Create the bot token in BotFather">Öffnen Sie Telegram und chatten Sie mit **@BotFather** ([Direktlink](https://t.me/BotFather)). Bestätigen Sie, dass der Handle exakt `@BotFather` ist.

    ```
    Führen Sie `/newbot` aus und folgen Sie den Anweisungen (Name + Benutzername mit Endung `bot`).
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
    Env-Option: `TELEGRAM_BOT_TOKEN=...` (funktioniert für das Standardkonto).
    ```

  
</Step>

  <Step title="Start gateway and approve first DM">

```bash
`openclaw pairing approve telegram <CODE>`
```

    ```
    Pairing-Codes laufen nach 1 Stunde ab.
    ```

  
</Step>

  <Step title="Add the bot to a group">
    Füge den Bot zu deiner Gruppe hinzu und setze anschließend `channels.telegram.groups` und `groupPolicy` entsprechend deinem Zugriffsmodell.
  
</Step>
</Steps>

<Note>
Die Token-Auflösungsreihenfolge ist kontobewusst. In der Praxis haben Konfigurationswerte Vorrang vor dem Env-Fallback, und `TELEGRAM_BOT_TOKEN` gilt nur für das Standardkonto.
</Note>

## Telegram-seitige Einstellungen

<AccordionGroup>
  <Accordion title="Privacy mode and group visibility">**Bot sieht überhaupt keine Gruppennachrichten:**

    ```
    **Hinweis:** Wenn Sie den Privacy Mode umschalten, verlangt Telegram, den Bot
    aus jeder Gruppe zu entfernen und erneut hinzuzufügen, damit die Änderung wirksam wird.
    ```

  
</Accordion>

  <Accordion title="Group permissions">Der Admin-Status wird innerhalb der Gruppe (Telegram-UI) gesetzt.

    ```
    Den Bot als Gruppen-**Admin** hinzufügen (Admin-Bots erhalten alle Nachrichten).
    ```

  
</Accordion>

  <Accordion title="Helpful BotFather toggles">

    ```
    `/setjoingroups` — Hinzufügen des Bots zu Gruppen erlauben/verbieten.
    ```

  
</Accordion>
</AccordionGroup>

## Zugriffskontrolle und Aktivierung

<Tabs>
  <Tab title="DM policy">
    `channels.telegram.dmPolicy` steuert den Zugriff auf Direktnachrichten:


    ```
    `channels.telegram.allowFrom` akzeptiert numerische Benutzer-IDs (empfohlen) oder `@username`-Einträge. Es ist **nicht** der Bot-Benutzername; verwenden Sie die ID des menschlichen Absenders. Der Assistent akzeptiert `@username` und löst sie, wenn möglich, zur numerischen ID auf.
    ```

```bash
curl "https://api.telegram.org/bot<bot_token>/getUpdates"
```

    ```
    Drittanbieter-Methode (weniger privat): `@userinfobot` oder `@getidsbot`.
    ```

  
</Tab>

  <Tab title="Group policy and allowlists">Zwei unabhängige Kontrollen:

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

  <Tab title="Mention behavior">
    Gruppenantworten erfordern standardmäßig eine Erwähnung.


    ```
    Standardmäßig antwortet der Bot in Gruppen nur auf Erwähnungen (`@botname` oder Muster in `agents.list[].groupChat.mentionPatterns`). Um dieses Verhalten zu ändern:
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
    Leiten Sie eine beliebige Nachricht aus der Gruppe an `@userinfobot` oder `@getidsbot` auf Telegram weiter, um die Chat-ID zu sehen (negative Zahl wie `-1001234567890`).
    ```

  
</Tab>
</Tabs>

## Laufzeitverhalten

- Telegram wird vom Gateway-Prozess verwaltet.
- Deterministisches Routing: Antworten gehen zurück zu Telegram; das Modell wählt keine Kanäle.
- Eingehende Nachrichten werden in den gemeinsamen Kanal-Umschlag mit Antwortkontext und Medien-Platzhaltern normalisiert.
- Gruppensitzungen sind nach Gruppen-ID isoliert. Hängt `:topic:<threadId>` an den Sitzungs­schlüssel der Telegram-Gruppe an, sodass jedes Thema isoliert ist.
- Sendet Tippindikatoren und Antworten mit `message_thread_id`, damit Antworten im Thema bleiben.
- Long-Polling nutzt den grammY-Runner mit Sequenzierung pro Chat; die Gesamtkonkurrenz ist durch `agents.defaults.maxConcurrent` begrenzt. Die gesamte Runner-Sink-Konkurrenz verwendet `agents.defaults.maxConcurrent`.
- Die Telegram Bot API unterstützt keine Lesebestätigungen; es gibt keine Option `sendReadReceipts`.

## Funktionsreferenz

<AccordionGroup>
  <Accordion title="Live stream preview (message edits)">OpenClaw kann in Telegram-DMs partielle Antworten streamen, indem `sendMessageDraft` verwendet wird.

    ```
    Anforderung:
    
    - `channels.telegram.streamMode` ist nicht `"off"` (Standard: `"partial"`)
    
    Modi:
    
    - `off`: keine Live-Vorschau
    - `partial`: häufige Vorschau-Updates aus partiellem Text
    - `block`: blockweise Vorschau-Updates mithilfe von `channels.telegram.draftChunk`
    
    `draftChunk`-Standards für `streamMode: "block"`:
    
    - `minChars: 200`
    - `maxChars: 800`
    - `breakPreference: "paragraph"`
    
    `maxChars` wird durch `channels.telegram.textChunkLimit` begrenzt.
    
    Dies funktioniert in Direktchats sowie in Gruppen/Themen.
    
    Bei reinen Textantworten behält OpenClaw dieselbe Vorschau-Nachricht bei und führt am Ende eine Bearbeitung an Ort und Stelle durch (keine zweite Nachricht).
    
    Bei komplexen Antworten (z. B. Media-Payloads) greift OpenClaw auf die normale finale Zustellung zurück und bereinigt anschließend die Vorschau-Nachricht.
    
    `streamMode` ist unabhängig vom Block-Streaming. Wenn Block-Streaming für Telegram explizit aktiviert ist, überspringt OpenClaw den Vorschau-Stream, um doppeltes Streaming zu vermeiden.
    
    Telegram-spezifischer Reasoning-Stream:
    
    - `/reasoning stream` sendet das Reasoning während der Generierung an die Live-Vorschau
    - die endgültige Antwort wird ohne Reasoning-Text gesendet
    ```

  
</Accordion>

  <Accordion title="Formatting and HTML fallback">Ausgehender Telegram-Text verwendet `parse_mode: "HTML"` (Telegram-unterstützter Tag-Subset).

    ```
    - Markdown-ähnlicher Text wird in Telegram-sicheres HTML gerendert.
    - Rohes Modell-HTML wird maskiert, um Telegram-Parsing-Fehler zu reduzieren.
    - Wenn Telegram das geparste HTML ablehnt, versucht OpenClaw es erneut als Klartext.
    
    Link-Vorschauen sind standardmäßig aktiviert und können mit `channels.telegram.linkPreview: false` deaktiviert werden.
    ```

  
</Accordion>

  <Accordion title="Native commands and custom commands">Some commands can be handled by plugins/skills without being registered in Telegram’s command menu.

    ```
    OpenClaw registriert native Befehle (wie `/status`, `/reset`, `/model`) beim Start im Bot-Menü von Telegram. Sie können über die Konfiguration benutzerdefinierte Befehle zum Menü hinzufügen:
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
    Regeln:
    
    - Namen werden normalisiert (führendes `/` entfernen, kleinschreiben)
    - gültiges Muster: `a-z`, `0-9`, `_`, Länge `1..32`
    - benutzerdefinierte Befehle dürfen native Befehle nicht überschreiben
    - Konflikte/Duplikate werden übersprungen und protokolliert
    
    Hinweise:
    
    - benutzerdefinierte Befehle sind nur Menüeinträge; sie implementieren kein Verhalten automatisch
    - Plugin-/Skill-Befehle können weiterhin funktionieren, wenn sie eingegeben werden, auch wenn sie im Telegram-Menü nicht angezeigt werden
    
    Wenn native Befehle deaktiviert sind, werden integrierte Befehle entfernt. Benutzerdefinierte/Plugin-Befehle können weiterhin registriert werden, wenn sie konfiguriert sind.
    
    Häufiger Einrichtungsfehler:
    
    - `setMyCommands failed` bedeutet in der Regel, dass ausgehendes DNS/HTTPS zu `api.telegram.org` blockiert ist.
    
    ### Device-Pairing-Befehle (`device-pair` Plugin)
    
    Wenn das `device-pair` Plugin installiert ist:
    
    1. `/pair` erzeugt einen Einrichtungscode
    2. Code in die iOS-App einfügen
    3. `/pair approve` genehmigt die zuletzt ausstehende Anfrage
    
    Weitere Details: [Pairing](/channels/pairing#pair-via-telegram-recommended-for-ios).
    ```

  
</Accordion>

  <Accordion title="Inline buttons">
    Inline-Keyboard-Bereich konfigurieren:


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
    Pro-Konto-Überschreibung:
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
    `allowlist` — DMs + Gruppen, aber nur Absender, die durch `allowFrom`/`groupAllowFrom` erlaubt sind (gleiche Regeln wie Steuerbefehle)
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
    Wenn ein Benutzer einen Button anklickt, werden die Callback-Daten als Nachricht mit folgendem Format an den Agenten zurückgesendet:
    `callback_data: value`
    ```

  
</Accordion>

  <Accordion title="Telegram message actions for agents and automation">
    Telegram-Tool-Aktionen umfassen:


    ```
    Werkzeug: `telegram` mit Aktion `react` (`chatId`, `messageId`, `emoji`).
    ```

  
</Accordion>

  <Accordion title="Reply threading tags">Telegram unterstützt optionales Threading von Antworten über Tags:

    ```
    - `[[reply_to_current]]` antwortet auf die auslösende Nachricht
    - `[[reply_to:<id>]]` antwortet auf eine bestimmte Telegram-Nachrichten-ID
    
    `channels.telegram.replyToMode` steuert die Verarbeitung:
    
    - `off` (Standard)
    - `first`
    - `all`
    
    Hinweis: `off` deaktiviert implizites Antwort-Threading. Explizite `[[reply_to_*]]`-Tags werden weiterhin berücksichtigt.
    ```

  
</Accordion>

  <Accordion title="Forum topics and thread behavior">Themen (Forum-Supergroups)

    ```
    Allgemeines Thema (Thread-ID `1`) ist speziell: Nachrichtensendungen lassen `message_thread_id` weg (Telegram lehnt es ab), Tippindikatoren enthalten es weiterhin.
    ```

  
</Accordion>

  <Accordion title="Audio, video, and stickers">
    ### Audio-Nachrichten


    ```
    `[[audio_as_voice]]` — Audio als Sprachnotiz statt als Datei senden.
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
    Videonotizen unterstützen keine Beschriftungen; bereitgestellter Nachrichtentext wird separat gesendet.
    
    ### Sticker
    
    Eingehende Sticker-Verarbeitung:
    
    - statisches WEBP: wird heruntergeladen und verarbeitet (Platzhalter `<media:sticker>`)
    - animiertes TGS: wird übersprungen
    - Video-WEBM: wird übersprungen
    
    Sticker-Kontextfelder:
    
    - `Sticker.emoji`
    - `Sticker.setName`
    - `Sticker.fileId`
    - `Sticker.fileUniqueId`
    - `Sticker.cachedDescription`
    
    Sticker-Cache-Datei:
    
    - `~/.openclaw/telegram/sticker-cache.json`
    
    Sticker werden (wenn möglich) einmal beschrieben und zwischengespeichert, um wiederholte Vision-Aufrufe zu reduzieren.
    
    Sticker-Aktionen aktivieren:
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
    Sticker senden
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
    Sticker-Cache
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

  <Accordion title="Reaction notifications">Empfang des `message_reaction`-Updates von der Telegram API

    ```
    Wenn aktiviert, reiht OpenClaw Systemereignisse ein wie:
    
    - `Telegram reaction added: 👍 by Alice (@alice) on msg 42`
    
    Konfiguration:
    
    - `channels.telegram.reactionNotifications`: `off | own | all` (Standard: `own`)
    - `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` (Standard: `minimal`)
    
    Hinweise:
    
    - `own` bedeutet Benutzerreaktionen nur auf vom Bot gesendete Nachrichten (Best-Effort über gesendete-Nachrichten-Cache).
    - Telegram stellt in Reaktions-Updates keine Thread-IDs bereit.
      - Nicht-Forum-Gruppen werden zur Gruppenchat-Sitzung geleitet
      - Forum-Gruppen werden zur allgemeinen Gruppen-Topic-Sitzung (`:topic:1`) geleitet, nicht zum exakt ursprünglichen Topic
    
    `allowed_updates` für Polling/Webhook enthalten automatisch `message_reaction`.
    ```

  
</Accordion>

  <Accordion title="Ack reactions">
    `ackReaction` sendet ein Bestätigungs-Emoji, während OpenClaw eine eingehende Nachricht verarbeitet.


    ```
    Auflösungsreihenfolge:
    
    - `channels.telegram.accounts.<accountId>.ackReaction`
    - `channels.telegram.ackReaction`
    - `messages.ackReaction`
    - Agent-Identitäts-Emoji-Fallback (`agents.list[].identity.emoji`, sonst "👀")
    
    Hinweise:
    
    - Telegram erwartet Unicode-Emoji (zum Beispiel "👀").
    - Verwende `""`, um die Reaktion für einen Kanal oder ein Konto zu deaktivieren.
    ```

  
</Accordion>

  <Accordion title="Config writes from Telegram events and commands">
    Kanal-Konfigurationsschreibvorgänge sind standardmäßig aktiviert (`configWrites !== false`).


    ```
    Telegram-ausgelöste Schreibvorgänge umfassen:
    
    - Gruppenmigrationsereignisse (`migrate_to_chat_id`) zur Aktualisierung von `channels.telegram.groups`
    - `/config set` und `/config unset` (erfordert Aktivierung der Befehle)
    
    Deaktivieren:
    ```

```json5
{
  channels: { telegram: { configWrites: false } },
}
```

  
</Accordion>

  <Accordion title="Long polling vs webhook">
    Standard: Long Polling.


    ```
    Wenn Ihre öffentliche URL abweicht, verwenden Sie einen Reverse Proxy und zeigen Sie `channels.telegram.webhookUrl` auf den öffentlichen Endpunkt.
    ```

  
</Accordion>

  <Accordion title="Limits, retry, and CLI targets">
    Ausgehender Text wird auf `channels.telegram.textChunkLimit` segmentiert (Standard 4000).
    Optionales Newline-Chunking: Setzen Sie `channels.telegram.chunkMode="newline"`, um vor der Längen-Segmentierung an Leerzeilen (Absatzgrenzen) zu teilen.
    Medien-Downloads/Uploads sind durch `channels.telegram.mediaMaxMb` begrenzt (Standard 5).
    Telegram Bot API-Anfragen laufen nach `channels.telegram.timeoutSeconds` ab (Standard 500 über grammY).
    Gruppenverlaufs-Kontext nutzt `channels.telegram.historyLimit` (oder `channels.telegram.accounts.*.historyLimit`) und fällt auf `messages.groupChat.historyLimit` zurück. Setzen Sie `0`, um zu deaktivieren (Standard 50).
    DM-Verlauf kann mit `channels.telegram.dmHistoryLimit` begrenzt werden (Benutzer-Turns). Pro-Benutzer-Overrides: `channels.telegram.dms["<user_id>"].historyLimit`.<user_id>Standardmäßig darf Telegram durch Kanalereignisse oder `/config set|unset` ausgelöste Konfigurationsupdates schreiben.

    ```
    CLI-Sendeziel kann eine numerische Chat-ID oder ein Benutzername sein:
    ```

```bash
Beispiel: `openclaw message send --channel telegram --target 123456789 --message "hi"`.
```

  
</Accordion>
</AccordionGroup>

## Fehlerbehebung

<AccordionGroup>
  <Accordion title="Bot does not respond to non mention group messages">

    ```
    Wenn Sie `channels.telegram.groups.*.requireMention=false` gesetzt haben, muss der **Privacy Mode** der Telegram Bot API deaktiviert sein.
    ```

  
</Accordion>

  <Accordion title="Bot not seeing group messages at all">

    ```
    Wenn `channels.telegram.groups` gesetzt ist, muss die Gruppe gelistet sein oder `"*"` verwenden
    ```

  
</Accordion>

  <Accordion title="Commands work partially or not at all">

    ```
    `setMyCommands failed` in Logs bedeutet meist, dass ausgehendes HTTPS/DNS zu `api.telegram.org` blockiert ist.
    ```

  
</Accordion>

  <Accordion title="Polling or network instability">

    ```
    - Node 22+ + benutzerdefiniertes fetch/Proxy kann ein sofortiges Abbruchverhalten auslösen, wenn AbortSignal-Typen nicht übereinstimmen.
    - Einige Hosts lösen `api.telegram.org` zuerst zu IPv6 auf; fehlerhafter IPv6-Egress kann zu intermittierenden Telegram-API-Fehlern führen.
    - DNS-Antworten überprüfen:
    ```

```bash
Schnellcheck: `dig +short api.telegram.org A` und `dig +short api.telegram.org AAAA`, um zu bestätigen, was DNS zurückgibt.
```

  
</Accordion>
</AccordionGroup>

Weitere Hilfe: [Channel troubleshooting](/channels/troubleshooting).

## Konfigurationsreferenz (Telegram)

Primäre Referenz:

- `channels.telegram.enabled`: Kanalstart aktivieren/deaktivieren.

- `channels.telegram.botToken`: Bot-Token (BotFather).

- `channels.telegram.tokenFile`: Token aus Dateipfad lesen.

- `channels.telegram.dmPolicy`: `pairing | allowlist | open | disabled` (Standard: Pairing).

- `channels.telegram.allowFrom`: DM-Allowlist (IDs/Benutzernamen). `open` erfordert `"*"`. `openclaw doctor --fix` kann veraltete `@username`-Einträge in IDs auflösen.

- `channels.telegram.groupPolicy`: `open | allowlist | disabled` (Standard: Allowlist).

- `channels.telegram.groupAllowFrom`: Gruppen-Absender-Allowlist (IDs/Benutzernamen). `openclaw doctor --fix` kann veraltete `@username`-Einträge in IDs auflösen.

- `channels.telegram.groups`: Gruppen-Standards + Allowlist (verwenden Sie `"*"` für globale Defaults).
  - `channels.telegram.groups.<id>.groupPolicy`: Gruppen­spezifisches Override für groupPolicy (`open | allowlist | disabled`).
  - `channels.telegram.groups.<id>.requireMention`: Standard für Mention-Gating.
  - `channels.telegram.groups.<id>.skills`: Skill-Filter (weglassen = alle Skills, leer = keine).
  - `channels.telegram.groups.<id>.allowFrom`: Gruppen­spezifisches Absender-Allowlist-Override.
  - `channels.telegram.groups.<id>.systemPrompt`: Zusätzliches System-Prompt für die Gruppe.
  - `channels.telegram.groups.<id>.enabled`: Deaktiviert die Gruppe, wenn `false`.
  - .topics.<threadId>`channels.telegram.groups.<id>.*`: Themen­spezifische Overrides (gleiche Felder wie Gruppe).
  - `channels.telegram.groups.<id>.topics.<threadId>.groupPolicy`: Themen­spezifisches Override für groupPolicy (`open | allowlist | disabled`).
  - .topics.<threadId>`channels.telegram.groups.<id>.requireMention`: Themen­spezifisches Mention-Gating-Override.

- `channels.telegram.capabilities.inlineButtons`: `off | dm | group | all | allowlist` (Standard: Allowlist).

- `channels.telegram.accounts.<account>.capabilities.inlineButtons`: Konto­spezifisches Override.

- `channels.telegram.replyToMode`: `off | first | all` (Standard: `first`).

- `channels.telegram.textChunkLimit`: Ausgehende Chunk-Größe (Zeichen).

- `channels.telegram.chunkMode`: `length` (Standard) oder `newline`, um vor der Längen-Segmentierung an Leerzeilen (Absatzgrenzen) zu teilen.

- `channels.telegram.linkPreview`: Link-Vorschauen für ausgehende Nachrichten umschalten (Standard: true).

- `channels.telegram.streamMode`: `off | partial | block` (Entwurfs-Streaming).

- `channels.telegram.mediaMaxMb`: Eingehende/ausgehende Medienbegrenzung (MB).

- `channels.telegram.retry`: Wiederholungsrichtlinie für ausgehende Telegram-API-Aufrufe (Versuche, minDelayMs, maxDelayMs, Jitter).

- `channels.telegram.network.autoSelectFamily`: Override für Node autoSelectFamily (true=aktivieren, false=deaktivieren). Standardmäßig auf Node 22 deaktiviert, um Happy-Eyeballs-Timeouts zu vermeiden.

- `channels.telegram.proxy`: Proxy-URL für Bot-API-Aufrufe (SOCKS/HTTP).

- `channels.telegram.webhookUrl`: Webhook-Modus aktivieren (erfordert `channels.telegram.webhookSecret`).

- `channels.telegram.webhookSecret`: Webhook-Secret (erforderlich, wenn webhookUrl gesetzt ist).

- `channels.telegram.webhookPath`: Lokaler Webhook-Pfad (Standard `/telegram-webhook`).

- Der lokale Listener bindet an `0.0.0.0:8787` und stellt standardmäßig `POST /telegram-webhook` bereit.

- `channels.telegram.actions.reactions`: Telegram-Werkzeugreaktionen steuern.

- `channels.telegram.actions.sendMessage`: Telegram-Werkzeug-Nachrichtensendungen steuern.

- `channels.telegram.actions.deleteMessage`: Telegram-Werkzeug-Nachrichtenlöschungen steuern.

- `channels.telegram.actions.sticker`: Telegram-Sticker-Aktionen steuern — Senden und Suchen (Standard: false).

- `channels.telegram.reactionNotifications`: `off | own | all` — steuert, welche Reaktionen Systemereignisse auslösen (Standard: `own`, wenn nicht gesetzt).

- `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` — steuert die Reaktionsfähigkeit des Agenten (Standard: `minimal`, wenn nicht gesetzt).

- Vollständige Konfiguration: [Konfiguration](/gateway/configuration)

Telegram-spezifische High-Signal-Felder:

- startup/auth: `enabled`, `botToken`, `tokenFile`, `accounts.*`
- Befehle erfordern Autorisierung, selbst in Gruppen mit `groupPolicy: "open"`
- command/menu: `commands.native`, `customCommands`
- threading/replies: `replyToMode`
- Optional (nur für `streamMode: "block"`):
- formatting/delivery: `textChunkLimit`, `chunkMode`, `linkPreview`, `responsePrefix`
- media/network: `mediaMaxMb`, `timeoutSeconds`, `retry`, `network.autoSelectFamily`, `proxy`
- Webhook-Modus: Setzen Sie `channels.telegram.webhookUrl` und `channels.telegram.webhookSecret` (optional `channels.telegram.webhookPath`).
- actions/capabilities: `capabilities.inlineButtons`, `actions.sendMessage|editMessage|deleteMessage|reactions|sticker`
- Reaktionsbenachrichtigungen
- writes/history: `configWrites`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`

## Verwandt

- Details: [Pairing](/channels/pairing)
- More details: [Pairing](/channels/pairing#pair-via-telegram-recommended-for-ios).
- Setup-Fehlerbehebung (Befehle)

