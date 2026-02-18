---
title: "Telegram"
---

# Telegram (Bot API)

Status: produktionsreif für Bot-DMs + Gruppen über grammY. Long-Polling standardmäßig; Webhook optional.

## Schnellstart (Einsteiger)

1. Erstellen Sie einen Bot mit **@BotFather** ([Direktlink](https://t.me/BotFather)). Bestätigen Sie, dass der Handle exakt `@BotFather` ist, und kopieren Sie anschließend das Token.
2. Setzen Sie das Token:
   - Env: `TELEGRAM_BOT_TOKEN=...`
   - Oder Konfiguration: `channels.telegram.botToken: "..."`.
   - Wenn beide gesetzt sind, hat die Konfiguration Vorrang (Env-Fallback gilt nur für das Standardkonto).
3. Starten Sie das Gateway.
4. DM-Zugriff ist standardmäßig per Pairing aktiviert; genehmigen Sie beim ersten Kontakt den Pairing-Code.

Minimale Konfiguration:

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

## Was es ist

- Ein Telegram Bot API-Kanal, der dem Gateway gehört.
- Deterministisches Routing: Antworten gehen zurück zu Telegram; das Modell wählt keine Kanäle.
- DMs teilen sich die Hauptsitzung des Agenten; Gruppen bleiben isoliert (`agent:<agentId>:telegram:group:<chatId>`).

## Einrichtung (Schnellpfad)

### 1. Bot-Token erstellen (BotFather)

1. Öffnen Sie Telegram und chatten Sie mit **@BotFather** ([Direktlink](https://t.me/BotFather)). Bestätigen Sie, dass der Handle exakt `@BotFather` ist.
2. Führen Sie `/newbot` aus und folgen Sie den Anweisungen (Name + Benutzername mit Endung `bot`).
3. Kopieren Sie das Token und bewahren Sie es sicher auf.

Optionale BotFather-Einstellungen:

- `/setjoingroups` — Hinzufügen des Bots zu Gruppen erlauben/verbieten.
- `/setprivacy` — Steuern, ob der Bot alle Gruppennachrichten sieht.

### 2. Token konfigurieren (Env oder Konfiguration)

Beispiel:

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

Env-Option: `TELEGRAM_BOT_TOKEN=...` (funktioniert für das Standardkonto).
Wenn Env und Konfiguration gesetzt sind, hat die Konfiguration Vorrang.

Multi-Account-Unterstützung: Verwenden Sie `channels.telegram.accounts` mit konto­spezifischen Tokens und optional `name`. Siehe [`gateway/configuration`](/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) für das gemeinsame Muster.

3. Starten Sie das Gateway. Telegram startet, sobald ein Token aufgelöst ist (zuerst Konfiguration, dann Env-Fallback).
4. DM-Zugriff ist standardmäßig Pairing. Genehmigen Sie den Code beim ersten Kontakt mit dem Bot.
5. Für Gruppen: Fügen Sie den Bot hinzu, entscheiden Sie über Datenschutz-/Admin-Verhalten (unten) und setzen Sie dann `channels.telegram.groups`, um Mention-Gating + Allowlists zu steuern.

## Token + Datenschutz + Berechtigungen (Telegram-Seite)

### Token-Erstellung (BotFather)

- `/newbot` erstellt den Bot und gibt das Token zurück (geheim halten).
- Wenn ein Token kompromittiert wird, widerrufen/regenerieren Sie es über @BotFather und aktualisieren Sie Ihre Konfiguration.

### Sichtbarkeit von Gruppennachrichten (Privacy Mode)

Telegram-Bots verwenden standardmäßig den **Privacy Mode**, der einschränkt, welche Gruppennachrichten sie empfangen.
Wenn Ihr Bot _alle_ Gruppennachrichten sehen muss, haben Sie zwei Optionen:

- Privacy Mode mit `/setprivacy` deaktivieren **oder**
- Den Bot als Gruppen-**Admin** hinzufügen (Admin-Bots erhalten alle Nachrichten).

**Hinweis:** Wenn Sie den Privacy Mode umschalten, verlangt Telegram, den Bot
aus jeder Gruppe zu entfernen und erneut hinzuzufügen, damit die Änderung wirksam wird.

### Gruppenberechtigungen (Admin-Rechte)

Der Admin-Status wird innerhalb der Gruppe (Telegram-UI) gesetzt. Admin-Bots erhalten immer alle
Gruppennachrichten; verwenden Sie Admin, wenn Sie volle Sichtbarkeit benötigen.

## Funktionsweise (Verhalten)

- Eingehende Nachrichten werden in den gemeinsamen Kanal-Umschlag mit Antwortkontext und Medien-Platzhaltern normalisiert.
- Gruppenantworten erfordern standardmäßig eine Erwähnung (native @Erwähnung oder `agents.list[].groupChat.mentionPatterns` / `messages.groupChat.mentionPatterns`).
- Multi-Agent-Override: Setzen Sie agentenspezifische Muster auf `agents.list[].groupChat.mentionPatterns`.
- Antworten werden immer in denselben Telegram-Chat zurückgeroutet.
- Long-Polling nutzt den grammY-Runner mit Sequenzierung pro Chat; die Gesamtkonkurrenz ist durch `agents.defaults.maxConcurrent` begrenzt.
- Die Telegram Bot API unterstützt keine Lesebestätigungen; es gibt keine Option `sendReadReceipts`.

## Entwurfs-Streaming

OpenClaw kann in Telegram-DMs partielle Antworten streamen, indem `sendMessageDraft` verwendet wird.

Voraussetzungen:

- Threaded Mode für den Bot in @BotFather aktiviert (Forum-Topic-Modus).
- Nur private Chat-Threads (Telegram enthält `message_thread_id` in eingehenden Nachrichten).
- `channels.telegram.streamMode` nicht auf `"off"` gesetzt (Standard: `"partial"`; `"block"` aktiviert chunked Draft-Updates).

Entwurfs-Streaming ist nur für DMs verfügbar; Telegram unterstützt es nicht in Gruppen oder Kanälen.

## Formatierung (Telegram HTML)

- Ausgehender Telegram-Text verwendet `parse_mode: "HTML"` (Telegram-unterstützter Tag-Subset).
- Markdown-ähnliche Eingaben werden in **Telegram-sicheres HTML** gerendert (fett/kursiv/durchgestrichen/Code/Links); Block-Elemente werden zu Text mit Zeilenumbrüchen/Aufzählungen abgeflacht.
- Rohes HTML aus Modellen wird escaped, um Telegram-Parsefehler zu vermeiden.
- Wenn Telegram die HTML-Payload ablehnt, versucht OpenClaw dieselbe Nachricht erneut als Klartext zu senden.

## Befehle (nativ + benutzerdefiniert)

OpenClaw registriert native Befehle (wie `/status`, `/reset`, `/model`) beim Start im Bot-Menü von Telegram.
Sie können über die Konfiguration benutzerdefinierte Befehle zum Menü hinzufügen:

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

## Setup-Fehlerbehebung (Befehle)

- `setMyCommands failed` in Logs bedeutet meist, dass ausgehendes HTTPS/DNS zu `api.telegram.org` blockiert ist.
- Wenn Sie Fehler `sendMessage` oder `sendChatAction` sehen, prüfen Sie IPv6-Routing und DNS.

Weitere Hilfe: [Channel troubleshooting](/channels/troubleshooting).

Hinweise:

- Benutzerdefinierte Befehle sind **nur Menüeinträge**; OpenClaw implementiert sie nicht, sofern Sie sie nicht anderweitig behandeln.
- Some commands can be handled by plugins/skills without being registered in Telegram’s command menu. These still work when typed (they just won't show up in `/commands` / the menu).
- Befehlsnamen werden normalisiert (führendes `/` entfernt, kleingeschrieben) und müssen `a-z`, `0-9`, `_` entsprechen (1–32 Zeichen).
- Benutzerdefinierte Befehle **können native Befehle nicht überschreiben**. Konflikte werden ignoriert und protokolliert.
- Wenn `commands.native` deaktiviert ist, werden nur benutzerdefinierte Befehle registriert (oder gelöscht, falls keine vorhanden sind).

### Device pairing commands (`device-pair` plugin)

If the `device-pair` plugin is installed, it adds a Telegram-first flow for pairing a new phone:

1. `/pair` generates a setup code (sent as a separate message for easy copy/paste).
2. Paste the setup code in the iOS app to connect.
3. `/pair approve` approves the latest pending device request.

More details: [Pairing](/channels/pairing#pair-via-telegram-recommended-for-ios).

## Limits

- Ausgehender Text wird auf `channels.telegram.textChunkLimit` segmentiert (Standard 4000).
- Optionales Newline-Chunking: Setzen Sie `channels.telegram.chunkMode="newline"`, um vor der Längen-Segmentierung an Leerzeilen (Absatzgrenzen) zu teilen.
- Medien-Downloads/Uploads sind durch `channels.telegram.mediaMaxMb` begrenzt (Standard 5).
- Telegram Bot API-Anfragen laufen nach `channels.telegram.timeoutSeconds` ab (Standard 500 über grammY). Setzen Sie niedrigere Werte, um lange Hänger zu vermeiden.
- Gruppenverlaufs-Kontext nutzt `channels.telegram.historyLimit` (oder `channels.telegram.accounts.*.historyLimit`) und fällt auf `messages.groupChat.historyLimit` zurück. Setzen Sie `0`, um zu deaktivieren (Standard 50).
- DM-Verlauf kann mit `channels.telegram.dmHistoryLimit` begrenzt werden (Benutzer-Turns). Pro-Benutzer-Overrides: `channels.telegram.dms["<user_id>"].historyLimit`.

## Gruppenaktivierungsmodi

Standardmäßig antwortet der Bot in Gruppen nur auf Erwähnungen (`@botname` oder Muster in `agents.list[].groupChat.mentionPatterns`). Um dieses Verhalten zu ändern:

### Über Konfiguration (empfohlen)

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

**Wichtig:** Das Setzen von `channels.telegram.groups` erstellt eine **Allowlist** – nur aufgelistete Gruppen (oder `"*"`) werden akzeptiert.
Forum-Themen erben die Konfiguration ihrer übergeordneten Gruppe (allowFrom, requireMention, Skills, Prompts), sofern Sie keine topic-spezifischen Overrides unter `channels.telegram.groups.<groupId>.topics.<topicId>` hinzufügen.

Alle Gruppen mit immer-antworten erlauben:

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

Mention-only für alle Gruppen beibehalten (Standardverhalten):

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

### Über Befehl (Sitzungsebene)

Senden Sie in der Gruppe:

- `/activation always` – auf alle Nachrichten antworten
- `/activation mention` – Erwähnungen erforderlich (Standard)

**Hinweis:** Befehle aktualisieren nur den Sitzungszustand. Für persistentes Verhalten über Neustarts hinweg verwenden Sie die Konfiguration.

### Gruppen-Chat-ID ermitteln

Leiten Sie eine beliebige Nachricht aus der Gruppe an `@userinfobot` oder `@getidsbot` auf Telegram weiter, um die Chat-ID zu sehen (negative Zahl wie `-1001234567890`).

**Tipp:** Für Ihre eigene Benutzer-ID senden Sie dem Bot eine DM; er antwortet mit Ihrer Benutzer-ID (Pairing-Nachricht), oder verwenden Sie `/whoami`, sobald Befehle aktiviert sind.

**Datenschutzhinweis:** `@userinfobot` ist ein Drittanbieter-Bot. Alternativ fügen Sie den Bot der Gruppe hinzu, senden eine Nachricht und verwenden `openclaw logs --follow`, um `chat.id` zu lesen, oder nutzen Sie die Bot API `getUpdates`.

## Konfigurationsschreibzugriffe

Standardmäßig darf Telegram durch Kanalereignisse oder `/config set|unset` ausgelöste Konfigurationsupdates schreiben.

Dies geschieht, wenn:

- Eine Gruppe zu einer Supergroup aufgewertet wird und Telegram `migrate_to_chat_id` sendet (Chat-ID ändert sich). OpenClaw kann `channels.telegram.groups` automatisch migrieren.
- Sie `/config set` oder `/config unset` in einem Telegram-Chat ausführen (erfordert `commands.config: true`).

Deaktivieren mit:

```json5
{
  channels: { telegram: { configWrites: false } },
}
```

## Themen (Forum-Supergroups)

Telegram-Forum-Themen enthalten pro Nachricht eine `message_thread_id`. OpenClaw:

- Hängt `:topic:<threadId>` an den Sitzungs­schlüssel der Telegram-Gruppe an, sodass jedes Thema isoliert ist.
- Sendet Tippindikatoren und Antworten mit `message_thread_id`, damit Antworten im Thema bleiben.
- Allgemeines Thema (Thread-ID `1`) ist speziell: Nachrichtensendungen lassen `message_thread_id` weg (Telegram lehnt es ab), Tippindikatoren enthalten es weiterhin.
- Stellt `MessageThreadId` + `IsForum` im Template-Kontext für Routing/Templating bereit.
- Themen­spezifische Konfiguration ist unter `channels.telegram.groups.<chatId>.topics.<threadId>` verfügbar (Skills, Allowlists, Auto-Reply, System-Prompts, Deaktivieren).
- Themen­konfigurationen erben Gruppeneinstellungen (requireMention, Allowlists, Skills, Prompts, Enabled), sofern nicht pro Thema überschrieben.

Private Chats können in einigen Randfällen `message_thread_id` enthalten. OpenClaw behält den DM-Sitzungs­schlüssel unverändert bei, nutzt die Thread-ID jedoch weiterhin für Antworten/Entwurfs-Streaming, wenn vorhanden.

## Inline-Buttons

Telegram unterstützt Inline-Keyboards mit Callback-Buttons.

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

Für konto­spezifische Konfiguration:

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

Genehmigen über:

- `off` — Inline-Buttons deaktiviert
- `dm` — nur DMs (Gruppen-Ziele blockiert)
- `group` — nur Gruppen (DM-Ziele blockiert)
- `all` — DMs + Gruppen
- `allowlist` — DMs + Gruppen, aber nur Absender, die durch `allowFrom`/`groupAllowFrom` erlaubt sind (gleiche Regeln wie Steuerbefehle)

Standard: `allowlist`.
Legacy: `capabilities: ["inlineButtons"]` = `inlineButtons: "all"`.

### Buttons senden

Verwenden Sie das Nachrichten-Werkzeug mit dem Parameter `buttons`:

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

Wenn ein Benutzer einen Button anklickt, werden die Callback-Daten als Nachricht mit folgendem Format an den Agenten zurückgesendet:
`callback_data: value`

### Konfigurationsoptionen

Telegram-Fähigkeiten können auf zwei Ebenen konfiguriert werden (Objektform oben gezeigt; Legacy-String-Arrays werden weiterhin unterstützt):

- `channels.telegram.capabilities`: Globale Standard-Fähigkeitskonfiguration, die auf alle Telegram-Konten angewendet wird, sofern nicht überschrieben.
- `channels.telegram.accounts.<account>.capabilities`: Konto­spezifische Fähigkeiten, die die globalen Standards für dieses Konto überschreiben.

Verwenden Sie die globale Einstellung, wenn sich alle Telegram-Bots/Konten gleich verhalten sollen. Verwenden Sie konto­spezifische Konfiguration, wenn unterschiedliche Bots unterschiedliche Verhaltensweisen benötigen (z. B. ein Konto nur DMs, ein anderes auch Gruppen).

## Zugriffskontrolle (DMs + Gruppen)

### DM-Zugriff

- Standard: `channels.telegram.dmPolicy = "pairing"`. Unbekannte Absender erhalten einen Pairing-Code; Nachrichten werden ignoriert, bis sie genehmigt sind (Codes verfallen nach 1 Stunde).
- Genehmigung über:
  - `openclaw pairing list telegram`
  - `openclaw pairing approve telegram <CODE>`
- Pairing ist der Standard-Tokenaustausch für Telegram-DMs. Details: [Pairing](/channels/pairing)
- `channels.telegram.allowFrom` akzeptiert numerische Benutzer-IDs (empfohlen) oder `@username`-Einträge. Es ist **nicht** der Bot-Benutzername; verwenden Sie die ID des menschlichen Absenders. Der Assistent akzeptiert `@username` und löst sie, wenn möglich, zur numerischen ID auf.

#### Ihre Telegram-Benutzer-ID finden

Sicherer (kein Drittanbieter-Bot):

1. Starten Sie das Gateway und senden Sie Ihrem Bot eine DM.
2. Führen Sie `openclaw logs --follow` aus und suchen Sie nach `from.id`.

Alternative (offizielle Bot API):

1. Senden Sie Ihrem Bot eine DM.
2. Rufen Sie Updates mit Ihrem Bot-Token ab und lesen Sie `message.from.id`:

   ```bash
   curl "https://api.telegram.org/bot<bot_token>/getUpdates"
   ```

Drittanbieter (weniger privat):

- Senden Sie eine DM an `@userinfobot` oder `@getidsbot` und verwenden Sie die zurückgegebene Benutzer-ID.

### Gruppenzugriff

Zwei unabhängige Kontrollen:

**1. Welche Gruppen erlaubt sind** (Gruppen-Allowlist über `channels.telegram.groups`):

- Keine `groups`-Konfiguration = alle Gruppen erlaubt
- Mit `groups`-Konfiguration = nur aufgelistete Gruppen oder `"*"` sind erlaubt
- Beispiel: `"groups": { "-1001234567890": {}, "*": {} }` erlaubt alle Gruppen

**2. Welche Absender erlaubt sind** (Absenderfilterung über `channels.telegram.groupPolicy`):

- `"open"` = alle Absender in erlaubten Gruppen können schreiben
- `"allowlist"` = nur Absender in `channels.telegram.groupAllowFrom` können schreiben
- `"disabled"` = keine Gruppennachrichten werden akzeptiert
  Standard ist `groupPolicy: "allowlist"` (blockiert, sofern Sie nicht `groupAllowFrom` hinzufügen).

Die meisten Nutzer wollen: `groupPolicy: "allowlist"` + `groupAllowFrom` + spezifische Gruppen in `channels.telegram.groups` gelistet

Um **jedem Gruppenmitglied** in einer bestimmten Gruppe das Schreiben zu erlauben (während Steuerbefehle weiterhin auf autorisierte Absender beschränkt bleiben), setzen Sie ein gruppen­spezifisches Override:

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

## Long-Polling vs. Webhook

- Standard: Long-Polling (keine öffentliche URL erforderlich).
- Webhook-Modus: Setzen Sie `channels.telegram.webhookUrl` und `channels.telegram.webhookSecret` (optional `channels.telegram.webhookPath`).
  - Der lokale Listener bindet an `0.0.0.0:8787` und stellt standardmäßig `POST /telegram-webhook` bereit.
  - Wenn Ihre öffentliche URL abweicht, verwenden Sie einen Reverse Proxy und zeigen Sie `channels.telegram.webhookUrl` auf den öffentlichen Endpunkt.

## Antwort-Threading

Telegram unterstützt optionales Threading von Antworten über Tags:

- `[[reply_to_current]]` – Antwort auf die auslösende Nachricht.
- `[[reply_to:<id>]]` – Antwort auf eine bestimmte Nachrichten-ID.

Gesteuert durch `channels.telegram.replyToMode`:

- `first` (Standard), `all`, `off`.

## Audionachrichten (Sprachnotiz vs. Datei)

Telegram unterscheidet **Sprachnotizen** (runde Blase) von **Audiodateien** (Metadatenkarte).
OpenClaw verwendet aus Gründen der Abwärtskompatibilität standardmäßig Audiodateien.

Um in Agentenantworten eine Sprachnotiz-Blase zu erzwingen, fügen Sie dieses Tag irgendwo in der Antwort ein:

- `[[audio_as_voice]]` — Audio als Sprachnotiz statt als Datei senden.

Das Tag wird aus dem ausgelieferten Text entfernt. Andere Kanäle ignorieren dieses Tag.

Für Sends über das Nachrichten-Werkzeug setzen Sie `asVoice: true` mit einer sprach­kompatiblen Audio-`media`-URL
(`message` ist optional, wenn Medien vorhanden sind):

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

## Sticker

OpenClaw unterstützt das Empfangen und Senden von Telegram-Stickern mit intelligenter Zwischenspeicherung.

### Sticker empfangen

Wenn ein Benutzer einen Sticker sendet, verarbeitet OpenClaw ihn je nach Stickertyp:

- **Statische Sticker (WEBP):** Heruntergeladen und über Vision verarbeitet. Der Sticker erscheint als `<media:sticker>`-Platzhalter im Nachrichteninhalt.
- **Animierte Sticker (TGS):** Übersprungen (Lottie-Format wird für die Verarbeitung nicht unterstützt).
- **Video-Sticker (WEBM):** Übersprungen (Videoformat wird für die Verarbeitung nicht unterstützt).

Verfügbares Template-Kontextfeld beim Empfangen von Stickern:

- `Sticker` — Objekt mit:
  - `emoji` — dem Sticker zugeordnetes Emoji
  - `setName` — Name des Sticker-Sets
  - `fileId` — Telegram-Datei-ID (denselben Sticker zurücksenden)
  - `fileUniqueId` — stabile ID für Cache-Lookup
  - `cachedDescription` — zwischengespeicherte Vision-Beschreibung, sofern verfügbar

### Sticker-Cache

Sticker werden über die Vision-Fähigkeiten der KI verarbeitet, um Beschreibungen zu erzeugen. Da dieselben Sticker häufig wiederholt gesendet werden, speichert OpenClaw diese Beschreibungen zwischen, um redundante API-Aufrufe zu vermeiden.

**So funktioniert es:**

1. **Erstkontakt:** Das Stickerbild wird zur Vision-Analyse an die KI gesendet. Die KI erzeugt eine Beschreibung (z. B. „Eine Cartoon-Katze, die enthusiastisch winkt“).
2. **Cache-Speicherung:** Die Beschreibung wird zusammen mit der Datei-ID des Stickers, dem Emoji und dem Set-Namen gespeichert.
3. **Weitere Kontakte:** Wird derselbe Sticker erneut gesehen, wird die zwischengespeicherte Beschreibung direkt verwendet. Das Bild wird nicht erneut an die KI gesendet.

**Cache-Speicherort:** `~/.openclaw/telegram/sticker-cache.json`

**Cache-Eintragsformat:**

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

**Vorteile:**

- Reduziert API-Kosten durch Vermeidung wiederholter Vision-Aufrufe für denselben Sticker
- Schnellere Antwortzeiten für gecachte Sticker (keine Vision-Verarbeitungsverzögerung)
- Ermöglicht Sticker-Suchfunktionen auf Basis zwischengespeicherter Beschreibungen

Der Cache wird automatisch befüllt, sobald Sticker empfangen werden. Es ist keine manuelle Cache-Verwaltung erforderlich.

### Sticker senden

Der Agent kann Sticker senden und suchen, indem er die Aktionen `sticker` und `sticker-search` verwendet. Diese sind standardmäßig deaktiviert und müssen in der Konfiguration aktiviert werden:

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

**Sticker senden:**

```json5
{
  action: "sticker",
  channel: "telegram",
  to: "123456789",
  fileId: "CAACAgIAAxkBAAI...",
}
```

Parameter:

- `fileId` (erforderlich) — die Telegram-Datei-ID des Stickers. Erhalten Sie diese aus `Sticker.fileId` beim Empfangen eines Stickers oder aus einem `sticker-search`-Ergebnis.
- `replyTo` (optional) — Nachrichten-ID, auf die geantwortet werden soll.
- `threadId` (optional) — Nachrichten-Thread-ID für Forum-Themen.

**Sticker suchen:**

Der Agent kann zwischengespeicherte Sticker nach Beschreibung, Emoji oder Set-Namen durchsuchen:

```json5
{
  action: "sticker-search",
  channel: "telegram",
  query: "cat waving",
  limit: 5,
}
```

Gibt passende Sticker aus dem Cache zurück:

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

Die Suche verwendet Fuzzy-Matching über Beschreibungstext, Emoji-Zeichen und Set-Namen.

**Beispiel mit Threading:**

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

## Streaming (Entwürfe)

Telegram kann **Entwurfsblasen** streamen, während der Agent eine Antwort generiert.
OpenClaw verwendet die Bot API `sendMessageDraft` (keine echten Nachrichten) und sendet anschließend die
finale Antwort als normale Nachricht.

Voraussetzungen (Telegram Bot API 9.3+):

- **Private Chats mit aktivierten Themen** (Forum-Topic-Modus für den Bot).
- Eingehende Nachrichten müssen `message_thread_id` enthalten (privater Topic-Thread).
- Streaming wird für Gruppen/Supergroups/Kanäle ignoriert.

Konfiguration:

- `channels.telegram.streamMode: "off" | "partial" | "block"` (Standard: `partial`)
  - `partial`: Aktualisiert die Entwurfsblase mit dem neuesten Streaming-Text.
  - `block`: Aktualisiert die Entwurfsblase in größeren Blöcken (chunked).
  - `off`: Deaktiviert Entwurfs-Streaming.
- Optional (nur für `streamMode: "block"`):
  - `channels.telegram.draftChunk: { minChars?, maxChars?, breakPreference? }`
    - Standardwerte: `minChars: 200`, `maxChars: 800`, `breakPreference: "paragraph"` (begrenzt auf `channels.telegram.textChunkLimit`).

Hinweis: Entwurfs-Streaming ist getrennt vom **Block-Streaming** (Kanalnachrichten).
Block-Streaming ist standardmäßig deaktiviert und erfordert `channels.telegram.blockStreaming: true`,
wenn Sie frühe Telegram-Nachrichten statt Entwurfs-Updates wünschen.

Reasoning-Stream (nur Telegram):

- `/reasoning stream` streamt die Begründung in die Entwurfsblase, während die Antwort
  generiert wird, und sendet anschließend die finale Antwort ohne Begründung.
- Wenn `channels.telegram.streamMode` auf `off` gesetzt ist, ist der Reasoning-Stream deaktiviert.
  Weitere Informationen: [Streaming + Chunking](/concepts/streaming).

## Retry-Richtlinie

Ausgehende Telegram-API-Aufrufe werden bei transienten Netzwerk-/429-Fehlern mit exponentiellem Backoff und Jitter wiederholt. Konfigurieren Sie dies über `channels.telegram.retry`. Siehe [Retry policy](/concepts/retry).

## Agenten-Werkzeug (Nachrichten + Reaktionen)

- Werkzeug: `telegram` mit Aktion `sendMessage` (`to`, `content`, optional `mediaUrl`, `replyToMessageId`, `messageThreadId`).
- Werkzeug: `telegram` mit Aktion `react` (`chatId`, `messageId`, `emoji`).
- Werkzeug: `telegram` mit Aktion `deleteMessage` (`chatId`, `messageId`).
- Semantik zum Entfernen von Reaktionen: siehe [/tools/reactions](/tools/reactions).
- Werkzeug-Gating: `channels.telegram.actions.reactions`, `channels.telegram.actions.sendMessage`, `channels.telegram.actions.deleteMessage` (Standard: aktiviert) und `channels.telegram.actions.sticker` (Standard: deaktiviert).

## Reaktionsbenachrichtigungen

**So funktionieren Reaktionen:**
Telegram-Reaktionen treffen als **separate `message_reaction`-Ereignisse** ein, nicht als Eigenschaften in Nachrichten-Payloads. Wenn ein Benutzer eine Reaktion hinzufügt, führt OpenClaw aus:

1. Empfang des `message_reaction`-Updates von der Telegram API
2. Umwandlung in ein **Systemereignis** mit Format: `"Telegram reaction added: {emoji} by {user} on msg {id}"`
3. Ruft das Systemereignis mit dem **gleichen Sitzungsschlüssel** als reguläre Nachrichten ein
4. Wenn die nächste Nachricht in dieser Konversation eintrifft, werden Systemereignisse abgearbeitet und dem Kontext des Agenten vorangestellt

Der Agent sieht Reaktionen als **Systembenachrichtigungen** im Konversationsverlauf, nicht als Nachrichten-Metadaten.

**Konfiguration:**

- `channels.telegram.reactionNotifications`: Steuert, welche Reaktionen Benachrichtigungen auslösen
  - `"off"` — alle Reaktionen ignorieren
  - `"own"` — benachrichtigen, wenn Benutzer auf Bot-Nachrichten reagieren (Best-Effort; In-Memory) (Standard)
  - `"all"` — für alle Reaktionen benachrichtigen

- `channels.telegram.reactionLevel`: Steuert die Reaktionsfähigkeit des Agenten
  - `"off"` — Agent kann nicht reagieren
  - `"ack"` — Bot sendet Bestätigungsreaktionen (👀 während der Verarbeitung) (Standard)
  - `"minimal"` — Agent kann sparsam reagieren (Richtwert: 1 pro 5–10 Austauschvorgänge)
  - `"extensive"` — Agent kann bei Bedarf großzügig reagieren

**Forum-Gruppen:** Reaktionen in Forum-Gruppen enthalten `message_thread_id` und verwenden Sitzungs­schlüssel wie `agent:main:telegram:group:{chatId}:topic:{threadId}`. Dadurch bleiben Reaktionen und Nachrichten im selben Thema zusammen.

**Beispielkonfiguration:**

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

**Voraussetzungen:**

- Telegram-Bots müssen explizit `message_reaction` in `allowed_updates` anfordern (automatisch durch OpenClaw konfiguriert)
- Im Webhook-Modus sind Reaktionen im Webhook `allowed_updates` enthalten
- Im Polling-Modus sind Reaktionen im `getUpdates` `allowed_updates` enthalten

## Zustellziele (CLI/Cron)

- Verwenden Sie eine Chat-ID (`123456789`) oder einen Benutzernamen (`@name`) als Ziel.
- Beispiel: `openclaw message send --channel telegram --target 123456789 --message "hi"`.

## Fehlerbehebung

**Bot antwortet in einer Gruppe nicht auf Nachrichten ohne Erwähnung:**

- Wenn Sie `channels.telegram.groups.*.requireMention=false` gesetzt haben, muss der **Privacy Mode** der Telegram Bot API deaktiviert sein.
  - BotFather: `/setprivacy` → **Disable** (anschließend Bot aus der Gruppe entfernen und erneut hinzufügen)
- `openclaw channels status` zeigt eine Warnung, wenn die Konfiguration nicht erwähnte Gruppennachrichten erwartet.
- `openclaw channels status --probe` kann zusätzlich die Mitgliedschaft für explizite numerische Gruppen-IDs prüfen (Wildcard-`"*"`-Regeln können nicht geprüft werden).
- Schnelltest: `/activation always` (nur Sitzung; für Persistenz Konfiguration verwenden)

**Bot sieht überhaupt keine Gruppennachrichten:**

- Wenn `channels.telegram.groups` gesetzt ist, muss die Gruppe gelistet sein oder `"*"` verwenden
- Prüfen Sie die Datenschutzeinstellungen in @BotFather → „Group Privacy“ sollte **OFF** sein
- Verifizieren Sie, dass der Bot tatsächlich Mitglied ist (nicht nur Admin ohne Lesezugriff)
- Prüfen Sie die Gateway-Logs: `openclaw logs --follow` (nach „skipping group message“ suchen)

**Bot antwortet auf Erwähnungen, aber nicht auf `/activation always`:**

- Der Befehl `/activation` aktualisiert den Sitzungszustand, persistiert aber nicht in die Konfiguration
- Für persistentes Verhalten fügen Sie die Gruppe zu `channels.telegram.groups` mit `requireMention: false` hinzu

**Befehle wie `/status` funktionieren nicht:**

- Stellen Sie sicher, dass Ihre Telegram-Benutzer-ID autorisiert ist (über Pairing oder `channels.telegram.allowFrom`)
- Befehle erfordern Autorisierung, selbst in Gruppen mit `groupPolicy: "open"`

**Long-Polling bricht unter Node 22+ sofort ab (oft mit Proxys/Custom Fetch):**

- Node 22+ ist strenger bei `AbortSignal`-Instanzen; fremde Signale können `fetch`-Aufrufe sofort abbrechen.
- Aktualisieren Sie auf einen OpenClaw-Build, der Abort-Signale normalisiert, oder betreiben Sie das Gateway auf Node 20, bis Sie upgraden können.

**Bot startet und reagiert dann stillschweigend nicht mehr (oder loggt `HttpError: Network request ... failed`):**

- Einige Hosts lösen `api.telegram.org` zuerst zu IPv6 auf. Wenn Ihr Server keinen funktionierenden IPv6-Egress hat, kann grammY bei IPv6-only-Anfragen hängen bleiben.
- Beheben Sie dies, indem Sie IPv6-Egress aktivieren **oder** IPv4-Auflösung für `api.telegram.org` erzwingen (z. B. einen `/etc/hosts`-Eintrag mit dem IPv4-A-Record hinzufügen oder IPv4 im OS-DNS-Stack bevorzugen) und starten Sie das Gateway neu.
- Schnellcheck: `dig +short api.telegram.org A` und `dig +short api.telegram.org AAAA`, um zu bestätigen, was DNS zurückgibt.

## Konfigurationsreferenz (Telegram)

Vollständige Konfiguration: [Konfiguration](/gateway/configuration)

Anbieteroptionen:

- `channels.telegram.enabled`: Kanalstart aktivieren/deaktivieren.
- `channels.telegram.botToken`: Bot-Token (BotFather).
- `channels.telegram.tokenFile`: Token aus Dateipfad lesen.
- `channels.telegram.dmPolicy`: `pairing | allowlist | open | disabled` (Standard: Pairing).
- `channels.telegram.allowFrom`: DM-Allowlist (IDs/Benutzernamen). `open` erfordert `"*"`.
- `channels.telegram.groupPolicy`: `open | allowlist | disabled` (Standard: Allowlist).
- `channels.telegram.groupAllowFrom`: Gruppen-Absender-Allowlist (IDs/Benutzernamen).
- `channels.telegram.groups`: Gruppen-Standards + Allowlist (verwenden Sie `"*"` für globale Defaults).
  - `channels.telegram.groups.<id>.groupPolicy`: Gruppen­spezifisches Override für groupPolicy (`open | allowlist | disabled`).
  - `channels.telegram.groups.<id>.requireMention`: Standard für Mention-Gating.
  - `channels.telegram.groups.<id>.skills`: Skill-Filter (weglassen = alle Skills, leer = keine).
  - `channels.telegram.groups.<id>.allowFrom`: Gruppen­spezifisches Absender-Allowlist-Override.
  - `channels.telegram.groups.<id>.systemPrompt`: Zusätzliches System-Prompt für die Gruppe.
  - `channels.telegram.groups.<id>.enabled`: Deaktiviert die Gruppe, wenn `false`.
  - `channels.telegram.groups.<id>.topics.<threadId>.*`: Themen­spezifische Overrides (gleiche Felder wie Gruppe).
  - `channels.telegram.groups.<id>.topics.<threadId>.groupPolicy`: Themen­spezifisches Override für groupPolicy (`open | allowlist | disabled`).
  - `channels.telegram.groups.<id>.topics.<threadId>.requireMention`: Themen­spezifisches Mention-Gating-Override.
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
- `channels.telegram.actions.reactions`: Telegram-Werkzeugreaktionen steuern.
- `channels.telegram.actions.sendMessage`: Telegram-Werkzeug-Nachrichtensendungen steuern.
- `channels.telegram.actions.deleteMessage`: Telegram-Werkzeug-Nachrichtenlöschungen steuern.
- `channels.telegram.actions.sticker`: Telegram-Sticker-Aktionen steuern — Senden und Suchen (Standard: false).
- `channels.telegram.reactionNotifications`: `off | own | all` — steuert, welche Reaktionen Systemereignisse auslösen (Standard: `own`, wenn nicht gesetzt).
- `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` — steuert die Reaktionsfähigkeit des Agenten (Standard: `minimal`, wenn nicht gesetzt).

Zugehörige globale Optionen:

- `agents.list[].groupChat.mentionPatterns` (Mention-Gating-Muster).
- `messages.groupChat.mentionPatterns` (globaler Fallback).
- `commands.native` (Standard: `"auto"` → an für Telegram/Discord, aus für Slack), `commands.text`, `commands.useAccessGroups` (Befehlsverhalten). Überschreiben mit `channels.telegram.commands.native`.
- `messages.responsePrefix`, `messages.ackReaction`, `messages.ackReactionScope`, `messages.removeAckAfterReply`.


