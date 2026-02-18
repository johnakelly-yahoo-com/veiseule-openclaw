---
title: "Sub-agenten"
---

# Sub-agenten

Sub-agents zijn achtergrond-agentruns die worden gestart vanuit een bestaande agentrun. Ze draaien in hun eigen sessie (`agent:<agentId>:subagent:<uuid>`) en **kondigen** hun resultaat na voltooiing aan terug in het chatkanaal van de aanvrager.

## Slash-opdracht

Gebruik `/subagents` om sub-agentruns voor de **huidige sessie** te inspecteren of te beheren:

- `/subagents list`
- `/subagents kill <id|#|all>`
- `/subagents log <id|#> [limit] [tools]`
- `/subagents info <id|#>`
- `/subagents send <id|#> <message>`

`/subagents info` toont run-metadata (status, tijdstempels, sessie-id, transcriptpad, opschoning).

Primaire doelen:

- Paralleliseer "research / lange taak / trage tool"-werk zonder de hoofdrun te blokkeren.
- Houd sub-agents standaard geïsoleerd (sessiescheiding + optionele sandboxing).
- Houd het tooloppervlak moeilijk te misbruiken: sub-agents krijgen **geen** sessietools standaard.
- Ondersteun configureerbare nestingdiepte voor orchestrator-patronen.

Kostenopmerking: elke sub-agent heeft zijn **eigen** context en tokenverbruik. Voor zware of repetitieve taken stel je een goedkoper model in voor sub-agents en houd je je hoofdagent op een model van hogere kwaliteit. Je kunt dit configureren via `agents.defaults.subagents.model` of per-agent-overschrijvingen.

## Tool

Gebruik `sessions_spawn`:

- Start een sub-agentrun (`deliver: false`, globale lane: `subagent`)
- Voert daarna een announce-stap uit en plaatst het announce-antwoord in het chatkanaal van de aanvrager
- Standaardmodel: erft van de aanroeper tenzij je `agents.defaults.subagents.model` instelt (of per-agent `agents.list[].subagents.model`); een expliciete `sessions_spawn.model` heeft nog steeds voorrang.
- Standaard thinking: erft van de aanroeper tenzij je `agents.defaults.subagents.thinking` instelt (of per-agent `agents.list[].subagents.thinking`); een expliciete `sessions_spawn.thinking` heeft nog steeds voorrang.

Toolparameters:

- `task` (verplicht)
- `label?` (optioneel)
- `agentId?` (optioneel; start onder een andere agent-id indien toegestaan)
- `model?` (optioneel; overschrijft het sub-agentmodel; ongeldige waarden worden overgeslagen en de sub-agent draait op het standaardmodel met een waarschuwing in het toolresultaat)
- `thinking?` (optioneel; overschrijft het denkniveau voor de sub-agentrun)
- `runTimeoutSeconds?` (standaard `0`; indien ingesteld wordt de sub-agentrun na N seconden afgebroken)
- `cleanup?` (`delete|keep`, standaard `keep`)

Allowlist:

- `agents.list[].subagents.allowAgents`: lijst met agent-id’s die via `agentId` mogen worden getarget (`["*"]` om alles toe te staan). Standaard: alleen de aanvragende agent.

Discovery:

- Gebruik `agents_list` om te zien welke agent-id’s momenteel zijn toegestaan voor `sessions_spawn`.

Auto-archive:

- Sub-agentsessies worden automatisch gearchiveerd na `agents.defaults.subagents.archiveAfterMinutes` (standaard: 60).
- Archiveren gebruikt `sessions.delete` en hernoemt het transcript naar `*.deleted.<timestamp>` (dezelfde map).
- `cleanup: "delete"` archiveert onmiddellijk na announce (het transcript blijft behouden via hernoemen).
- Auto-archive is best-effort; wachtende timers gaan verloren als de gateway opnieuw wordt gestart.
- `runTimeoutSeconds` archiveert **niet** automatisch; het stopt alleen de run. De sessie blijft bestaan tot auto-archive.
- Auto-archive geldt zowel voor depth-1- als depth-2-sessies.

## Geneste sub-agents

Standaard kunnen sub-agents geen eigen sub-agents starten (`maxSpawnDepth: 1`). Je kunt één niveau nesting inschakelen door `maxSpawnDepth: 2` in te stellen, wat het **orchestrator-patroon** mogelijk maakt: main → orchestrator-sub-agent → worker-sub-sub-agents.

