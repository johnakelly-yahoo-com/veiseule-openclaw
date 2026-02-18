---
title: "Sub-agenter"
---

# Sub-agenter

Sub-agenter er baggrundskørsler af agenter, der startes fra en eksisterende agentkørsel. De kører i deres egen session (`agent:<agentId>:subagent:<uuid>`) og, når de er færdige, **annoncerer** deres resultat tilbage til den anmodende chatkanal.

## Slash-kommando

Brug `/subagents` til at inspicere eller styre sub-agent-kørsler for den **nuværende session**:

- `/subagents list`
- `/subagents kill <id|#|all>`
- `/subagents log <id|#> [limit] [tools]`
- `/subagents info <id|#>`
- `/subagents send <id|#> <message>`

`/subagents info` viser kørselsmetadata (status, tidsstempler, session id, transskriptsti, oprydning).

Primære mål:

- Parallelisér "research / lang opgave / langsomt værktøj"-arbejde uden at blokere hovedkørslen.
- Hold sub-agenter isoleret som standard (sessionsadskillelse + valgfri sandboxing).
- Gør værktøjsfladen svær at misbruge: sub-agenter får **ikke** sessionsværktøjer som standard.
- Understøt konfigurerbar indlejringsdybde til orkestrator-mønstre.

Omkostningsnote: hver sub-agent har sin **egen** kontekst og sit eget tokenforbrug. For tunge eller gentagne opgaver bør du sætte en billigere model for sub-agenter og beholde din hovedagent på en model af højere kvalitet.
Du kan konfigurere dette via `agents.defaults.subagents.model` eller per-agent overrides.

## Værktøj

Brug `sessions_spawn`:

- Starter en sub-agent-kørsel (`deliver: false`, global lane: `subagent`)
- Kører derefter et announce-trin og poster announce-svaret til den anmodende chatkanal
- Standardmodel: arver fra kaldende agent, medmindre du sætter `agents.defaults.subagents.model` (eller per-agent `agents.list[].subagents.model`); en eksplicit `sessions_spawn.model` har stadig forrang.
- Standard thinking: arver fra kaldende agent, medmindre du sætter `agents.defaults.subagents.thinking` (eller per-agent `agents.list[].subagents.thinking`); en eksplicit `sessions_spawn.thinking` har stadig forrang.

Værktøjsparametre:

- `task` (påkrævet)
- `label?` (valgfri)
- `agentId?` (valgfri; start under et andet agent-id hvis tilladt)
- `model?` (valgfri; tilsidesætter sub-agent-modellen; ugyldige værdier springes over, og sub-agenten kører på standardmodellen med en advarsel i værktøjsresultatet)
- `thinking?` (valgfri; tilsidesætter thinking-niveau for sub-agent-kørslen)
- `runTimeoutSeconds?` (standard `0`; når sat, afbrydes sub-agent-kørslen efter N sekunder)
- `cleanup?` (`delete|keep`, standard `keep`)

Allowlist:

- `agents.list[].subagents.allowAgents`: liste over agent-id’er, der kan målrettes via `agentId` (`["*"]` for at tillade alle). Standard: kun den anmodende agent.

Discovery:

- Brug `agents_list` for at se hvilke agent-id’er der aktuelt er tilladt for `sessions_spawn`.

Auto-arkivering:

- Sub-agent-sessioner arkiveres automatisk efter `agents.defaults.subagents.archiveAfterMinutes` (standard: 60).
- Arkivering bruger `sessions.delete` og omdøber transskriptet til `*.deleted.<timestamp>` (samme mappe).
- `cleanup: "delete"` arkiverer straks efter announce (beholder stadig transskriptet via omdøbning).
- Auto-arkivering er best-effort; afventende timere går tabt, hvis gatewayen genstarter.
- `runTimeoutSeconds` auto-arkiverer **ikke**; den stopper kun kørslen. Sessionen forbliver indtil auto-arkivering.
- Auto-arkivering gælder ens for depth-1 og depth-2 sessioner.

## Indlejrede sub-agenter

Som standard kan sub-agenter ikke starte deres egne sub-agenter (`maxSpawnDepth: 1`). Du kan aktivere ét niveau af indlejring ved at sætte `maxSpawnDepth: 2`, hvilket tillader **orkestrator-mønsteret**: main → orkestrator-sub-agent → worker-sub-sub-agenter.

