---
title: "„Konfiguration“"
---

# Konfiguration 🔧

OpenClaw liest eine optionale **JSON5**-Konfiguration aus `~/.openclaw/openclaw.json` (Kommentare + nachgestellte Kommata erlaubt).

Fehlt die Datei, verwendet OpenClaw sichere Standardwerte (eingebetteter Pi‑Agent + Sitzungen pro Absender + Workspace `~/.openclaw/workspace`). In der Regel benötigen Sie eine Konfiguration nur, um:

- einzuschränken, wer den Bot auslösen darf (`channels.whatsapp.allowFrom`, `channels.telegram.allowFrom` usw.)
- Gruppen-Allowlists und Mention-Verhalten zu steuern (`channels.whatsapp.groups`, `channels.telegram.groups`, `channels.discord.guilds`, `agents.list[].groupChat`)
- Nachrichtenpräfixe anzupassen (`messages`)
- den Workspace des Agenten festzulegen (`agents.defaults.workspace` oder `agents.list[].workspace`)
- die Standardwerte des eingebetteten Agenten (`agents.defaults`) und das Sitzungsverhalten (`session`) feinzujustieren
- eine agentenspezifische Identität festzulegen (`agents.list[].identity`)

> **Neu bei der Konfiguration?** Sehen Sie sich den Leitfaden [Configuration Examples](/gateway/configuration-examples) mit vollständigen Beispielen und detaillierten Erläuterungen an!

## Strikte Konfigurationsvalidierung

OpenClaw akzeptiert nur Konfigurationen, die vollständig dem Schema entsprechen.
Unbekannte Schlüssel, fehlerhafte Typen oder ungültige Werte führen dazu, dass das Gateway aus Sicherheitsgründen **nicht startet**.

Wenn die Validierung fehlschlägt:

- Das Gateway startet nicht.
- Es sind nur Diagnosebefehle erlaubt (z. B.: `openclaw doctor`, `openclaw logs`, `openclaw health`, `openclaw status`, `openclaw service`, `openclaw help`).
- Führen Sie `openclaw doctor` aus, um die genauen Probleme zu sehen.
- Führen Sie `openclaw doctor --fix` (oder `--yes`) aus, um Migrationen/Reparaturen anzuwenden.

Doctor schreibt niemals Änderungen, es sei denn, Sie entscheiden sich explizit für `--fix`/`--yes`.

## Schema- und UI-Hinweise

Das Gateway stellt eine JSON-Schema-Darstellung der Konfiguration über `config.schema` für UI-Editoren bereit.
Die Control UI rendert aus diesem Schema ein Formular, mit einem **Raw JSON**-Editor als Notausgang.

Kanal-Plugins und Erweiterungen können Schema- und UI-Hinweise für ihre Konfiguration registrieren, sodass Kanaleinstellungen schema‑getrieben über Apps hinweg bleiben, ohne hart codierte Formulare.

Hinweise (Beschriftungen, Gruppierung, sensible Felder) werden zusammen mit dem Schema ausgeliefert, damit Clients bessere Formulare rendern können, ohne Konfigurationswissen fest zu verdrahten.

## Anwenden + Neustart (RPC)

Verwenden Sie `config.apply`, um die vollständige Konfiguration in einem Schritt zu validieren, zu schreiben und das Gateway neu zu starten.
Dabei wird ein Neustart-Sentinel geschrieben und nach dem Wiederanlauf des Gateways die zuletzt aktive Sitzung angepingt.

Warnung: `config.apply` ersetzt die **gesamte Konfiguration**. Wenn Sie nur wenige Schlüssel ändern möchten,
verwenden Sie `config.patch` oder `openclaw config set`. Erstellen Sie eine Sicherung von `~/.openclaw/openclaw.json`.

Parameter:

- `raw` (string) — JSON5‑Payload für die gesamte Konfiguration
- `baseHash` (optional) — Konfigurations-Hash aus `config.get` (erforderlich, wenn bereits eine Konfiguration existiert)
- `sessionKey` (optional) — Schlüssel der zuletzt aktiven Sitzung für den Wake‑up‑Ping
- `note` (optional) — Notiz für das Neustart‑Sentinel
- `restartDelayMs` (optional) — Verzögerung vor dem Neustart (Standard 2000)

Beispiel (über `gateway call`):

```bash
openclaw gateway call config.get --params '{}' # capture payload.hash
openclaw gateway call config.apply --params '{
  "raw": "{\\n  agents: { defaults: { workspace: \\"~/.openclaw/workspace\\" } }\\n}\\n",
  "baseHash": "<hash-from-config.get>",
  "sessionKey": "agent:main:whatsapp:dm:+15555550123",
  "restartDelayMs": 1000
}'
```

## Partielle Updates (RPC)

Verwenden Sie `config.patch`, um ein partielles Update in die bestehende Konfiguration zu mergen, ohne
unverwandte Schlüssel zu überschreiben. Es gelten die Semantiken von JSON Merge Patch:

- Objekte werden rekursiv zusammengeführt
- `null` löscht einen Schlüssel
- Arrays werden ersetzt  
  Wie `config.apply` validiert, schreibt es die Konfiguration, speichert ein Neustart‑Sentinel
  und plant den Gateway‑Neustart (mit optionalem Wake‑up, wenn `sessionKey` angegeben ist).

Parameter:

- `raw` (string) — JSON5‑Payload mit nur den zu ändernden Schlüsseln
- `baseHash` (erforderlich) — Konfigurations-Hash aus `config.get`
- `sessionKey` (optional) — Schlüssel der zuletzt aktiven Sitzung für den Wake‑up‑Ping
- `note` (optional) — Notiz für das Neustart‑Sentinel
- `restartDelayMs` (optional) — Verzögerung vor dem Neustart (Standard 2000)

Beispiel:

```bash
openclaw gateway call config.get --params '{}' # capture payload.hash
openclaw gateway call config.patch --params '{
  "raw": "{\\n  channels: { telegram: { groups: { \\"*\\": { requireMention: false } } } }\\n}\\n",
  "baseHash": "<hash-from-config.get>",
  "sessionKey": "agent:main:whatsapp:dm:+15555550123",
  "restartDelayMs": 1000
}'
```

## Minimale Konfiguration (empfohlener Startpunkt)

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

Erstellen Sie das Standard‑Image einmalig mit:

```bash
scripts/sandbox-setup.sh
```

## Self‑Chat‑Modus (empfohlen für Gruppenkontrolle)

Um zu verhindern, dass der Bot in Gruppen auf WhatsApp‑@‑Mentions reagiert (nur auf bestimmte Text‑Trigger antworten):

```json5
{
  agents: {
    defaults: { workspace: "~/.openclaw/workspace" },
    list: [
      {
        id: "main",
        groupChat: { mentionPatterns: ["@openclaw", "reisponde"] },
      },
    ],
  },
  channels: {
    whatsapp: {
      // Allowlist is DMs only; including your own number enables self-chat mode.
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } },
    },
  },
}
```

## Config Includes (`$include`)

Teilen Sie Ihre Konfiguration mithilfe der Direktive `$include` in mehrere Dateien auf. Dies ist nützlich für:

- Organisation großer Konfigurationen (z. B. agentenspezifische Definitionen pro Client)
- Gemeinsame Nutzung von Einstellungen über Umgebungen hinweg
- Separates Halten sensibler Konfigurationen

### Grundlegende Verwendung

```json5
// ~/.openclaw/openclaw.json
{
  gateway: { port: 18789 },

  // Include a single file (replaces the key's value)
  agents: { $include: "./agents.json5" },

  // Include multiple files (deep-merged in order)
  broadcast: {
    $include: ["./clients/mueller.json5", "./clients/schmidt.json5"],
  },
}
```

```json5
// ~/.openclaw/agents.json5
{
  defaults: { sandbox: { mode: "all", scope: "session" } },
  list: [{ id: "main", workspace: "~/.openclaw/workspace" }],
}
```

### Merge‑Verhalten

- **Einzeldatei**: Ersetzt das Objekt, das `$include` enthält
- **Array von Dateien**: Deep‑Merge in Reihenfolge (spätere Dateien überschreiben frühere)
- **Mit Geschwisterschlüsseln**: Geschwisterschlüssel werden nach den Includes gemergt (überschreiben inkludierte Werte)
- **Geschwisterschlüssel + Arrays/Primitive**: Nicht unterstützt (inkludierter Inhalt muss ein Objekt sein)

```json5
// Sibling keys override included values
{
  $include: "./base.json5", // { a: 1, b: 2 }
  b: 99, // Result: { a: 1, b: 99 }
}
```

### Verschachtelte Includes

Eingebundene Dateien können selbst `$include`‑Direktiven enthalten (bis zu 10 Ebenen tief):

```json5
// clients/mueller.json5
{
  agents: { $include: "./mueller/agents.json5" },
  broadcast: { $include: "./mueller/broadcast.json5" },
}
```

### Pfadauflösung

- **Relative Pfade**: Relativ zur einbindenden Datei aufgelöst
- **Absolute Pfade**: Unverändert verwendet
- **Übergeordnete Verzeichnisse**: `../`‑Referenzen funktionieren wie erwartet

```json5
{ "$include": "./sub/config.json5" }      // relative
{ "$include": "/etc/openclaw/base.json5" } // absolute
{ "$include": "../shared/common.json5" }   // parent dir
```

### Fehlerbehandlung

- **Fehlende Datei**: Klarer Fehler mit aufgelöstem Pfad
- **Parse‑Fehler**: Zeigt an, welche eingebundene Datei fehlgeschlagen ist
- **Zirkuläre Includes**: Erkannt und mit Include‑Kette gemeldet

### Beispiel: Multi‑Client‑Rechtssetup

```json5
// ~/.openclaw/openclaw.json
{
  gateway: { port: 18789, auth: { token: "secret" } },

  // Common agent defaults
  agents: {
    defaults: {
      sandbox: { mode: "all", scope: "session" },
    },
    // Merge agent lists from all clients
    list: { $include: ["./clients/mueller/agents.json5", "./clients/schmidt/agents.json5"] },
  },

  // Merge broadcast configs
  broadcast: {
    $include: ["./clients/mueller/broadcast.json5", "./clients/schmidt/broadcast.json5"],
  },

  channels: { whatsapp: { groupPolicy: "allowlist" } },
}
```

```json5
// ~/.openclaw/clients/mueller/agents.json5
[
  { id: "mueller-transcribe", workspace: "~/clients/mueller/transcribe" },
  { id: "mueller-docs", workspace: "~/clients/mueller/docs" },
]
```

```json5
// ~/.openclaw/clients/mueller/broadcast.json5
{
  "120363403215116621@g.us": ["mueller-transcribe", "mueller-docs"],
}
```

## Häufige Optionen

### Umgebungsvariablen + `.env`

OpenClaw liest Umgebungsvariablen aus dem übergeordneten Prozess (Shell, launchd/systemd, CI usw.).

Zusätzlich lädt es:

- `.env` aus dem aktuellen Arbeitsverzeichnis (falls vorhanden)
- eine globale Fallback‑Datei `.env` aus `~/.openclaw/.env` (alias `$OPENCLAW_STATE_DIR/.env`)

Keine der `.env`‑Dateien überschreibt bestehende Umgebungsvariablen.