### Hoe inschakelen

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxSpawnDepth: 2, // sta sub-agents toe kinderen te starten (standaard: 1)
        maxChildrenPerAgent: 5, // max actieve kinderen per agentsessie (standaard: 5)
        maxConcurrent: 8, // globale concurrency-limiet voor de lane (standaard: 8)
      },
    },
  },
}
```

### Diepteniveaus

| Depth | Session key shape                            | Rol                                           | Kan starten?                 |
| ----- | -------------------------------------------- | --------------------------------------------- | ---------------------------- |
| 0     | `agent:<id>:main`                            | Hoofdagent                                   | Altijd                      |
| 1     | `agent:<id>:subagent:<uuid>`                 | Sub-agent (orchestrator wanneer depth 2 is toegestaan) | Alleen als `maxSpawnDepth >= 2` |
| 2     | `agent:<id>:subagent:<uuid>:subagent:<uuid>` | Sub-sub-agent (leaf worker)                  | Nooit                       |

### Announce-keten

Resultaten stromen terug omhoog in de keten:

1. Depth-2 worker voltooit → kondigt aan bij zijn ouder (depth-1 orchestrator)
2. Depth-1 orchestrator ontvangt de announce, synthetiseert resultaten, voltooit → kondigt aan bij main
3. De hoofdagent ontvangt de announce en levert deze aan de gebruiker

Elk niveau ziet alleen announces van zijn directe kinderen.

### Toolbeleid per diepte

- **Depth 1 (orchestrator, wanneer `maxSpawnDepth >= 2`)**: Krijgt `sessions_spawn`, `subagents`, `sessions_list`, `sessions_history` zodat hij zijn kinderen kan beheren. Andere sessie-/systeemtools blijven geweigerd.
- **Depth 1 (leaf, wanneer `maxSpawnDepth == 1`)**: Geen sessietools (huidig standaardgedrag).
- **Depth 2 (leaf worker)**: Geen sessietools — `sessions_spawn` is altijd geweigerd op depth 2. Kan geen verdere kinderen starten.

### Per-agent spawn-limiet

Elke agentsessie (op elk niveau) kan maximaal `maxChildrenPerAgent` (standaard: 5) actieve kinderen tegelijk hebben. Dit voorkomt runaway fan-out vanuit één orchestrator.

### Cascade stop

Het stoppen van een depth-1-orchestrator stopt automatisch al zijn depth-2-kinderen:

- `/stop` in de hoofdchat stopt alle depth-1-agents en cascadeert naar hun depth-2-kinderen.
- `/subagents kill <id>` stopt een specifieke sub-agent en cascadeert naar zijn kinderen.
- `/subagents kill all` stopt alle sub-agents voor de aanvrager en cascadeert.

## Authenticatie

Sub-agentauthenticatie wordt bepaald op **agent-id**, niet op sessietype:

- De sub-agentsessiesleutel is `agent:<agentId>:subagent:<uuid>`.
- De auth-store wordt geladen vanuit de `agentDir` van die agent.
- De auth-profielen van de hoofdagent worden samengevoegd als **fallback**; agentprofielen overschrijven hoofdprofielen bij conflicten.

Opmerking: de samenvoeging is additief, dus hoofdprofielen zijn altijd beschikbaar als fallbacks. Volledig geïsoleerde authenticatie per agent wordt nog niet ondersteund.

## Announce

Sub-agents rapporteren terug via een announce-stap:

- De announce-stap draait binnen de sub-agentsessie (niet de sessie van de aanvrager).
- Als de sub-agent exact `ANNOUNCE_SKIP` antwoordt, wordt er niets geplaatst.
- Anders wordt het announce-antwoord geplaatst in het chatkanaal van de aanvrager via een follow-up `agent`-aanroep (`deliver=true`).
- Announce-antwoorden behouden thread-/topicroutering wanneer beschikbaar (Slack-threads, Telegram-topics, Matrix-threads).
- Announce-berichten worden genormaliseerd naar een stabiel sjabloon:
  - `Status:` afgeleid van de run-uitkomst (`success`, `error`, `timeout`, of `unknown`).
  - `Result:` de samenvattingsinhoud van de announce-stap (of `(not available)` indien ontbrekend).
  - `Notes:` foutdetails en andere nuttige context.
- `Status` wordt niet afgeleid van modeloutput; het komt van runtime-uitkomstsignalen.

Announce-payloads bevatten aan het einde een statistiekregel (ook wanneer ingepakt):

- Runtime (bijv. `runtime 5m12s`)
- Tokengebruik (input/output/totaal)
- Geschatte kosten wanneer modelprijzen zijn geconfigureerd (`models.providers.*.models[].cost`)
- `sessionKey`, `sessionId` en transcriptpad (zodat de hoofdagent geschiedenis kan ophalen via `sessions_history` of het bestand op schijf kan inspecteren)

## Toolbeleid (sub-agenttools)

Standaard krijgen sub-agents **alle tools behalve sessietools** en systeemtools:

- `sessions_list`
- `sessions_history`
- `sessions_send`
- `sessions_spawn`

Wanneer `maxSpawnDepth >= 2`, krijgen depth-1-orchestrator-sub-agents aanvullend `sessions_spawn`, `subagents`, `sessions_list` en `sessions_history` zodat zij hun kinderen kunnen beheren.

Overschrijven via configuratie:

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxConcurrent: 1,
      },
    },
  },
  tools: {
    subagents: {
      tools: {
        // deny wint
        deny: ["gateway", "cron"],
        // als allow is ingesteld, wordt het allow-only (deny wint nog steeds)
        // allow: ["read", "exec", "process"]
      },
    },
  },
}
```

## Concurrency

Sub-agents gebruiken een aparte in-process queue-lane:

- Lane-naam: `subagent`
- Concurrency: `agents.defaults.subagents.maxConcurrent` (standaard `8`)

## Stoppen

- Het versturen van `/stop` in de chat van de aanvrager breekt de aanvragersessie af en stopt alle actieve sub-agentruns die daaruit zijn gestart, inclusief geneste kinderen.
- `/subagents kill <id>` stopt een specifieke sub-agent en cascadeert naar zijn kinderen.

## Beperkingen

- Sub-agent announce is **best-effort**. Als de gateway opnieuw wordt gestart, gaat wachtend "announce back"-werk verloren.
- Sub-agents delen nog steeds dezelfde gatewayprocesresources; behandel `maxConcurrent` als veiligheidsventiel.
- `sessions_spawn` is altijd non-blocking: het retourneert onmiddellijk `{ status: "accepted", runId, childSessionKey }`.
- Sub-agentcontext injecteert alleen `AGENTS.md` + `TOOLS.md` (geen `SOUL.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md` of `BOOTSTRAP.md`).
- Maximale nestingdiepte is 5 (`maxSpawnDepth` bereik: 1–5). Depth 2 wordt aanbevolen voor de meeste use cases.
- `maxChildrenPerAgent` begrenst actieve kinderen per sessie (standaard: 5, bereik: 1–20).