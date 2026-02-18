---
title: "Hooks"
---

# Hooks

Hooks bieten ein erweiterbares, ereignisgesteuertes System zur Automatisierung von Aktionen als Reaktion auf Agent-Befehle und -Ereignisse. Hooks werden automatisch aus Verzeichnissen erkannt und können über CLI-Befehle verwaltet werden, ähnlich wie Skills in OpenClaw funktionieren.

## Orientierung

Hooks sind kleine Skripte, die ausgeführt werden, wenn etwas passiert. Es gibt zwei Arten:

- **Hooks** (diese Seite): werden im Gateway ausgeführt, wenn Agent-Ereignisse ausgelöst werden, z. B. `/new`, `/reset`, `/stop` oder Lifecycle-Ereignisse.
- **Webhooks**: externe HTTP-Webhooks, mit denen andere Systeme Arbeit in OpenClaw auslösen können. Siehe [Webhook Hooks](/automation/webhook) oder verwenden Sie `openclaw webhooks` für Gmail-Hilfsbefehle.

Hooks können auch in Plugins gebündelt werden; siehe [Plugins](/tools/plugin#plugin-hooks).

Häufige Anwendungsfälle:

- Einen Memory-Snapshot speichern, wenn Sie eine Sitzung zurücksetzen
- Einen Audit-Trail von Befehlen zur Fehlerbehebung oder Compliance führen
- Folgeautomatisierungen auslösen, wenn eine Sitzung beginnt oder endet
- Dateien in den Agent-Arbeitsbereich schreiben oder externe APIs aufrufen, wenn Ereignisse ausgelöst werden

Wenn Sie eine kleine TypeScript-Funktion schreiben können, können Sie auch einen Hook schreiben. Hooks werden automatisch erkannt, und Sie aktivieren oder deaktivieren sie über die CLI.

## Überblick

Das Hook-System ermöglicht Ihnen:

- Sitzungs-Kontext in den Speicher zu sichern, wenn `/new` ausgeführt wird
- Alle Befehle für Audit-Zwecke zu protokollieren
- Benutzerdefinierte Automatisierungen bei Agent-Lifecycle-Ereignissen auszulösen
- Das Verhalten von OpenClaw zu erweitern, ohne den Core-Code zu verändern

## Erste Schritte

### Gebündelte Hooks

OpenClaw wird mit vier gebündelten Hooks ausgeliefert, die automatisch erkannt werden:

- **💾 session-memory**: Speichert den Sitzungs-Kontext in Ihrem Agent-Arbeitsbereich (Standard `~/.openclaw/workspace/memory/`), wenn Sie `/new` ausführen
- **📝 command-logger**: Protokolliert alle Befehlsereignisse in `~/.openclaw/logs/commands.log`
- **🚀 boot-md**: Führt `BOOT.md` aus, wenn das Gateway startet (erfordert aktivierte interne Hooks)
- **😈 soul-evil**: Tauscht injizierten `SOUL.md`-Inhalt während eines Purge-Fensters oder mit zufälliger Wahrscheinlichkeit gegen `SOUL_EVIL.md` aus

Verfügbare Hooks auflisten:

```bash
openclaw hooks list
```

Einen Hook aktivieren:

```bash
openclaw hooks enable session-memory
```

Hook-Status prüfen:

```bash
openclaw hooks check
```

Detaillierte Informationen abrufen:

```bash
openclaw hooks info session-memory
```

### Einführung

Während des Onboardings (`openclaw onboard`) werden Sie aufgefordert, empfohlene Hooks zu aktivieren. Der Assistent erkennt automatisch geeignete Hooks und stellt sie zur Auswahl.

## Hook-Erkennung

Hooks werden automatisch aus drei Verzeichnissen erkannt (in der Reihenfolge der Priorität):

1. **Workspace-Hooks**: `<workspace>/hooks/` (pro Agent, höchste Priorität)
2. **Verwaltete Hooks**: `~/.openclaw/hooks/` (benutzerinstalliert, gemeinsam über Workspaces hinweg)
3. **Gebündelte Hooks**: `<openclaw>/dist/hooks/bundled/` (mit OpenClaw ausgeliefert)

Verwaltete Hook-Verzeichnisse können entweder ein **einzelner Hook** oder ein **Hook-Pack** (Paketverzeichnis) sein.

Jeder Hook ist ein Verzeichnis mit folgendem Inhalt:

```
my-hook/
├── HOOK.md          # Metadata + documentation
└── handler.ts       # Handler implementation
```

## Hook-Packs (npm/Archive)

Hook-Packs sind standardmäßige npm-Pakete, die einen oder mehrere Hooks über `openclaw.hooks` in
`package.json` exportieren. Installieren Sie sie mit:

```bash
openclaw hooks install <path-or-spec>
```

Beispiel `package.json`:

```json
{
  "name": "@acme/my-hooks",
  "version": "0.1.0",
  "openclaw": {
    "hooks": ["./hooks/my-hook", "./hooks/other-hook"]
  }
}
```

Jeder Eintrag verweist auf ein Hook-Verzeichnis, das `HOOK.md` und `handler.ts` (oder `index.ts`) enthält.
Hook-Packs können Abhängigkeiten mitbringen; diese werden unter `~/.openclaw/hooks/<id>` installiert.

## Hook-Struktur

### HOOK.md-Format

Die Datei `HOOK.md` enthält Metadaten im YAML-Frontmatter sowie Markdown-Dokumentation:

```markdown
---
name: my-hook
description: "Short description of what this hook does"
homepage: https://docs.openclaw.ai/hooks#my-hook
metadata:
  { "openclaw": { "emoji": "🔗", "events": ["command:new"], "requires": { "bins": ["node"] } } }
---

# My Hook

Detailed documentation goes here...

## What It Does

- Listens for `/new` commands
- Performs some action
- Logs the result

## Requirements

- Node.js must be installed

## Configuration

No configuration needed.
```

### Metadatenfelder

Das Objekt `metadata.openclaw` unterstützt:

- **`emoji`**: Anzeige-Emoji für die CLI (z. B. `"💾"`)
- **`events`**: Array von Ereignissen, auf die gehört wird (z. B. `["command:new", "command:reset"]`)
- **`export`**: Zu verwendender benannter Export (Standard: `"default"`)
- **`homepage`**: Dokumentations-URL
- **`requires`**: Optionale Anforderungen
  - **`bins`**: Erforderliche Binaries im PATH (z. B. `["git", "node"]`)
  - **`anyBins`**: Mindestens eines dieser Binaries muss vorhanden sein
  - **`env`**: Erforderliche Umgebungsvariablen
  - **`config`**: Erforderliche Konfigurationspfade (z. B. `["workspace.dir"]`)
  - **`os`**: Erforderliche Plattformen (z. B. `["darwin", "linux"]`)
- **`always`**: Eignungsprüfungen umgehen (boolean)
- **`install`**: Installationsmethoden (für gebündelte Hooks: `[{"id":"bundled","kind":"bundled"}]`)

### Handler-Implementierung

Die Datei `handler.ts` exportiert eine Funktion `HookHandler`:

```typescript
import type { HookHandler } from "../../src/hooks/hooks.js";

const myHandler: HookHandler = async (event) => {
  // Only trigger on 'new' command
  if (event.type !== "command" || event.action !== "new") {
    return;
  }

  console.log(`[my-hook] New command triggered`);
  console.log(`  Session: ${event.sessionKey}`);
  console.log(`  Timestamp: ${event.timestamp.toISOString()}`);

  // Your custom logic here

  // Optionally send message to user
  event.messages.push("✨ My hook executed!");
};

export default myHandler;
```

#### Ereigniskontext

Jedes Ereignis enthält:

```typescript
{
  type: 'command' | 'session' | 'agent' | 'gateway',
  action: string,              // e.g., 'new', 'reset', 'stop'
  sessionKey: string,          // Session identifier
  timestamp: Date,             // When the event occurred
  messages: string[],          // Push messages here to send to user
  context: {
    sessionEntry?: SessionEntry,
    sessionId?: string,
    sessionFile?: string,
    commandSource?: string,    // e.g., 'whatsapp', 'telegram'
    senderId?: string,
    workspaceDir?: string,
    bootstrapFiles?: WorkspaceBootstrapFile[],
    cfg?: OpenClawConfig
  }
}
```

## Ereignistypen

### Befehlsereignisse

Ausgelöst, wenn Agent-Befehle ausgeführt werden:

- **`command`**: Alle Befehlsereignisse (allgemeiner Listener)
- **`command:new`**: Wenn der Befehl `/new` ausgeführt wird
- **`command:reset`**: Wenn der Befehl `/reset` ausgeführt wird
- **`command:stop`**: Wenn der Befehl `/stop` ausgeführt wird

### Agent-Ereignisse

- **`agent:bootstrap`**: Bevor Workspace-Bootstrap-Dateien injiziert werden (Hooks dürfen `context.bootstrapFiles` verändern)

### Gateway-Ereignisse

Ausgelöst, wenn das Gateway startet:

- **`gateway:startup`**: Nachdem Kanäle gestartet sind und Hooks geladen wurden

### Tool-Result-Hooks (Plugin-API)

Diese Hooks sind keine Event-Stream-Listener; sie ermöglichen es Plugins, Tool-Ergebnisse synchron anzupassen, bevor OpenClaw sie persistiert.

- **`tool_result_persist`**: Transformiert Tool-Ergebnisse, bevor sie in das Sitzungsprotokoll geschrieben werden. Muss synchron sein; geben Sie die aktualisierte Tool-Ergebnis-Payload zurück oder `undefined`, um sie unverändert zu lassen. Siehe [Agent Loop](/concepts/agent-loop).

### Zukünftige Ereignisse

Geplante Ereignistypen:

- **`session:start`**: Wenn eine neue Sitzung beginnt
- **`session:end`**: Wenn eine Sitzung endet
- **`agent:error`**: Wenn ein Agent auf einen Fehler stößt
- **`message:sent`**: Wenn eine Nachricht gesendet wird
- **`message:received`**: Wenn eine Nachricht empfangen wird

## Eigene Hooks erstellen

### 1. Ort wählen

- **Workspace-Hooks** (`<workspace>/hooks/`): Pro Agent, höchste Priorität
- **Verwaltete Hooks** (`~/.openclaw/hooks/`): Gemeinsam über Workspaces hinweg

### 2. Verzeichnisstruktur erstellen

```bash
mkdir -p ~/.openclaw/hooks/my-hook
cd ~/.openclaw/hooks/my-hook
```

### 3. HOOK.md erstellen

```markdown
---
name: my-hook
description: "Does something useful"
metadata: { "openclaw": { "emoji": "🎯", "events": ["command:new"] } }
---

# My Custom Hook

This hook does something useful when you issue `/new`.
```

### 4. handler.ts erstellen

```typescript
import type { HookHandler } from "../../src/hooks/hooks.js";

const handler: HookHandler = async (event) => {
  if (event.type !== "command" || event.action !== "new") {
    return;
  }

  console.log("[my-hook] Running!");
  // Your logic here
};

export default handler;
```

### 5. Aktivieren und testen

```bash
# Verify hook is discovered
openclaw hooks list

# Enable it
openclaw hooks enable my-hook

# Restart your gateway process (menu bar app restart on macOS, or restart your dev process)

# Trigger the event
# Send /new via your messaging channel
```

## Konfiguration

### Neues Konfigurationsformat (empfohlen)

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "session-memory": { "enabled": true },
        "command-logger": { "enabled": false }
      }
    }
  }
}
```

### Pro-Hook-Konfiguration

Hooks können eine benutzerdefinierte Konfiguration haben:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "my-hook": {
          "enabled": true,
          "env": {
            "MY_CUSTOM_VAR": "value"
          }
        }
      }
    }
  }
}
```

