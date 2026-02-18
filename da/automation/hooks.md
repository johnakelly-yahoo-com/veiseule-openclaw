---
title: "Hooks"
---

# Hooks

Hooks giver et omfattende eventdrevet system til automatisering af handlinger som reaktion på agent kommandoer og begivenheder. Kroge bliver automatisk opdaget fra mapper og kan styres via CLI kommandoer, svarende til hvordan færdigheder fungerer i OpenClaw.

## Overblik

Kroge er små scripts, der kører, når der sker noget. Der er to slags:

- **Hooks** (denne side): kører inde i Gateway, når agenthændelser udløses, såsom `/new`, `/reset`, `/stop` eller livscyklus-hændelser.
- **Webhooks**: eksterne HTTP webhooks, der lader andre systemer udløse arbejde i OpenClaw. Se [Webhook Hooks](/automation/webhook) eller brug 'openclaw webhooks' for Gmail helper kommandoer.

Hooks kan også pakkes inde i plugins; se [Plugins](/tools/plugin#plugin-hooks).

Almindelige anvendelser:

- Gem et hukommelses-snapshot, når du nulstiller en session
- Bevar et revisionsspor af kommandoer til fejlfinding eller compliance
- Udløs opfølgende automatisering, når en session starter eller slutter
- Skriv filer ind i agentens workspace eller kald eksterne API’er, når hændelser udløses

Hvis du kan skrive en lille TypeScript-funktion, kan du skrive en krog. Kroge bliver opdaget automatisk, og du aktiverer eller deaktiverer dem via CLI.

## Overblik

Hooks-systemet giver dig mulighed for at:

- Gemme sessionskontekst i hukommelsen, når `/new` udstedes
- Logge alle kommandoer til revision
- Udløse brugerdefineret automatisering ved agentens livscyklus-hændelser
- Udvide OpenClaws adfærd uden at ændre kernekode

## Kom godt i gang

### Medfølgende hooks

OpenClaw leveres med fire medfølgende hooks, som opdages automatisk:

- **💾 session-memory**: Gemmer sessionskontekst i dit agent-workspace (standard `~/.openclaw/workspace/memory/`), når du udsteder `/new`
- **📝 command-logger**: Logger alle kommandohændelser til `~/.openclaw/logs/commands.log`
- **🚀 boot-md**: Kører `BOOT.md`, når gatewayen starter (kræver, at interne hooks er aktiveret)
- **😈 soul-evil**: Udskifter injiceret `SOUL.md`-indhold med `SOUL_EVIL.md` under et purge-vindue eller ved tilfældig chance

List tilgængelige hooks:

```bash
openclaw hooks list
```

Aktivér et hook:

```bash
openclaw hooks enable session-memory
```

Tjek hook-status:

```bash
openclaw hooks check
```

Få detaljerede oplysninger:

```bash
openclaw hooks info session-memory
```

### Introduktion

Under onboarding (`openclaw onboard`), vil du blive bedt om at aktivere anbefalede kroge. Guiden opdager automatisk kvalificerede kroge og præsenterer dem til udvælgelse.

## Hook-opdagelse

Hooks opdages automatisk fra tre mapper (i prioriteret rækkefølge):

1. **Workspace-hooks**: `<workspace>/hooks/` (pr. agent, højeste prioritet)
2. **Managed hooks**: `~/.openclaw/hooks/` (brugerinstalleret, delt på tværs af workspaces)
3. **Medfølgende hooks**: `<openclaw>/dist/hooks/bundled/` (leveret med OpenClaw)

Managed hook-mapper kan være enten et **enkelt hook** eller en **hook-pakke** (pakke-mappe).

Hvert hook er en mappe, der indeholder:

```
my-hook/
├── HOOK.md          # Metadata + documentation
└── handler.ts       # Handler implementation
```

## Hook-pakker (npm/arkiver)

Krogpakker er standard npm pakker, der eksporterer en eller flere kroge via `openclaw.hooks` i
`package.json`. Installér dem med:

```bash
openclaw hooks install <path-or-spec>
```

Eksempel på `package.json`:

```json
{
  "name": "@acme/my-hooks",
  "version": "0.1.0",
  "openclaw": {
    "hooks": ["./hooks/my-hook", "./hooks/other-hook"]
  }
}
```

Hver post peger på en krog mappe indeholdende `HOOK.md` og `handler.ts` (eller `index.ts`).
Krogpakker kan sende afhængigheder; de vil blive installeret under `~/.openclaw/hooks/<id>`.

## Hook-struktur

### HOOK.md-format

Filen `HOOK.md` indeholder metadata i YAML-frontmatter plus Markdown-dokumentation:

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

### Metadatafelter

Objektet `metadata.openclaw` understøtter:

- **`emoji`**: Display emoji for CLI (f.eks. `"💾"`)
- **`begivenheder`**: Array af begivenheder der skal lyttes til (fx, `["kommando:ny", "kommando:reset"]`)
- **`export`**: Navngivet eksport, der bruges (standard er `"default"`)
- **`homepage`**: Dokumentations-URL
- **`requires`**: Valgfrie krav
  - **`binder`**: Krævede binære filer på PATH (f.eks. `["git", "node"]`)
  - **`anyBins`**: Mindst én af disse binære filer skal være til stede
  - **`env`**: Påkrævede miljøvariabler
  - **`config`**: Krævede konfigurationsstier (f.eks. `["workspace.dir"]`)
  - **`os`**: Påkrævede platforme (f.eks. `["darwin", "linux"]`)
- **`always`**: Omgå egnethedstjek (boolean)
- **`install`**: Installationsmetoder (for medfølgende hooks: `[{"id":"bundled","kind":"bundled"}]`)

### Handler-implementering

Filen `handler.ts` eksporterer en `HookHandler`-funktion:

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

#### Hændelseskontekst

Hver hændelse indeholder:

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

## Hændelsestyper

### Kommandohændelser

Udløses, når agentkommandoer udstedes:

- **`command`**: Alle kommandohændelser (generel lytter)
- **`command:new`**: Når kommandoen `/new` udstedes
- **`command:reset`**: Når kommandoen `/reset` udstedes
- **`command:stop`**: Når kommandoen `/stop` udstedes

### Agenthændelser

- **`agent:bootstrap`**: Før workspace-bootstrapfiler injiceres (hooks kan mutere `context.bootstrapFiles`)

### Gateway-hændelser

Udløses, når gatewayen starter:

- **`gateway:startup`**: Efter kanaler starter, og hooks er indlæst

### Tool-result hooks (Plugin API)

Disse hooks er ikke hændelsesstream-lyttere; de lader plugins synkront justere tool-resultater, før OpenClaw persisterer dem.

- **`tool_result_persist`**: transformere værktøj resultater, før de er skrevet til sessions-transkriptionen. Skal være synkron; returnere den opdaterede værktøj resultat nyttelast eller `undefined` for at holde det som-is. Se [Agent Loop](/concepts/agent-loop).

### Fremtidige hændelser

Planlagte hændelsestyper:

- **`session:start`**: Når en ny session begynder
- **`session:end`**: Når en session slutter
- **`agent:error`**: Når en agent støder på en fejl
- **`message:sent`**: Når en besked sendes
- **`message:received`**: Når en besked modtages

## Oprettelse af brugerdefinerede hooks

### 1. Vælg Placering

- **Workspace-hooks** (`<workspace>/hooks/`): Pr. agent, højeste prioritet
- **Managed hooks** (`~/.openclaw/hooks/`): Delt på tværs af workspaces

### 2. Opret Mappestruktur

```bash
mkdir -p ~/.openclaw/hooks/my-hook
cd ~/.openclaw/hooks/my-hook
```

### 3. Opret HOOK.md

```markdown
---
name: my-hook
description: "Does something useful"
metadata: { "openclaw": { "emoji": "🎯", "events": ["command:new"] } }
---

# My Custom Hook

This hook does something useful when you issue `/new`.
```

### 4. Opret handler.ts

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

### 5. Aktiver og test

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

### Nyt konfigurationsformat (anbefalet)

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

### Konfiguration pr. hook

Hooks kan have brugerdefineret konfiguration:

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

### Ekstra mapper

Indlæs hooks fra yderligere mapper:

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

### Legacy-konfigurationsformat (stadig understøttet)

Det gamle konfigurationsformat fungerer stadig af hensyn til bagudkompatibilitet:

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

**Migration**: Brug det nye opdagelsesbaserede system til nye kroge. Legacy handlers er indlæst efter mappebaserede kroge.

## CLI-kommandoer

### List hooks

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

### Hook-oplysninger

```bash
# Show detailed info about a hook
openclaw hooks info session-memory

# JSON output
openclaw hooks info session-memory --json
```

### Tjek egnethed

```bash
# Show eligibility summary
openclaw hooks check

# JSON output
openclaw hooks check --json
```

### Aktivér/deaktivér

```bash
# Enable a hook
openclaw hooks enable session-memory

# Disable a hook
openclaw hooks disable command-logger
```

## Reference for medfølgende hooks

### session-memory

Gemmer sessionskontekst i hukommelsen, når du udsteder `/new`.

**Hændelser**: `command:new`

**Krav**: `workspace.dir` skal være konfigureret

**Output**: `<workspace>/memory/YYYY-MM-DD-slug.md` (standard er `~/.openclaw/workspace`)

**Hvad den gør**:

1. Bruger session-indgangen før reset til at finde den korrekte udskrift
2. Udtrækker de sidste 15 linjer af samtalen
3. Bruger LLM til at generere en beskrivende filnavnsslug
4. Gemmer sessionsmetadata i en dateret hukommelsesfil

**Eksempel på output**:

```markdown
# Session: 2026-01-16 14:30:00 UTC

- **Session Key**: agent:main:main
- **Session ID**: abc123def456
- **Source**: telegram
```

**Eksempler på filnavne**:

- `2026-01-16-vendor-pitch.md`
- `2026-01-16-api-design.md`
- `2026-01-16-1430.md` (fallback-tidsstempel, hvis slug-generering mislykkes)

**Aktivér**:

```bash
openclaw hooks enable session-memory
```

### command-logger

Logger alle kommandohændelser til en central revisionsfil.

**Hændelser**: `command`

**Krav**: Ingen

**Output**: `~/.openclaw/logs/commands.log`

**Hvad den gør**:

1. Indfanger hændelsesdetaljer (kommandohandling, tidsstempel, sessionsnøgle, afsender-ID, kilde)
2. Tilføjer til logfil i JSONL-format
3. Kører lydløst i baggrunden

**Eksempel på logposter**:

```jsonl
{"timestamp":"2026-01-16T14:30:00.000Z","action":"new","sessionKey":"agent:main:main","senderId":"+1234567890","source":"telegram"}
{"timestamp":"2026-01-16T15:45:22.000Z","action":"stop","sessionKey":"agent:main:main","senderId":"user@example.com","source":"whatsapp"}
```

**Se logs**:

```bash
# View recent commands
tail -n 20 ~/.openclaw/logs/commands.log

# Pretty-print with jq
cat ~/.openclaw/logs/commands.log | jq .

# Filter by action
grep '"action":"new"' ~/.openclaw/logs/commands.log | jq .
```

**Aktivér**:

```bash
openclaw hooks enable command-logger
```

### soul-evil

Udskifter injiceret `SOUL.md`-indhold med `SOUL_EVIL.md` under et purge-vindue eller ved tilfældig chance.

**Hændelser**: `agent:bootstrap`

**Docs**: [SOUL Evil Hook](/hooks/soul-evil)

**Output**: Ingen filer skrives; udskiftninger sker kun i hukommelsen.

**Aktivér**:

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

Kører `BOOT.md`, når gateway’en starter (efter kanaler starter).
Interne kroge skal være aktiveret for at dette kan køre.

**Hændelser**: `gateway:startup`

**Krav**: `workspace.dir` skal være konfigureret

**Hvad den gør**:

1. Læser `BOOT.md` fra dit workspace
2. Kører instruktionerne via agent-runneren
3. Sender eventuelle ønskede udgående beskeder via message-værktøjet

**Aktivér**:

```bash
openclaw hooks enable boot-md
```

## Bedste praksis

### Hold handlere hurtige

Hooks kører under kommando behandling. Behold dem letvægt:

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

### Håndtér fejl på en robust måde

Indpak altid risikable operationer:

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

### Filtrér hændelser tidligt

Returnér tidligt, hvis hændelsen ikke er relevant:

```typescript
const handler: HookHandler = async (event) => {
  // Only handle 'new' commands
  if (event.type !== "command" || event.action !== "new") {
    return;
  }

  // Your logic here
};
```

### Brug specifikke hændelsesnøgler

Angiv præcise hændelser i metadata, når det er muligt:

```yaml
metadata: { "openclaw": { "events": ["command:new"] } } # Specific
```

I stedet for:

```yaml
metadata: { "openclaw": { "events": ["command"] } } # General - more overhead
```

## Fejlfinding

### Aktivér hook-logging

Gatewayen logger indlæsning af hooks ved opstart:

```
Registered hook: session-memory -> command:new
Registered hook: command-logger -> command
Registered hook: boot-md -> gateway:startup
```

### Tjek opdagelse

List alle opdagede hooks:

```bash
openclaw hooks list --verbose
```

### Tjek registrering

Log i din handler, når den kaldes:

```typescript
const handler: HookHandler = async (event) => {
  console.log("[my-handler] Triggered:", event.type, event.action);
  // Your logic
};
```

### Verificér egnethed

Tjek, hvorfor et hook ikke er egnet:

```bash
openclaw hooks info my-hook
```

Se efter manglende krav i outputtet.

## Test

### Gateway-logs

Overvåg gateway-logs for at se hook-udførelse:

```bash
# macOS
./scripts/clawlog.sh -f

# Other platforms
tail -f ~/.openclaw/gateway.log
```

### Test hooks direkte

Test dine handlere isoleret:

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

## Arkitektur

### Kernekomponenter

- **`src/hooks/types.ts`**: Typedefinitioner
- **`src/hooks/workspace.ts`**: Mappe-scanning og indlæsning
- **`src/hooks/frontmatter.ts`**: Parsing af HOOK.md-metadata
- **`src/hooks/config.ts`**: Egnethedstjek
- **`src/hooks/hooks-status.ts`**: Statusrapportering
- **`src/hooks/loader.ts`**: Dynamisk modulindlæser
- **`src/cli/hooks-cli.ts`**: CLI-kommandoer
- **`src/gateway/server-startup.ts`**: Indlæser hooks ved gateway-start
- **`src/auto-reply/reply/commands-core.ts`**: Udløser kommandohændelser

### Opdagelsesflow

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

### Hændelsesflow

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

## Fejlfinding

### Hook ikke opdaget

1. Tjek mappestruktur:

   ```bash
   ls -la ~/.openclaw/hooks/my-hook/
   # Should show: HOOK.md, handler.ts
   ```

2. Verificér HOOK.md-format:

   ```bash
   cat ~/.openclaw/hooks/my-hook/HOOK.md
   # Should have YAML frontmatter with name and metadata
   ```

3. List alle opdagede hooks:

   ```bash
   openclaw hooks list
   ```

### Hook ikke egnet

Tjek krav:

```bash
openclaw hooks info my-hook
```

Se efter manglende:

- Binære filer (tjek PATH)
- Miljøvariabler
- Konfigurationsværdier
- OS-kompatibilitet

### Hook udføres ikke

1. Verificér, at hooket er aktiveret:

   ```bash
   openclaw hooks list
   # Should show ✓ next to enabled hooks
   ```

2. Genstart din gateway-proces, så hooks genindlæses.

3. Tjek gateway-logs for fejl:

   ```bash
   ./scripts/clawlog.sh | grep hook
   ```

### Handler-fejl

Tjek for TypeScript-/importfejl:

```bash
# Test import directly
node -e "import('./path/to/handler.ts').then(console.log)"
```

## Migreringsguide

### Fra legacy-konfiguration til opdagelse

**Før**:

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

**Efter**:

1. Opret hook-mappe:

   ```bash
   mkdir -p ~/.openclaw/hooks/my-hook
   mv ./hooks/handlers/my-handler.ts ~/.openclaw/hooks/my-hook/handler.ts
   ```

2. Opret HOOK.md:

   ```markdown
   ---
   name: my-hook
   description: "My custom hook"
   metadata: { "openclaw": { "emoji": "🎯", "events": ["command:new"] } }
   ---

   # My Hook

   Does something useful.
   ```

3. Opdatér konfiguration:

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

4. Verificér og genstart din gateway-proces:

   ```bash
   openclaw hooks list
   # Should show: 🎯 my-hook ✓
   ```

**Fordele ved migrering**:

- Automatisk opdagelse
- CLI-administration
- Egnethedstjek
- Bedre dokumentation
- Konsistent struktur

## Se også

- [CLI Reference: hooks](/cli/hooks)
- [Bundled Hooks README](https://github.com/openclaw/openclaw/tree/main/src/hooks/bundled)
- [Webhook Hooks](/automation/webhook)
- [Konfiguration](/gateway/configuration#hooks)