### Sådan aktiveres det

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxSpawnDepth: 2, // tillad at sub-agenter starter børn (standard: 1)
        maxChildrenPerAgent: 5, // maks. aktive børn pr. agent-session (standard: 5)
        maxConcurrent: 8, // global samtidighedsgrænse for lane (standard: 8)
      },
    },
  },
}
```

### Dybdeniveauer

| Depth | Session key shape                            | Rolle                                          | Kan starte?                   |
| ----- | -------------------------------------------- | ---------------------------------------------- | ----------------------------- |
| 0     | `agent:<id>:main`                            | Hovedagent                                     | Altid                         |
| 1     | `agent:<id>:subagent:<uuid>`                 | Sub-agent (orkestrator når depth 2 er tilladt) | Kun hvis `maxSpawnDepth >= 2` |
| 2     | `agent:<id>:subagent:<uuid>:subagent:<uuid>` | Sub-sub-agent (leaf worker)                    | Aldrig                        |

### Announce-kæde

Resultater flyder tilbage op gennem kæden:

1. Depth-2 worker afslutter → annoncerer til sin parent (depth-1 orkestrator)
2. Depth-1 orkestrator modtager announce, syntetiserer resultater, afslutter → annoncerer til main
3. Hovedagenten modtager announce og leverer til brugeren

Hvert niveau ser kun announces fra sine direkte børn.

### Værktøjspolitik pr. dybde

- **Depth 1 (orkestrator, når `maxSpawnDepth >= 2`)**: Får `sessions_spawn`, `subagents`, `sessions_list`, `sessions_history`, så den kan administrere sine børn. Andre sessions-/systemværktøjer forbliver afvist.
- **Depth 1 (leaf, når `maxSpawnDepth == 1`)**: Ingen sessionsværktøjer (nuværende standardadfærd).
- **Depth 2 (leaf worker)**: Ingen sessionsværktøjer — `sessions_spawn` er altid afvist ved depth 2. Kan ikke starte yderligere børn.

### Per-agent spawn-grænse

Hver agent-session (på ethvert niveau) kan højst have `maxChildrenPerAgent` (standard: 5) aktive børn ad gangen. Dette forhindrer ukontrolleret fan-out fra en enkelt orkestrator.

### Kaskadestop

Stop af en depth-1 orkestrator stopper automatisk alle dens depth-2 børn:

- `/stop` i hovedchatten stopper alle depth-1 agenter og kaskaderer til deres depth-2 børn.
- `/subagents kill <id>` stopper en specifik sub-agent og kaskaderer til dens børn.
- `/subagents kill all` stopper alle sub-agenter for den anmodende og kaskaderer.

## Autentificering

Sub-agent-autentificering afgøres af **agent-id**, ikke af sessionstype:

- Sub-agentens sessionsnøgle er `agent:<agentId>:subagent:<uuid>`.
- Auth store indlæses fra den pågældende agents `agentDir`.
- Hovedagentens auth-profiler flettes ind som en **fallback**; agentprofiler har forrang ved konflikter.

Note: Fletningen er additiv, så hovedprofiler er altid tilgængelige som fallback. Fuldt isoleret auth pr. agent understøttes endnu ikke.

## Announce

Sub-agenter rapporterer tilbage via et announce-trin:

- Announce-trinnet kører inde i sub-agent-sessionen (ikke i den anmodende session).
- Hvis sub-agenten svarer præcist `ANNOUNCE_SKIP`, postes der intet.
- Ellers postes announce-svaret til den anmodende chatkanal via et opfølgende `agent`-kald (`deliver=true`).
- Announce-svar bevarer tråd-/emnerouting, når det er tilgængeligt (Slack-tråde, Telegram-emner, Matrix-tråde).
- Announce-beskeder normaliseres til en stabil skabelon:
  - `Status:` afledt af kørselsresultatet (`success`, `error`, `timeout` eller `unknown`).
  - `Result:` resuméindholdet fra announce-trinnet (eller `(not available)` hvis mangler).
  - `Notes:` fejldetaljer og anden nyttig kontekst.
- `Status` udledes ikke fra modeloutput; den kommer fra runtime-signaler.

Announce-payloads inkluderer en stats-linje til sidst (selv når indpakket):

- Runtime (f.eks. `runtime 5m12s`)
- Tokenforbrug (input/output/total)
- Estimeret omkostning når modelpriser er konfigureret (`models.providers.*.models[].cost`)
- `sessionKey`, `sessionId` og transskriptsti (så hovedagenten kan hente historik via `sessions_history` eller inspicere filen på disk)

## Værktøjspolitik (sub-agent-værktøjer)

Som standard får sub-agenter **alle værktøjer undtagen sessionsværktøjer** og systemværktøjer:

- `sessions_list`
- `sessions_history`
- `sessions_send`
- `sessions_spawn`

Når `maxSpawnDepth >= 2`, får depth-1 orkestrator-sub-agenter derudover `sessions_spawn`, `subagents`, `sessions_list` og `sessions_history`, så de kan administrere deres børn.

Tilsidesæt via konfiguration:

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

## Samtidighed

Sub-agenter bruger en dedikeret in-process queue lane:

- Lane-navn: `subagent`
- Samtidighed: `agents.defaults.subagents.maxConcurrent` (standard `8`)

## Stop

- Afsendelse af `/stop` i den anmodende chat afbryder den anmodende session og stopper alle aktive sub-agent-kørsler startet fra den, med kaskade til indlejrede børn.
- `/subagents kill <id>` stopper en specifik sub-agent og kaskaderer til dens børn.

## Begrænsninger

- Sub-agent announce er **best-effort**. Hvis gatewayen genstarter, går afventende "announce back"-arbejde tabt.
- Sub-agenter deler stadig de samme gateway-procesressourcer; behandl `maxConcurrent` som en sikkerhedsventil.
- `sessions_spawn` er altid non-blocking: den returnerer `{ status: "accepted", runId, childSessionKey }` med det samme.
- Sub-agent-kontekst injicerer kun `AGENTS.md` + `TOOLS.md` (ingen `SOUL.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md` eller `BOOTSTRAP.md`).
- Maksimal indlejringsdybde er 5 (`maxSpawnDepth` interval: 1–5). Depth 2 anbefales til de fleste use cases.
- `maxChildrenPerAgent` begrænser aktive børn pr. session (standard: 5, interval: 1–20).