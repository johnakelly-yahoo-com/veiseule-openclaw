---
status: experimental
title: "Broadcast-grupper"
---

# Broadcast-grupper

**Status:** Experimentell  
**Version:** Tillagd i 2026.1.9

## Översikt

Sändningsgrupper gör det möjligt för flera agenter att bearbeta och svara på samma meddelande samtidigt. Detta gör att du kan skapa specialiserade agentgrupper som arbetar tillsammans i en enda WhatsApp grupp eller DM — alla med ett telefonnummer.

Nuvarande omfattning: **endast WhatsApp** (webbkanal).

Sändningsgrupper utvärderas efter kanaltillåtna listor och gruppaktiveringsregler. I WhatsApp-grupper innebär detta att sändningar sker när OpenClaw normalt skulle svara (till exempel: omnämnande, beroende på dina gruppinställningar).

## Användningsfall

### 1. Specialiserade agentteam

Driftsätt flera agenter med atomära, fokuserade ansvarsområden:

```
Group: "Development Team"
Agents:
  - CodeReviewer (reviews code snippets)
  - DocumentationBot (generates docs)
  - SecurityAuditor (checks for vulnerabilities)
  - TestGenerator (suggests test cases)
```

Varje agent bearbetar samma meddelande och bidrar med sitt specialiserade perspektiv.

### 2. Stöd för flera språk

```
Group: "International Support"
Agents:
  - Agent_EN (responds in English)
  - Agent_DE (responds in German)
  - Agent_ES (responds in Spanish)
```

### 3. Arbetsflöden för kvalitetssäkring

```
Group: "Customer Support"
Agents:
  - SupportAgent (provides answer)
  - QAAgent (reviews quality, only responds if issues found)
```

### 4. Automatiserad Uppgift

```
Group: "Project Management"
Agents:
  - TaskTracker (updates task database)
  - TimeLogger (logs time spent)
  - ReportGenerator (creates summaries)
```

## Konfiguration

### Grundläggande konfigurering

Lägg till en toppnivå `broadcast`-sektion (bredvid `bindings`). Nycklar är WhatsApp kamrat-ids:

- grupp chattar: grupp JID (t.ex. `120363403215116621@g.us`)
- DMs: E.164 telefonnummer (t.ex. `+15551234567`)

```json
{
  "broadcast": {
    "120363403215116621@g.us": ["alfred", "baerbel", "assistant3"]
  }
}
```

**Resultat:** När OpenClaw skulle svara i denna chatt körs alla tre agenterna.

### Bearbetningsstrategi

Styr hur agenter bearbetar meddelanden:

#### Parallellt (standard)

Alla agenter bearbetar samtidigt:

```json
{
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": ["alfred", "baerbel"]
  }
}
```

#### Sekventiellt

Agenter bearbetar i ordning (en väntar tills föregående är klar):

```json
{
  "broadcast": {
    "strategy": "sequential",
    "120363403215116621@g.us": ["alfred", "baerbel"]
  }
}
```

### Fullständigt exempel

```json
{
  "agents": {
    "list": [
      {
        "id": "code-reviewer",
        "name": "Code Reviewer",
        "workspace": "/path/to/code-reviewer",
        "sandbox": { "mode": "all" }
      },
      {
        "id": "security-auditor",
        "name": "Security Auditor",
        "workspace": "/path/to/security-auditor",
        "sandbox": { "mode": "all" }
      },
      {
        "id": "docs-generator",
        "name": "Documentation Generator",
        "workspace": "/path/to/docs-generator",
        "sandbox": { "mode": "all" }
      }
    ]
  },
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": ["code-reviewer", "security-auditor", "docs-generator"],
    "120363424282127706@g.us": ["support-en", "support-de"],
    "+15555550123": ["assistant", "logger"]
  }
}
```

## Hur det fungerar

### Meddelandeflöde

1. **Inkommande meddelande** anländer i en WhatsApp-grupp
2. **Broadcast-kontroll**: Systemet kontrollerar om peer-ID finns i `broadcast`
3. **Om i broadcast-listan**:
   - Alla listade agenter bearbetar meddelandet
   - Varje agent har sin egen sessionsnyckel och isolerad kontext
   - Agenter bearbetar parallellt (standard) eller sekventiellt