### Zusätzliche Verzeichnisse

Hooks aus zusätzlichen Verzeichnissen laden:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "load": {
        "extraDirs": ["/path/to/more/hooks"]
      }
    }
  }
}
```

### Legacy-Konfigurationsformat (weiterhin unterstützt)

Das alte Konfigurationsformat funktioniert aus Gründen der Abwärtskompatibilität weiterhin:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "handlers": [
        {
          "event": "command:new",
          "module": "./hooks/handlers/my-handler.ts",
          "export": "default"
        }
      ]
    }
  }
}
```

**Migration**: Verwenden Sie für neue Hooks das neue, erkennungsgestützte System. Legacy-Handler werden nach den verzeichnisbasierten Hooks geladen.

## CLI-Befehle

### Hooks auflisten

```bash
# List all hooks
openclaw hooks list

# Show only eligible hooks
openclaw hooks list --eligible

# Verbose output (show missing requirements)
openclaw hooks list --verbose

# JSON output
openclaw hooks list --json
```

### Hook-Informationen

```bash
# Show detailed info about a hook
openclaw hooks info session-memory

# JSON output
openclaw hooks info session-memory --json
```

### Eignung prüfen

```bash
# Show eligibility summary
openclaw hooks check

# JSON output
openclaw hooks check --json
```

