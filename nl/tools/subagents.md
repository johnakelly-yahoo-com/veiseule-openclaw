---
summary: "Sub-agents: het starten van geïsoleerde agentruns die resultaten terugmelden aan de aanvragende chat"
read_when:
  - Je wilt achtergrond-/parallel werk via de agent
  - Je wijzigt sessions_spawn of het toolbeleid voor sub-agents
title: "Sub-agenten"
---

# Sub-agenten

Sub-agents let you run background tasks without blocking the main conversation. When you spawn a sub-agent, it runs in its own isolated session, does its work, and announces the result back to the chat when finished.

## Slash-commando

Gebruik de slash-opdracht `/subagents` om subagent-runs voor de huidige sessie te inspecteren en te beheren:

- `/subagents list`
- \`/subagents stop <id\\
- \`/subagents log <id\\
- \`/subagents info <id\\
- \`/subagents send <id\\

`/subagents info` toont run-metadata (status, tijdstempels, sessie-id, transcriptpad, opschoning).

Primaire doelen:

- Paralleliseer "research / lange taak / trage tool"-werk zonder de hoofd-run te blokkeren.
- Houd sub-agents standaard geïsoleerd (sessiescheiding + optionele sandboxing).
- Houd het tooloppervlak moeilijk te misbruiken: sub-agents krijgen standaard **geen** sessietools.
- Ondersteun configureerbare nesting-diepte voor orchestrator-patronen.

Each sub-agent has its **own** context and token usage. Voor zware of repetitieve
taken stel je een goedkoper model in voor sub-agents en houd je je hoofd-agent op een model van hogere kwaliteit.
Per-agentconfiguratie: `agents.list[].subagents.model`

## #> [limit] [tools]\`

Expliciete `thinking`-parameter in de `sessions_spawn`-aanroep

- Start een sub-agent-run (`deliver: false`, globale lane: `subagent`)
- Voert vervolgens een announce-stap uit en plaatst het announce-antwoord in het chatkanaal van de aanvrager
- Model: target agent’s normal model selection (unless `subagents.model` is set)
- Per-agentconfiguratie: `agents.list[].subagents.thinking`

Toolparameters:

- `task`
- `label`
- Starten onder een andere agent-id (moet zijn toegestaan)
- Ongeldige modelwaarden worden stilzwijgend overgeslagen — de sub-agent draait op de eerstvolgende geldige standaard met een waarschuwing in het toolresultaat.
- Thinking: no sub-agent override (unless `subagents.thinking` is set)
- De sub-agent afbreken na N seconden
- "delete" \\

Allowlist:

- `agents.list[].subagents.allowAgents`: lijst met agent-id's die via `agentId` kunnen worden getarget (`["*"]` om elke toe te staan). Standaard: alleen de aanvragende agent.

Detectie:

- Gebruik de `agents_list`-tool om te ontdekken welke agent-id's momenteel zijn toegestaan voor `sessions_spawn`.

Auto-Archive

- Sub-agent sessions are automatically archived after a configurable period:
- <Note>
Archiveren hernoemt het transcript naar `*.deleted.<timestamp>` (same folder).
- "delete" archiveert onmiddellijk na aankondiging
- Timers voor automatisch archiveren zijn best-effort; wachtende timers gaan verloren als de gateway opnieuw wordt gestart.
- `runTimeoutSeconds` archiveert de sessie **niet** automatisch. De sessie blijft bestaan totdat de normale archieftimer afgaat.
- Auto-archivering is gelijk van toepassing op depth-1- en depth-2-sessies.

## Sub-agents stoppen

Standaard kunnen sub-agents alleen worden gestart onder hun eigen agent-id. Je kunt één niveau van nesting inschakelen door `maxSpawnDepth: 2` in te stellen, wat het **orchestrator-patroon** mogelijk maakt: main → orchestrator sub-agent → worker sub-sub-agents.

### Hoe in te schakelen

```json5
{
  agents: {
    defaults: {
      subagents: {
        model: "minimax/MiniMax-M2.1",
      },
    },
  },
}
```

### Diepteniveaus

| Diepte | Sleutelvorm van sessie                                                                                                                                                                                                                                                                                                                                                                                                                | Rol                                                                       | Kan spawnen?                    |
| ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------- | ------------------------------- |
| 0      | `agentId`                                                                                                                                                                                                                                                                                                                                                                                                                             | Per-agent-overschrijvingen                                                | Altijd                          |
| 1      | {&#xA;agents: {&#xA;defaults: {&#xA;subagents: {&#xA;thinking: "low",&#xA;},&#xA;},&#xA;},&#xA;}                                                                                                                                                                                                                                                                      | Sub-agent (orchestrator wanneer depth 2 is toegestaan) | Alleen als `maxSpawnDepth >= 2` |
| 2      | {&#xA;agents: {&#xA;list: [&#xA;{&#xA;id: "orchestrator",&#xA;subagents: {&#xA;allowAgents: ["researcher", "coder"], // of ["\*"] om alles toe te staan&#xA;},&#xA;},&#xA;],&#xA;},&#xA;} | Subagents beheren (`/subagents`)                       | Nooit                           |

### Announce-keten

Resultaten stromen terug omhoog in de keten:

1. Depth-2 worker voltooit → kondigt aan bij zijn ouder (depth-1 orchestrator)
2. Depth-1 orchestrator ontvangt de aankondiging, synthetiseert resultaten, voltooit → kondigt aan bij main
3. Main agent ontvangt de aankondiging en levert deze aan de gebruiker

Elk niveau ziet alleen aankondigingen van zijn directe kinderen.

### Toolbeleid

- **Depth 1 (orchestrator, wanneer `maxSpawnDepth >= 2`)**: Krijgt `sessions_spawn`, `subagents`, `sessions_list`, `sessions_history` zodat het zijn kinderen kan beheren. Andere session-/systeemtools blijven geweigerd.
- **Depth 1 (leaf, wanneer `maxSpawnDepth == 1`)**: Geen session tools (huidig standaardgedrag).
- **Depth 2 (leaf worker)**: Geen session tools — `sessions_spawn` wordt altijd geweigerd op depth 2. Kan geen verdere kinderen starten.

### Cross-agent spawning

Elke agent-sessie (op elk niveau) mag maximaal `maxChildrenPerAgent` (standaard: 5) actieve kinderen tegelijk hebben. Dit voorkomt ongecontroleerde vertakking vanuit één enkele orchestrator.

### Cascade stop

Het stoppen van een depth-1 orchestrator stopt automatisch al zijn depth-2 kinderen:

- `/stop` in de hoofdchat stopt alle depth-1 agents en cascadeert naar hun depth-2 kinderen.
- `) on the dedicated `subagent\` queue lane.
- `/subagents kill all` stopt alle sub-agents voor de aanvrager en cascadeert.

## Authenticatie

Sub-agentauthenticatie wordt bepaald op **agent-id**, niet op sessietype:

- Je kunt subagents refereren via lijstindex (`1`, `2`), run-id-prefix, volledige sessiesleutel of `last`.
- De auth-store wordt geladen vanuit de `agentDir` van de doelagent
- De auth-profielen van de hoofdagent worden samengevoegd als **fallback** (agentprofielen winnen bij conflicten)

De samenvoeging is additief — hoofdprofielen zijn altijd beschikbaar als fallbacks <Note>
Volledig geïsoleerde authenticatie per sub-agent wordt momenteel niet ondersteund.
</Note>

## Opdracht

Sub-agents rapporteren terug via een announce-stap:

- De announce-stap wordt uitgevoerd binnen de sub-agent-sessie (niet de aanvragerssessie).
- Als de sub-agent exact `ANNOUNCE_SKIP` antwoordt, wordt er niets geplaatst.
- When the sub-agent finishes, it announces its findings back to the requester chat.
- Announce-antwoorden behouden thread-/topicroutering wanneer beschikbaar (Slack-threads, Telegram-topics, Matrix-threads).
- Announce-berichten worden genormaliseerd naar een stabiel sjabloon:
  - `Status:` afgeleid van de uitvoeruitkomst (`success`, `error`, `timeout` of `unknown`).
  - `Result:` de samenvattende inhoud van de announce-stap (of `(not available)` indien ontbrekend).
  - `Notes:` foutdetails en andere nuttige context.
- Het aankondigingsbericht bevat een status die is afgeleid van de runtime-uitkomst (niet van de modeloutput):

Announce-payloads bevatten aan het einde een statistiekenregel (zelfs wanneer verpakt):

- Runtime (bijv. `runtime 5m12s`)
- Tokengebruik (invoer/uitvoer/totaal)
- Geschatte kosten (wanneer modelprijzen zijn geconfigureerd via `models.providers.*.models[].cost`)
- `sessionKey`, `sessionId` en transcriptpad (zodat de main agent geschiedenis kan ophalen via `sessions_history` of het bestand op schijf kan inspecteren)

## Subagent-tools aanpassen

Standaard krijgen subagents **alle tools behalve** een set geweigerde tools die onveilig of onnodig zijn voor achtergrondtaken:

- Expliciete `model`-parameter in de `sessions_spawn`-aanroep
- `sessions_history`
- `sessions_send`
- De tool `sessions_spawn`

Wanneer `maxSpawnDepth >= 2`, ontvangen depth-1 orchestrator sub-agents daarnaast `sessions_spawn`, `subagents`, `sessions_list` en `sessions_history` zodat zij hun kinderen kunnen beheren.

Overschrijven via configuratie:

```json5
{
  tools: {
    subagents: {
      tools: {
        allow: ["read", "exec", "process", "write", "edit", "apply_patch"],
        // deny wint nog steeds als ingesteld
      },
    },
  },
}
```

## Concurrency

Sub-agents gebruiken een toegewezen in-process wachtrij:

- :subagent:
- {
  agents: {
  defaults: {
  subagents: {
  maxConcurrent: 4, // default: 8
  },
  },
  },
  }

## Stoppen

- The agent will call the `sessions_spawn` tool behind the scenes. When the sub-agent finishes, it announces its findings back into your chat.
- `/subagents stop <id>`

## Beperkingen

- Sub-agent announce is **best-effort**. Als de gateway herstart, gaat in behandeling zijnd "announce back"-werk verloren.
- Sub-agents delen nog steeds dezelfde gateway-procesresources; beschouw `maxConcurrent` als een veiligheidsklep.
- The main agent calls `sessions_spawn` with a task description. The call is **non-blocking** — the main agent gets back `{ status: "accepted", runId, childSessionKey }` immediately.
- Sub-agentcontext injecteert alleen `AGENTS.md` + `TOOLS.md` (geen `SOUL.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md` of `BOOTSTRAP.md`).
- Maximale nestingdiepte is 5 (`maxSpawnDepth` bereik: 1–5). Depth 2 wordt aanbevolen voor de meeste use cases.
- `maxChildrenPerAgent` beperkt het aantal actieve kinderen per sessie (standaard: 5, bereik: 1–20).