4. **Om inte i broadcast-listan**:
   - Normal routning gäller (första matchande bindning)

Obs: sändningsgrupper förbigår inte kanaltillåtna listor eller gruppaktiveringsregler (omnämnande/kommandon/etc). De ändrar bara _vilka agenter som kör _ när ett meddelande är berättigat till behandling.

### Sessionsisolering

Varje agent i en broadcast-grupp upprätthåller helt separata:

- **Sessionsnycklar** (`agent:alfred:whatsapp:group:120363...` vs `agent:baerbel:whatsapp:group:120363...`)
- **Konversationshistorik** (agenten ser inte andra agenters meddelanden)
- **Arbetsyta** (separata sandboxar om konfigurerat)
- **Verktygsåtkomst** (olika tillåt-/nekalistor)
- **Minne/kontext** (separata IDENTITY.md, SOUL.md, etc.)
- **Gruppkontextbuffert** (senaste gruppmeddelanden som används som kontext) delas per peer, så alla broadcast-agenter ser samma kontext när de triggas

Detta gör att varje agent kan ha:

- Olika personligheter
- Olika verktygsåtkomst (t.ex. skrivskyddad vs. läs–skriv)
- Olika modeller (t.ex. opus vs. sonnet)
- Olika Skills installerade

### Exempel: Isolerade sessioner

I grupp `120363403215116621@g.us` med agenterna `["alfred", "baerbel"]`:

**Alfreds kontext:**

```
Session: agent:alfred:whatsapp:group:120363403215116621@g.us
History: [user message, alfred's previous responses]
Workspace: /Users/pascal/openclaw-alfred/
Tools: read, write, exec
```

**Bärbels kontext:**

```
Session: agent:baerbel:whatsapp:group:120363403215116621@g.us
History: [user message, baerbel's previous responses]
Workspace: /Users/pascal/openclaw-baerbel/
Tools: read only
```

## Bästa praxis

### 1. Håll agenter fokuserade

Designa varje agent med ett enda, tydligt ansvar:

```json
{
  "broadcast": {
    "DEV_GROUP": ["formatter", "linter", "tester"]
  }
}
```

✅ **Bra:** Varje agent har ett jobb  
❌ **Dåligt:** En generisk ”dev-helper”-agent

### 2. Använd beskrivande namn

Gör det tydligt vad varje agent gör:

```json
{
  "agents": {
    "security-scanner": { "name": "Security Scanner" },
    "code-formatter": { "name": "Code Formatter" },
    "test-generator": { "name": "Test Generator" }
  }
}
```

### 3. Konfigurera olika verktyg tillgång

Ge agenter bara de verktyg de behöver:

```json
{
  "agents": {
    "reviewer": {
      "tools": { "allow": ["read", "exec"] } // Read-only
    },
    "fixer": {
      "tools": { "allow": ["read", "write", "edit", "exec"] } // Read-write
    }
  }
}
```

### 4. Övervaka prestanda

Med många agenter, överväg:

- Att använda `"strategy": "parallel"` (standard) för hastighet
- Att begränsa broadcast-grupper till 5–10 agenter
- Att använda snabbare modeller för enklare agenter

### 5. Hantera misslyckanden Gracfully

Agenter misslyckas självständigt. Ett agentfel blockerar inte andra:

```
Message → [Agent A ✓, Agent B ✗ error, Agent C ✓]
Result: Agent A and C respond, Agent B logs error
```

## Kompatibilitet

### Leverantörer

Broadcast-grupper fungerar för närvarande med:

- ✅ WhatsApp (implementerat)
- 🚧 Telegram (planerat)
- 🚧 Discord (planerat)
- 🚧 Slack (planerat)

### Routning

Broadcast-grupper fungerar sida vid sida med befintlig routning:

```json
{
  "bindings": [
    {
      "match": { "channel": "whatsapp", "peer": { "kind": "group", "id": "GROUP_A" } },
      "agentId": "alfred"
    }
  ],
  "broadcast": {
    "GROUP_B": ["agent1", "agent2"]
  }
}
```

