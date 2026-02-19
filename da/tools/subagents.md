---
summary: "Sub-agenter: opstart af isolerede agentkørsler, der annoncerer resultater tilbage til anmoderens chat"
read_when:
  - Du ønsker baggrunds-/parallelarbejde via agenten
  - Du ændrer sessions_spawn eller politik for sub-agent-værktøjer
title: "Sub-agenter"
---

# Sub-agenter

Sub-agenter er baggrunds-agentkørsler, der startes fra en eksisterende agentkørsel. De kører i deres egen session (`agent:<agentId>:subagent:<uuid>`) og, når de er færdige, **annoncerer** de deres resultat tilbage til den anmodende chatkanal.

## Slash-kommando

Brug `/subagents` til at inspicere eller styre sub-agentkørsler for den **nuværende session**:

- `/subagents list`
- `/subagents kill <id|#|all>`
- `/subagents log <id|#> [limit] [tools]`
- `/subagents info <id|#>`
- `/subagents send <id|#> <message>`

Den nemmeste måde at bruge sub-agenter på er at bede din agent naturligt:

Primære mål:

- Parallelisér "research / lang opgave / langsomt værktøj"-arbejde uden at blokere hovedkørslen.
- Hold sub-agenter isolerede som standard (sessionsadskillelse + valgfri sandboxing).
- Hold værktøjsfladen svær at misbruge: sub-agenter får **ikke** session-værktøjer som standard.
- Understøt konfigurerbar indlejringsdybde til orchestrator-mønstre.

Omkostningsnote: hver sub-agent har sin **egen** kontekst og tokenforbrug. For tunge eller gentagne opgaver skal du angive en billigere model til sub-agenter og beholde din hovedagent på en model af højere kvalitet.
Du kan konfigurere dette via `agents.defaults.subagents.model` eller pr.-agent-overskrivninger.

## Værktøj

Brug `sessions_spawn`:

- Starter en sub-agentkørsel (`deliver: false`, global lane: `subagent`)
- Kører derefter et announce-trin og sender announce-svaret til den anmodende chatkanal
- Standardmodel: arver fra kalderen, medmindre du angiver `agents.defaults.subagents.model` (eller pr.-agent `agents.list[].subagents.model`); en eksplicit `sessions_spawn.model` har stadig forrang.
- Standard thinking: arver fra kalderen, medmindre du angiver `agents.defaults.subagents.thinking` (eller pr.-agent `agents.list[].subagents.thinking`); en eksplicit `sessions_spawn.thinking` har stadig forrang.

Værktøjsparametre:

- `task` (påkrævet)
- `label?` (valgfri)
- `agentId?` (valgfri; start under et andet agent-id, hvis tilladt)
- `model?` (valgfri; tilsidesætter sub-agentens model; ugyldige værdier springes over, og sub-agenten kører på standardmodellen med en advarsel i værktøjsresultatet)
- `thinking?` (valgfri; tilsidesætter thinking-niveauet for sub-agentkørslen)
- `runTimeoutSeconds?` (standard `0`; når sat, afbrydes sub-agentens kørsel efter N sekunder)
- `cleanup?` (`delete|keep`, standard `keep`)

Allowlist:

- `agents.list[].subagents.allowAgents`: liste over agent-id'er, der kan målrettes via `agentId` (`["*"]` for at tillade alle). Standard: kun den anmodende agent.

Discovery:

- Brug `agents_list` for at se, hvilke agent-id'er der aktuelt er tilladt for `sessions_spawn`.

Use a cheaper model for sub-agents to save on token costs:

- Sub-agent-sessioner arkiveres automatisk efter `agents.defaults.subagents.archiveAfterMinutes` (standard: 60).
- Arkivering bruger `sessions.delete` og omdøber transskriptionen til `*.deleted.<timestamp>` (samme mappe).
- `cleanup: "delete"` arkiverer straks efter announce (beholder stadig transskriptionen via omdøbning).
- Auto-arkivering er best-effort; ventende timere går tabt, hvis gatewayen genstartes.
- `runTimeoutSeconds` medfører **ikke** automatisk arkivering; den stopper kun kørslen. Sessionen forbliver, indtil auto-arkivering sker.
- Auto-arkivering gælder både for depth-1- og depth-2-sessioner.

## Setting a Default Thinking Level

Som standard kan sub-agenter ikke starte deres egne sub-agenter (`maxSpawnDepth: 1`). Du kan aktivere ét niveau af nesting ved at sætte `maxSpawnDepth: 2`, hvilket tillader **orchestrator pattern**: main → orchestrator sub-agent → worker sub-sub-agenter.

