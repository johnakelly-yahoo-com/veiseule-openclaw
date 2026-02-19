---
summary: "Sub-agenter: starta isolerade agentkörningar som annonserar resultat tillbaka till den begärande chatten"
read_when:
  - Du vill ha bakgrunds-/parallellt arbete via agenten
  - Du ändrar sessions_spawn eller policy för sub-agentverktyg
title: "Sub-agenter"
---

# Sub-agenter

Sub‑agenter är bakgrundskörningar av agenter som startas från en befintlig agentkörning. De körs i sin egen session (`agent:<agentId>:subagent:<uuid>`) och när de är klara **meddelar** de sitt resultat tillbaka till den begärande chattkanalen.

## Snedstreckskommando

Använd `/subagents` för att inspektera eller styra sub‑agentkörningar för den **aktuella sessionen**:

- `/subagents list`
- `/subagents kill <id|#|all>`
- `/subagents log <id|#> [limit] [tools]`
- `/subagents info <id|#>`
- `/subagents send <id|#> <message>`

The simplest way to use sub-agents is to ask your agent naturally:

Huvudmål:

- Parallellisera "research / lång uppgift / långsamt verktyg" utan att blockera huvudkörningen.
- Håll sub‑agenter isolerade som standard (sessionsseparering + valfri sandboxing).
- Gör verktygsytan svår att missbruka: sub‑agenter får **inte** sessionverktyg som standard.
- Stöd konfigurerbart nästningsdjup för orkestreringsmönster.

Kostnadsnotering: varje sub‑agent har sin **egen** kontext och tokenanvändning. För tunga eller repetitiva
uppgifter, ställ in en billigare modell för sub‑agenter och behåll din huvudagent på en modell med högre kvalitet.
Du kan konfigurera detta via `agents.defaults.subagents.model` eller åsidosättningar per agent.

## Verktyg

Använd `sessions_spawn`:

- Startar en sub‑agentkörning (`deliver: false`, global lane: `subagent`)
- Kör sedan ett announce‑steg och publicerar announce‑svaret till den begärande chattkanalen
- Standardmodell: ärver anroparen om du inte sätter `agents.defaults.subagents.model` (eller per agent `agents.list[].subagents.model`); en explicit `sessions_spawn.model` gäller fortfarande.
- Standard thinking: ärver anroparen om du inte sätter `agents.defaults.subagents.thinking` (eller per agent `agents.list[].subagents.thinking`); en explicit `sessions_spawn.thinking` gäller fortfarande.

Verktygsparametrar:

- `task` (obligatorisk)
- `label?` (valfri)
- `agentId?` (valfri; starta under ett annat agent‑id om tillåtet)
- `model?` (valfri; åsidosätter sub‑agentens modell; ogiltiga värden hoppas över och sub‑agenten körs på standardmodellen med en varning i verktygsresultatet)
- `thinking?` (valfri; åsidosätter thinking‑nivå för sub‑agentkörningen)
- `runTimeoutSeconds?` (standard `0`; när satt avbryts sub‑agentkörningen efter N sekunder)
- `cleanup?` (`delete|keep`, standard `keep`)

Tillåtslista:

- `agents.list[].subagents.allowAgents`: lista över agent-ID:n som kan adresseras via `agentId` (`["*"]` för att tillåta alla). Standard: endast den begärande agenten.

Upptäckt:

- Använd `agents_list` för att se vilka agent-ID:n som för närvarande är tillåtna för `sessions_spawn`.

Automatisk arkivering:

- Underagentsessioner arkiveras automatiskt efter `agents.defaults.subagents.archiveAfterMinutes` (standard: 60).
- Arkivering använder `sessions.delete` och byter namn på transkriptet till `*.deleted.<timestamp>` (samma mapp).
- `cleanup: "delete"` arkiverar omedelbart efter announce (behåller fortfarande transkriptet genom namnbyte).
- Automatisk arkivering är en best-effort-funktion; väntande timers försvinner om gatewayen startas om.
- `runTimeoutSeconds` **inte** automatisk arkivering; den stoppar endast körningen. Sessionen kvarstår tills automatisk arkivering sker.
- Automatisk arkivering gäller lika för sessioner på djup 1 och djup 2.

## Nästlade underagenter

Som standard kan underagenter inte skapa egna underagenter (`maxSpawnDepth: 1`). Du kan aktivera en nivå av nästling genom att ange `maxSpawnDepth: 2`, vilket möjliggör **orchestrator-mönstret**: main → orchestrator-underagent → worker-under-underagenter.

### Så aktiverar du

```json5
{
  agents: {
    list: [
      {
        id: "researcher",
        subagents: {
          model: "anthropic/claude-sonnet-4",
        },
      },
      {
        id: "assistant",
        subagents: {
          model: "minimax/MiniMax-M2.1",
        },
      },
    ],
  },
}
```

### Samtidighet

| Djup | Sessionsnyckelns form                        | Roll                                                                | Kan skapa?                     |
| ---- | -------------------------------------------- | ------------------------------------------------------------------- | ------------------------------ |
| 0    | `agent:&lt;id&gt;:main`                            | Huvudagent                                                          | Alltid                         |
| 1    | `agent:&lt;id&gt;:subagent:<uuid>`                 | Underagent (orchestrator när djup 2 är tillåtet) | Endast om `maxSpawnDepth >= 2` |
| 2    | `agent:&lt;id&gt;:subagent:<uuid>:subagent:<uuid>` | Under-underagent (löv‑worker)                    | Aldrig                         |

### Announce-kedja

Underagenter använder en dedikerad köfil (lane) (`subagent`) som är separerad från huvudagentens kö, så att körningar av underagenter inte blockerar inkommande svar.