- `GROUP_A`: Endast alfred svarar (normal routning)
- `GROUP_B`: agent1 OCH agent2 svarar (broadcast)

**Företräde:** `broadcast` har prioritet över `bindings`.

## Felsökning

### Agenter svarar inte

**Kontrollera:**

1. Agent-ID:n finns i `agents.list`
2. Det andra ID-formatet är korrekt (t.ex., '120363403215116621@g.us')
3. Agenterna finns inte i nekalistor

**Debugga:**

```bash
tail -f ~/.openclaw/logs/gateway.log | grep broadcast
```

### Endast en agent svarar

**Orsak:** Peer-ID kan finnas i `bindings` men inte i `broadcast`.

**Åtgärd:** Lägg till i broadcast-konfigen eller ta bort från bindningar.

### Prestandaproblem

**Om det är långsamt med många agenter:**

- Minska antalet agenter per grupp
- Använd lättare modeller (sonnet i stället för opus)
- Kontrollera starttiden för sandbox

## Exempel

### Exempel 1: Team för kodgranskning

```json
{
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": [
      "code-formatter",
      "security-scanner",
      "test-coverage",
      "docs-checker"
    ]
  },
  "agents": {
    "list": [
      {
        "id": "code-formatter",
        "workspace": "~/agents/formatter",
        "tools": { "allow": ["read", "write"] }
      },
      {
        "id": "security-scanner",
        "workspace": "~/agents/security",
        "tools": { "allow": ["read", "exec"] }
      },
      {
        "id": "test-coverage",
        "workspace": "~/agents/testing",
        "tools": { "allow": ["read", "exec"] }
      },
      { "id": "docs-checker", "workspace": "~/agents/docs", "tools": { "allow": ["read"] } }
    ]
  }
}
```

**Användaren skickar:** Kodsnutt  
**Svar:**

- code-formatter: ”Fixade indrag och lade till typanvisningar”
- security-scanner: ”⚠️ SQL-injektionssårbarhet på rad 12”
- test-coverage: ”Täckningen är 45 %, saknar tester för felhanteringsfall”
- docs-checker: ”Saknar docstring för funktionen `process_data`”

### Exempel 2: Flerspråksstöd

```json
{
  "broadcast": {
    "strategy": "sequential",
    "+15555550123": ["detect-language", "translator-en", "translator-de"]
  },
  "agents": {
    "list": [
      { "id": "detect-language", "workspace": "~/agents/lang-detect" },
      { "id": "translator-en", "workspace": "~/agents/translate-en" },
      { "id": "translator-de", "workspace": "~/agents/translate-de" }
    ]
  }
}
```

## API-referens

### Konfigschema

```typescript
interface OpenClawConfig {
  broadcast?: {
    strategy?: "parallel" | "sequential";
    [peerId: string]: string[];
  };
}
```

### Fält

- `strategy` (valfritt): Hur agenter ska bearbetas
  - `"parallel"` (standard): Alla agenter bearbetar samtidigt
  - `"sequential"`: Agenter bearbetar i array-ordning
- `[peerId]`: WhatsApp-grupp-JID, E.164-nummer eller annat peer-ID
  - Värde: Array av agent-ID:n som ska bearbeta meddelanden

## Begränsningar

1. **Max antal agenter:** Ingen hård gräns, men 10+ agenter kan vara långsamt
2. **Delad kontext:** Agenter ser inte varandras svar (avsiktligt)
3. **Meddelandeordning:** Parallella svar kan komma i valfri ordning
4. **Hastighetsbegränsningar:** Alla agenter räknas mot WhatsApps rate limits

## Framtida förbättringar

Planerade funktioner:

- [ ] Delat kontextläge (agenter ser varandras svar)
- [ ] Agentkoordination (agenter kan signalera till varandra)
- [ ] Dynamiskt agentval (välj agenter baserat på meddelandets innehåll)
- [ ] Agentprioriteter (vissa agenter svarar före andra)

## Se även

- [Konfiguration för flera agenter](/tools/multi-agent-sandbox-tools)
- [Routningskonfiguration](/channels/channel-routing)
- [Sessionshantering](/concepts/sessions)


