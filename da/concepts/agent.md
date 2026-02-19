---
summary: "Agent-runtime (indlejret pi-mono), workspace-kontrakt og session-bootstrap"
read_when:
  - Ændring af agent-runtime, workspace-bootstrap eller sessionsadfærd
title: "Agent-runtime"
---

# Agent-runtime 🤖

OpenClaw kører en enkelt indlejret agent-runtime afledt af **pi-mono**.

## Workspace (påkrævet)

OpenClaw bruger én agent-workspace-mappe (`agents.defaults.workspace`) som agentens **eneste** arbejdsmappe (`cwd`) for værktøjer og kontekst.

Anbefalet: brug `openclaw setup` til at oprette `~/.openclaw/openclaw.json`, hvis den mangler, og initialisér workspace-filerne.

Fuld workspace-layout + backup-guide: [Agent workspace](/concepts/agent-workspace)

Hvis `agents.defaults.sandbox` er aktiveret, kan ikke-hovedsessioner tilsidesætte dette med
per-session workspaces under `agents.defaults.sandbox.workspaceRoot` (se
[Gateway-konfiguration](/gateway/configuration)).

## Bootstrap-filer (injekteret)

Inde i `agents.defaults.workspace` forventer OpenClaw disse brugeredigerbare filer:

- `AGENTS.md` — driftsinstruktioner + “hukommelse”
- `SOUL.md` — persona, grænser, tone
- `TOOLS.md` — brugervedligeholdte værktøjsnoter (f.eks. `imsg`, `sag`, konventioner)
- `BOOTSTRAP.md` — engangsritual ved første kørsel (slettes efter gennemførelse)
- `IDENTITY.md` — agentnavn/stemning/emoji
- `USER.md` — brugerprofil + foretrukken tiltaleform

Ved første tur i en ny session injicerer OpenClaw indholdet af disse filer direkte i agentens kontekst.

Blanke filer springes over. Store filer trimmes og afkortes med en markør, så beder dig forblive magert (læs filen for fuldt indhold).

Hvis en fil mangler, injicerer OpenClaw en enkelt “manglende fil”-markørlinje (og `openclaw setup` opretter en sikker standardskabelon).

`BOOTSTRAP.md` er kun oprettet til et \*\* helt nyt arbejdsområde \*\* (ingen andre bootstrap filer til stede). Hvis du sletter det efter at have afsluttet ritualet, bør det ikke genskabes ved senere genstarter.

For helt at deaktivere oprettelse af bootstrap-filer (for forudseedede workspaces), sæt:

```json5
{ agent: { skipBootstrap: true } }
```

## Indbyggede værktøjer

Kerneværktøjer (læse / eksekvere / redigere / skrive og relaterede systemværktøjer) er altid tilgængelige,
underlagt værktøjspolitik. `apply_patch` er valgfri og gated af
`tools.exec.applyPatch`. `TOOLS.md` gør **ikke** styrer hvilke værktøjer, der findes; det er
vejledning for, hvordan _you_ vil have dem brugt.

## Færdigheder

OpenClaw indlæser Skills fra tre placeringer (workspace vinder ved navnekonflikt):

- Bundlet (leveret med installationen)
- Administreret/lokal: `~/.openclaw/skills`
- Arbejdsområde: `<workspace>/skills`

Skills kan styres via config/env (se `skills` i [Gateway-konfiguration](/gateway/configuration)).

## pi-mono-integration

OpenClaw genbruger dele af pi-mono-kodebasen (modeller/værktøjer), men **sessionsstyring, discovery og værktøjsfortrådning ejes af OpenClaw**.

- Ingen pi-coding agent-runtime.
- Ingen `~/.pi/agent`- eller `<workspace>/.pi`-indstillinger konsulteres.

## Sessioner

Sessionsudskrifter gemmes som JSONL på:

- `~/.openclaw/agents/<agentId>/sessions/<SessionId>.jsonl`

Sessions-ID er stabil og valgt af OpenClaw.
Ældre Pi/Tau sessionsmapper er **ikke** læst.

## Styring under streaming

Når køtilstand er `styre`, injiceres indgående beskeder i den aktuelle kørsel.
Kø er tjekket **efter hvert værktøjs kald**; hvis en meddelelse i kø er til stede
resterende værktøjs opkald fra den aktuelle assisterende meddelelse springes over (fejlværktøj
resultater med "Sprunget over på grund af brugerbeskeden i køen. ), derefter køen bruger
beskeden injiceres før næste assistent respons.

Når køtilstand er 'opfølgning' eller 'collect', holdes indgående beskeder indtil
nuværende vending slutter. derefter en ny agent turn starter med de kø nyttelast. Se
[Queue](/concepts/queue) for tilstand + debounce/cap adfærd.

Blokstreaming sender afsluttede assistentblokke, så snart de er færdig; det er
**slukket som standard** (`agents.defaults.blockStreamingStandard: "slukket"`).
Indstil grænsen via `agents.defaults.blockStreamingBreak` (`text_end` vs `message_end`; standard text_end).
Styr blød blok chunking med `agents.defaults.blockStreamingChunk` (standard er
800–1200 tegn; foretrækker afsnit breaks, derefter newlines; sætninger sidste).
Coalesce streamed chunks with `agents.defaults.blockStreamingCoalesce` to reduce
single-line spam (idle-based merging before send). Ikke-Telegram kanaler kræver
eksplicit `*.blockStreaming: true` for at aktivere blok svar.
Overordnede værktøjs resuméer udsendes ved værktøjsstart (ingen debounce); Control UI
streams værktøjs output via agent begivenheder, når de er tilgængelige.
Flere detaljer: [Streaming + chunking](/concepts/streaming).

## Modelreferencer

Modelreferencer i config (for eksempel `agents.defaults.model` og `agents.defaults.models`) parses ved at splitte på den **første** `/`.

- Brug `provider/model` ved konfiguration af modeller.
- Hvis selve model-ID’et indeholder `/` (OpenRouter-stil), inkludér udbyderpræfikset (eksempel: `openrouter/moonshotai/kimi-k2`).
- Hvis du udelader udbyderen, behandler OpenClaw inputtet som et alias eller en model for **standardudbyderen** (virker kun, når der ikke er `/` i model-ID’et).

## Konfiguration (minimum)

Som minimum skal du sætte:

- `agents.defaults.workspace`
- `channels.whatsapp.allowFrom` (kraftigt anbefalet)

---

_Næste: [Group Chats](/channels/group-messages)_ 🦞