### Aktivieren/Deaktivieren

```bash
# Enable a hook
openclaw hooks enable session-memory

# Disable a hook
openclaw hooks disable command-logger
```

## Referenz der gebündelten Hooks

### session-memory

Speichert den Sitzungs-Kontext im Speicher, wenn Sie `/new` ausführen.

**Ereignisse**: `command:new`

**Anforderungen**: `workspace.dir` muss konfiguriert sein

**Ausgabe**: `<workspace>/memory/YYYY-MM-DD-slug.md` (Standard: `~/.openclaw/workspace`)

**Was es tut**:

1. Verwendet den Vor-Reset-Sitzungseintrag, um das korrekte Transkript zu finden
2. Extrahiert die letzten 15 Zeilen der Konversation
3. Verwendet ein LLM, um einen beschreibenden Dateinamen-Slug zu generieren
4. Speichert Sitzungsmetadaten in einer datierten Memory-Datei

**Beispielausgabe**:

```markdown
# Session: 2026-01-16 14:30:00 UTC

- **Session Key**: agent:main:main
- **Session ID**: abc123def456
- **Source**: telegram
```

**Beispiele für Dateinamen**:

- `2026-01-16-vendor-pitch.md`
- `2026-01-16-api-design.md`
- `2026-01-16-1430.md` (Fallback-Zeitstempel, falls die Slug-Generierung fehlschlägt)

