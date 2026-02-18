---
summary: "CLI-referens för `openclaw hooks` (agent-hooks)"
read_when:
  - Du vill hantera agent-hooks
  - Du vill installera eller uppdatera hooks
title: "hooks"
---

# `openclaw hooks`

Hantera agent-hooks (händelsedrivna automatiseringar för kommandon som `/new`, `/reset` och gateway-start).

Relaterat:

- Krokar: [Krokar](/automation/hooks)
- Plugin-hooks: [Plugins](/tools/plugin#plugin-hooks)

## Lista alla hooks

```bash
openclaw hooks list
```

Lista alla upptäckta hooks från arbetsytans, hanterade och paketerade kataloger.

**Alternativ:**

- `--eligible`: Visa endast behöriga hooks (krav uppfyllda)
- `--json`: Utdata som JSON
- `-v, --verbose`: Visa detaljerad information inklusive saknade krav

**Exempelutdata:**

```
Hooks (4/4 ready)

Ready:
  🚀 boot-md ✓ - Run BOOT.md on gateway startup
  📝 command-logger ✓ - Log all command events to a centralized audit file
  💾 session-memory ✓ - Save session context to memory when /new command is issued
  😈 soul-evil ✓ - Swap injected SOUL content during a purge window or by random chance
```

**Exempel (utförlig):**

```bash
openclaw hooks list --verbose
```

Visar saknade krav för obehöriga hooks.

**Exempel (JSON):**

```bash
openclaw hooks list --json
```

Returnerar strukturerad JSON för programmatisk användning.

## Hämta hook-information

```bash
openclaw hooks info <name>
```

Visa detaljerad information om en specifik hook.

**Argument:**

- `<name>`: Kroknamn (t.ex., `session-memory`)

**Alternativ:**

- `--json`: Utdata som JSON

**Exempel:**

```bash
openclaw hooks info session-memory
```

**Utdata:**

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

## Kontrollera hooks behörighet

```bash
openclaw hooks check
```

Visa en sammanfattning av hooks behörighetsstatus (hur många som är redo jämfört med inte redo).

**Alternativ:**

- `--json`: Utdata som JSON

**Exempelutdata:**

```
Hooks Status

Total hooks: 4
Ready: 4
Not ready: 0
```

## Aktivera en hook

```bash
openclaw hooks enable <name>
```

Aktivera en specifik hook genom att lägga till den i din konfig (`~/.openclaw/config.json`).

**Observera:** Krokar som hanteras av plugins visar `plugin:<id>` i `openclaw hooks list` och
kan inte aktiveras/inaktiveras här. Aktivera/inaktivera plugin istället.

**Argument:**

- `<name>`: Kroknamn (t.ex., `session-memory`)

**Exempel:**

```bash
openclaw hooks enable session-memory
```

**Utdata:**

```
✓ Enabled hook: 💾 session-memory
```

**Vad den gör:**

- Kontrollerar om hooken finns och är behörig
- Uppdaterar `hooks.internal.entries.<name>.enabled = true` i din konfiguration
- Sparar konfig till disk

**Efter aktivering:**

- Starta om gateway (nätverksgateway) så att hooks laddas om (starta om menyradsappen på macOS, eller starta om din gateway-process i utveckling).

## Inaktivera en hook

```bash
openclaw hooks disable <name>
```

Inaktivera en specifik hook genom att uppdatera din konfig.

**Argument:**

- `<name>`: Kroknamn (t.ex., `command-logger`)

**Exempel:**

```bash
openclaw hooks disable command-logger
```

**Utdata:**

```
⏸ Disabled hook: 📝 command-logger
```

**Efter inaktivering:**

- Starta om gateway (nätverksgateway) så att hooks laddas om

## Installera hooks

```bash
openclaw hooks install <path-or-spec>
```

Installera ett hook-paket från en lokal mapp/arkiv eller npm.

**Vad den gör:**

- Kopierar hook-paketet till `~/.openclaw/hooks/<id>`
- Aktiverar de installerade hooks i `hooks.internal.entries.*`
- Registrerar installationen under `hooks.internal.installs`

**Alternativ:**

- `-l, --link`: Länka en lokal katalog i stället för att kopiera (lägger till den i `hooks.internal.load.extraDirs`)

**Stödda arkiv:** `.zip`, `.tgz`, `.tar.gz`, `.tar`

**Exempel:**

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

## Uppdatera hooks

```bash
openclaw hooks update <id>
openclaw hooks update --all
```

Uppdatera installerade hook-paket (endast npm-installationer).

**Alternativ:**

- `--all`: Uppdatera alla spårade hook-paket
- `--dry-run`: Visa vad som skulle ändras utan att skriva

## Medföljande hooks

### session-memory

Sparar sessionskontext till minne när du utfärdar `/new`.

**Aktivera:**

```bash
openclaw hooks enable session-memory
```

**Utdata:** `~/.openclaw/workspace/memory/YYYY-MM-DD-slug.md`

**Se:** [session-memory-dokumentation](/automation/hooks#session-memory)

### command-logger

Loggar alla kommandohändelser till en centraliserad revisionsfil.

**Aktivera:**

```bash
openclaw hooks enable command-logger
```

**Utdata:** `~/.openclaw/logs/commands.log`

**Visa loggar:**

```bash
# Recent commands
tail -n 20 ~/.openclaw/logs/commands.log

# Pretty-print
cat ~/.openclaw/logs/commands.log | jq .

# Filter by action
grep '"action":"new"' ~/.openclaw/logs/commands.log | jq .
```

**Se:** [command-logger-dokumentation](/automation/hooks#command-logger)

### soul-evil

Byter ut injicerat `SOUL.md`-innehåll mot `SOUL_EVIL.md` under ett rensningsfönster eller av slump.

**Aktivera:**

```bash
openclaw hooks enable soul-evil
```

**Se:** [SOUL Evil Hook](/hooks/soul-evil)

### boot-md

Kör `BOOT.md` när gateway (nätverksgateway) startar (efter att kanalerna startat).

**Händelser**: `gateway:startup`

**Aktivera**:

```bash
openclaw hooks enable boot-md
```

**Se:** [boot-md-dokumentation](/automation/hooks#boot-md)
