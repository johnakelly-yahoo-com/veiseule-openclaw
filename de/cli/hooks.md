---
title: "Hooks"
---

# `openclaw hooks`

Verwalten Sie Agent-Hooks (ereignisgesteuerte Automatisierungen für Befehle wie `/new`, `/reset` und den Gateway-Start).

Verwandt:

- Hooks: [Hooks](/automation/hooks)
- Plugin-Hooks: [Plugins](/tools/plugin#plugin-hooks)

## Alle Hooks auflisten

```bash
openclaw hooks list
```

Listet alle erkannten Hooks aus Workspace-, verwalteten und gebündelten Verzeichnissen auf.

**Optionen:**

- `--eligible`: Nur geeignete Hooks anzeigen (Voraussetzungen erfüllt)
- `--json`: Ausgabe als JSON
- `-v, --verbose`: Detaillierte Informationen einschließlich fehlender Voraussetzungen anzeigen

**Beispielausgabe:**

```
Hooks (4/4 ready)

Ready:
  🚀 boot-md ✓ - Run BOOT.md on gateway startup
  📝 command-logger ✓ - Log all command events to a centralized audit file
  💾 session-memory ✓ - Save session context to memory when /new command is issued
  😈 soul-evil ✓ - Swap injected SOUL content during a purge window or by random chance
```

**Beispiel (ausführlich):**

```bash
openclaw hooks list --verbose
```

Zeigt fehlende Voraussetzungen für ungeeignete Hooks an.

**Beispiel (JSON):**

```bash
openclaw hooks list --json
```

Gibt strukturiertes JSON zur programmgesteuerten Verwendung zurück.

## Hook-Informationen abrufen

```bash
openclaw hooks info <name>
```

Zeigt detaillierte Informationen zu einem bestimmten Hook an.

**Argumente:**

- `<name>`: Hook-Name (z. B. `session-memory`)

**Optionen:**

- `--json`: Ausgabe als JSON

**Beispiel:**

```bash
openclaw hooks info session-memory
```

**Ausgabe:**

```
💾 session-memory ✓ Ready

Save session context to memory when /new command is issued

Details:
  Source: openclaw-bundled
  Path: /path/to/openclaw/hooks/bundled/session-memory/HOOK.md
  Handler: /path/to/openclaw/hooks/bundled/session-memory/handler.ts
  Homepage: https://docs.openclaw.ai/hooks#session-memory
  Events: command:new

Requirements:
  Config: ✓ workspace.dir
```

## Hook-Eignung prüfen

```bash
openclaw hooks check
```

Zeigt eine Zusammenfassung des Eignungsstatus der Hooks (wie viele bereit sind vs. nicht bereit).

**Optionen:**

- `--json`: Ausgabe als JSON

**Beispielausgabe:**

```
Hooks Status

Total hooks: 4
Ready: 4
Not ready: 0
```

## Einen Hook aktivieren

```bash
openclaw hooks enable <name>
```

Aktiviert einen bestimmten Hook, indem er zu Ihrer Konfiguration (`~/.openclaw/config.json`) hinzugefügt wird.

**Hinweis:** Von Plugins verwaltete Hooks zeigen `plugin:<id>` in `openclaw hooks list` an und
können hier nicht aktiviert/deaktiviert werden. Aktivieren/deaktivieren Sie stattdessen das Plugin.

**Argumente:**

- `<name>`: Hook-Name (z. B. `session-memory`)

**Beispiel:**

```bash
openclaw hooks enable session-memory
```

**Ausgabe:**

```
✓ Enabled hook: 💾 session-memory
```

**Was es tut:**

- Prüft, ob der Hook existiert und geeignet ist
- Aktualisiert `hooks.internal.entries.<name>.enabled = true` in Ihrer Konfiguration
- Speichert die Konfiguration auf der Festplatte

**Nach dem Aktivieren:**

- Starten Sie das Gateway neu, damit Hooks neu geladen werden (Neustart der Menüleisten-App unter macOS oder Neustart Ihres Gateway-Prozesses in der Entwicklung).

## Einen Hook deaktivieren

```bash
openclaw hooks disable <name>
```

Deaktiviert einen bestimmten Hook durch Aktualisieren Ihrer Konfiguration.

**Argumente:**

- `<name>`: Hook-Name (z. B. `command-logger`)

**Beispiel:**

```bash
openclaw hooks disable command-logger
```

**Ausgabe:**

```
⏸ Disabled hook: 📝 command-logger
```

**Nach dem Deaktivieren:**

- Starten Sie das Gateway neu, damit Hooks neu geladen werden

## Hooks installieren

```bash
openclaw hooks install <path-or-spec>
```

Installiert ein Hook-Paket aus einem lokalen Ordner/Archiv oder von npm.

**Was es tut:**

- Kopiert das Hook-Paket nach `~/.openclaw/hooks/<id>`
- Aktiviert die installierten Hooks in `hooks.internal.entries.*`
- Erfasst die Installation unter `hooks.internal.installs`

**Optionen:**

- `-l, --link`: Lokales Verzeichnis verknüpfen statt kopieren (fügt es zu `hooks.internal.load.extraDirs` hinzu)

**Unterstützte Archive:** `.zip`, `.tgz`, `.tar.gz`, `.tar`

**Beispiele:**

```bash
# Local directory
openclaw hooks install ./my-hook-pack

# Local archive
openclaw hooks install ./my-hook-pack.zip

# NPM package
openclaw hooks install @openclaw/my-hook-pack

# Link a local directory without copying
openclaw hooks install -l ./my-hook-pack
```

## Hooks aktualisieren

```bash
openclaw hooks update <id>
openclaw hooks update --all
```

Aktualisiert installierte Hook-Pakete (nur npm-Installationen).

**Optionen:**

- `--all`: Alle verfolgten Hook-Pakete aktualisieren
- `--dry-run`: Anzeigen, was sich ändern würde, ohne zu schreiben

## Gebündelte Hooks

### session-memory

Speichert Sitzungskontext im Speicher, wenn Sie `/new` ausführen.

**Aktivieren:**

```bash
openclaw hooks enable session-memory
```

**Ausgabe:** `~/.openclaw/workspace/memory/YYYY-MM-DD-slug.md`

**Siehe:** [session-memory-Dokumentation](/automation/hooks#session-memory)

### command-logger

Protokolliert alle Befehlsereignisse in eine zentrale Audit-Datei.

**Aktivieren:**

```bash
openclaw hooks enable command-logger
```

**Ausgabe:** `~/.openclaw/logs/commands.log`

**Protokolle anzeigen:**

```bash
# Recent commands
tail -n 20 ~/.openclaw/logs/commands.log

# Pretty-print
cat ~/.openclaw/logs/commands.log | jq .

# Filter by action
grep '"action":"new"' ~/.openclaw/logs/commands.log | jq .
```

**Siehe:** [command-logger-Dokumentation](/automation/hooks#command-logger)

### soul-evil

Tauscht injizierte `SOUL.md`-Inhalte während eines Bereinigungsfensters oder zufällig gegen `SOUL_EVIL.md` aus.

**Aktivieren:**

```bash
openclaw hooks enable soul-evil
```

**Siehe:** [SOUL Evil Hook](/hooks/soul-evil)

### boot-md

Führt `BOOT.md` aus, wenn das Gateway startet (nachdem Kanäle gestartet sind).

**Ereignisse**: `gateway:startup`

**Aktivieren**:

```bash
openclaw hooks enable boot-md
```

**Siehe:** [boot-md-Dokumentation](/automation/hooks#boot-md)