**Aktivieren**:

```bash
openclaw hooks enable session-memory
```

### command-logger

Protokolliert alle Befehlsereignisse in eine zentrale Audit-Datei.

**Ereignisse**: `command`

**Anforderungen**: Keine

**Ausgabe**: `~/.openclaw/logs/commands.log`

**Was es tut**:

1. Erfasst Ereignisdetails (Befehlsaktion, Zeitstempel, Sitzungs-Schlüssel, Absender-ID, Quelle)
2. Hängt sie im JSONL-Format an die Logdatei an
3. Läuft unauffällig im Hintergrund

**Beispiel-Logeinträge**:

```jsonl
{"timestamp":"2026-01-16T14:30:00.000Z","action":"new","sessionKey":"agent:main:main","senderId":"+1234567890","source":"telegram"}
{"timestamp":"2026-01-16T15:45:22.000Z","action":"stop","sessionKey":"agent:main:main","senderId":"user@example.com","source":"whatsapp"}
```

**Logs anzeigen**:

```bash
# View recent commands
tail -n 20 ~/.openclaw/logs/commands.log

# Pretty-print with jq
cat ~/.openclaw/logs/commands.log | jq .

# Filter by action
grep '"action":"new"' ~/.openclaw/logs/commands.log | jq .
```

**Aktivieren**:

```bash
openclaw hooks enable command-logger
```

### soul-evil

Tauscht injizierten `SOUL.md`-Inhalt während eines Purge-Fensters oder mit zufälliger Wahrscheinlichkeit gegen `SOUL_EVIL.md` aus.

**Ereignisse**: `agent:bootstrap`

**Docs**: [SOUL Evil Hook](/hooks/soul-evil)

**Ausgabe**: Es werden keine Dateien geschrieben; die Tauschaktionen erfolgen ausschließlich im Speicher.