1. Worker på djup 2 slutför → annonserar till sin förälder (orchestrator på djup 1)
2. Orchestrator på djup 1 tar emot announce, sammanställer resultat, slutför → annonserar till main
3. Huvudagenten tar emot announce och levererar till användaren

Underagentsessioner arkiveras automatiskt efter en konfigurerbar tidsperiod:

### Verktygspolicy per djup

- **Djup 1 (orchestrator, när `maxSpawnDepth >= 2`)**: Får `sessions_spawn`, `subagents`, `sessions_list`, `sessions_history` så att den kan hantera sina underagenter. Andra sessions-/systemverktyg förblir nekade.
- **Djup 1 (löv, när `maxSpawnDepth == 1`)**: Inga sessionsverktyg (nuvarande standardbeteende).
- **Djup 2 (löv‑worker)**: Inga sessionsverktyg — `sessions_spawn` nekas alltid på djup 2. Kan inte skapa ytterligare barn.

### Verktyget `sessions_spawn`

Varje agentsession (på valfritt djup) kan ha högst `maxChildrenPerAgent` (standard: 5) aktiva barn samtidigt. Detta förhindrar okontrollerad spridning från en enskild orkestrerare.

### Parametrar

Att stoppa en orkestrerare på djup 1 stoppar automatiskt alla dess barn på djup 2:

- `/stop` i huvudchatten stoppar alla agenter på djup 1 och fortplantas till deras barn på djup 2.
- `/subagents kill <id>` stoppar en specifik underagent och fortplantas till dess barn.
- `/subagents kill all` stoppar alla underagenter för begäraren och fortplantas vidare.

## Autentisering

Sub-agentautentisering löses via **agent-id**, inte via sessionstyp:

- Sessionsnyckeln för underagenten är `agent:<agentId>:subagent:<uuid>`.
- Auth‑lagret laddas från den agentens `agentDir`.
- Huvudagentens auth‑profiler slås samman som en **reserv**; agentprofiler åsidosätter huvudprofiler vid konflikter.

Obs: sammanslagningen är additiv, så huvudprofiler är alltid tillgängliga som reserv. Fullständigt isolerad auth per agent stöds ännu inte.

## Meddela

Underagenter rapporterar tillbaka via ett announce‑steg:

- Announce‑steget körs inuti underagentens session (inte i begärarens session).
- Om underagenten svarar exakt `ANNOUNCE_SKIP` publiceras ingenting.
- Annars publiceras announce‑svaret i begärarens chattkanal via ett uppföljande `agent`‑anrop (`deliver=true`).
- Announce-svar bevarar tråd-/ämnesroutning när det finns (Slack-trådar, Telegram-ämnen, Matrix-trådar).
- Announce‑meddelanden normaliseras till en stabil mall:
  - `Status:` härledd från körningens utfall (`success`, `error`, `timeout` eller `unknown`).
  - `Result:` sammanfattningsinnehållet från announce‑steget (eller `(not available)` om det saknas).
  - `Notes:` feldetaljer och annan användbar kontext.
- `Status` härleds inte från modellens utdata; den kommer från signaler om körningsutfall.

Announce‑payloads inkluderar en statistikrad i slutet (även när de är inbäddade):

- Körtid (t.ex. `runtime 5m12s`)
- Tokenförbrukning (in/ut/totalt)
- Uppskattad kostnad när modellprissättning är konfigurerad (`models.providers.*.models[].cost`)
- `sessionKey`, `sessionId` och sökväg till transkriptet (så att huvudagenten kan hämta historik via `sessions_history` eller inspektera filen på disk)

## Verktygspolicy (underagentverktyg)

Som standard får underagenter **alla verktyg utom sessionsverktyg** och systemverktyg:

- `sessions_list`
- `sessions_history`
- `sessions_send`
- `sessions_spawn`

När `maxSpawnDepth >= 2` får orkestrerande underagenter på djup 1 dessutom `sessions_spawn`, `subagents`, `sessions_list` och `sessions_history` så att de kan hantera sina barn.

Åsidosätt via konfiguration:

`````json5
````
13. ```
🧭 Subagents (current session)
Active: 1 · Done: 2
1) ✅ · research logs · 2m31s · run a1b2c3d4 · agent:main:subagent:...
2) ✅ · check deps · 45s · run e5f6g7h8 · agent:main:subagent:...
3) 🔄 · deploy staging · 1m12s · run i9j0k1l2 · agent:main:subagent:...
```

```
/subagents stop 3
```

```
⚙️ Stop requested for deploy staging.
```
````
`````

## Samtidighet

Underagenter använder en dedikerad intern kö‑lane:

- Lane‑namn: `subagent`
- Samtidighet: `agents.defaults.subagents.maxConcurrent` (standard `8`)

## Stoppar

- Att skicka `/stop` i begärarchatten avbryter begärarsessionen och stoppar alla aktiva underagenter som har startats från den, och sprider sig till nästlade barn.
- `/subagents kill <id>` stoppar en specifik underagent och sprider sig till dess barn.

## Begränsningar

- Underagentens annonsering är **best-effort**. Om gatewayen startas om förloras väntande "announce back"-arbete.
- Underagenter delar fortfarande samma gatewayprocessresurser; behandla `maxConcurrent` som en säkerhetsventil.
- `sessions_spawn` är alltid icke-blockerande: den returnerar `{ status: "accepted", runId, childSessionKey }` omedelbart.
- Underagentens kontext injicerar endast `AGENTS.md` + `TOOLS.md` (ingen `SOUL.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md` eller `BOOTSTRAP.md`).
- Maximalt nästlingsdjup är 5 (`maxSpawnDepth` intervall: 1–5). Djup 2 rekommenderas för de flesta användningsfall.
- `maxChildrenPerAgent` begränsar aktiva barn per session (standard: 5, intervall: 1–20).