Sie können Umgebungsvariablen auch inline in der Konfiguration angeben. Diese werden nur angewendet, wenn
die Prozess‑Umgebung den Schlüssel nicht enthält (gleiche Nicht‑Überschreibungsregel):

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: {
      GROQ_API_KEY: "gsk-...",
    },
  },
}
```

Siehe [/environment](/help/environment) für vollständige Prioritäten und Quellen.

### `env.shellEnv` (optional)

Opt‑in‑Komfortfunktion: Wenn aktiviert und noch keiner der erwarteten Schlüssel gesetzt ist,
führt OpenClaw Ihre Login‑Shell aus und importiert nur die fehlenden erwarteten Schlüssel (überschreibt nie).
Dies entspricht effektiv dem Sourcen Ihres Shell‑Profils.

```json5
{
  env: {
    shellEnv: {
      enabled: true,
      timeoutMs: 15000,
    },
  },
}
```

Env var Äquivalent:

- `OPENCLAW_LOAD_SHELL_ENV=1`
- `OPENCLAW_SHELL_ENV_TIMEOUT_MS=15000`

### Env var Substitution in der Konfiguration

Sie können Umgebungsvariablen direkt in jedem String‑Wert der Konfiguration mit der Syntax
`${VAR_NAME}` referenzieren. Die Variablen werden beim Laden der Konfiguration,
vor der Validierung, ersetzt.

```json5
{
  models: {
    providers: {
      "vercel-gateway": {
        apiKey: "${VERCEL_GATEWAY_API_KEY}",
      },
    },
  },
  gateway: {
    auth: {
      token: "${OPENCLAW_GATEWAY_TOKEN}",
    },
  },
}
```

**Regeln:**

- Es werden nur großgeschriebene Umgebungsvariablennamen gematcht: `[A-Z_][A-Z0-9_]*`
- Fehlende oder leere env vars werfen einen Fehler bei der Konfigurationslast
- Mit `$${VAR}` escapen, um ein literales `${VAR}` auszugeben
- Funktioniert mit `$include` (auch eingebundene Dateien erhalten Substitution)

**Inline‑Substitution:**

```json5
{
  models: {
    providers: {
      custom: {
        baseUrl: "${CUSTOM_API_BASE}/v1", // → "https://api.example.com/v1"
      },
    },
  },
}
```

### Auth‑Speicher (OAuth + API‑Schlüssel)

OpenClaw speichert **pro Agent** Auth‑Profile (OAuth + API‑Schlüssel) in:

- `<agentDir>/auth-profiles.json` (Standard: `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`)

Siehe auch: [/concepts/oauth](/concepts/oauth)

Legacy‑OAuth‑Importe:

- `~/.openclaw/credentials/oauth.json` (oder `$OPENCLAW_STATE_DIR/credentials/oauth.json`)

Der eingebettete Pi‑Agent verwaltet einen Laufzeit‑Cache unter:

- `<agentDir>/auth.json` (automatisch verwaltet; nicht manuell bearbeiten)

Legacy‑Agentenverzeichnis (vor Multi‑Agent):

- `~/.openclaw/agent/*` (von `openclaw doctor` nach `~/.openclaw/agents/<defaultAgentId>/agent/*` migriert)

Overrides:

- OAuth‑Verzeichnis (nur Legacy‑Import): `OPENCLAW_OAUTH_DIR`
- Agentenverzeichnis (Standard‑Agent‑Root‑Override): `OPENCLAW_AGENT_DIR` (bevorzugt), `PI_CODING_AGENT_DIR` (Legacy)

Bei der ersten Verwendung importiert OpenClaw `oauth.json`‑Einträge nach `auth-profiles.json`.

### `auth`

Optionale Metadaten für Auth‑Profile. Speichert **keine** Geheimnisse; ordnet
Profil‑IDs einem Anbieter + Modus (und optionaler E‑Mail) zu und definiert die
Anbieter‑Rotationsreihenfolge für Failover.

```json5
{
  auth: {
    profiles: {
      "anthropic:me@example.com": { provider: "anthropic", mode: "oauth", email: "me@example.com" },
      "anthropic:work": { provider: "anthropic", mode: "api_key" },
    },
    order: {
      anthropic: ["anthropic:me@example.com", "anthropic:work"],
    },
  },
}
```

### `agents.list[].identity`

Optionale agentenspezifische Identität für Standardwerte und UX. Diese wird vom macOS‑Onboarding‑Assistenten geschrieben.

Wenn gesetzt, leitet OpenClaw Standardwerte ab (nur wenn Sie diese nicht explizit gesetzt haben):

- `messages.ackReaction` aus dem `identity.emoji` des **aktiven Agenten** (Fallback 👀)
- `agents.list[].groupChat.mentionPatterns` aus dem `identity.name`/`identity.emoji` des Agenten (damit „@Samantha“ in Gruppen über Telegram/Slack/Discord/Google Chat/iMessage/WhatsApp funktioniert)
- `identity.avatar` akzeptiert einen workspace‑relativen Bildpfad oder eine Remote‑URL/Data‑URL. Lokale Dateien müssen innerhalb des Agent‑Workspace liegen.

`identity.avatar` akzeptiert:

- Workspace‑relativen Pfad (muss innerhalb des Agent‑Workspace bleiben)
- `http(s)`‑URL
- `data:`‑URI

```json5
{
  agents: {
    list: [
      {
        id: "main",
        identity: {
          name: "Samantha",
          theme: "helpful sloth",
          emoji: "🦥",
          avatar: "avatars/samantha.png",
        },
      },
    ],
  },
}
```

### `wizard`

Metadaten, die von CLI‑Assistenten geschrieben werden (`onboard`, `configure`, `doctor`).

```json5
{
  wizard: {
    lastRunAt: "2026-01-01T00:00:00.000Z",
    lastRunVersion: "2026.1.4",
    lastRunCommit: "abc1234",
    lastRunCommand: "configure",
    lastRunMode: "local",
  },
}
```

### `logging`

- Standard‑Logdatei: `/tmp/openclaw/openclaw-YYYY-MM-DD.log`
- Wenn Sie einen stabilen Pfad möchten, setzen Sie `logging.file` auf `/tmp/openclaw/openclaw.log`.
- Konsolenausgabe kann separat angepasst werden über:
  - `logging.consoleLevel` (Standard `info`, erhöht auf `debug` bei `--verbose`)
  - `logging.consoleStyle` (`pretty` | `compact` | `json`)
- Werkzeug‑Zusammenfassungen können redigiert werden, um das Leaken von Geheimnissen zu vermeiden:
  - `logging.redactSensitive` (`off` | `tools`, Standard: `tools`)
  - `logging.redactPatterns` (Array aus Regex‑Strings; überschreibt Standardwerte)

```json5
{
  logging: {
    level: "info",
    file: "/tmp/openclaw/openclaw.log",
    consoleLevel: "info",
    consoleStyle: "pretty",
    redactSensitive: "tools",
    redactPatterns: [
      // Example: override defaults with your own rules.
      "\\bTOKEN\\b\\s*[=:]\\s*([\"']?)([^\\s\"']+)\\1",
      "/\\bsk-[A-Za-z0-9_-]{8,}\\b/gi",
    ],
  },
}
```

### `channels.whatsapp.dmPolicy`

Steuert wie WhatsApp direkte Chats (DMs) behandelt werden:

- `"Paarung"` (Standard): Unbekannte Absender erhalten einen Paarcode; Besitzer muss genehmigen
- `"allowlist"`: erlaubt nur Absender in `channels.whatsapp.allowFrom` (oder gepaart mit "allow store" )
- `"open"`: Erlaube allen eingehenden DMs (**erforderlich** `channels.whatsapp.allowFrom` `"*"`)
- `"deaktiviert"`: Alle eingehenden DMs ignorieren

Kopplungscodes verfallen nach 1 Stunde; der Bot sendet nur einen Paarcode wenn eine neue Anfrage erstellt wird. Ausstehende DM-Paaranfragen sind standardmäßig auf **3 pro Kanal** beschränkt.

Paarungsgenehmigungen:

- `openclaw pairing list whatsapp`
- `openclaw Paarung genehmigt Whatsapp <code>`

### `channels.whatsapp.allowFrom`

Erlaubte Liste von E.164 Telefonnummern, die WhatsApp automatische Antworten auslösen können (**DMs nur**).
Wenn leer und `channels.whatsapp.dmPolicy="paaren"`, werden unbekannte Absender einen Paarcode erhalten.
Benutze `channels.whatsapp.groupPolicy` + `channels.whatsapp.groupAllowFrom`.

```json5
{
  Kanäle: {
    whatsapp: {
      dmPolicy: "Paarung", // Paarung | allowlist | offen | deaktiviert
      allowvon: ["+15555550123", "+447700900123"],
      textChunkLimit: 4000, // Optionale ausgehende Chunk-Größe (Zeichen)
      chunkModus: "Länge", // Optionaler Chunking-Modus (Länge | newline)
      mediaMaxMb: 50, // optionale eingehende Medienkappe (MB)
    },
  },
}
```

### `channels.whatsapp.sendReadReceipts`

Legt fest, ob eingehende WhatsApp-Nachrichten als gelesen markiert werden (blaue Ticks). Standard: `true`.

Der Selbst-Chat-Modus überspringt immer Lesebelege, auch wenn aktiviert.

Per-Account überschreiben: `channels.whatsapp.accounts.<id>.sendReadReceipts`.

```json5
{
  Kanäle: {
    whatsapp: { sendReadReceipts: false },
  },
}
```

### `channels.whatsapp.accounts` (Multi-Konto)

Mehrere WhatsApp-Konten in einem Gateway ausführen:

```json5
{
  Kanäle: {
    whatsapp: {
      Konten: {
        default: {}, // optional; hält die Standard ID stabil
        Person: {},
        biz: {
          // Optional override. Standard: ~/.openclaw/credentials/whatsapp/biz
          // authDir: "~/. penclaw/credentials/whatsapp/biz",
        },
      },
    },
  },
}
```

Hinweise:

- Ausgehende Befehle standardmäßig auf Konto `default`, wenn vorhanden; andernfalls die erste konfigurierte Konto-ID (sortiert).
- Das Legacy Single-Account Baileys auth dir wird von `openclaw doctor` in `whatsapp/default` migriert.

### `channels.telegram.accounts` / `channels.discord.accounts` / `channels.googlechat.accounts` / `channels.slack.accounts` / `channels.mattermost.accounts` / `channels.signal.accounts` / `channels.imessage.accounts`

Führe mehrere Konten pro Kanal aus (jedes Konto hat seinen eigenen `accountId` und optionalen `name`):

```json5
{
  Kanäle: {
    telegram: {
      Konten: {
        default: {
          name: "Primary bot",
          botToken: "123456:ABC... ,
        },
        Benachrichtigungen: {
          Name: "Alarme Bot",
          botToken: "987654:XYZ. .",
        },
      },
    },
  },
}
```

Hinweise:

- `default` wird verwendet, wenn `accountId` weggelassen wird (CLI + Routing).
- Env-Tokens gelten nur für das **Standard** Konto.
- Basis-Kanal-Einstellungen (Gruppenrichtlinien, Gürtel erwähnen usw.) auf alle Konten anzuwenden, es sei denn, sie werden pro Konto überschrieben.
- Benutze `bindings[].match.accountId` um jedes Konto zu einem anderen agents.defaults zu leiten.

### Gruppenchat erwähnt Gating (`agents.list[].groupChat` + `messages.groupChat`)

Gruppieren von Nachrichten standardmäßig **Erwähnung benötigen** (entweder Metadaten Erwähnung oder Regex Pattern). Gilt für WhatsApp, Telegram, Discord, Google Chat und iMessage Gruppen-Chats.

**Erwähnungstypen:**

- **Metadaten**: Native Plattform @-Erwähnungen (z.B. WhatsApp Tap-to-mention). Ignoriert im WhatsApp Selbst-Chat-Modus (siehe `channels.whatsapp.allowVrom`).
- **Textmuster**: Regex-Muster, definiert in `agents.list[].groupChat.mentionPatterns`. Immer überprüft, unabhängig vom Selbst-Chat-Modus.
- Erwähnung-Gating wird nur erzwungen, wenn die Erkennung von Erwähnungen möglich ist (natives Erwähnungsmuster oder mindestens ein "Erwähnungsmuster").

```json5
{
  Nachrichten: {
    GruppeChat: { historyLimit: 50 },
  },
  Agenten: {
    Liste: [{ id: "main", groupChat: { mentionPatterns: ["@openclaw", "openclaw"] } }],
  },
}
```

`messages.groupChat.historyLimit` setzt den globalen Standard für den Gruppenverlauf-Kontext. Kanäle können mit `channels überschrieben werden.<channel>.historyLimit` (oder `channels.<channel>.accounts.*.historyLimit` für Mehrfachkonten). Setze `0` um History Packing zu deaktivieren.

#### DM-Verlaufsgrenzen

DM Konversationen verwenden session-basierte Geschichte, die vom Agent verwaltet wird. Sie können die Anzahl der gespeicherten Benutzerdrehungen pro DM-Sitzung begrenzen:

```json5
{
  Kanäle: {
    telegram: {
      dmHistoryLimit: 30, // Begrenzung der DM-Sitzungen auf 30 Benutzer wird
      dms: {
        "123456789": { historyLimit: 50 }, // per-user override (user ID)
      },
    },
  },
}
```

Auflösungsreihenfolge:

1. Per-DM überschreiben: `channels.<provider>.dms[userId].historyLimit`
2. Provider Standard: \`channels.<provider>.dmHistoryLimit
3. Kein Limit (alle Historie gespeichert)

Unterstützte Anbieter: `telegram`, `whatsapp`, `discord`, `slack`, `signal`, `imessage`, `msteams`.

Per-Agent überschreiben (hat Vorrang, wenn gesetzt, sogar `[]`):

```json5
{
  Agenten: {
    Liste: [
      { id: "work", groupChat: { mentionPatterns: ["@workbot", "\\+15555550123"] }
      { id: "personal", groupChat: { mentionPatterns: ["@homebot", "\\\+1555555550999"] },
    ],
  },
}
```

Erwähnen Sie Gating standardmäßig live pro Kanal (`channels.whatsapp.groups`, `channels.telegram.groups`, `channels.imessage.groups`, `channels.discord.guilds`). Wenn `*.groups` gesetzt ist, fungiert sie auch als Gruppenerlaubnisliste; füge `"*"` hinzu, um alle Gruppen zu erlauben.

Um **nur** auf bestimmte Textauslöser zu reagieren (native @-Erwähnungen):

```json5
{
  Kanäle: {
    whatsapp: {
      // Fügen Sie Ihre eigene Nummer ein, um den Selbst-Chat-Modus zu aktivieren (native @-Erwähnungen).
      allowVon: ["+15555550123"],
      Gruppen: { "*": { requireMention: true } },
    },
  },
  Agenten: {
    list: [
      {
        id: "main",
        groupChat: {
          // Nur diese Textmuster werden Antworten
          Erwähnungsmuster auslösen: ["reisponde", "@openclaw"],
        },
      },
    ],
  },
}
```

### Gruppenrichtlinien (pro Kanal)

Benutze `channels.*.groupPolicy` um festzulegen, ob Gruppen- und Raumnachrichten überhaupt akzeptiert werden:

```json5
{
  Kanäle: {
    whatsapp: {
      groupPolicy: "allowlist",
      groupAllowVon: ["+15551234567"],
    },
    telegram: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["tg:123456789", "@alice"],
    },
    Signal: {
      groupPolicy: "allowlist",
      groupAllowVon: ["+15551234567"],
    },
    imessage: {
      groupPolicy: "allowlist",
      groupAllowVrom: ["chat_id:123"],
    },
    msteams: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["user@org. om"],
    },
    discord: {
      groupPolicy: "allowlist",
      Gilden: {
        GUILD_ID: {
          Kanäle: { help: { allow: true } },
        },
      },
    },
    slack: {
      groupPolicy: "allowlist",
      Kanäle: { "#general": { allow: true } },
    },
  },
}
```

Hinweise:

- `"offen"`: Gruppen umgehen Erlaubnislisten; Erwähnung-Gating gilt noch immer.
- `"deaktiviert"`: Blockiere alle Gruppen/Raum-Nachrichten.
- `"allowlist"`: erlaubt nur Gruppen/Räume, die mit der konfigurierten Berechtigungsliste übereinstimmen.
- `channels.defaults.groupPolicy` setzt die Standardeinstellung, wenn die `groupPolicy` eines Providers nicht gesetzt ist.
- WhatsApp/Telegram/Signal/iMessage/Microsoft Teams verwenden `groupAllowFrom` (Fallback: explizit `allowFrom`).
- Discord/Slack verwenden Channel allowlists (`channels.discord.guilds.*.channels`, `channels.slack.channels`).
- Group DMs (Discord/Slack) werden immer noch von `dm.groupEnabled` + `dm.groupChannels` kontrolliert.
- Standard ist `groupPolicy: "allowlist"` (unless overridden by `channels.defaults.groupPolicy`); if no allowlist is configured, group messages are blocked.

### Multi-Agent Routing (`agents.list` + `bindings`)

Führen Sie mehrere isolierte Agenten (separater Arbeitsbereich, `agentDir`, Sitzungen) innerhalb eines Gateways aus.
Eingehende Nachrichten werden über Bindungen an einen Agent weitergeleitet.

- `agents.list[]`: per-agent überschreibt.
  - `id`: stable agent id (erforderlich).
  - `default`: optional; wenn mehrere gesetzt sind, wird der erste gewinnt und eine Warnung protokolliert.
    Wenn keiner gesetzt ist, ist der **erste Eintrag** in der Liste der Standardagent.
  - `name`: Anzeigename für den Agenten.
  - `workspace`: Standard `~/.openclaw/workspace-<agentId>` (für `main`, fällt zurück auf `agents.defaults.workspace`).
  - `agentDir`: Standard `~/.openclaw/agents/<agentId>/agent`.
  - `model`: Standardmodell, überschreibt `agents.defaults.model` für diesen Agent.
    - string Formular: `"provider/model"`, überschreibt nur `agents.defaults.model.primary`
    - Objektform: `{ primary, fallbacks }` (Fallbacks überschreiben `agents.defaults.model.fallbacks`; `[]` deaktiviert globale Fallbacks für diesen Agenten)
  - `identity`: per-agent name/theme/emoji (zur Erwähnung von Mustern + ack reactions).
  - `groupChat`: per-agent mention-gating (`mentionPatterns`).
  - `sandbox`: per-agent sandbox config (überschreibt `agents.defaults.sandbox`).
    - `mode`: `"off"` | `"non-main"` | `"all"`
    - `workspaceAccess`: `"none"` | `"ro"` | `"rw"`
    - `scope`: `"session"` | `"agent"` | `"shared"`
    - `workspaceRoot`: Benutzerdefinierte Sandbox-Arbeitsbereichs-Root
    - `docker`: per-agent docker überschreibt (z.B. `image`, `network`, `env`, `setupCommand`, limits; ignoriert wenn `scope: "shared"`)
    - `browser`: Per-agent überschreibt den Browser in einer Sandbox (ignoriert wenn `scope: "shared"`)
    - `prune`: Per-agent Sandbox überschreibt (ignoriert wenn `Geltungsbereich: "Shared"`)
  - `subagents`: Per-agent Sub-Agent Standardeinstellungen.
    - `allowAgents`: allowlist of agent ids for `sessions_spawn` from this agent (`["*"]` = allow any; default: only same agent)
  - `tools`: Einschränkungen der Werkzeuge pro Agent (angewendet vor der Sandbox-Tool-Richtlinie).
    - Profil: Basiswerkzeugprofil (vor Erlaubt/Verweigerung angewendet)
    - `allow`: Array der erlaubten Werkzeugnamen
    - `deny`: Array der verweigerten Werkzeugnamen (verweigert Siege)
- `agents.defaults`: shared agent defaults (modell, workspace, sandbox, etc.).
- `bindings[]`: ruft eingehende Nachrichten an eine `agentId`.
  - `match.channel` (erforderlich)
  - `match.accountId` (optional; `*` = irgendein Konto; weggelassen = Standardkonto)
  - `match.peer` (optional; `{ kind: direct|group|channel, id }`)
  - `match.guildId` / `match.teamId` (optional; kanalspezifisch)

Deterministische Match-Reihenfolge:

1. `match.peer`
2. `match.guildId`
3. `match.teamId`
4. `match.accountId` (genau, kein Peer/Gild/Team)
5. `match.accountId: "*"` (kanalweit, kein Peer/Gild/Team)
6. default agent (`agents.list[].default`, sonst der erste Listeneintrag, sonst `"main"`)

Innerhalb jeder Spielstufe gewinnt der erste passende Eintrag in 'bindings'.

#### Pro‑Agent‑Zugriffsprofile (Multi‑Agent)

Jeder Agent kann seine eigene Sandbox + Werkzeugpolitik tragen. Verwende dies, um Zugriff auf
Ebenen in einem Gateway zu vermischen:

- \*\*Vollzugriff \*\* (persönlicher Agent)
- **Nur lesen** Werkzeuge + Arbeitsbereich
- **Kein Zugriff auf das Dateisystem** (nur auf Messaging/Session-Tools)

Siehe [Multi-Agent Sandbox & Tools](/tools/multi-agent-sandbox-tools) für Vorrang und
zusätzliche Beispiele.

Voller Zugriff (keine Sandbox):

```json5
{
  agents: {
    list: [
      {
        id: "personal",
        workspace: "~/.openclaw/workspace-personal",
        sandbox: { mode: "off" },
      },
    ],
  },
}
```

Nur-Lese-Werkzeuge + schreibgeschützter Arbeitsbereich:

```json5
{
  Agenten: {
    Liste: [
      {
        id: "Familie",
        Arbeitsbereich: "~/. penclaw/workspace-family",
        sandbox: {
          Modus: "all",
          Bereich: "Agent",
          Arbeitsbereichszugriff: "ro",
        },
        Tools: {
          allow: [
            "read",
            "sessions_list",
            "sessions_history",
            "sessions_send",
            "sessions_spawn",
            "session_status",
          ],
          verweigern: ["schreiben", "bearbeiten", "apply_patch", "exec", "process", "browser"],
        },
      },
    ],
  },
}
```

Kein Zugriff auf das Dateisystem (Messaging/Session-Tools aktiviert):

```json5
{
  Agenten: {
    Liste: [
      {
        id: "public",
        Arbeitsbereich: "~/. penclaw/workspace-public",
        sandbox: {
          Modus: "all",
          Bereich: "Agent",
          Arbeitsbereichszugriff: "keine",
        },
        Tools: {
          allow: [
            "sessions_list",
            "sessions_history",
            "sessions_send",
            "sessions_spawn",
            "session_status",
            "whatsapp",
            "Telegramm",
            "slack",
            "Diskord",
            "Gateway",
          ],
          Ablehnung: [
            "gelesen",
            "Schreiben",
            "Bearbeiten",
            "apply_patch",
            "ausführen",
            "Prozess",
            "Browser",
            "Leinwas",
            "Knoten",
            "cron",
            "Gateway",
            "Bild",
          ],
        },
      },
    ],
  },
}
```

Beispiel: zwei WhatsApp-Konten → zwei Agenten:

```json5
{
  Agenten: {
    Liste: [
      { id: "home", default: true workspace: "~/. penclaw/workspace-home" },
      { id: "work", workspace: "~/. penclaw/workspace-work" },
    ],
  },
  Bindungen: [
    { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
    { agentId: "work", match: { channel: "whatsapp", accountId: "biz" } },
  ],
  Kanäle: {
    whatsapp: {
      Accounts: {
        personal: {},
        Biz: {},
      },
    },
  },
}
```

### `tools.agentToAgent` (optional)

Agent-To-Agent-Nachrichten sind aktiviert:

```json5
{
  tools: {
    agentToAgent: {
      aktiviert: false,
      erlaubt: ["home", "work"],
    },
  },
}
```

### `messages.queue`

Legt fest, wie sich eingehende Nachrichten verhalten, wenn ein Agent bereits aktiv ist.

```json5
{
  Nachrichten: {
    Warteschlange: {
      Modus: "collect", // steer | follow up | sammle | steer-backlog (steer+backlog ok) | interrupt (queue=steer legacy)
      debounceMs: 1000,
      Cap: 20,
      Drop: "summarize", // alt | neu | Zusammenfassung
      byChannel: {
        whatsapp: "collect",
        Telegramm: "Sammeln",
        Diskord: "Sammeln",
        Bild: "Sammeln",
        Webchat: "Sammeln",
      },
    },
  },
}
```

### `messages.inbound`

Entprelle schnelle eingehende Nachrichten vom **gleichen Absender**, sodass mehrere unmittelbar aufeinanderfolgende Nachrichten zu einem einzelnen Agenten-Zug zusammengefasst werden. Debouncing ist pro Kanal + Unterhaltung begrenzt
und verwendet die zuletzt eingegangene Nachricht für Antwort-Threading/IDs.

```json5
{
  Nachrichten: {
    inbound: {
      debounceMs: 2000, // 0 deaktiviert
      byChannel: {
        whatsapp: 5000,
        Slack: 1500,
        Diskord: 1500,
      },
    },
  },
}
```

Hinweise:

- Debounce Batches **text-only** Nachrichten; Medien/Anhänge werden sofort gelöscht.
- Kontrollbefehle (z.B. `/queue`, `/new`) umgehen die Debouncing, so dass sie allein bleiben.

### `commands` (Chat-Befehlsbehandlung)

Legt fest, wie Chat-Befehle über Konnektoren hinweg aktiviert werden.

```json5
{
  Befehle: {
    nativ: "auto", // native Befehle registrieren wenn unterstützt (auto)
    Text: true // Schrägstrich Befehle in Chatnachrichten
    Bash: false, // zulassen ! (alias: /bash) (host-only; benötigt Werkzeuge. levated allowlists)
    bashForegroundMs: 2000, // bash Vordergrundfenster (0 Hintergründe sofort)
    config: falsch, // Erlaube /config (schreibt auf die Festplatte)
    debug: falsch, // Erlaube /debug (nur runtime-overrides)
    Neustart: false, // Erlaube /restart + Gateway Neustartwerkzeug
    useAccessGroups: true // Zugriffsgruppen-Erlaubnislisten/-Richtlinien für Befehle
  },
}
```

Hinweise:

- Text-Befehle müssen als **standalone** Nachricht gesendet werden und verwenden Sie die führende `/` (keine Klartext-Aliase).
- `commands.text: false` deaktiviert das Parsen von Chat-Nachrichten für Befehle.
- `commands.native: "auto"` (default) schaltet native Befehle für Discord/Telegram ein und lässt Slack ausschalten; nicht unterstützte Kanäle bleiben text-only.
- Setze `commands.native: true|false` um alle zu erzwingen, oder überschreiben pro Kanal mit `channels.discord.commands.native`, `channels.telegram.commands.native`, `channels.slack.commands.native` (bool oder `"auto"`). `false` löscht beim Start zuvor registrierte Befehle auf Discord/Telegram; Slack Befehle werden in der Slack App verwaltet.
- `channels.telegram.customCommands` fügt zusätzliche Telegram Bot-Menüeinträge hinzu. Namen werden normalisiert, Konflikte mit nativen Befehlen werden ignoriert.
- `commands.bash: true` aktiviert `! <cmd>` um Host-Shell-Befehle auszuführen (`/bash <cmd>funktioniert auch als Alias). Benötigt `tools.elevated.enabled`und erlaubt die Auflistung des Absenders in`tools.elevated.allowFrom.<channel>\`.
- `commands.bashForegroundMs` legt fest, wie lange bash vor dem Hintergrund wartet. Während ein Bash-Job läuft, neu `! <cmd>` Anfragen werden abgelehnt (jeweils ein).
- `commands.config: true` aktiviert `/config` (reads/writes `openclaw.json`).
- `Kanäle.<provider>.configWrites` gates config mutations initiiert durch diesen Kanal (Standard: true). Dies gilt für `/config set|unset` plus provider-spezifische Auto-Migrationen (Änderungen der Telegram-Supergruppen-ID-Änderungen, Änderungen der Slack Channel ID).
- `commands.debug: true` aktiviert `/debug` (nur runtime-overrides).
- `commands.restart: true` aktiviert `/restart` und die Neustart-Aktion des Gateway-Tools.
- `commands.useAccessGroups: false` erlaubt es Befehlen, Zugriffsgruppen-Erlaubnislisten/-Richtlinien zu umgehen.
- Slash‑Befehle und Direktiven werden nur für **autorisierte Absender** berücksichtigt. Die Autorisierung wird von
  Kanal allowlists/pairing plus `commands.useAccessGroups` abgeleitet.

### `web` (WhatsApp Web Channel Laufzeit)

WhatsApp läuft über den Webkanal (Baileys Web). Es startet automatisch, wenn eine verknüpfte Sitzung existiert.
Legen Sie `web.enabled: false` fest, um es standardmäßig auszuschalten.

```json5
{
  web: {
    aktiviert: true
    heartbeatSeconds: 60,
    Verbinde neu: {
      initialMs: 2000,
      maxMs: 120000,
      Faktor: 1. ,
      jitter: 0. ,
      Maximale Versuche: 0,
    },
  },
}
```

### `channels.telegram` (Bot-Transport)

OpenClaw startet Telegram nur, wenn ein `channels.telegram` Konfigurations-Abschnitt existiert. Der Bot Token wird von `channels.telegram.botToken` (oder `channels.telegram.tokenFile`) aufgelöst, mit `TELEGRAM_BOT_TOKEN` als Fallback für den Standardkonto.
Setze `channels.telegram.enabled: false` um den automatischen Start zu deaktivieren.
Der Multi-Account-Support lebt unter `channels.telegram.accounts` (siehe den oben genannten Multi-Account-Abschnitt). Env-Tokens gelten nur für das Standardkonto.
Setze `channels.telegram.configWrites: false` um Telegram-initiierte Konfigurationsschreibungen zu blockieren (einschließlich der Supergruppen-ID-Migrationen und `/config set|unset`).

```json5
{
  Kanäle: {
    telegram: {
      aktiviert: true
      botToken: "your-bot-token",
      dmPolicy: "Paaren", // Paarung | allowlist | offen | deaktiviert
      allowVrom: ["tg:123456789"], // optional; "offen" erfordert ["*"]
      Gruppen: {
        "*": { requireMention: true },
        "-1001234567890": {
          allowVrom: ["@admin"],
          systemPrompt: "Behalten Sie die Antworten kurz. ,
          Themen: {
            "99": {
              requireMention: false,
              Fähigkeiten: ["Suche"],
              systemPrompt: "Bleiben Sie auf dem Thema. ,
            },
          },
        },
      },
      CustomCommands: [
        { Befehl: "backup", Beschreibung: "Git backup" },
        { Befehl: "generate", Beschreibung: "Ein Bild erstellen" },
      ],
      Verlauflimit: 50, // füge letzte N Gruppen-Nachrichten als Kontext hinzu (0 deaktiviert ab)
      replyToMode: "first", // off | zuerst | alle
      linkPreview: true // Ausgehende Link-Vorschau ein/aus
      StreamModus: "partiell", // aus | partiell | Block (Entwurf Streaming; getrennt vom Blockstreaming)
      draftChunk: {
        // optional; nur für streamMode=block
        minZeichen: 200,
        maxChars: 800,
        breakPreference: "paragraph", // Absatz | newline | Satz
      },
      Aktionen: { reactions: true, sendMessage: true }, // Werkzeug-Aktionstore (falsch deaktiviert)
      ReaktionBenachrichtigungen: "eigen", // off | Eigenes | alle
      mediaMaxMb: 5,
      Wiederholung: {
        // Ausgehende Wiederholungsrichtlinie
        Versuche: 3,
        minVerzögerungen: 400,
        maxDelayMs: 30000,
        jitter: 0. ,
      },
      Netzwerk: {
        // Transport überschreibt
        autoSelectFamily: falsch,
      },
      Proxy: "socks5://localhost:9050",
      webhookUrl: "https://example. om/telegram-webhook", // erfordert webhookSecret
      webhookSecret: "secret",
      webhookPath: "/telegram-webhook",
    },
  },
}
```

Entwurf Streaming-Notizen:

- Verwendet Telegramm `sendMessageDraft` (entwerfen Blase, keine echte Nachricht).
- Benötigt **private Chatthemen** (message_thread_id in DMs; Bot hat Themen aktiviert).
- `/reasoning stream` streams die Argumentation in den Entwurf, sendet dann die endgültige Antwort.
  Standardwerte und Verhalten der Wiederholungsrichtlinien werden in [Wiederholungsrichtlinien](/concepts/retry) dokumentiert.

### `channels.discord` (Bot-Transport)

Konfiguriere den Discord Bot, indem du den Bot Token und optionales Gating setzst:
Multi-Account Support lebt unter `channels.discord.accounts` (siehe den Abschnitt oben mehrfach). Env-Tokens gelten nur für das Standardkonto.

```json5
{
  Kanäle: {
    Diskord: {
      aktiviert: true
      Token: "your-bot-token",
      mediaMaxMb: 8, // Pratzen eingehende Medien Größe
      erlaubt: falsch, // Erlaube Bot-Authored Nachrichten
      Aktionen: {
        // Werkzeug Action Gates (false disables)
        Reaktionen: true
        Aufkleber: true
        Umfragen: true
        Berechtigungen: wahr
        Nachrichten: true
        Threads: true
        Pins: true
        search: true
        memberInfo: true
        roleInfo: true
        Rollen: falsch,
        channelInfo: true
        Sprachstatus: wahr
        Ereignisse: wahr,
        Moderation: falsch,
      },
      replyToMode: "off", // off | zuerst | alle
      dm: {
        aktiviert: true // Deaktiviere alle DMs wenn false
        Richtlinie: "Paaren", // Paarung | allowlist | offen | deaktiviert
        allowvon: ["1234567890", "steipe"], // Optionale DM allowlist ("offen" erfordert ["*"])
        gruppAktiviert: false, // Gruppe DMs
        GroupChannels: ["openclaw-dm"], // Optionale Gruppe DM Erlaubnisliste
      },
      Gilden: {
        "123456789012345678": {
          // Gilde id (bevorzugt) oder slug
          slug: "friends-of-openclaw",
          Voraussetzung: falsch, // pro Gilde Standard
          ReaktionBenachrichtigungen: "eigene", // off | own | Alle | allowlist
          Benutzer: ["987654321098432"], // optionale Per-Guild User allowlist
          Kanäle: {
            general: { allow: true },
            Hilfe: {
              erlaubt: true
              requireMention: true
              Benutzer: ["98765432"],
              Fähigkeiten: ["docs"],
              systemPrompt: "Nur kurze Antworten. ,
            },
          },
        },
      },
      historyLimit: 20, // die letzten N Gildennachrichten als Kontext
      textChunkLimit: 2000, // Optionale ausgehende Textchunk-Größe (Zeichen)
      chunkModus: "Länge", // Optionaler Chunking-Modus (Länge | newline)
      maxLinesPerMessage: 17, // soft max lines per message (Discord UI clipping)
      retry y: {
        // outbound retry policy
        attempts: 3,
        minVerzögerungen: 500,
        maxDelayMs: 30000,
        jitter: 0. ,
      },
    },
  },
}
```

OpenClaw startet Discord nur, wenn ein `channels.discord` Konfigurations-Abschnitt existiert. Der Token wird von `channels.discord.token` aufgelöst, mit `DISCORD_BOT_TOKEN` als Fallback für das Standardkonto (es sei denn, `channels.discord.enabled` ist `false`). Benutze `user:<id>` (DM) oder `channel:<id>` (Gildenkanal) wenn du Lieferziele für Cron/CLI Befehle angibst; blanke numerische IDs sind mehrdeutig und abgelehnt.
Gildenschnecken sind Kleinbuchstaben mit Leerzeichen durch `-`; Kanaltasten verwenden den Namen des verschlungenen Kanals (kein führendes `#`). Bevorzuge Gilden-IDs als Schlüssel, um Mehrdeutigkeit zu vermeiden.
Bot-Authored Nachrichten werden standardmäßig ignoriert. Aktivieren mit `channels.discord.allowBots` (eigene Nachrichten werden immer noch gefiltert, um Self-Antwort-Schleifen zu verhindern).
Reaktionsbenachrichtigungsmodus:

- `off`: keine Reaktionsereignisse.
- `own`: Reaktionen auf eigene Bot-Nachrichten (Standard).
- `all`: alle Reaktionen auf allen Nachrichten.
- `allowlist`: Reaktionen von `guilds.<id>.users` auf allen Nachrichten (leere Liste deaktiviert).
  Ausgehender Text wird durch `channels.discord.textChunkLimit` gechunkelt (Standard 2000). Setze `channels.discord.chunkMode="newline"` um Leerzeilen (Absatzgrenzen) vor dem Chunking zu unterteilen. Discord Clients können sehr hohe Nachrichten clippen, so dass `channels.discord.maxLinesPerMessage` (Standard 17) lange mehrzeilige Antworten auch unter 2000 Zeichen aufteilt.
  Standardwerte und Verhalten der Wiederholungsrichtlinien werden in [Wiederholungsrichtlinien](/concepts/retry) dokumentiert.

### `channels.googlechat` (Chat API webhook)

Google Chat läuft über HTTP-Webhooks mit app-level auth (Service Account).
Der Multi-Account-Support lebt unter `channels.googlechat.accounts` (siehe den oben genannten Multi-Account-Abschnitt). Env vars gelten nur für das Standardkonto.

```json5
{
  Kanäle: {
    googlechat: {
      aktiviert: true
      serviceAccountFile: "/path/to/service-account. son",
      audienceType: "app-url", // app-url | project-number
      audience: "https://gateway.example. om/googlechat",
      webhookPath: "/googlechat",
      botUser: "users/1234567890", // optional; verbessert die Namenserkennung
      dm: {
        aktiviert: true
        Richtlinie: "Paarung", // Paarung | allowlist | offen | deaktiviert
        allowVrom: ["users/1234567890"], // optional; "offen" erfordert ["*"]
      },
      GroupPolicy: "allowlist",
      Gruppen: {
        "spaces/AAAA": { allow: true, requireMention: true },
      },
      Aktionen: { reactions: true },
      Tippen Indikator: "message",
      mediaMaxMb: 20,
    ,
  },
}
```

Hinweise:

- JSON Service Account kann inline (`serviceAccount`) oder dateibasiert (`serviceAccountFile`) sein.
- Env Fallbacks für das Standardkonto: `GOOGLE_CHAT_SERVICE_ACCOUNT` oder `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE`.
- `audienceType` + `audience` muss mit der webhook auth Konfiguration der Chat-App übereinstimmen.
- Verwende `spaces/<spaceId>` oder `users/<userId|email>` beim Setzen von Lieferzielen.

### `channels.slack` (Socket-Modus)

Slack läuft im Socket-Modus und benötigt sowohl einen Bot-Token als auch einen App-Token:

```json5
{
  Kanäle: {
    slack: {
      aktiviert: true
      botToken: "xoxb-. .",
      appToken: "xapp-... ,
      dm: {
        aktiviert: true
        Richtlinie: "Paarung" // Paarung | allowlist | offen | deaktiviert
        allowVrom: ["U123", "U456", "*"], // optional; "offen" erfordert ["*"]
        gruppiert: false,
        GroupChannels: ["G123"],
      },
      Kanäle: {
        C123: { allow: true, requireMention: true, allowBots: false },
        "#general": {
          erlaubt: true
          requireMention: true
          allowBots: falsch,
          Benutzer: ["U123"],
          Fähigkeiten: ["docs"],
          systemPrompt: "Nur kurze Antworten. ,
        },
      },
      Verlauflimit: 50, // füge letzte N Channel/Gruppen-Nachrichten als Kontext hinzu (0 deaktiviert ab)
      allowBots: false,
      Reaktionsbenachrichtigungen: "eigene", // off | own | Alle | allowlist
      reactionAllowlist: ["U123"],
      replyToModus: "aus", // off | zuerst | alle
      Thread: {
        historyScope: "thread", // Thread | Kanal
        inheritParent: falsch,
      },
      Aktionen: {
        Reaktionen: true
        Nachrichten: true
        Pins: true
        memberInfo: true
        emojiList: true
      },
      slashCommand: {
        aktiviert: true
        Name: "openclaw",
        sessionPrefix: "slack:slash",
        ephemeral: true
      },
      textChunkLimit: 4000,
      chunkModus: "length",
      mediaMaxMb: 20,
    },
  },
}
```

Der Multi-Account-Support lebt unter `channels.slack.accounts` (siehe oben der Multi-Account-Abschnitt). Env-Tokens gelten nur für das Standardkonto.

OpenClaw startet Slack wenn der Provider aktiviert ist und beide Token gesetzt sind (via config oder `SLACK_BOT_TOKEN` + `SLACK_APP_TOKEN`). Benutze `user:<id>` (DM) oder `channel:<id>` wenn du Lieferziele für Cron/CLI Befehle angibst.
Setze `channels.slack.configWrites: false` um Slack-initiierte Konfigurationsschreibungen zu blockieren (inklusive Channel-ID-Migrationen und `/config set|unset`).

Bot-Authored Nachrichten werden standardmäßig ignoriert. Aktiviere mit `channels.slack.allowBots` oder `channels.slack.channels.<id>.allowBots`.

Reaktionsbenachrichtigungsmodus:

- `off`: keine Reaktionsereignisse.
- `own`: Reaktionen auf eigene Bot-Nachrichten (Standard).
- `all`: alle Reaktionen auf allen Nachrichten.
- `allowlist`: Reaktionen von `channels.slack.reactionAllowlist` auf alle Nachrichten (leere Liste deaktiviert).

Thread-Sitzungs-Isolierung:

- `channels.slack.thread.historyScope` legt fest, ob der Threadverlauf pro Thread (`thread`, default) oder über den Channel geteilt wird (`channel`).
- `channels.slack.thread.inheritParent` kontrolliert, ob neue Thread-Sessions das übergeordnete Senderprotokoll übernehmen (Standard: falsch).

Slack Aktionengruppen (Schiebe `slack` Werkzeug-Aktionen):

| Aktionsgruppe | Default   | Notes                            |
| ------------- | --------- | -------------------------------- |
| reactions     | aktiviert | Reagieren + Reaktionen auflisten |
| messages      | aktiviert | Lesen/Senden/Bearbeiten/Löschen  |
| pins          | aktiviert | Anpinnen/Entpinnen/Auflisten     |
| memberInfo    | aktiviert | Mitgliederinformationen          |
| emojiList     | aktiviert | Benutzerdefinierte Emoji-Liste   |

### `channels.mattermost` (Bot-Token)

Mattermost wird als Plugin ausgeliefert und ist nicht im Core-Install enthalten.
Installieren Sie es zuerst: `openclaw plugins installieren @openclaw/mattermost` (oder `./extensions/mattermost` aus einem git checkout).

Mattermost benötigt einen Bot-Token plus die Basis-URL für Ihren Server:

```json5
{
  Kanäle: {
    ist wichtig: {
      aktiviert: true
      botToken: "mm-token",
      baseUrl: "https://chat. xample. om",
      dmPolicy: "Paaren",
      Chatmodus: "oncall", // oncall | onmessage | onchar
      oncharPrefixes: [">", "! ],
      textChunkLimit: 4000,
      chunkModus: "length",
    },
  },
}
```

OpenClaw startet Mattermost wenn das Konto konfiguriert ist (Bot-Token + Base-URL) und aktiviert ist. Die Token + Basis-URL wird von `channels.mattermost.botToken` + `channels.mattermost.baseUrl` oder `MATTERMOST_BOT_TOKEN` + `MATTERMOST_URL` für den Standard-Account aufgelöst (es sei denn, `channels.mattermost.enabled` ist `false`).

Chat-Modus:

- `oncall` (Standard): antworte auf Nachrichten nur wenn @mentioned ist.
- `onmessage`: antwortet auf jede Nachricht im Kanal.
- `onchar`: antworten, wenn eine Nachricht mit einem Trigger-Präfix beginnt (`channels.mattermost.oncharPrefixes`, default `[">", "!"]`).

Zugriffskontrolle:

- Standard DMs: `channels.mattermost.dmPolicy="pairing"` (unbekannte Absender erhalten einen Paarcode).
- Öffentliche DMs: `channels.mattermost.dmPolicy="open"` plus `channels.mattermost.allowFrom=["*"]`.
- Gruppen: `channels.mattermost.groupPolicy="allowlist"` standardmäßig (mention-gated). Benutze `channels.mattermost.groupAllowFrom` um Absender zu beschränken.

Der Multi-Account-Support lebt unter `channels.mattermost.accounts` (siehe oben der Multi-Account-Abschnitt). Env vars gelten nur für das Standardkonto.
Benutze `channel:<id>` oder `user:<id>` (oder `@username`), wenn du Lieferziele angibt; blanke Ids werden als Kanal-Ids behandelt.

### `channels.signal` (signal-cli)

Signalreaktionen können Systemereignisse emittieren (gemeinsame Reaktions-Tooling):

```json5
{
  Kanäle: {
    signal: {
      reactionNotifications: "own", // off | own | Alle | allowlist
      reactionAllowlist: ["+15551234567", "uuid:123e4567-e89b-12d3-a456-426614174000"],
      historyLimit: 50, // Füge letzte N Gruppen-Nachrichten als Kontext hinzu (0 deaktiviert)
    },
  },
}
```

Reaktionsbenachrichtigungsmodus:

- `off`: keine Reaktionsereignisse.
- `own`: Reaktionen auf eigene Bot-Nachrichten (Standard).
- `all`: alle Reaktionen auf allen Nachrichten.
- `allowlist`: Reaktionen von `channels.signal.reactionAllowlist` auf alle Nachrichten (leere Liste deaktiviert).

### `channels.imessage` (imsg CLI)

OpenClaw erzeugt `imsg rpc` (JSON-RPC über stdio). Kein Daemon oder Port erforderlich.

```json5
{
  Kanäle: {
    imessage: {
      aktiviert: true
      cliPath: "imsg",
      dbPath: "~/Library/Messages/chat. b",
      remoteHost: "user@gateway-host", // SCP für externe Anhänge bei Verwendung von SSH-Wrapper
      dmPolicy: "Paaren", // Paarung | allowlist | open | deaktiviert
      allowVrom: ["+15555550123", "user@example. om", "chat_id:123"],
      historyLimit: 50, // die letzten N-Gruppen-Nachrichten als Kontext einbeziehen (0 deaktiviert
      includeAttachments: false,
      mediaMaxMb: 16,
      Service: "auto",
      Region: "US",
    },
  },
}
```

Der Multi-Account-Support lebt unter `channels.imessage.accounts` (siehe oben der Multi-Account-Abschnitt).

Hinweise:

- Benötigt vollen Festplattenzugriff auf die Nachrichten DB.
- Der erste Sendevorgang wird nach der Automatisierungsberechtigung für Nachrichten gefragt.
- Bevorzuge `chat_id:<id>` Ziele. Verwende `imsg chats --limit 20` um Chats anzuzeigen.
- `channels.imessage.cliPath` kann auf ein Wrapper-Skript verweisen (z.B. `ssh` auf einen anderen Mac, der `imsg rpc` läuft); benutze SSH-Schlüssel, um Passwort-Eingaben zu vermeiden.
- Setze `channels.imessage.remoteHost` für entfernte SSH-Wrapper, um Anhänge über SCP abzurufen, wenn `includeAttachments` aktiviert ist.

Beispiel-Wrapper:

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

### `agents.defaults.workspace`

Legt das **einzige globale Arbeitsbereichsverzeichnis** fest, das vom Agent für Dateioperationen verwendet wird.

Standard: `~/.openclaw/workspace`.

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
}
```

Wenn `agents.defaults.sandbox` aktiviert ist, können nicht-Haupt-Sitzungen dies mit ihren
eigenen Arbeitsbereichen für jeden Bereich unter `agents.defaults.sandbox.workspaceRoot` überschreiben.

### `agents.defaults.repoRoot`

Optionales Repository-Root, das in der Laufzeitzeile der System-Eingabeaufforderung angezeigt wird. Falls nicht gesetzt, versucht OpenClaw
ein `.git`-Verzeichnis zu erkennen, indem man aus dem Arbeitsbereich (und dem aktuellen
Arbeitsverzeichnis) nach oben geht. Der Pfad muss vorhanden sein, um ihn nutzen zu können.

```json5
{
  Agenten: { defaults: { repoRoot: "~/Projects/openclaw" } },
}
```

### `agents.defaults.skipBootstrap`

Deaktiviert die automatische Erstellung von Bootstrap-Dateien (`AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md` und `BOOTSTRAP.md`).

Verwenden Sie dies für vorinstallierte Installationen, bei denen Ihre Arbeitsbereichsdateien von einem Repo stammen.

```json5
{
  Agenten: { defaults: { skipBootstrap: true } },
}
```

### `agents.defaults.bootstrapMaxChars`

Maximale Anzahl von Zeichen für jeden Workspace-Bootstrap-Datei, die in die System-Eingabeaufforderung
eingefügt wird, bevor sie abgeschnitten wird. Standard: `20000`.

Wenn eine Datei dieses Limit überschreitet, protokolliert OpenClaw eine Warnung und injiziert einen abgeschnittenen
Kopf/Schwanz mit einem Marker.

```json5
{
  Agenten: { defaults: { bootstrapMaxChars: 20000 } },
}
```

### `agents.defaults.userTimezone`

Legt die Zeitzone des Benutzers für den **System-Eingabeaufforderungskontext** fest (nicht für Zeitstempeln in
Nachrichtenumschlägen). Wenn nicht gesetzt, verwendet OpenClaw die Host-Zeitzone zur Laufzeit.

```json5
{
  agents: { defaults: { userTimezone: "America/Chicago" } },
}
```

### `agents.defaults.timeFormat`

Steuert das **Zeitformat** des aktuellen Datums- und Zeitabschnitts der Systemabfrage an.
Standard: `auto` (OS-Einstellung).

```json5
{
  Agenten: { defaults: { timeFormat: "auto" } }, // auto | 12 | 24
}
```

### `nachrichten`

Steuert Inbo/Ausgangspräfixe und optionale ack-Reaktionen.
Siehe [Messages](/concepts/messages) für Warteschlange, Sitzungen und Streaming-Kontext.

```json5
{
  Nachrichten: {
    responsePrefix: "🦞", // oder "auto"
    ackReaction: "👀",
    ackReactionScope: "group-mentions",
    removeAckAfterReply: false,
  },
}
```

`responsePrefix` wird auf **alle ausgehenden Antworten** angewendet (Tool-Zusammenfassungen,
Streaming, endgültige Antworten) über Kanäle hinweg sofern nicht bereits vorhanden ist.

Überschreibungen können pro Kanal und pro Konto konfiguriert werden:

- `channels.<channel>.responsePrefix`
- `channels.<channel>.accounts.<id>.responsePrefix`

Auflösungsreihenfolge (das Spezifischste gewinnt):

1. `channels.<channel>.accounts.<id>.responsePrefix`
2. `channels.<channel>.responsePrefix`
3. `messages.responsePrefix`

Semantik:

- `undefined` fällt auf das nächste Level.
- `""` deaktiviert explizit das Präfix und stoppt die Kaskaden.
- `"auto"` leitet `[{identity.name}]` für den gerouteten Agent ab.

Overrides gelten für alle Kanäle, einschließlich Erweiterungen und für jede ausgehende Antwortart.

Wenn `messages.responsePrefix` nicht gesetzt ist, wird standardmäßig kein Präfix angewendet. WhatsApp Self-Chat
Antworten sind die Ausnahme: Sie sind standardmäßig `[{identity.name}]` wenn gesetzt, andernfalls
`[openclaw]`, damit gleiche Telefongespräche lesbar bleiben.
Setze es auf `"auto"` um `[{identity.name}]` für den Routed Agent abzuleiten (wenn gesetzt).

#### Template-Variablen

Der `responsePrefix` String kann Template-Variablen enthalten, die dynamisch auflösen:

| Variable          | Description                | Beispiel                                          |
| ----------------- | -------------------------- | ------------------------------------------------- |
| `{model}`         | Kurzer Modellname          | `claude-opus-4-6`, `gpt-4o`                       |
| `{modelFull}`     | Vollständige Model-Kennung | `anthropic/claude-opus-4-6`                       |
| `{provider}`      | Name des Anbieters         | `anthropic`, `openai`                             |
| `{thinkingLevel}` | Aktuelle Denkstufe         | `high`, `low`, `off`                              |
| `{identity.name}` | Agenten-Identitätsname     | (identisch mit `"auto"` Modus) |

Variablen sind Groß- und Kleinschreibung (`{MODEL}` = `{model}`). `{think}` ist ein Alias für `{thinkingLevel}`.
Ungelöste Variablen bleiben als wörtlicher Text.

```json5
{
  Nachrichten: {
    responsePrefix: "[{model} | denken:{thinkingLevel}]",
  },
}
```

Beispiel Ausgabe: `[claude-opus-4-6 | think:high] Hier ist meine Antwort...`

WhatsApp eingehendes Präfix ist über `channels.whatsapp.messagePrefix` konfiguriert (veraltet:
`messages.messagePrefix`). Standard bleibt **unverändert**: `"[openclaw]"` wenn
`channels.whatsapp.allowFrom` leer ist, ansonsten `""` (kein Präfix). Wenn
`"[openclaw]"`, wird OpenClaw stattdessen `[{identity.name}]` verwenden, wenn der verteilte
Agent `identity.name` gesetzt hat.

`ackReaction` sendet eine bestmögliche Emoji-Reaktion, um eingehende Nachrichten
auf Kanälen anzuerkennen, die Reaktionen unterstützen (Slack/Discord/Telegram/Google Chat). Standardmäßig wird die `identity.emoji` des aktiven Agenten
gesetzt, andernfalls `"👀"`. Zum Deaktivieren auf `""` gesetzt.

"ackReactionScope" steuert beim Feuern von Reaktionen:

- `group-mentions` (Standard): Nur wenn eine Gruppe/ein Raum Erwähnungen benötigt **und** der Bot erwähnt wurde
- `group-all`: alle Gruppen/Raumnachrichten
- "direct": nur Direktnachrichten
- `alle`: alle Nachrichten

`removeAckAfterReply` entfernt die Antwort des Bots nachdem eine Antwort
gesendet wurde (nur lack/Discord/Telegram/Google Chat). Standard: "falsch".

#### `messages.tts`

Text-zu-Sprache für ausgehende Antworten aktivieren. Wenn aktiviert, generiert OpenClaw Audio
mit ElevenLabs oder OpenAI und fügt es an Antworten an. Telegram verwendet Opus
Sprachnotizen und andere Kanäle senden MP3-Audio.

```json5
{
  Nachrichten: {
    tts: {
      auto: "immer", // off | immer | inbound | tagged
      Modus: "final", // final | all (include tool/block replies)
      provider: "elevenlabs",
      summaryModel: "openai/gpt-4. -mini",
      modelOverrides: {
        enabled: true,
      },
      maxTextLänge: 4000,
      TimeoutMs: 30000,
      PrefsPath: "~/. penclaw/settings/tts. son",
      elvenlabs: {
        apiKey: "elevenlabs_api_key",
        baseUrl: "https://api. levenlabs. o",
        VoiceId: "voice_id",
        modelId: "eleven_multilingual_v2",
        seed: 42,
        applyTextNormalization: "auto",
        Sprachcode: "en",
        Spracheinstellungen: {
          Stabilität: 0. ,
          similarityBoost: 0. 5,
          Stil: 0. ,
          useSpeakerBoost: true
          Geschwindigkeit: 1. ,
        },
      },
      openai: {
        apiKey: "openai_api_key",
        Modell: "gpt-4o-mini-tts",
        Stimme: "alloy",
      },
    },
  },
}
```

Hinweise:

- `messages.tts.auto` steuert die automatische Steuerung des TTS (`off`, `immers`, `inbound`, `tagged`).
- `/tts off|immer|inbound|tagged` setzt den Auto-Modus der Per-Session (überschreibt die Konfiguration).
- `messages.tts.enabled` ist veraltet; Arzt migriert es auf `messages.tts.auto`.
- "prefsPath" speichert lokale Überschreibungen (Provider/Limitierung/Summarize).
- `maxTextLength` ist ein harter Deckel für TTTS-Eingabe. Zusammenfassungen werden abgeschnitten um passend zu sein.
- `summaryModel` überschreibt `agents.defaults.model.primary` für die automatische Zusammenfassung.
  - Akzeptiert `provider/model` oder einen Alias von `agents.defaults.models`.
- `modelOverrides` aktiviert modellgetriebene Überschreibungen wie `[[tts:...]]` Tags (standardmäßig ein).
- `/tts limit` und `/tts summary` steuern die Zusammenfassungseinstellungen pro Benutzer.
- `apiKey` Werte fallen zurück auf `ELEVENLABS_API_KEY`/`XI_API_KEY` und `OPENAI_API_KEY`.
- `elevenlabs.baseUrl` überschreibt die ElevenLabs API Basis-URL.
- `elevenlabs.voiceSettings` unterstützt `stability`/`similarityBoost`/`style` (0..1),
  `useSpeakerBoost` und `speed` (0.5..2.0).

### "Talk"

Standard für den Talk-Modus (macOS/iOS/Android). Sprach-IDs fallen zurück auf `ELEVENLABS_VOICE_ID` oder `SAG_VOICE_ID` wenn sie nicht gesetzt werden.
`apiKey` fällt zurück auf `ELEVENLABS_API_KEY` (oder das Shellprofil des Gateways) wenn nicht gesetzt.
`voiceAliases` lässt Talk Direktiven benutzerfreundliche Namen verwenden (z.B. `"voice":"Clawd"`).

```json5
{
  Talk: {
    voiceId: "elevenlabs_voice_id",
    VoiceAliases: {
      Clawd: "EXAVITQu4vr4xnSDxMaL",
      Roger: "CwhRBWXzGAHq8TQ4Fs17",
    },
    modelId: "eleven_v3",
    outputFormat: "mp3_44100_128",
    apiKey: "elevenlabs_api_key",
    interruptOnSpeech: true
  },
}
```

### `agents.defaults`

Steuert die Embedded Agent Laufzeit (model/thinking/verbose/timeouts).
`agents.defaults.models` definiert den konfigurierten Modellkatalog (und fungiert als allowlist für `/model`).
`agents.defaults.model.primary` setzt das Standardmodell; `agents.defaults.model.fallbacks` sind globale Ausfälle.
`agents.defaults.imageModel` ist optional und wird **nur verwendet, wenn das primäre Modell keine Bildeingabe** hat.
Jeder Eintrag `agents.defaults.models` kann beinhalten:

- `alias` (optionale Modellverknüpfung, z.B. `/opus`).
- `params` (optionale provider-spezifische API-Params wurden an die Modellanfrage weitergeleitet).

`params` wird auch für Streaming Run angewendet (Embedded Agent + Compaction). Unterstützte Schlüssel heute: `temperature`, `maxTokens`. Diese verschmelzen mit Anrufoptionen; die von Anrufern gelieferten Werte gewinnen. `temperature` ist ein fortgeschrittener Knopf. Lassen Sie den Wert außer Sie kennen die Standardwerte des Modells und benötigen eine Änderung.

Beispiel:

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-sonnet-4-5-20250929": {
          params: { temperature: 0.6 },
        },
        "openai/gpt-5. ": {
          Parameter: { maxTokens: 8192 },
        },
      },
    },
  },
}
```

Z.AI GLM-4.x Modelle aktivieren automatisch Denkmodus, außer Sie:

- setze `--thinking off`, oder
- definieren Sie `agents.defaults.models["zai/<model>"].params.thinking` selbst.

OpenClaw liefert auch ein paar integrierte Alias Shorthands. Standardwerte gelten nur, wenn das Modell
bereits in `agents.defaults.models` vorhanden ist:

- `opus` -> `anthropic/claude-opus-4-6`
- `sonnet` -> `anthropic/claude-sonnet-4-5`
- `gpt` -> `openai/gpt-5.2`
- `gpt-mini` -> `openai/gpt-5-mini`
- `gemini` -> `google/gemini-3-pro-preview`
- `gemini-flash` -> `google/gemini-3-flash-preview`

Wenn Sie den gleichen Alias-Namen (Groß- und Kleinschreibung) selbst konfigurieren, gewinnt Ihr Wert (Standardwerte werden nie überschrieben).

Beispiel: Opus 4.6 primär mit MiniMax M2.1 Fallback (gehostetes MiniMax):

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": { alias: "opus" }, { alias: "opus"
        "Minimax/MiniMax-M2. ": { alias: "minimax" },
      },
      Modell: {
        primär: "anthropic/claude-opus-4-6",
        Fallbacks: ["Minimax/MiniMax-M2. "],
      },
    },
  },
}
```

MiniMax auth: Setze `MINIMAX_API_KEY` (env) oder konfiguriere `models.providers.minimax`.

#### `agents.defaults.cliBackends` (CLI Fallback)

Optionale CLI Backends für nur Text-Fallback läuft (keine Werkzeugaufrufe). Diese sind als
Sicherungspfad nützlich, wenn API-Anbieter fehlschlagen. Das Durchführen von Bildern wird unterstützt, wenn Sie
ein `imageArg` konfigurieren, das Dateipfade akzeptiert.

Hinweise:

- CLI Backends sind **text-first**; Werkzeuge sind immer deaktiviert.
- Sessions werden unterstützt, wenn `sessionArg` gesetzt ist; Session-IDs werden pro Backend fortgesetzt.
- Bei `claude-cli`, werden die Standardwerte eingerahmt. Befehlspfad überschreiben, wenn PATH minimale
  ist (start/system).

Beispiel:

```json5
{
  Agenten: {
    defaults: {
      cliBackends: {
        "claude-cli": {
          command: "/opt/homebrew/bin/claude",
        },
        "my-cli": {
          Befehl: "my-cli",
          Args: ["--json"],
          Ausgabe: "json",
          modelArg: "--model",
          sessionArg: "--session",
          Sitzungsmodus: "Bestehen",
          systemPromptArg: "--system",
          systemPromptWannen: "ersten",
          imageArg: "--image",
          Bildmodus: "Wiederholen",
        },
      },
    ,
  },
}
```

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": { alias: "Opus" },
        "anthropic/claude-sonnet-4-1": { alias: "Sonnet" },
        "openrouter/deepseek/deepseek-r1:free": {},
        "zai/glm-4. ": {
          Alias: "GLM",
          Parameter: {
            thinking: {
              type: "enabled",
              clear_thinking: falsch,
            },
          },
        },
      },
      Modell: {
        primär: "anthropic/claude-opus-4-6",
        Fallbacks: [
          "openrouter/deepseek/deepseek-r1:free",
          "openrouter/meta-llama/llama-3. -70b-instruct:free",
        ],
      },
      imageModel: {
        primary: "openrouter/qwen/qwen-2. -vl-72b-instruct:free",
        Fallbacks: ["openrouter/google/gemini-2. -flash-vision:free"],
      },
      thinkingDefault: "low",
      verboseDefault: "off",
      erhöhter Standard: "on",
      TimeoutSekunden: 600,
      mediaMaxMb: 5,
      Herzbeat: {
        every: "30m",
        Ziel: "letzt",
      },
      maxConcurrent: 3,
      Subagenten: {
        model: "minimax/MiniMax-M2. ",
        maxConcurrent: 1,
        archiveAfterMinutes: 60,
      },
      exec: {
        backgroundMs: 10000,
        TimeoutSec: 1800,
        CleanupMs: 1800000,
      },
      KontextTokens: 200000,
    },
  },
}
```

#### `agents.defaults.contextPruning` (Tool-Ergebnis-Schnitt)

`agents.defaults.contextPruning` prallt **alte Werkzeugergebnis** aus dem in-memory Kontext direkt ab, bevor eine Anfrage an die LLM gesendet wird.
Es ändert **nicht** den Sitzungsverlauf auf der Festplatte (`*.jsonl` bleibt vollständig).

Dies soll die Verwendung von Token für Chat-Agenten verringern, die im Laufe der Zeit große Werkzeugausgänge anhäufen.

Hohes Level:

- Benutzer-/Assistent-Nachrichten nicht berühren.
- Schützt die letzten `keepLastAssistants` Nachrichten (keine Tool-Ergebnisse nach diesem Punkt sind gekürzt).
- Schützt das Bootstrap-Präfix (nichts bevor die erste Benutzer-Nachricht gekürzt wird).
- Modus:
  - `adaptive`: Soft-trims oversized tool results (keep head/tail) when the estimated context ratio crosses `softTrimRatio`.
    Hard löscht dann das älteste berechtigte Werkzeugergebnis, wenn das geschätzte Kontextverhältnis `hardClearRatio` **und**
    überquert wird und es genügend prunable Werkzeugergebnisse gibt (`minPrunableToolChars`).
  - `aggressiv`: ersetzt immer berechtigte Werkzeugergebnisse vor dem Ausschneiden durch den `hardClear.placeholder` (keine Ratio-Prüfung).

Weiche gegen harte Beschneidungen (was ändert sich im Kontext an die LLM):

- **Soft-trim**: nur für _überdimensionierte_ Werkzeugergebnisse. Hält den Anfang + Ende und fügt `...` in die Mitte ein.
  - Vorteile: `toolResult("…sehr lange Ausgabe…")`
  - Nach: `toolResult("HEAD…\n...\n…TAIL\n\n[Tool Ergebnis beschnitten: …]")`
- **Hard-clear**: Ersetzt das gesamte Werkzeugergebnis durch den Platzhalter.
  - Vorteile: `toolResult("…sehr lange Ausgabe…")`
  - Nachher: `toolResult("[Old tool result content cleared]")`

Notizen / aktuelle Einschränkungen:

- Werkzeugergebnisse, die **Bildblöcke enthalten, werden übersprungen** (nie geschnitten/geleert) im Moment übersprungen.
- Das geschätzte „Kontext-Verhältnis“ basiert auf **Zeichen** (ungefähr), nicht auf exakten Token.
- Falls die Sitzung noch keine "keepLastAssistants"-Assistentennachrichten enthält, wird das Beschneiden übersprungen.
- Im `aggressive` Modus wird `hardClear.enabled` ignoriert (berechtigte Tool-Ergebnisse werden immer durch `hardClear.placeholder` ersetzt.

Standard (adaptiv):

```json5
{
  Agenten: { defaults: { contextPruning: { mode: "adaptive" } } },
}
```

Zum Deaktivieren:

```json5
{
  Agenten: { defaults: { contextPruning: { mode: "off" } } },
}
```

Standard (wenn `mode` `"adaptive"` oder `"aggressive"`):

- `keepLastAssistants`: `3`
- `softTrimRatio`: `0.3` (nur adaptiv)
- `hardClearRatio`: `0.5` (nur adaptiv)
- `minPrunableToolChars`: `50000` (nur adaptiv)
- `softTrim`: `{ maxChars: 4000, headChars: 1500, tailChars: 1500 }` (nur adaptiv)
- `hardClear`: `{ enabled: true, placeholder: "[Old tool result content cleared]" }`

Beispiel (aggressiv, minimal):

```json5
{
  Agenten: { defaults: { contextPruning: { mode: "aggressive" } } },
}
```

Beispiel (adaptiv getuned):

```json5
{
  Agenten: {
    defaults: {
      contextPruning: {
        Modus: "adaptive",
        keepLastAssistants: 3,
        softTrimRatio: 0. ,
        hardClearRatio: 0. ,
        minPrunableToolChars: 50000,
        SoftTrim: { maxChars: 4000, headChars: 1500, tailChars: 1500 },
        HardClear: { enabled: true Platzhalter: "[Altes Werkzeug Ergebnis gelöscht]" },
        // Optional: Beschneiden auf bestimmte Werkzeuge beschränken (Siege verweigern; unterstützt "*" wildcards)
        tools: { deny: ["browser", "canvas"] },
      },
    },
  },
}
```

Siehe [/concepts/session-pruning](/concepts/session-pruning) für Verhaltensdetails.

#### `agents.defaults.compaction` (Reservekopf + Speicher flush)

`agents.defaults.compaction.mode` wählt die Verdichtungsstrategie aus. Standard ist `default`; Setze `safeguard` um die chunked Zusammenfassung für sehr lange Geschichte zu aktivieren. Siehe [/concepts/compaction](/concepts/compaction).

`agents.defaults.compaction.reserveTokensFloor` erzwingen ein Minimum an `reserveTokens`
Wert für Pi Verdichtung (Standard: `20000`). Setze es auf `0` um den Boden zu deaktivieren.

`agents.defaults.compaction.memoryFlush` führt eine **silent** agentic turn vor
automatische Verdichtung durch, die das Modell anleitet, dauerhafte Speicher auf der Festplatte zu speichern (z.B.
`memory/YYYY-MM-DD.md`). Es wird ausgelöst, wenn die Schätzung des Session-Token einen weichen Schwellenwert von
überschreitet, der unter dem Verdichtungslimit liegt.

Legacy-Standards:

- `memoryFlush.enabled`: `true`
- `memoryFlush.softThresholdTokens`: `4000`
- `memoryFlush.prompt` / `memoryFlush.systemPrompt`: Standard mit `NO_REPLY`
- Hinweis: Memory Flush wird übersprungen, wenn der Arbeitsbereich der Sitzung schreibgeschützt ist
  (`agents.defaults.sandbox.workspaceAccess: "ro"` oder `"none"`).

Beispiel (getun):

```json5
{
  Agenten: {
    defaults: {
      compaction: {
        Modus: "safeguard",
        ReserveTokensFloor: 24000,
        memoryFlush: {
          aktiviert: true
          SoftThresholdToken: 6000,
          systemPrompt: "Session near compaction. Haltbare Erinnerungen jetzt speichern.",
          Prompt: "Schreibe alle bleibenden Notizen in Memory/JJJ-MM-TT. d; antworten Sie mit NO_REPLY, wenn nichts zu speichern. ,
        },
      },
    },
  },
}
```

Block-Streaming:

- `agents.defaults.blockStreamingDefault`: `"on"`/`"off"` (standardmäßig aus).

- Kanal überschreibt: `*.blockStreaming` (und Variante), um Streaming ein/aus zu blockieren.
  Nicht-Telegram-Kanäle benötigen eine explizite `*.blockStreaming: true` um Blockantworten zu aktivieren.

- `agents.defaults.blockStreamingBreak`: `"text_end"` oder `"message_end"` (default: text_end).

- `agents.defaults.blockStreamingChunk`: Weiches Chunken für gestreamte Blöcke. Standardmäßig
  800–1200 Zeichen, bevorzugt Absatz-Breaks (`\n\n`), dann Zeilen, dann Sätze.
  Beispiel:

  ```json5
  {
    Agenten: { defaults: { blockStreamingChunk: { minChars: 800, maxChars: 1200 } } },
  }
  ```

- `agents.defaults.blockStreamingCoalesce`: Blöcke vor dem Senden zusammenführen.
  Standard ist `{ idleMs: 1000 }` und erbt `minChars` von `blockStreamingChunk`
  mit `maxChars` auf das Kanaltext beschränkt. Signal/Slack/Discord/Google Chat Standard
  bis `minChars: 1500` falls nicht überschrieben wird.
  Channel Overrides: `channels.whatsapp.blockStreamingCoalesce`, `channels.telegram.blockStreamingCoalesce`,
  `channels.discord.blockStreamingCoalesce`, `channels.slack.blockStreamingCoalesce`, `channels.mattermost.blockStreamingCoalesce`,
  `channels.signal.blockStreamingCoalesce`, `channels.imessage.blockStreamingCoalesce`, `channels.msteams.blockamingCoalesce`,
  `channels.googlechat.blockamingCoalesce`
  (und per-account Variante).

- `agents.defaults.humanDelay`: zufällige Pause zwischen **Block Antworten** nach dem ersten
  Mode: `off` (Standard), `natural` (800–2500ms), `custom` (benutze `minMs`/`maxMs`).
  Per-agent überschreiben: `agents.list[].humanDelay`.
  Beispiel:

  ```json5
  {
    agents: { defaults: { humanDelay: { mode: "natural" } } },
  }
  ```

  Siehe [/concepts/streaming](/concepts/streaming) für Verhalten + chunking Details.

Eingabeindikatoren:

- `agents.defaults.typingMode`: `"never" | "instant" | "thinking" | "message"`. Standardmäßig
  `instant` für direkte Chats / Erwähnungen und `message` für unerwähnte Gruppenchats.
- `session.typingMode`: Überschreiben von Sitzungen für den Modus.
- `agents.defaults.typingIntervalSeconds`: wie oft das Schreibsignal aktualisiert wird (Standard: 6s).
- `session.typingIntervalSeconds`: pro Session überschreiben für das Aktualisierungsintervall.
  Siehe [/concepts/typing-indicators](/concepts/typing-indicators) für Details zum Verhalten.

`agents.defaults.model.primary` sollte als `provider/model` gesetzt werden (z.B. `anthropic/claude-opus-4-6`).
Aliase kommen von `agents.defaults.models.*.alias` (z.B. `Opus`).
Wenn du den Provider weggelassen hast, nimmt OpenClaw derzeit `anthropic` als temporären
Deprecation Fallback an.
Z.AI Modelle sind als `zai/<model>` (z.B. `zai/glm-4.7`) verfügbar und erfordern
`ZAI_API_KEY` (oder Legacy `Z_AI_API_KEY`) in der Umgebung.

`agents.defaults.heartbeat` konfiguriert periodische Herzbeat-Ausführungen:

- `every`: duration string (`ms`, `s`, `m`, `h`); default unit minutes. Standard:
  `30m`. Setze `0m` zum Deaktivieren.
- `model`: optionales Überschreiben des Modells für Heartbeat Run (`provider/model`).
- `includeReasoning`: Wenn `true`, werden Herzbeats auch die separate `Vernunft:` Nachricht liefern, wenn verfügbar (gleiche Form wie `/reasoning on`). Standard: "falsch".
- `session`: optionaler Sitzungsschlüssel, um zu kontrollieren, in welcher Sitzung das Heartbeat läuft. Standard: `main`.
- `zu`: optionale Empfängerüberschreibung (kanalspezifische ID, z.B. E.164 für WhatsApp, Chat-ID für Telegram).
- `target`: optionaler Lieferkanal (`last`, `whatsapp`, `telegram`, `discord`, `slack`, `msteams`, `signal`, `imessage`, `none`). Standard: `letzt`.
- `prompt`: optionale Überschreibung für den Heartbeat (Standard: `Lesen Sie HEARTBEAT.md wenn er existiert (Arbeitsbereich). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`). Overrides werden wörtlich gesendet. Füge eine `Read HEARTBEAT.md` Zeile hinzu, wenn du die Datei noch lesen möchtest.
- `ackMaxChars`: Maximal erlaubte Zeichen nach `HEARTBEAT_OK` vor Auslieferung (Standard: 300).

Pro-Agent-Heartbeats:

- Setze `agents.list[].heartbeat` um die heartbeat-Einstellungen für einen bestimmten Agent zu aktivieren oder zu überschreiben.
- Wenn irgendein Agenteneintrag `heartbeat` definiert, führen **nur diese Agenten** Herzbeats aus; Standardwert ist
  die gemeinsame Baseline für diese Agenten.

Heartbeats führen vollständige Agent-Turns aus. Kürzere Intervalle brennen mehr Tokens; sei vorsichtig mit
von `all`, halte `HEARTBEAT.md` klein und/oder wähle ein billigeres `Modell`.

`tools.exec` konfiguriert Hintergrund exec Standards:

- `backgroundMs`: Zeit vor dem automatischen Hintergrund (ms, Standard 10000)
- `timeoutSec`: Auto-kill nach dieser Laufzeit (Sekunden, Standard 1800)
- `cleanupMs`: wie lange Sie fertige Sitzungen im Speicher behalten möchten (ms, Standard 1800000)
- `notifyOnExit`: enqueue ein System-Ereignis + request heartbeat wenn backgrounded exec beendet (Standard true)
- `applyPatch.enabled`: Aktiviere experimentelle `apply_patch` (nur OpenAI/OpenAI Codex; Standard falsch)
- `applyPatch.allowModels`: optionale Liste der Model-Ids (z.B. `gpt-5.2` oder `openai/gpt-5.2`)
  Hinweis: `applyPatch` ist nur unter `tools.exec`.

`tools.web` konfiguriert Websuche + Tools abrufen:

- `tools.web.search.enabled` (Standard: true wenn Schlüssel vorhanden ist)
- `tools.web.search.apiKey` (empfohlen: über `openclaw configure --section web` setzen oder `BRAVE_API_KEY` env var) verwenden
- `tools.web.search.maxResults` (1–10, Standard 5)
- `tools.web.search.timeoutSeconds` (Standard 30)
- `tools.web.search.cacheTtlMinutes` (Standard 15)
- `tools.web.fetch.enabled` (Standard true)
- `tools.web.fetch.maxChars` (Standard 50000)
- `tools.web.fetch.maxCharsCap` (Standard 50000; Pratzen maxChars von config/tool calls)
- `tools.web.fetch.timeoutSeconds` (Standard 30)
- `tools.web.fetch.cacheTtlMinutes` (Standard 15)
- `tools.web.fetch.userAgent` (optional Override)
- `tools.web.fetch.readability` (Standard true; Deaktivieren um nur die einfache HTML-Aufräumung zu verwenden)
- `tools.web.fetch.firecrawl.enabled` (Standard true wenn ein API-Schlüssel gesetzt ist)
- `tools.web.fetch.firecrawl.apiKey` (optional; Standard ist `FIRECRAWL_API_KEY`)
- `tools.web.fetch.firecrawl.baseUrl` (Standard [https://api.firecrawl.dev](https://api.firecrawl.dev))
- `tools.web.fetch.firecrawl.onlyMainContent` (Standard true)
- `tools.web.fetch.firecrawl.maxAgeMs` (optional)
- `tools.web.fetch.firecrawl.timeoutSeconds` (optional)

`tools.media` konfiguriert das eingehende Medienverständnis (Bild/Audio/Video):

- `tools.media.models`: shared model list (capability-tagged; used after per-cap lists).
- `tools.media.concurrency`: Maximale gleichzeitige Fähigkeit läuft (Standard 2).
- `tools.media.image` / `tools.media.audio` / `tools.media.video`:
  - `enabled`: Opt-out-Schalter (Standard true wenn Modelle konfiguriert sind).
  - `prompt`: optionale Eingabeaufforderung (image/video append a `maxChars` hint automatically.
  - `maxChars`: Maximale Ausgabezeichen (Standard 500 für Bild/Video; für Audio deaktiviert).
  - `maxBytes`: Maximale Mediengröße zum Senden (Standard: Bild 10MB, Audio 20MB, Video 50MB).
  - `timeoutseconds`: Request Timeout (Standard: Bild 60s, Audio 60s, Video 120s).
  - `language`: optionaler Audio-Hint.
  - `attachments`: Attachment Policy (`mode`, `maxAttachments`, `prefer`).
  - `scope`: optionales Gating (erstes Spiel gewinnt) mit `match.channel`, `match.chatType` oder `match.keyPrefix`.
  - `models`: geordnete Liste der Modelleinträge; Fehler oder übergroße Medien fallen zurück auf den nächsten Eintrag.
- Jeder `models[]` Eintrag:
  - Anbietereintrag (`type: "provider"` oder weggelassen):
    - `provider`: API Provider ID (`openai`, `anthropic`, `google`/`gemini`, `groq`, etc).
    - `model`: model id override (required for image; defaults to `gpt-4o-mini-transcribe`/`whisper-large v3-turbo` for audio providers, and `gemini-3-flash-preview` for video).
    - `profile` / `preferredProfile`: auth profile selection.
  - CLI-Eintrag (`type: "cli"`):
    - `Kommando`: ausführbar zum Ausführen.
    - `args`: Template-args (unterstützt `{{MediaPath}}`, `{{Prompt}}`, `{{MaxChars}}`, etc).
  - `capabilities`: optionale Liste (`image`, `audio`, `video`), um einen freigegebenen Eintrag zu verteilen. Standardeinstellungen: `openai`/`anthropic`/`minimax` → Bild, `google` → image+audio+video, `groq` → Audio.
  - `prompt`, `maxChars`, `maxBytes`, `timeoutSeconds`, `language` kann pro Eintrag überschrieben werden.

Wenn keine Modelle konfiguriert sind (oder `aktiviert: false`) wird das Verständnis übersprungen, das Modell erhält immer noch die ursprünglichen Anhänge.

Der Provider auth order folgt der Standardauth Order (auth profifiles, env vars like `OPENAI_API_KEY`/`GROQ_API_KEY`/`GEMINI_API_KEY`, or `models.providers.*.apiKey`).

Beispiel:

```json5
{
  Tools: {
    media: {
      audio: {
        aktiviert: true
        MaxBytes: 20971520,
        Bereich: {
          default: "deny",
          Regeln: [{ action: "allow", treffen: { chatType: "direct" } }],
        },
        Modelle: [
          { Provider: "openai", model: "gpt-4o-mini-transcribe" },
          { type: "cli", command: "Whisper", args: ["--model", "base", "{{MediaPath}}"] },
        ],
      },
      Video: {
        aktiviert: true
        maxBytes: 52428800,
        Modelle: [{ provider: "google", model: "gemini-3-flash-preview" }],
      },
    },
  },
}
```

`agents.defaults.subagents` konfiguriert Subagent Standards:

- `model`: default model for spawned sub-agents (string or `{ primary, fallbacks }`). Wenn dies weggelassen wird, erben Sub-Agenten das Modell des Anrufers, es sei denn, es wird pro Agent oder pro Aufruf überschrieben.
- `maxConcurrent`: Max. gleichzeitige Sub-Agent läuft (Standard 1)
- `archiveAfterMinutes`: Auto-Archiv-Unter-Agenten-Sitzungen nach N Minuten (Standard 60; Setze `0` auf deaktiviert)
- Per-subagent Werkzeugrichtlinie: `tools.subagents.tools.allow` / `tools.subagents.tools.deny` (Siege verweigern)

`tools.profile` setzt eine **base tool allowlist** bevor `tools.allow`/`tools.deny`:

- `minimal`: nur `session_status`
- `coding`: `group:fs`, `group:runtime`, `group:sessions`, `group:memory`, `image`
- `messaging`: `group:messaging`, `sessions_list`, `sessions_history`, `sessions_send`, `session_status`
- `full`: keine Einschränkung (wie nicht gesetzt)

Pro-Agent-Override: `agents.list[].tools.profile`.

Beispiel (standardmäßig nur Messaging, zusätzlich Slack- und Discord-Werkzeuge erlauben):

```json5
{
  tools: {
    profile: "messaging",
    allow: ["slack", "discord"],
  },
}
```

Beispiel (Coding-Profil, aber exec/process überall verbieten):

```json5
{
  tools: {
    profile: "coding",
    deny: ["group:runtime"],
  },
}
```

`tools.byProvider` erlaubt dir **weitere Einschränkungen** Werkzeuge für bestimmte Provider (oder einen einzelnen `Provider/Modell`).
Pro-Agent-Override: `agents.list[].tools.byProvider`.

Bestellung: Basisprofil → Anbieterprofil → Zulassen/Leugnen von Richtlinien.
Provider-Schlüssel akzeptieren entweder `provider` (z.B. `google-antigravity`) oder `provider/model`
(z.B. `openai/gpt-5.2`).

Beispiel (globales Coding-Profil beibehalten, aber minimale Werkzeuge für Google Antigravity):

```json5
{
  tools: {
    profile: "coding",
    byProvider: {
      "google-antigravity": { profile: "minimal" },
    },
  },
}
```

Beispiel (Provider/modellspezifische Erlaubnisliste):

```json5
{
  tools: {
    allow: ["group:fs", "group:runtime", "sessions_list"],
    byProvider: {
      "openai/gpt-5.2": { allow: ["group:fs", "sessions_list"] },
    },
  },
}
```

`tools.allow` / `tools.deny` konfigurieren ein globales Werkzeug erlauben/leugnen Richtlinie (Siege verweigern).
Passend ist Groß-/Kleinschreibung unbeachtet und unterstützt `*` Platzhalter (`"*"` bedeutet alle Tools).
Dies wird auch dann angewandt, wenn die Docker-Sandbox **aus ist** ist.

Beispiel (Browser/Leinwand überall deaktivieren):

```json5
{
  tools: { deny: ["browser", "canvas"] },
}
```

Werkzeuggruppen (Kurzanhänge) funktionieren in **global** und **pro Agent** Werkzeugrichtlinien:

- `group:runtime`: `exec`, `bash`, `process`
- `group:fs`: `read`, `write`, `edit`, `apply_patch`
- `group:sessions`: `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
- `group:memory`: `memory_search`, `memory_get`
- `group:web`: `web_search`, `web_fetch`
- `group:ui`: `browser`, `canvas`
- `group:automation`: `cron`, `gateway`
- `group:messaging`: `message`
- `group:nodes`: `nodes`
- `group:openclaw`: alle integrierten OpenClaw‑Werkzeuge (ohne Provider‑Plugins)

`tools.elevated` Steuerelemente erhöht (Host) exec Zugriff:

- `aktiviert`: Erlaube erhöhten Modus (Standard true)
- `allowVrom`: Per-channel allowlists (leer = deaktiviert)
  - `whatsapp`: E.164 Zahlen
  - `telegram`: Chat-Ids oder Benutzernamen
  - `discord`: Benutzer-IDs oder Benutzernamen (falls weggelassen) zurück zu `channels.discord.dm.allowFrom`
  - `signal`: E.164 Zahlen
  - `imessage`: handles/chat ids
  - "webchat": Session-IDs oder Benutzernamen

Beispiel:

```json5
{
  Tools: {
    erhöht: {
      aktiviert: true
      allowVrom: {
        whatsapp: ["+15555550123"],
        Diskord: ["steipete", "1234567890123"],
      },
    },
  },
}
```

Per-Agent überschreiben (weitere Einschränkung):

```json5
{
  Agenten: {
    Liste: [
      {
        id: "Familie",
        Werkzeuge: {
          erhöht: { enabled: false },
        },
      },
    ],
  },
}
```

Hinweise:

- `tools.elevated` ist die globale Baseline. `agents.list[].tools.elevated` kann nur weiter eingeschränkt werden (beide müssen es zulassen).
- `/elevated on|off|ask|full` speichert den Status pro Sitzungsschlüssel; Inline-Direktiven gelten für eine einzelne Nachricht.
- Erhöhte `exec` läuft auf dem Host und umgeht Sandboxen.
- Die Tool-Richtlinie gilt noch immer; wenn `exec` verweigert wird, kann erhöhte nicht verwendet werden.

`agents.defaults.maxConcurrent` legt die maximale Anzahl von Embedded-Agenten fest, die
parallel über Sitzungen hinweg ausführen können. Jede Sitzung ist immer noch serialisiert (ein Ausführen
pro Sitzungsschlüssel gleichzeitig). Standard: 1.

### `agents.defaults.sandbox`

Optionales **Docker Sandbox** für den eingebetteten Agent. Geplant für nicht-Haupt-
-Sitzungen, so dass sie nicht auf Ihr Host-System zugreifen können.

Details: [Sandboxing](/gateway/sandboxing)

Standardwerte (wenn aktiviert):

- umfang: `"agent"` (ein Container + Arbeitsbereich pro Agent)
- Debian Bookworm-Slim-basiertes Bild
- agent Arbeitsbereich-Zugriff: `workspaceAccess: "none"` (Standard)
  - `"Keine"`: verwende einen pro scope sandbox Arbeitsbereich unter `~/.openclaw/sandboxes`
- `"ro"`: behalte den sandbox-Arbeitsbereich bei `/workspace`, und mounte den Agent-Arbeitsbereich schreibgeschützt bei `/agent` (deaktiviert `write`/`edit`/`apply_patch`)
  - `"rw"`: mounte den Agent-Arbeitsbereich lesen/schreiben auf `/workspace`
- auto-Prune: Idle > 24 h ODER Alter > 7 d
- tool Policy: allow only `exec`, `process`, `read`, `write`, `edit`, `apply_patch`, `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status` (deny wins)
  - via `tools.sandbox.tools` konfigurieren, über `agents.list[].tools.sandbox.tools` pro Agent überschreiben
  - Werkzeug Gruppe Kurzbefehle unterstützt in der Sandbox-Richtlinie: `group:runtime`, `group:fs`, `group:sessions`, `group:memory` (siehe [Sandbox vs Tool Policy vs Elevated](/gateway/sandbox-vs-tool-policy-vs-elevated#tool-groups-shorthands))
- optional belegter Browser (Chromium + CDP, noVNC Beobachter)
- hardening knobs: `network`, `user`, `pidsLimit`, `memory`, `cpus`, `ulimits`, `seccompProfile`, `apparmorProfile`

Warnung: `scope: "shared"` means a shared container and shared workspace. Keine
Sitzungsübergreifende Isolierung. Verwende `scope: "session"` für die Isolierung pro Sitzung.

Legacy: `perSession` wird immer noch unterstützt (`true` → `scope: "session"`,
`false` → `scope: "shared"`).

`setupCommand` läuft **einmal** nachdem der Container erstellt wurde (innerhalb des Containers über `sh -lc`).
Bei Paketinstallationen stellen Sie sicher, dass Netzwerk-Egress, ein beschreibbarer root FS und ein root-Benutzer installiert werden.

```json5
{
  Agenten: {
    defaults: {
      sandbox: {
        Modus: "non-main", // off | non-main | alle
        Bereich: "agent", // Session | Agent | shared (Agent ist Standard)
        WorkspaceAccess: "Keine", // none | ro | rw
        workspaceRoot: "~/. penclaw/sandboxes",
        docker: {
          image: "openclaw-sandbox:bookworm-slim",
          containerPrefix: "openclaw-sbx-",
          Arbeitsverzeichnis: "/workspace",
          readOnlyRoot: true
          tmpfs: ["/tmp", "/var/tmp", "/run"],
          Netzwerk: "keine",
          Benutzer: "1000:1000",
          capDrop: ["ALL"],
          env: { LANG: "C. TF-8" },
          setupCommand: "apt-get update && apt-get install -y git curl jq",
          // Per-Agent überschreiben (Multi-Agent): Agenten. ist[].sandbox.docker.
          pidsLimit: 256,
          Speicher: "1g",
          memorySwap: "2g",
          cpus: 1,
          Ulimits: {
            nofile: { soft: 1024, hard: 2048 },
            nproc: 256,
          },
          seccompProfile: "/path/to/seccomp. son",
          apparmorProfile: "openclaw-sandbox",
          dns: ["1. .1.1", "8.8.8. "],
          extraHosts: ["internal.service:10.0.0. "],
          Bindungen: ["/var/run/docker.sock:/var/run/docker. ock", "/home/user/source:/source:rw"],
        },
        Browser: {
          aktiviert: falsch,
          Bild: "openclaw-sandbox-browser:bookworm-slim",
          containerPrefix: "openclaw-sbx-browser-",
          cdpPort: 9222,
          vncPort: 5900,
          noVncPort: 6080,
          kopflos: falsch,
          aktivieren NoVnc: true
          allowHostControl: falsch,
          allowedControlUrls: ["http://10. .0.42:18791"],
          allowedControlHosts: ["browser.lab.local", "10.0.0. 2"],
          erlaubte ControlPorts: [18791],
          AutoStart: wahr
          autoStartTimeoutMs: 12000,
        },
        prune: {
          idleHours: 24, // 0 deaktiviert Leerlaufschnitt
          maxAgeDays: 7, // 0 deaktiviert Max-Age-Abschneiden
        },
      },
    },
  },
  tools: {
    sandbox: {
      tools: {
        allow: [
          "exec",
          "Prozess",
          "gelesen",
          "Schreiben",
          "Bearbeiten",
          "apply_patch",
          "sessions_list",
          "sessions_history",
          "sessions_send",
          "sessions_spawn",
          "session_status",
        ],
        verweigern: ["Browser", "canvas", "nodes", "cron", "discord", "gateway"],
      },
    },
  },
}
```

Erstellen Sie einmal das Standard-Sandbox-Bild mit:

```bash
scripts/sandbox-setup.sh
```

Hinweis: Sandbox-Container standardmäßig zu `network: "none"; setze `agents.defaults.sandbox.docker.network`auf`"bridge"\` (oder dein benutzerdefiniertes Netzwerk), wenn der Agent ausgehenden Zugriff benötigt.

Hinweis: Eingehende Anhänge werden im aktiven Arbeitsbereich unter `media/inboand/*` inszeniert. Mit \`workspaceAccess: "rw" bedeutet, dass Dateien in den Agenten-Arbeitsbereich geschrieben werden.

Hinweis: `docker.binds` mounts additional host directories; global and per-agent binds are merged.

Erstelle das optionale Browserbild mit:

```bash
scripts/sandbox-browser-setup.sh
```

Wenn `agents.defaults.sandbox.browser.enabled=true` verwendet das Browser-Tool eine Sandboxed
Chromium-Instanz (CDP). Wenn noVNC aktiviert ist (Standard, wenn headless=false), wird die noVNC-URL in den System-Prompt eingefügt, damit der Agent darauf Bezug nehmen kann.
Dies erfordert keine `browser.enabled` in der Hauptkonfiguration; die Sandbox-Steuerung
URL wird pro Sitzung injiziert.

`agents.defaults.sandbox.browser.allowHostControl` (Standard: falsch) erlaubt es
gesandelte Sessions explizit auf den **host** Browser-Kontrollserver
über das Browser-Tool (`target: "host"`). Lassen Sie dies aus, wenn Sie eine strikte
Sandbox Isolierung wünschen.

Erlaubte Listen für Fernbedienung:

- `allowedControlUrls`: exakte Kontroll-URLs erlaubt für `target: "custom"`.
- `allowedControlHosts`: Hostnamen erlaubt (nur Hostname, kein Port).
- `allowedControlPorts`: Ports erlaubt (Standard: http=80, https=443).
  Standards: Alle Zulassungslisten sind nicht gesetzt (ohne Einschränkung). `allowHostControl` Standard ist falsch.

### `models` (benutzerdefinierte Provider + Basis-URLs)

OpenClaw verwendet den **pi-coding-Agent** Modellkatalog. Sie können benutzerdefinierte Anbieter
(LiteLLM, lokale OpenAI-kompatible Server, Anthropische Proxies, etc.) hinzufügen indem Sie
`~/.openclaw/agents/<agentId>/agent/models.json` schreiben oder das gleiche Schema innerhalb Ihrer
OpenClaw-Konfiguration unter `models.providers` definieren.
Provider-by-Provider Übersicht + Beispiele: [/concepts/model-providers](/concepts/model-providers).

Wenn `models.providers` vorhanden ist, schreibt OpenClaw eine `models.json` in
`~/.openclaw/agents/<agentId>/agent/` beim Start:

- Standardverhalten: **merge** (bestehende Provider, Überschreibungen beim Namen)
- setze `models.mode: "replace"` um den Dateiinhalt zu überschreiben

Wählen Sie das Modell über `agents.defaults.model.primary` (Provider/Modell).

```json5
{
  Agenten: {
    defaults: {
      model: { primary: "custom-proxy/llama-3. -8b" },
      Modelle: {
        "custom-proxy/llama-3. -8b": {},
      },
    },
  },
  Modelle: {
    Modus: "merge",
    Providers: {
      "custom-proxy": {
        baseUrl: "http://localhost:4000/v1",
        apiKey: "LITELLM_KEY",
        api: "openai-completions",
        Modelle: [
          {
            id: "llama-3. -8b",
            Name: "Llama 3. 8B",
            Argumentation: falsch,
            Eingabe: ["Text"],
            Kosten: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            KontextFenster: 128000,
            maxTokens: 32000,
          },
        ],
      },
    },
  },
}
```

### OpenCode Zen (Multi-Model-Proxy)

OpenCode Zen ist ein multimodelles Gateway mit modellen Endpunkten. OpenClaw verwendet
den eingebauten `opencode` Provider von pi-ai; setze `OPENCODE_API_KEY` (oder
`OPENCODE_ZEN_API_KEY`) von [https://opencode.ai/auth](https://opencode.ai/auth).

Hinweise:

- Modell verweigert `opencode/<modelId>` (Beispiel: `opencode/claude-opus-4-6`).
- Wenn Sie eine allowlist über `agents.defaults.models` aktivieren, fügen Sie jedes Modell hinzu, das Sie verwenden möchten.
- Shortcut: `openclaw onboard --auth-choice opencode-zen`.

```json5
{
  agents: {
    defaults: {
      model: { primary: "opencode/claude-opus-4-6" },
      Modelle: { "opencode/claude-opus-4-6": { alias: "Opus" } },
    },
  },
}
```

### Z.AI (GLM-4.7) — Provider-Alias-Unterstützung

Z.AI-Modelle sind über den integrierten "zai"-Anbieter erhältlich. Setze `ZAI_API_KEY`
in deiner Umgebung ein und referenziere das Modell nach Provider/Modell.

Shortcut: `openclaw onboard --auth-choice zai-api-key`.

```json5
{
  agents: {
    defaults: {
      model: { primary: "zai/glm-4.7" },
      models: { "zai/glm-4.7": {} },
    },
  },
}
```

Hinweise:

- `z.ai/*` und `z-ai/*` sind akzeptierte Aliase und normalisieren sich zu `zai/*`.
- Falls `ZAI_API_KEY` fehlt, werden Anfragen an `zai/*` mit einem Authentifizierungsfehler zur Laufzeit fehlschlagen.
- Beispielfehler: `Kein API-Schlüssel für Anbieter "zai".` gefunden
- Der allgemeine API-Endpunkt von Z.AI ist `https://api.z.ai/api/paas/v4`. GLM-Codierung
  Anfragen verwenden den dedizierten Coding Endpunkt `https://api.z.ai/api/coding/paas/v4`.
  Der eingebaute `zai` Provider verwendet den Coding Endpunkt. Wenn du den allgemeinen
  Endpunkt brauchst, definiere einen benutzerdefinierten Provider in `models.providers` mit der Basis-URL
  (s. Abschnitt über benutzerdefinierte Provider oben).
- Verwenden Sie einen gefälschten Platzhalter in docs/configs; begeben Sie niemals echte API-Schlüssel.

### Moonshot AI (Kimi)

Verwenden Sie den OpenAI-kompatiblen Endpunkt von Moonshot:

```json5
{
  env: { MOONSHOT_API_KEY: "sk-... },
  Agenten: {
    defaults: {
      model: { primary: "moonshot/kimi-k2. },
      Modelle: { "moonshot/kimi-k2. ": { alias: "Kimi K2. " } },
    },
  },
  Modelle: {
    Modus: "merge",
    Providers: {
      moonshot: {
        baseUrl: "https://api. oonshot. i/v1",
        apiKey: "${MOONSHOT_API_KEY}",
        api: "openai-completions",
        Modelle: [
          {
            id: "kimi-k2. ",
            Name: "Kimi K2. ",
            Argumentation: falsch,
            Eingabe: ["Text"],
            Kosten: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            KontextFenster: 256000,
            MaxToken: 8192,
          },
        ],
      },
    },
  },
}
```

Hinweise:

- Setze `MOONSHOT_API_KEY` in der Umgebung oder verwende `openclaw an Bord --auth-choice moonshot-api-key`.
- Modell ref: `moonshot/kimi-k2.5`.
- Auch für den Endpunkt China:
  - Führe `openclaw an Bord --auth-choice moonshot-api-key-cn` aus (Assistent setzt `https://api.moonshot.cn/v1`), oder
  - Lege `baseUrl: "https://api.moonshot.cn/v1"` in `models.providers.moonshot`.

### Kimi Coding

Verwenden Sie Moonshot AI's Kimi Coding Endpunkt (Anthropic-kompatibel, integrierter Provider):

```json5
{
  env: { KIMI_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "kimi-coding/k2p5" },
      models: { "kimi-coding/k2p5": { alias: "Kimi K2.5" } },
    },
  },
}
```

Hinweise:

- Setze `KIMI_API_KEY` in der Umgebung oder verwende `openclaw an Bord --auth-choice kimi-code-api-key`.
- Modell ref: `kimi-coding/k2p5`.

### Synthetisch (Anthropic-kompatibel)

Synthetic's Anthropic-kompatiblen Endpunkt verwenden:

```json5
{
  env: { SYNTHETIC_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "synthetic/hf:MiniMaxAI/MiniMax-M2.1" },
      models: { "synthetic/hf:MiniMaxAI/MiniMax-M2.1": { alias: "MiniMax M2.1" } },
    },
  },
  models: {
    mode: "merge",
    providers: {
      synthetic: {
        baseUrl: "https://api.synthetic.new/anthropic",
        apiKey: "${SYNTHETIC_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "hf:MiniMaxAI/MiniMax-M2.1",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 192000,
            maxTokens: 65536,
          },
        ],
      },
    },
  },
}
```

Hinweise:

- Setze `SYNTHETIC_API_KEY` oder benutze `openclaw an board --auth-choice synthetic-api-key`.
- Modell ref: `synthetic/hf:MiniMaxAI/MiniMax-M2.1`.
- Die Basis-URL sollte `/v1` weglassen, da der anthropische Client sie anfügt.

### Lokale Modelle (LM Studio) — empfohlene Einrichtung

Siehe [/gateway/local-models](/gateway/local-models) für die aktuelle lokale Anleitung. TL;DR: Starten Sie MiniMax M2.1 über LM Studio Responses API auf seriöser Hardware; halten Sie gehostete Modelle fusioniert für Fallback.

### MiniMax M2.1

MiniMax M2.1 direkt ohne LM Studio verwenden:

```json5
{
  agent: {
    model: { primary: "minimax/MiniMax-M2. " },
    Modelle: {
      "anthropic/claude-opus-4-6": { alias: "Opus" },
      "Minimax/MiniMax-M2. ": { alias: "Minimax" },
    },
  },
  Modelle: {
    Modus: "merge",
    Providers: {
      minimax: {
        baseUrl: "https://api. inimax. o/anthropic",
        apiKey: "${MINIMAX_API_KEY}",
        api: "anthropic-messages",
        Modelle: [
          {
            id: "MiniMax-M2. ",
            Name: "MiniMax M2. ",
            Argumentation: falsch,
            Eingabe: ["Text"],
            // Preise: Update in Modellen. son wenn Sie eine exakte Kostenverfolgung benötigen.
            Kosten: { input: 15, output: 60, cacheRead: 2, cacheWrite: 10 },
            contextWindow: 200000,
            MaxToken: 8192,
          },
        ],
      },
    },
  },
}
```

Hinweise:

- Setze `MINIMAX_API_KEY` Umgebungsvariable oder verwende `openclaw an Bord --auth-choice minimax-api`.
- Verfügbares Modell: `MiniMax-M2.1` (Standard).
- Aktualisiere die Preisgestaltung in `models.json` wenn du eine exakte Kostenverfolgung benötigst.

### Cerebras (GLM 4.6 / 4.7)

Cerebras über ihren OpenAI-kompatiblen Endpunkt verwenden:

```json5
{
  env: { CEREBRAS_API_KEY: "sk-... },
  Agenten: {
    defaults: {
      model: {
        primary: "cerebras/zai-glm-4. ",
        Fallbacks: ["cerebras/zai-glm-4. "],
      },
      Modelle: {
        "cerebras/zai-glm-4. ": { alias: "GLM 4.7 (Cerebras)" },
        "cerebras/zai-glm-4.6": { alias: "GLM 4. (Cerebras)" },
      },
    },
  },
  Modelle: {
    Modus: "merge",
    Providers: {
      cerebras: {
        baseUrl: "https://api. erebras. i/v1",
        apiKey: "${CEREBRAS_API_KEY}",
        api: "openai-completions",
        Modelle: [
          { id: "zai-glm-4. ", name: "GLM 4. (Cerebras)" },
          { id: "zai-glm-4.6", name: "GLM 4. (Cerebras)" },
        ],
      },
    },
  },
}
```

Hinweise:

- Benutze `cerebras/zai-glm-4.7` für Cerebras; verwende `zai/glm-4.7` für Z.AI direct.
- Setze `CEREBRAS_API_KEY` in der Umgebung oder in der Konfiguration.

Hinweise:

- Unterstützte APIs: `openai-completions`, `openai-responses`, `anthropic-messages`,
  `google-generative-ai`
- Benutze `authHeader: true` + `headers` für benutzerdefinierte Authentifizierungsbedürfnisse.
- Überschreiben Sie das Agent config root mit `OPENCLAW_AGENT_DIR` (oder `PI_CODING_AGENT_DIR`)
  wenn Sie `models.json` woanders gespeichert haben möchten (Standard: `~/.openclaw/agents/main/agent`).

### `sitzung`

Steuert Session-scoping, Resetrationsrichtlinie, Trigger zurücksetzen und wo der Session-Shop geschrieben wird.

```json5
{
  session: {
    scope: "per-sender",
    dmScope: "main",
    identityLinks: {
      alice: ["telegram:123456789", "discord:987654321012345678"],
    },
    reset: {
      mode: "daily",
      atHour: 4,
      idleMinutes: 60,
    },
    resetByType: {
      thread: { mode: "daily", atHour: 4 },
      direct: { mode: "idle", idleMinutes: 240 },
      group: { mode: "idle", idleMinutes: 120 },
    },
    resetTriggers: ["/new", "/reset"],
    // Default is already per-agent under ~/.openclaw/agents/<agentId>/sessions/sessions.json
    // You can override with {agentId} templating:
    store: "~/.openclaw/agents/{agentId}/sessions/sessions.json",
    // Direct chats collapse to agent:<agentId>:<mainKey> (default: "main").
    mainKey: "main",
    agentToAgent: {
      // Max ping-pong reply turns between requester/target (0–5).
      maxPingPongTurns: 5,
    },
    sendPolicy: {
      rules: [{ action: "deny", match: { channel: "discord", chatType: "group" } }],
      default: "allow",
    },
  },
}
```

Felder:

- `mainKey`: Direkt-Chat Bucket Key (Standard: `"main"`). Nützlich, wenn Sie den primären DM-Thread „umbenennen“ wollen, ohne `agentId` zu ändern.
  - Sandbox-Notiz: `agents.defaults.sandbox.mode: "non-main"` verwendet diesen Schlüssel, um die Hauptsitzung zu erkennen. Jeder Session-Schlüssel, der nicht mit `mainKey` (Gruppen/Kanäle) übereinstimmt, ist in Sandkasten.
- `dmScope`: Wie DM-Sitzungen gruppiert werden (Standard: `"main"`).
  - `main`: alle DMs teilen die Haupt-Sitzung für Kontinuität.
  - `per-peer`: Isoliere DMs durch Absender-ID über Kanäle.
  - `per-channel-peer`: Isolate DMs pro Kanal + Sender (empfohlen für Multi User Inboxes).
  - `per-account-channel-peer`: Isolate DMs pro Konto + Kanal + Absender (empfohlen für Multikonto-Postfach).
  - Sicherer DM-Modus (empfohlen): Setze `session.dmScope: "per-channel-peer"` wenn mehrere Personen den Bot DM können (freigegebene Posteingänge, Mehrpersonen-Erlaubnislisten oder `dmPolicy: "open"`).
- `identityLinks`: Ordnen Sie kanonische Ids den Provider-Prefix Peers zu, so dass dieselbe Person eine DM-Sitzung über Kanäle hinweg teilt, wenn sie `per-peer`, `per-channel-peer`, oder `per-account-channel-peer`.
  - Beispiel: `alice: ["telegram:123456789", "discord:987654321012345678"]`.
- `reset`: Primäres Reset-Richtlinie. Standardmäßig wird täglich um 4:00 Uhr Ortszeit auf dem Gateway-Host zurückgesetzt.
  - `mode`: `daily` oder `idle` (Standard: `daily` wenn `reset` vorhanden ist).
  - `atHour`: local hour (0-23) for the daily reset boundary.
  - `idleMinutes`: Schiebe inaktives Fenster in Minuten. Wenn täglich + Leerlauf beide konfiguriert sind, gewinnt der zuerst ablaufende.
- `resetByType`: per-session overrides for `direct`, `group`, and `thread`. Legacy `dm` key is accepted as an alias for `direct`.
  - Wenn du nur Legacy `session.idleMinutes` ohne `reset`/`resetByType` setzst, bleibt OpenClaw im Nur-Modus für Abwärtskompatibilität.
- `heartbeatIdleMinutes`: optionale Leerlauf-Überschreibung für Heartbeat-Prüfungen (tägliches Zurücksetzen gilt immer noch, wenn aktiviert).
- `agentToAgent.maxPingPongTurns`: Maximale Antwortrückgänge zwischen Anfrage/Ziel (0–5, Standard 5).
- `sendPolicy.default`: `allow` oder `deny` Fallback wenn keine Regel übereinstimmt.
- `sendPolicy.rules[]`: Übereinstimmung mit `channel`, `chatType` (`direct|group|room`) oder `keyPrefix` (z.B. `cron:`). Zuerst leugnen Siege; andernfalls erlauben.

### `skills` (Skill-Konfiguration)

Steuert gebündelte Erlaubnisliste, Installationseinstellungen, zusätzliche Fähigkeitsordner und Überschreibungen für Fertigkeiten
. Gilt für **gebündelte** Fähigkeiten und `~/.openclaw/skills` (Arbeitsbereichsfähigkeiten
gewinnen immer noch bei Namenskonflikten).

Felder:

- `allowBundled`: optionale Allowlist nur für **gebündelte** Skills. Wenn aktiviert, sind nur diese
  gebündelten Fähigkeiten förderfähig (bewirtschaftet/bewirtschaftet ohne Beeinträchtigung).
- `load.extraDirs`: zusätzliche Skill-Verzeichnisse, die gescannt werden (niedrigste Priorität).
- `install.preferBrew`: bevorzugt Brew-Installer, wenn verfügbar (Standard: true).
- `install.nodeManager`: node installer Einstellungen (`npm` | `pnpm` | `yarn`, default: npm).
- `entries.<skillKey>`: Per-Skill config überschreibt.

Skill-spezifische Felder:

- `enabled`: setzen Sie `false`, um einen Skill zu deaktivieren, auch wenn er gebündelt/installiert ist.
- `env`: Umgebungsvariablen, die für den Agent-Lauf injiziert werden (nur wenn sie noch nicht gesetzt sind).
- `apiKey`: optionale Bequemlichkeit für Fähigkeiten, die eine primäre env var erklären (z.B. `nano-banana-pro` → `GEMINI_API_KEY`).

Beispiel:

```json5
{
  Skills: {
    allowBundled: ["gemini", "peekaboo"],
    Laden: {
      ExtraDirs: ["~/Projects/agent-scripts/skills", "~/Projects/oss/some skill-pack/skills"],
    },
    install: {
      preferBrew: true
      nodeManager: "npm",
    },
    Einträge: {
      "nano-banana-pro": {
        apiKey: "GEMINI_KEY_HERE",
        env: {
          GEMINI_API_KEY: "GEMINI_KEY_HERE",
        },
      },
      peekaboo: { enabled: true },
      sag: { enabled: false },
    },
  },
}
```

### `plugins` (Erweiterung)

Steuert das Plugin entdecken, zulassen/verweigern und die Konfiguration pro Plugin. Plugins werden
von `~/.openclaw/extensions`, `<workspace>/.openclaw/extensions`, sowie allen
`plugins.load.paths` Einträgen geladen. **Änderungen der Konfiguration erfordern einen Neustart des Gateways.**
Siehe [/plugin](/tools/plugin) für die volle Nutzung.

Felder:

- `enabled`: Master-Schalter für das Laden des Plugins (Standard: true).
- `allow`: optional allowlist of plugin ids; when set only listed plugins load.
- `deny`: optionale Denylist der Plugin Ids (deny wins).
- `load.paths`: zusätzliche Plugin-Dateien oder Verzeichnisse zum Laden (absolut oder `~`).
- `Einträge.<pluginId>`: pro Plugin überschreibt.
  - `enabled`: Setze `false` auf deaktivieren.
  - `config`: plugin-specific config object (validated by the plugin if provided).

Beispiel:

```json5
{
  Plugins: {
    aktiviert: true,
    erlaubt: ["Voice-call"],
    load: {
      paths: ["~/Projects/oss/voice-call-extension"],
    },
    Einträge: {
      "Voice-call": {
        aktiviert: true
        config: {
          Provider: "twilio",
        },
      },
    },
  },
}
```

### "browser" (openclaw-verwalteter Browser)

OpenClaw kann eine **engagierte, isolierte** Chrome/Brave/Edge/Chromium Instanz für openclaw starten und einen kleinen Schleifenkontrolldienst ausblenden.
Profile können auf einen **entfernten** Chromium-Browser über `Profilen verweisen.<name>.cdpUrl`. Remote
Profile sind nur angehängt (starten/stoppen/zurücksetzen sind deaktiviert).

`browser.cdpUrl` bleibt für Legacy Single-Profil-Konfigurationen und als Basis
Schema/Host für Profile, die nur `cdpPort` setzen.

Standardwerte:

- aktiviert: `true`
- evaluiert: `true` (setze `false` auf `act:evaluate` und `wait --fn`)
- Kontrolldienst: nur loopback (Port abgeleitet von `gateway.port`, Standard `18791`)
- CDP-URL: `http://127.0.0.1:18792` (Kontrolldienst + 1, Legacy Single-Profil)
- Profilfarbe: `#FF4500` (lobster-orange)
- Hinweis: Der Kontrollserver wird über das laufende Gateway (OpenClaw.app Menüleiste oder "openclaw gateway") gestartet.
- Reihenfolge automatisch erkennen: Standardbrowser wenn Chromium-basiert; sonst Chrome → Brave → Kante → Chromium → Chrome Canary.

```json5
{
  Browser: {
    aktiviert: true
    ausgewertet Aktiviert: true
    // cdpUrl: "http://127. .0. :18792", // Legacy Einzelprofil überschreiben
    defaultProfil: "chrome",
    Profile: {
      openclaw: { cdpPort: 18800, color: "#FF4500" },
      work: { cdpPort: 18801, color: "#0066CC" },
      remote: { cdpUrl: "http://10. .0.42:9222", Farbe: "#00AA00" },
    },
    Farbe: "#FF4500",
    // Erweitert:
    // kopflos: falsch,
    // noSandbox: falsch,
    // executablePath: "/Applications/Brave Browser. pp/Contents/MacOS/Brave Browser",
    // attachly: falsch, // Setze true beim Tunnelieren eines entfernten CDP auf localhost
  },
}
```

### `ui` (Erscheinung)

Optionale Akzentfarbe, die von den nativen Apps für UI-Chrom verwendet wird (z.B. Sprechblase im Talk-Modus).

Wenn nicht gesetzt, fallen Kunden zurück auf ein stummgeschaltetes hellblau.

```json5
{
  ui: {
    seamColor: "#FF4500", // hex (RRGGBB oder #RRGGBB)
    // Optional: Kontrolle der UI-Assistentenidentität überschreiben.
    // Wenn nicht gesetzt, verwendet das Kontroll-UI die aktive Agenten-Identität (Konfiguration oder IDENTITY. d).
    Assistent: {
      Name: "OpenClaw",
      Avatar: "CB", // Emoji, Kurztext oder Bild URL/Daten URI
    },
  },
}
```

### `gateway` (Gateway Server Mode + bind)

Benutze `gateway.mode` um explizit anzugeben, ob diese Maschine das Gateway ausführen soll.

Standardwerte:

- Modus: **unset** (behandelt als “Nicht automatisch starten”)
- bind: `loopback`
- port: `18789` (einzelner Port für WS + HTTP)

```json5
{
  Gateway: {
    Modus: "local", // oder "remote"
    Port: 18789, // WS + HTTP Multiplex
    Bind: "loopback",
    // controlUi: { enabled: true basePath: "/openclaw" }
    // auth: { mode: "token", token: "your-token" } // token gates WS + Control UI access
    // tailscale: { mode: "off" | "serve" | "funnel" }
  },
}
```

UI Basispfad kontrollieren:

- `gateway.controlUi.basePath` legt das URL-Präfix fest, in dem das Control UI verwendet wird.
- Beispiele: `"/ui"`, `"/openclaw"`, `"/apps/openclaw"`.
- Standard: root (`/`) (unverändert).
- `gateway.controlUi.root` setzt das Dateisystem root für Control UI Assets (Standard: `dist/control-ui`).
- `gateway.controlUi.allowInsecureAuth` erlaubt token-only auth für die Kontroll-Benutzeroberfläche, wenn die Identität des
  Geräts weggelassen wird (typischerweise über HTTP). Standard: "falsch". Bevorzuge HTTPS
  (Tailscale Serve) oder `127.0.0.1`.
- `gateway.controlUi.dangerlyDisableDeviceAuth` deaktiviert die Überprüfung der Geräteidentität für die
  Kontroll-UI (nur Token / Passwort). Standard: "falsch". Nur Break-Glas.

Verwandte Dokumente:

- [Control UI](/web/control-ui)
- [Web-Übersicht](/web)
- [Tailscale](/gateway/tailscale)
- [Remote-Zugriff](/gateway/remote)

Vertrauenswürdige Proxies:

- `gateway.trustedProxies`: Liste der Reverse Proxy IPs, die TLS vor dem Gateway beenden.
- Wenn eine Verbindung von einer dieser IPs kommt, OpenClaw verwendet `x-forwarded-for` (oder `x-real-ip`), um die Client-IP für lokale Paarungsüberprüfungen und HTTP-Authentifizierung/lokale Prüfungen zu ermitteln.
- Listet nur Proxies auf, die du vollständig kontrollierst und stellt sicher, dass sie **überschreiben** eingehende `x-forwarded-for` sind.

Hinweise:

- `openclaw gateway` weigert sich, zu starten, es sei denn, `gateway.mode` ist auf `local` gesetzt (oder du überschreibst die Flag).
- `gateway.port` steuert den einzelnen Multiplex-Port für WebSocket + HTTP (Kontroll-UI, Haken, A2UI).
- OpenAI Chat Completions Endpunkt: **standardmäßig deaktiviert**; aktivieren Sie mit `gateway.http.endpoints.chatCompletions.enabled: true`.
- Precedence: `--port` > `OPENCLAW_GATEWAY_PORT` > `gateway.port` > default `18789`.
- Gateway auth is required by default (token/password or tailscale Serve identity). Nicht-Loopback-Binds benötigen einen gemeinsamen Token/Passwort.
- Der Onboarding-Assistent erzeugt standardmäßig ein Gateway-Token (auch bei Schleifback).
- `gateway.remote.token` ist **nur** für entfernte CLI-Anrufe; es aktiviert keine lokale Gateway-Authentifizierung. `gateway.token` wird ignoriert.

Auth und Maßstab:

- `gateway.auth.mode` setzt die Handshake-Anforderungen (`token` oder `password`). Wenn nicht gesetzt, wird Tokenauth angenommen.
- `gateway.auth.token` speichert das Shared Token für Token auth (wird von der CLI auf demselben Rechner verwendet).
- Wenn `gateway.auth.mode` gesetzt ist, wird nur diese Methode akzeptiert (plus optionale Maßstab-Header).
- `gateway.auth.password` kann hier gesetzt werden, oder über `OPENCLAW_GATEWAY_PASSWORD` (empfohlen).
- `gateway.auth.allowTailscale` erlaubt Tailscale-Serve-Identitäts-Headern (`tailscale-user-login`), die Authentifizierung zu erfüllen, wenn die Anfrage über Loopback mit `x-forwarded-for`, `x-forwarded-proto` und `x-forwarded-host` eintrifft. OpenClaw
  überprüft die Identität durch das Lösen der `x-forwarded-for`-Adresse über
  `tailscale whois` bevor sie akzeptiert wird. Wenn `true`, benötigt Serve Requests nicht
  ein Token/Passwort; setzen Sie `false` um explizite Zugangsdaten zu erfordern. Standardmäßig
  `true` wenn `tailscale.mode = "serve"` und auth mode nicht `password` ist.
- `gateway.tailscale.mode: "serve"` verwendet maßstabsgetreue Serve (nur tailnet, loopback bind).
- `gateway.tailscale.mode: "funnel"` zeigt das Dashboard öffentlich auf; erfordert auth.
- `gateway.tailscale.resetOnExit` setzt Serve/Funnel Konfiguration beim Herunterfahren zurück.

Remote-Client-Standardwerte (CLI):

- `gateway.remote.url` setzt die Standard-Gateway-WebSocket-URL für CLI-Aufrufe, wenn `gateway.mode = "remote"`.
- `gateway.remote.transport` wählt den macOS Remote-Transport aus (`ssh` default, `direct` für ws/wss). Wenn `direct`, muss `gateway.remote.url` `ws://` oder `wss://` sein. `ws://host` defaults to port `18789`.
- `gateway.remote.token` liefert das Token für entfernte Aufrufe (lassen Sie es nicht gesetzt für kein auth).
- `gateway.remote.password` liefert das Passwort für entfernte Anrufe (lassen Sie nicht gesetzt für kein auth).

macOS-App-Verhalten:

- OpenClaw.app beobachtet `~/.openclaw/openclaw.json` und wechselt Modi live wenn sich `gateway.mode` oder `gateway.remote.url` ändert.
- Wenn `gateway.mode` nicht gesetzt ist, aber `gateway.remote.url` gesetzt ist, behandelt die macOS-App sie als Remote-Modus.
- Wenn du den Verbindungsmodus in der macOS-App änderst, schreibt er `gateway.mode` (und `gateway.remote.url` + `gateway.remote.transport` in den Remote-Modus) zurück in die Konfigurationsdatei.

```json5
{
  gateway: {
    Modus: "remote",
    remote: {
      url: "ws://gateway.tailnet:18789",
      token: "your-token",
      Passwort: "your-password",
    },
  },
}
```

Beispiel für Direkttransport (macOS App):

```json5
{
  gateway: {
    Modus: "remote",
    remote: {
      transport: "direct",
      url: "wss://gateway.example.ts.net",
      token: "your-token",
    },
  },
}
```

### `gateway.reload` (Config hot reload)

Das Gateway überwacht `~/.openclaw/openclaw.json` (oder `OPENCLAW_CONFIG_PATH`) und führt automatisch Änderungen durch.

Modi:

- `hybrid` (Standard): Hot-apply safe changes; starten Sie das Gateway für kritische Änderungen neu.
- `hot`: nur het-safe Änderungen anwenden; loggen, wenn ein Neustart erforderlich ist.
- `restart`: Starten Sie das Gateway bei jeder Konfigurationsänderung neu.
- 'off': Deaktiviere heißes Neuladen.

```json5
{
  Gateway: {
    reload: {
      Modus: "hybrid",
      debounceMs: 300,
    },
  },
}
```

#### Matrix für heißes Nachladen (Dateien + Auswirkung)

Gesehene Dateien:

- `~/.openclaw/openclaw.json` (oder `OPENCLAW_CONFIG_PATH`)

Hot-angewendet (kein vollständiger Gateway Neustart):

- `hooks` (webhook auth/path/mappings) + `hooks.gmail` (Gmail Watcher neugestartet)
- `browser` (Browser Server neustarten)
- `cron` (cron service restart + concurrency update)
- `agents.defaults.heartbeat` (Heartbeat Runner Neustart)
- `web` (WhatsApp Webkanal neustarten)
- `telegram`, `discord`, `signal`, `imessage` (Kanal Neustart)
- `agent`, `models`, `routing`, `messages`, `session`, `whatsapp`, `logging`, `skills`, `ui`, `talk`, `identity`, `wizard` (dynamische Lesen)

Erfordert vollen Gateway-Neustart:

- `gateway` (port/bind/auth/control UI/tailscale)
- `bridge` (Legacy)
- `discovery`
- `canvasHost`
- `plugins`
- Jeder unbekannte/nicht unterstützte Konfigurationspfad (Standardeinstellung für Sicherheit neustarten)

### Multi-Instanz-Isolierung

Um mehrere Gateways auf einem Host auszuführen (für Redundanz oder einen Rettungsbot), isolieren Sie jeden Instanz-Status + Konfiguration und verwenden Sie einzigartige Ports:

- `OPENCLAW_CONFIG_PATH` (pro Instanz config)
- `OPENCLAW_STATE_DIR` (Sitzungen/Credits)
- `agents.defaults.workspace` (Memories)
- `gateway.port` (einzigartig pro Instanz)

Convenience Flags (CLI):

- `openclaw --dev …` → verwendet `~/.openclaw-dev` + verschiebt Ports von Basis `19001`
- `openclaw --profile <name> …` → verwendet `~/.openclaw-<name>` (Port über config/env/flags)

Siehe [Gateway runbook](/gateway) für die abgeleitete Port-Zuordnung (gateway/browser/canvas).
Siehe [Mehrere Gateways](/gateway/multiple-gateways) für Browser/CDP-Port-Isolation Details.

Beispiel:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json \
OPENCLAW_STATE_DIR=~/.openclaw-a \
openclaw gateway --Port 19001
```

### `Hooks` (Gateway Webhooks)

Aktivieren Sie einen einfachen HTTP-Webhook-Endpunkt auf dem Gateway HTTP-Server.

Standards:

- aktiviert: `false`
- pfad: `/hooks`
- maxBodyBytes: `262144` (256 KB)

```json5
{
  Haken: {
    aktiviert: true
    Token: "Shared-secret",
    Pfad: "/hooks",
    Voreinstellungen: ["gmail"],
    transformsDir: "~/. penclaw/haoks",
    Mappings: [
      {
        Übereinstimmung: { path: "gmail" },
        Aktion: "Agent",
        wakeMode: "jetzt",
        Name: "Gmail",
        Sitzungsschlüssel: "hook:gmail:{{messages[0].id}}",
        Nachrichtenvorlage: "Von: {{messages[0].from}}\nBetreff: {{messages[0].subject}}\n{{messages[0].snippet}}",
        Lieferung: wahr,
        Kanal: "letzt",
        Modell: "openai/gpt-5. -mini",
      },
    ],
  },
}
```

Anfragen müssen den Hook-Token enthalten:

- `Autorisierung: Bär <token>` **or**
- `x-openclaw-token: <token>`

Endpunkte:

- `POST /hooks/wake` → `{ text, mode?: "now"|"next-heartbeat" }`
- `POST /hooks/agent` → `{ message, name?, sessionKey?, wakeMode?, deliver?, channel?, to?, model?, thinking?, timeoutseconds? }` zurück
- `POST /hooks/<name>` → mittels `hooks.mappings` aufgelöst

`/hooks/agent` postet immer eine Zusammenfassung in die Hauptsitzung (und kann optional einen sofortigen Herzschlag über `wakeMode: "now"` auslösen.

Notizen zuordnen:

- `match.path` entspricht dem Unterpfad nach `/hooks` (z.B. `/hooks/gmail` → `gmail`).
- `match.source` entspricht einem Payload-Feld (z.B. `{ source: "gmail" }`), so dass du einen generischen `/hooks/ingest` Pfad verwenden kannst.
- Templates wie `{{messages[0].subject}}` aus der Payload.
- `transform` kann auf ein JS/TS Modul verweisen, das eine Hook-Aktion zurückgibt.
- `deliver: true` sendet die endgültige Antwort an einen Kanal; `channel` defaults to `last` (fällt zurück auf WhatsApp).
- Falls es keine vorherige Zustellung gibt, setze `channel` + `to` explizit (erforderlich für Telegram/Discord/Google Chat/Slack/Signal/iMessage/MS Teams).
- `model` überschreibt die LLM für diesen Hook Run (`provider/model` oder Alias; muss erlaubt sein, wenn `agents.defaults.models` gesetzt ist).

Gmail Helfer-Konfiguration (verwendet von `openclaw webhooks gmail setup` / `run`):

```json5
{
  Haken: {
    gmail: {
      Konto: "openclaw@gmail. om",
      Thema: "projects/<project-id>/topics/gog-gmail-watch",
      Abonnement: "gog-gmail-watch-push",
      pushToken: "shared-push-token",
      hookUrl: "http://127. .0.1:18789/haoks/gmail",
      includeBody: true
      maxBytes: 20000,
      reneEveryMinutes: 720,
      serve: { bind: "127. .0. ", port: 8788, path: "/" },
      tailscale: { mode: "funnel", path: "/gmail-pubsub" },

      // Optional: Verwenden Sie ein günstigeres Modell für die Verarbeitung von Google Mail-Haken
      // Rückfälle auf Agenten. efaults.model allbacks, dann primär auf auth/rate-limit/timeout
      Modell: "openrouter/meta-llama/llama-3. -70b-instruct:free",
      // Optional: Standard-Denklevel für Google Mail-Hooks
      Denken: "off",
    },
  },
}
```

Modell für Gmail-Haken überschreiben:

- `hooks.gmail.model` spezifiziert ein Modell, das für die Verarbeitung von Gmail verwendet werden soll (Standardeinstellung für Session-primär).
- Akzeptiert `provider/model` verweigert oder Aliase von `agents.defaults.models`.
- Falls Sie zu `agents.defaults.model.fallbacks` zurückkehren, dann zu `agents.defaults.model.primary`, auf auth/rate-limit/timeouts.
- Wenn `agents.defaults.models` gesetzt ist, füge das Hooks-Modell in die allowlist ein.
- Beim Start warnt, ob das konfigurierte Modell nicht im Modellkatalog oder in der Zulassungsliste ist.
- `hooks.gmail.thinking` setzt die Standard-Denkstufe für Gmail-Hooks und wird von per-hook `thinking` überschrieben.

Gateway-Autostart:

- Wenn `hooks.enabled=true` und `hooks.gmail.account` gesetzt sind, startet das Gateway
  `gog gmail watch serve` beim Booten und erneuert die Uhr automatisch.
- Setze `OPENCLAW_SKIP_GMAIL_WATCHER=1` um den Autostart zu deaktivieren (für manuelle Ausführungen).
- Vermeiden Sie, einen separaten `gog gmail watch serve` neben dem Gateway auszuführen; es wird
  scheitern mit `listen tcp 127.0.0.1:8788: bind: address already in use`.

Notiz: Wenn `tailscale.mode` eingeschaltet ist, wird OpenClaw standardmäßig `serve.path` zu `/` vorgeben, so dass
Maßstabstabsänderung `/gmail-pubsub` korrekt proxy kann (es entfernt das Set-Pfad-Präfix).
Wenn Sie das Backend benötigen, um den präfixierten Pfad zu erhalten, setzen Sie
`hooks.gmail.tailscale.target` auf eine vollständige URL (und richten Sie `serve.path` aus.

### `canvasHost` (LAN/tailnet Canvas Dateiserver + Live Reload)

Das Gateway liefert ein Verzeichnis von HTML/CSS/JS über HTTP, so dass iOS/Android-Knoten einfach `canvas.navigate` dorthin bringen können.

Standardroot: `~/. penclaw/workspace/canvas`  
Standardport: `18793` (gewählt, um den openclaw Browser CDP Port `18792` zu vermeiden)  
Der Server hört auf dem **Gateway bind host** (LAN oder Tailnet), so dass Knoten ihn erreichen können.

Der Server:

- dient Dateien unter `canvasHost.root`
- injiziert einen winzigen Live-Nachlade-Client in den bedienten HTML
- beobachtet das Verzeichnis und sendet über einen WebSocket-Endpunkt unter `/__openclaw__/ws` erneut
- auto-erstellt einen Starter `index.html` wenn das Verzeichnis leer ist (also siehst du etwas sofort)
- dient auch A2UI bei `/__openclaw__/a2ui/` und wird als `canvasHostUrl`
  beworben (immer von Knoten für Canvas/A2UI)

Deaktiviere Live-Nachladen (und Dateibeobachtung), wenn das Verzeichnis groß ist oder du `EMFILE` drückst:

- config: `canvasHost: { liveReload: false }`

```json5
{
  canvasHost: {
    root: "~/.openclaw/workspace/canvas",
    port: 18793,
    liveReload: true,
  },
}
```

Änderungen an `canvasHost.*` erfordern einen Gateway-Neustart (Konfigurations-Neustart wird neu geladen).

Deaktivieren mit:

- config: `canvasHost: { enabled: false }`
- dev: `OPENCLAW_SKIP_CANVAS_HOST=1`

### `bridge` (alte TCP Bridge, entfernt)

Aktuelle Builds beinhalten nicht mehr den TCP bridge Listener; `bridge.*` Konfigurationsschlüssel werden ignoriert.
Knoten verbinden sich über den Gateway WebSocket. Diese Sektion ist für die historische Referenz.

Legacy-Verhalten:

- Das Gateway könnte eine einfache TCP Bridge für Knoten (iOS/Android) aufdecken, typischerweise auf Port `18790`.

Standardwerte:

- aktiviert: `true`
- port: `18790`
- bind: `lan` (bindet zu `0.0.0.0`)

Bind modes:

- `lan`: `0.0.0.0` (erreichbar auf jeder Schnittstelle, inklusive LAN/Wi-Fi und Maßskala)
- `tailnet`: nur an die Tailscale IP der Maschine binden (empfohlen für Wien <unk> London)
- `loopback`: `127.0.0.1` (nur lokal)
- `auto`: bevorzuge tailnet IP wenn vorhanden, sonst `lan`

TLS:

- `bridge.tls.enabled`: Aktivieren Sie TLS für Bridge-Verbindungen (TLS-nur wenn aktiviert).
- `bridge.tls.autoGenerate`: Erzeugt ein selbst signiertes Zertifikat, wenn kein cert/key vorhanden ist (Standard: true).
- `bridge.tls.certPath` / `bridge.tls.keyPath`: PEM-Pfade für das Bridge-Zertifikat + privater Schlüssel.
- `bridge.tls.caPath`: optionales PEM CA Bundle (Custom roots or future mTLS).

Wenn TLS aktiviert ist, gibt das Gateway `bridgeTls=1` und `bridgeTlsSha256` zur Entdeckung von TXT
an, so dass Knoten das Zertifikat anpinnen können. Manuelle Verbindungen verwenden Vertrauen-zu-Erste Verwendung, wenn noch kein
Fingerabdruck gespeichert ist.
Automatisch generierte certs erfordern `openssl` auf PATH; falls die Erzeugung fehlschlägt, wird die Bridge nicht gestartet.

```json5
{
  Bridge: {
    aktiviert: true
    Port: 18790,
    Bind: "tailnet",
    tls: {
      aktiviert: true
      // Verwendet ~/. penclaw/bridge/tls/bridge-{cert,key}. em ausgelassen.
      // certPath: "~/.openclaw/bridge/tls/bridge-cert.pem",
      // Schlüsselpfad: "~/. penclaw/bridge/tls/bridge-key.pem"
    },
  },
}
```

### `discovery.mdns` (Bonjour / mDNS Broadcast-Modus)

Controls LAN mDNS discovery broadcasts (`_openclaw-gw._tcp`).

- `minimal` (Standard): Strenge `cliPath` + `sshPort` aus TXT-Einträgen
- `full`: include `cliPath` + `sshPort` in TXT records
- `off`: mDNS Broadcasts komplett deaktivieren
- Hostname: defaults to `openclaw` (advertises `openclaw.local`). Überschreiben mit `OPENCLAW_MDNS_HOSTNAME`.

```json5
{
  discovery: { mdns: { mode: "minimal" } },
}
```

### "discovery.wideArea" (ide-Area Bonjour / unicast DNS<unk> SD)

Wenn aktiviert, schreibt das Gateway unter `~/.openclaw/dns/` eine unicast DNS-SD-Zone für `_openclaw-gw._tcp` unter `~/.openclaw/dns/` unter Verwendung der konfigurierten Discovery Domain (Beispiel: `openclaw.internal.`).

Um iOS/Android in allen Netzwerken (Wien <unk> London) entdecken zu lassen, paaren Sie folgendes mit:

- ein DNS-Server auf dem Gateway-Host, der Ihre gewählte Domain bedient (CoreDNS wird empfohlen)
- Anpassungsmaßstab **teilt DNS** so dass Clients diese Domain über den Gateway-DNS-Server auflösen

Einmaliges Setup-Helfer (Gateway-Host):

```bash
openclaw dns setup --apply
```

```json5
{
  discovery: { wideArea: { enabled: true } },
}
```

## Medienmodellvorlagen-Variablen

Template-Platzhalter werden in `tools.media.*.models[].args` und `tools.media.models[].args` (und zukünftigen Template-Argument-Feldern) erweitert.

\| Variable | Beschreibung |
\| ----------------------------------------------------------------------------------------------- | -------- | ------- | ---------- | ------ | ------ | -------- | ------- | ------- | ------- | --- | --- |
\| `{{Body}}` | Ganzer Eingang der Nachricht |
\| `{{RawBody}}` | Roher Eingang der Nachricht (keine Historie/Absender Wrappers; best for command parsing) |
\| `{{BodyStripped}}` | Body mit Gruppenangaben gestrichen (bester Standard für Agenten) |
\| `{{From}}` | Absender Identifier (E. 64 für WhatsApp; kann sich je Kanal unterscheiden) |
\| `{{To}}` | Destination Identifier |
\| `{{MessageSid}}` | Channel Message id (wenn verfügbar) |
\| `{{SessionId}}` | Aktuelle Session UUID |
\| `{{IsNewSession}}` | `"true"` wenn eine neue Session erstellt wurde |
\| `{{MediaUrl}}` | Inbound media pseudo-URL (falls vorhanden) |
\| `{{MediaPath}}` | Lokaler Medienpfad (falls heruntergeladen) |
\| `{{MediaType}}` | Media type (image/audio/document/…)                                             |
\| `{{Transcript}}`   | Audio-Transkript (wenn aktiviert)                                                |
\| `{{Prompt}}`       | Aufgelöster Medien-Prompt für CLI-Einträge                                       |
\| `{{MaxChars}}`     | Aufgelöste maximale Ausgabezeichen für CLI-Einträge                               |
\| `{{ChatType}}`     | `"direct"` oder `"group"`                                                     |
\| `{{GroupSubject}}` | Gruppenthema (bestmöglicher Versuch)                                              |
\| `{{GroupMembers}}` | Vorschau der Gruppenmitglieder (bestmöglicher Versuch)                            |
\| `{{SenderName}}`   | Anzeigename des Absenders (bestmöglicher Versuch)                                 |
\| `{{SenderE164}}`   | Telefonnummer des Absenders (bestmöglicher Versuch)                               |
\| `{{Provider}}`     | Anbieter-Hinweis (whatsapp                                                         | telegram | discord | googlechat | slack | signal | imessage | msteams | webchat | …)  |

## Cron (Gateway Zeitplaner)

Cron ist ein Gateway-Scheduler für Weckungen und geplante Jobs. Siehe [Cron Jobs](/automation/cron-jobs) für die Funktionsübersicht und CLI Beispiele.

```json5
{
  cron: {
    enabled: true,
    maxConcurrentRuns: 2,
  },
}
```

---

_Nächste Seite: [Agent Runtime](/concepts/agent)_ 🦞