### Per-Agent Overrides

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxSpawnDepth: 2, // tillad sub-agenter at starte child-agenter (standard: 1)
        maxChildrenPerAgent: 5, // maks. aktive child-agenter pr. agentsession (standard: 5)
        maxConcurrent: 8, // global grænse for samtidige kørsler (standard: 8)
      },
    },
  },
}
```

### Depth-niveauer

| Depth | Sessionsnøgle-format                         | Rolle                                                              | Kan starte nye?               |
| ----- | -------------------------------------------- | ------------------------------------------------------------------ | ----------------------------- |
| 0     | `agent:&lt;id&gt;:main`                            | Hovedagent                                                         | Altid                         |
| 1     | `agent:&lt;id&gt;:subagent:<uuid>`                 | Sub-agent (orchestrator når depth 2 er tilladt) | Kun hvis `maxSpawnDepth >= 2` |
| 2     | `agent:&lt;id&gt;:subagent:<uuid>:subagent:<uuid>` | Sub-sub-agent (leaf worker)                     | Aldrig                        |

### Announce-kæde

Resultater flyder tilbage op gennem kæden:

1. Depth-2 worker afslutter → annoncerer til sin parent (depth-1 orchestrator)
2. Depth-1 orchestrator modtager announcen, sammenfatter resultaterne, afslutter → annoncerer til main
3. Main-agenten modtager announcen og leverer til brugeren

Hvert niveau ser kun annonceringer fra sine direkte børn.

### Værktøjspolitik efter dybde

- **Dybde 1 (orchestrator, når `maxSpawnDepth >= 2`)**: Får `sessions_spawn`, `subagents`, `sessions_list`, `sessions_history`, så den kan administrere sine børn. Andre session-/systemværktøjer forbliver afvist.
- **Dybde 1 (leaf, når `maxSpawnDepth == 1`)**: Ingen sessionsværktøjer (nuværende standardadfærd).
- **Dybde 2 (leaf worker)**: Ingen sessionsværktøjer — `sessions_spawn` afvises altid på dybde 2. Kan ikke oprette yderligere børn.

### Grænse for oprettelse pr. agent

Hver agentsession (på ethvert niveau) kan højst have `maxChildrenPerAgent` (standard: 5) aktive børn ad gangen. Dette forhindrer ukontrolleret fan-out fra en enkelt orchestrator.

### Kaskadestop

Stop af en dybde-1 orchestrator stopper automatisk alle dens dybde-2 børn:

- `/stop` i hovedchatten stopper alle dybde-1 agenter og kaskaderer til deres dybde-2 børn.
- `/subagents kill <id>` stopper en specifik sub-agent og kaskaderer til dens børn.
- `/subagents kill all` stopper alle sub-agenter for anmoderen og kaskaderer.

## Model Resolution Order

The sub-agent model is resolved in this order (first match wins):

- Explicit `model` parameter in the `sessions_spawn` call
- Per-agent config: `agents.list[].subagents.model`
- Global default: `agents.defaults.subagents.model`

Bemærk: sammenfletningen er additiv, så hovedprofiler er altid tilgængelige som fallback. Fuldstændigt isoleret godkendelse pr. agent understøttes endnu ikke.

## Annoncering

Sub-agenter rapporterer tilbage via et annonceringstrin:

- Annonceringstrinnet kører inde i sub-agentens session (ikke i anmoderens session).
- Hvis sub-agenten svarer præcist `ANNOUNCE_SKIP`, bliver der ikke sendt noget.
- Ellers sendes annonceringssvaret til anmoderens chatkanal via et opfølgende `agent`-kald (`deliver=true`).
- Announce-svar bevarer tråd-/emnerouting, når det er tilgængeligt (Slack-tråde, Telegram-emner, Matrix-tråde).
- Annonceringsmeddelelser normaliseres til en stabil skabelon:
  - `Status:` afledt af kørslens resultat (`success`, `error`, `timeout` eller `unknown`).
  - `Result:` resuméindholdet fra annonceringstrinnet (eller `(not available)` hvis det mangler).
  - `Notes:` fejldetaljer og anden nyttig kontekst.
- `Status` udledes ikke fra modeloutput; det kommer fra runtime-resultatsignaler.

Annonceringspayloads inkluderer en statistiklinje til sidst (selv når de er indkapslet):

- Runtime (f.eks. `runtime 5m12s`)
- Tokenforbrug (input/output/i alt)
- Estimeret omkostning når modelprissætning er konfigureret (`models.providers.*.models[].cost`)
- `sessionKey`, `sessionId` og sti til transskription (så hovedagenten kan hente historik via `sessions_history` eller inspicere filen på disken)

## Værktøjspolitik (sub-agent-værktøjer)

Som standard får sub-agenter **alle værktøjer undtagen sessionsværktøjer** og systemværktøjer:

- `sessions_list`
- `sessions_history`
- `sessions_send`
- `sessions_spawn`

Når `maxSpawnDepth >= 2`, modtager dybde-1 orchestrator-sub-agenter derudover `sessions_spawn`, `subagents`, `sessions_list` og `sessions_history`, så de kan administrere deres børn.

You can reference sub-agents by list index (`1`, `2`), run id prefix, full session key, or `last`.

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

Sub-agenter bruger en dedikeret in-process kø-lane:

- Lane-navn: `subagent`
- Samtidighed: `agents.defaults.subagents.maxConcurrent` (standard `8`)

## Stopning

- At sende `/stop` i requester-chatten afbryder requester-sessionen og stopper alle aktive sub-agent-kørsler, der er startet fra den, og kaskaderer til indlejrede underordnede.
- `/subagents kill <id>` stopper en specifik sub-agent og kaskaderer til dens underordnede.

## Begrænsninger

- Sub-agent-annoncering er **best-effort**. Hvis gatewayen genstarter, går afventende "announce back"-arbejde tabt.
- Sub-agents deler stadig de samme gateway-procesressourcer; behandl `maxConcurrent` som en sikkerhedsventil.
- `sessions_spawn` er altid ikke-blokerende: den returnerer `{ status: "accepted", runId, childSessionKey }` med det samme.
- Sub-agent-kontekst injicerer kun `AGENTS.md` + `TOOLS.md` (ingen `SOUL.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md` eller `BOOTSTRAP.md`).
- Maksimal indlejringsdybde er 5 (`maxSpawnDepth` interval: 1–5). Dybde 2 anbefales til de fleste anvendelsestilfælde.
- `maxChildrenPerAgent` begrænser aktive underordnede pr. session (standard: 5, interval: 1–20).