**Aktivieren**:

```bash
openclaw hooks enable soul-evil
```

**Konfiguration**:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "soul-evil": {
          "enabled": true,
          "file": "SOUL_EVIL.md",
          "chance": 0.1,
          "purge": { "at": "21:00", "duration": "15m" }
        }
      }
    }
  }
}
```

### boot-md

Führt `BOOT.md` aus, wenn das Gateway startet (nachdem die Kanäle gestartet sind).
Interne Hooks müssen aktiviert sein, damit dies ausgeführt wird.

**Ereignisse**: `gateway:startup`

**Anforderungen**: `workspace.dir` muss konfiguriert sein

**Was es tut**:

1. Liest `BOOT.md` aus Ihrem Workspace
2. Führt die Anweisungen über den Agent-Runner aus
3. Sendet angeforderte ausgehende Nachrichten über das Nachrichten-Werkzeug

**Aktivieren**:

```bash
openclaw hooks enable boot-md
```

## Best Practices

### Handler schnell halten

Hooks laufen während der Befehlsverarbeitung. Halten Sie sie leichtgewichtig:

```typescript
// ✓ Good - async work, returns immediately
const handler: HookHandler = async (event) => {
  void processInBackground(event); // Fire and forget
};

// ✗ Bad - blocks command processing
const handler: HookHandler = async (event) => {
  await slowDatabaseQuery(event);
  await evenSlowerAPICall(event);
};
```

### Fehler robust behandeln

Umschließen Sie riskante Operationen immer:

```typescript
const handler: HookHandler = async (event) => {
  try {
    await riskyOperation(event);
  } catch (err) {
    console.error("[my-handler] Failed:", err instanceof Error ? err.message : String(err));
    // Don't throw - let other handlers run
  }
};
```

### Ereignisse früh filtern

Beenden Sie frühzeitig, wenn das Ereignis nicht relevant ist:

```typescript
const handler: HookHandler = async (event) => {
  // Only handle 'new' commands
  if (event.type !== "command" || event.action !== "new") {
    return;
  }

  // Your logic here
};
```

### Spezifische Ereignisschlüssel verwenden

Geben Sie in den Metadaten nach Möglichkeit exakte Ereignisse an:

```yaml
metadata: { "openclaw": { "events": ["command:new"] } } # Specific
```

Stattdessen:

```yaml
metadata: { "openclaw": { "events": ["command"] } } # General - more overhead
```

## Debugging

### Hook-Logging aktivieren

Das Gateway protokolliert das Laden von Hooks beim Start:

```
Registered hook: session-memory -> command:new
Registered hook: command-logger -> command
Registered hook: boot-md -> gateway:startup
```

### Erkennung prüfen

Alle erkannten Hooks auflisten:

```bash
openclaw hooks list --verbose
```

### Registrierung prüfen

Protokollieren Sie in Ihrem Handler, wann er aufgerufen wird:

```typescript
const handler: HookHandler = async (event) => {
  console.log("[my-handler] Triggered:", event.type, event.action);
  // Your logic
};
```

### Eignung verifizieren

Prüfen Sie, warum ein Hook nicht geeignet ist:

```bash
openclaw hooks info my-hook
```

Achten Sie in der Ausgabe auf fehlende Anforderungen.

## Testen

### Gateway-Logs

Überwachen Sie die Gateway-Logs, um die Ausführung von Hooks zu sehen:

```bash
# macOS
./scripts/clawlog.sh -f

# Other platforms
tail -f ~/.openclaw/gateway.log
```

### Hooks direkt testen

Testen Sie Ihre Handler isoliert:

```typescript
import { test } from "vitest";
import { createHookEvent } from "./src/hooks/hooks.js";
import myHandler from "./hooks/my-hook/handler.js";

