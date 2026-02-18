---
title: "Sub-agenter"
---

# Sub-agenter

Sub-agenter är bakgrundskörningar som startas från en befintlig agentkörning. De kör i sin egen session (`agent:<agentId>:subagent:<uuid>`) och, när de är klara, **annonserar** sitt resultat tillbaka till den begärande chattkanalen.

## Snedstreckskommando

Använd `/subagents` för att inspektera eller styra underagentkörningar för den **aktuella sessionen**:

- `/subagents list`
- `/subagents kill <id|#|all>`
- `/subagents log <id|#> [limit] [tools]`
- `/subagents info <id|#>`
- `/subagents send <id|#> <message>`

`/subagents info` visar körningsmetadata (status, tidsstämplar, sessions-id, transkriptsökväg, rensning).

Primära mål:

- Parallellisera "research / lång uppgift / långsamt verktyg"-arbete utan att blockera huvudkörningen.
- Hålla underagenter isolerade som standard (sessionsseparation + valfri sandboxning).
- Göra verktygsytan svår att missbruka: underagenter får **inte** session-verktyg som standard.
- Stödja konfigurerbart nästlingsdjup för orkestreringsmönster.

Kostnadsnotering: varje underagent har sin **egen** kontext och tokenanvändning. För tunga eller repetitiva
uppgifter, sätt en billigare modell för underagenter och behåll din huvudagent på en modell med högre kvalitet.
Du kan konfigurera detta via `agents.defaults.subagents.model` eller per-agent-åsidosättningar.

## Verktyg

Använd `sessions_spawn`:

- Startar en underagentkörning (`deliver: false`, global lane: `subagent`)
- Kör sedan ett announce-steg och postar announce-svaret till den begärande chattkanalen
- Standardmodell: ärver anroparen om du inte sätter `agents.defaults.subagents.model` (eller per-agent `agents.list[].subagents.model`); en explicit `sessions_spawn.model` vinner fortfarande.
- Standard thinking: ärver anroparen om du inte sätter `agents.defaults.subagents.thinking` (eller per-agent `agents.list[].subagents.thinking`); en explicit `sessions_spawn.thinking` vinner fortfarande.

Verktygsparametrar:

- `task` (obligatorisk)
- `label?` (valfri)
- `agentId?` (valfri; starta under ett annat agent-id om tillåtet)
- `model?` (valfri; åsidosätter underagentens modell; ogiltiga värden hoppas över och underagenten körs på standardmodellen med en varning i verktygsresultatet)
- `thinking?` (valfri; åsidosätter thinking-nivå för underagentkörningen)
- `runTimeoutSeconds?` (standard `0`; när satt avbryts underagentkörningen efter N sekunder)
- `cleanup?` (`delete|keep`, standard `keep`)

Tillåtelselista:

- `agents.list[].subagents.allowAgents`: lista över agent-id:n som kan målas via `agentId` (`["*"]` för att tillåta alla). Standard: endast den begärande agenten.

Upptäckt:

- Använd `agents_list` för att se vilka agent-id:n som för närvarande är tillåtna för `sessions_spawn`.

Automatisk arkivering:

- Underagentsessioner arkiveras automatiskt efter `agents.defaults.subagents.archiveAfterMinutes` (standard: 60).
- Arkivering använder `sessions.delete` och byter namn på transkriptet till `*.deleted.<timestamp>` (samma mapp).
- `cleanup: "delete"` arkiverar omedelbart efter announce (behåller fortfarande transkriptet via namnbyte).
- Automatisk arkivering är best-effort; väntande timers går förlorade om gatewayen startas om.
- `runTimeoutSeconds` autoarkiverar **inte**; det stoppar endast körningen. Sessionen kvarstår tills autoarkivering.
- Automatisk arkivering gäller lika för djup-1 och djup-2-sessioner.

## Nästlade underagenter

Som standard kan underagenter inte skapa egna underagenter (`maxSpawnDepth: 1`). Du kan aktivera en nivå av nästling genom att sätta `maxSpawnDepth: 2`, vilket tillåter **orkestreringsmönstret**: main → orkestrator-underagent → worker-under-underagenter.