test("my handler works", async () => {
  const event = createHookEvent("command", "new", "test-session", {
    foo: "bar",
  });

  await myHandler(event);

  // Assert side effects
});
```

## Architektur

### Kernkomponenten

- **`src/hooks/types.ts`**: Typdefinitionen
- **`src/hooks/workspace.ts`**: Verzeichnisscans und Laden
- **`src/hooks/frontmatter.ts`**: Parsing der HOOK.md-Metadaten
- **`src/hooks/config.ts`**: Eignungsprüfung
- **`src/hooks/hooks-status.ts`**: Statusberichterstattung
- **`src/hooks/loader.ts`**: Dynamischer Modul-Loader
- **`src/cli/hooks-cli.ts`**: CLI-Befehle
- **`src/gateway/server-startup.ts`**: Lädt Hooks beim Gateway-Start
- **`src/auto-reply/reply/commands-core.ts`**: Löst Befehlsereignisse aus

### Erkennungsfluss

```
Gateway startup
    ↓
Scan directories (workspace → managed → bundled)
    ↓
Parse HOOK.md files
    ↓
Check eligibility (bins, env, config, os)
    ↓
Load handlers from eligible hooks
    ↓
Register handlers for events
```

### Ereignisfluss

```
User sends /new
    ↓
Command validation
    ↓
Create hook event
    ↓
Trigger hook (all registered handlers)
    ↓
Command processing continues
    ↓
Session reset
```

## Fehlerbehebung

### Hook nicht erkannt

1. Verzeichnisstruktur prüfen:

   ```bash
   ls -la ~/.openclaw/hooks/my-hook/
   # Should show: HOOK.md, handler.ts
   ```

2. HOOK.md-Format verifizieren:

   ```bash
   cat ~/.openclaw/hooks/my-hook/HOOK.md
   # Should have YAML frontmatter with name and metadata
   ```

3. Alle erkannten Hooks auflisten:

   ```bash
   openclaw hooks list
   ```

### Hook nicht geeignet

Anforderungen prüfen:

```bash
openclaw hooks info my-hook
```

Achten Sie auf fehlende:

- Binaries (PATH prüfen)
- Umgebungsvariablen
- Konfigurationswerte
- OS-Kompatibilität

### Hook wird nicht ausgeführt

1. Prüfen, ob der Hook aktiviert ist:

   ```bash
   openclaw hooks list
   # Should show ✓ next to enabled hooks
   ```

2. Starten Sie Ihren Gateway-Prozess neu, damit Hooks neu geladen werden.

3. Prüfen Sie die Gateway-Logs auf Fehler:

   ```bash
   ./scripts/clawlog.sh | grep hook
   ```

### Handler-Fehler

Auf TypeScript-/Import-Fehler prüfen:

```bash
# Test import directly
node -e "import('./path/to/handler.ts').then(console.log)"
```

## Migrationsleitfaden

### Von Legacy-Konfiguration zu Erkennung

**Vorher**:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "handlers": [
        {
          "event": "command:new",
          "module": "./hooks/handlers/my-handler.ts"
        }
      ]
    }
  }
}
```

**Nachher**:

1. Hook-Verzeichnis erstellen:

   ```bash
   mkdir -p ~/.openclaw/hooks/my-hook
   mv ./hooks/handlers/my-handler.ts ~/.openclaw/hooks/my-hook/handler.ts
   ```

2. HOOK.md erstellen:

   ```markdown
   ---
   name: my-hook
   description: "My custom hook"
   metadata: { "openclaw": { "emoji": "🎯", "events": ["command:new"] } }
   ---

   # My Hook

   Does something useful.
   ```

3. Konfiguration aktualisieren:

   ```json
   {
     "hooks": {
       "internal": {
         "enabled": true,
         "entries": {
           "my-hook": { "enabled": true }
         }
       }
     }
   }
   ```

4. Verifizieren und Ihren Gateway-Prozess neu starten:

   ```bash
   openclaw hooks list
   # Should show: 🎯 my-hook ✓
   ```

**Vorteile der Migration**:

- Automatische Erkennung
- CLI-Verwaltung
- Berechtigungsprüfung
- Bessere Dokumentation
- Konsistente Struktur

## Siehe auch

- [CLI-Referenz: hooks](/cli/hooks)
- [Bundled Hooks README](https://github.com/openclaw/openclaw/tree/main/src/hooks/bundled)
- [Webhook Hooks](/automation/webhook)
- [Konfiguration](/gateway/configuration#hooks)