### Så aktiverar du

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxSpawnDepth: 2, // tillåt underagenter att skapa barn (standard: 1)
        maxChildrenPerAgent: 5, // max aktiva barn per agentsession (standard: 5)
        maxConcurrent: 8, // global samtidighetsgräns för lane (standard: 8)
      },
    },
  },
}
```

### Djupnivåer

| Depth | Session key shape                            | Roll                                          | Kan skapa?                   |
| ----- | -------------------------------------------- | --------------------------------------------- | ---------------------------- |
| 0     | `agent:<id>:main`                            | Huvudagent                                    | Alltid                       |
| 1     | `agent:<id>:subagent:<uuid>`                 | Underagent (orkestrator när djup 2 är tillåtet) | Endast om `maxSpawnDepth >= 2` |
| 2     | `agent:<id>:subagent:<uuid>:subagent:<uuid>` | Under-underagent (lövarbetare)                   | Aldrig                        |

### Announce-kedja

Resultat flödar tillbaka upp i kedjan:

1. Djup-2 worker avslutas → annonserar till sin förälder (djup-1 orkestrator)
2. Djup-1 orkestrator tar emot announce, syntetiserar resultat, avslutas → annonserar till main
3. Huvudagenten tar emot announce och levererar till användaren

Varje nivå ser endast annonser från sina direkta barn.

### Verktygspolicy per djup

- **Djup 1 (orkestrator, när `maxSpawnDepth >= 2`)**: Får `sessions_spawn`, `subagents`, `sessions_list`, `sessions_history` så att den kan hantera sina barn. Övriga session-/systemverktyg förblir nekade.
- **Djup 1 (lövnivå, när `maxSpawnDepth == 1`)**: Inga sessionsverktyg (nuvarande standardbeteende).
- **Djup 2 (lövarbetare)**: Inga sessionsverktyg — `sessions_spawn` nekas alltid på djup 2. Kan inte skapa fler barn.

### Spawn-gräns per agent

Varje agentsession (på vilken nivå som helst) kan ha högst `maxChildrenPerAgent` (standard: 5) aktiva barn samtidigt. Detta förhindrar okontrollerad fan-out från en enskild orkestrator.

### Kaskadstopp

Att stoppa en djup-1-orkestrator stoppar automatiskt alla dess djup-2-barn:

- `/stop` i huvudchatten stoppar alla djup-1-agenter och kaskaderar till deras djup-2-barn.
- `/subagents kill <id>` stoppar en specifik underagent och kaskaderar till dess barn.
- `/subagents kill all` stoppar alla underagenter för den begärande och kaskaderar.

## Autentisering

Underagentautentisering löses via **agent-id**, inte via sessionstyp:

- Underagentens sessionsnyckel är `agent:<agentId>:subagent:<uuid>`.
- Autentiseringslagret laddas från den agentens `agentDir`.
- Huvudagentens autentiseringsprofiler slås samman som en **fallback**; agentprofiler åsidosätter huvudprofiler vid konflikter.

Obs: sammanslagningen är additiv, så huvudprofiler är alltid tillgängliga som fallbacks. Fullständigt isolerad autentisering per agent stöds ännu inte.

## Announce

Underagenter rapporterar tillbaka via ett announce-steg:

- Announce-steget körs inuti underagentsessionen (inte den begärande sessionen).
- Om underagenten svarar exakt `ANNOUNCE_SKIP` publiceras ingenting.
- Annars publiceras announce-svaret till den begärande chattkanalen via ett uppföljande `agent`-anrop (`deliver=true`).
- Announce-svar bevarar tråd-/ämnesroutning när det finns (Slack-trådar, Telegram-ämnen, Matrix-trådar).
- Announce-meddelanden normaliseras till en stabil mall:
  - `Status:` härleds från körningens utfall (`success`, `error`, `timeout` eller `unknown`).
  - `Result:` sammanfattningsinnehållet från announce-steget (eller `(not available)` om det saknas).
  - `Notes:` feldetaljer och annan användbar kontext.
- `Status` härleds inte från modellens utdata; den kommer från körningens utfallssignaler.

Announce-payloads inkluderar en statistikrad i slutet (även när de är wrapperade):

- Körtid (t.ex. `runtime 5m12s`)
- Tokenanvändning (input/output/total)
- Uppskattad kostnad när modellprissättning är konfigurerad (`models.providers.*.models[].cost`)
- `sessionKey`, `sessionId` och transkriptsökväg (så att huvudagenten kan hämta historik via `sessions_history` eller inspektera filen på disk)

## Verktygspolicy (underagentverktyg)

Som standard får underagenter **alla verktyg utom sessionverktyg** och systemverktyg:

- `sessions_list`
- `sessions_history`
- `sessions_send`
- `sessions_spawn`

När `maxSpawnDepth >= 2` får djup-1-orkestrator-underagenter dessutom `sessions_spawn`, `subagents`, `sessions_list` och `sessions_history` så att de kan hantera sina barn.

Åsidosätt via konfiguration:

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
        // deny wins
        deny: ["gateway", "cron"],
        // if allow is set, it becomes allow-only (deny still wins)
        // allow: ["read", "exec", "process"]
      },
    },
  },
}
```

## Samtidighet

Underagenter använder en dedikerad in-process queue lane:

- Lane-namn: `subagent`
- Samtidighet: `agents.defaults.subagents.maxConcurrent` (standard `8`)

## Stoppa

- Att skicka `/stop` i den begärande chatten avbryter den begärande sessionen och stoppar alla aktiva underagentkörningar som startats från den, med kaskad till nästlade barn.
- `/subagents kill <id>` stoppar en specifik underagent och kaskaderar till dess barn.

## Begränsningar

- Underagent-announce är **best-effort**. Om gatewayen startas om går väntande "announce back"-arbete förlorat.
- Underagenter delar fortfarande samma gateway-processresurser; behandla `maxConcurrent` som en säkerhetsventil.
- `sessions_spawn` är alltid icke-blockerande: det returnerar `{ status: "accepted", runId, childSessionKey }` omedelbart.
- Underagentens kontext injicerar endast `AGENTS.md` + `TOOLS.md` (inga `SOUL.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md` eller `BOOTSTRAP.md`).
- Maximalt nästlingsdjup är 5 (`maxSpawnDepth` intervall: 1–5). Djup 2 rekommenderas för de flesta användningsfall.
- `maxChildrenPerAgent` begränsar aktiva barn per session (standard: 5, intervall: 1–20).