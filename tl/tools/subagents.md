---
title: "Mga Sub-Agent"
---

# Mga Sub-Agent

Ang mga sub-agent ay mga background agent run na inilulunsad mula sa isang umiiral na agent run. Tumatakbo sila sa sarili nilang session (`agent:<agentId>:subagent:<uuid>`) at kapag natapos, **inaanunsyo** nila ang kanilang resulta pabalik sa chat channel ng humiling.

## Slash command

Gamitin ang `/subagents` upang siyasatin o kontrolin ang mga sub-agent run para sa **kasalukuyang session**:

- `/subagents list`
- `/subagents kill <id|#|all>`
- `/subagents log <id|#> [limit] [tools]`
- `/subagents info <id|#>`
- `/subagents send <id|#> <message>`

Ipinapakita ng `/subagents info` ang metadata ng run (status, mga timestamp, session id, transcript path, cleanup).

Mga pangunahing layunin:

- I-parallelize ang mga gawaing “research / mahabang task / mabagal na tool” nang hindi hinaharangan ang pangunahing run.
- Panatilihing isolated ang mga sub-agent bilang default (hiwalay na session + opsyonal na sandboxing).
- Panatilihing mahirap maabuso ang tool surface: ang mga sub-agent ay **hindi** nakakakuha ng session tools bilang default.
- Suportahan ang configurable na nesting depth para sa mga orchestrator pattern.

Tala sa gastos: bawat sub-agent ay may **sarili nitong** context at paggamit ng token. Para sa mabibigat o paulit-ulit na gawain, magtakda ng mas murang modelo para sa mga sub-agent at panatilihin ang iyong pangunahing agent sa mas mataas na kalidad na modelo. Maaari mo itong i-configure sa pamamagitan ng `agents.defaults.subagents.model` o per-agent overrides.

## Tool

Gamitin ang `sessions_spawn`:

- Nagsisimula ng sub-agent run (`deliver: false`, global lane: `subagent`)
- Pagkatapos ay nagpapatakbo ng announce step at nagpo-post ng announce reply sa chat channel ng humiling
- Default model: minamana ang model ng tumatawag maliban kung magtatakda ka ng `agents.defaults.subagents.model` (o per-agent `agents.list[].subagents.model`); mas mananaig pa rin ang tahasang `sessions_spawn.model`.
- Default thinking: minamana ang thinking ng tumatawag maliban kung magtatakda ka ng `agents.defaults.subagents.thinking` (o per-agent `agents.list[].subagents.thinking`); mas mananaig pa rin ang tahasang `sessions_spawn.thinking`.

Mga parameter ng tool:

- `task` (kinakailangan)
- `label?` (opsyonal)
- `agentId?` (opsyonal; mag-spawn sa ilalim ng ibang agent id kung pinapayagan)
- `model?` (opsyonal; i-o-override ang model ng sub-agent; ang mga hindi wastong value ay lalaktawan at tatakbo ang sub-agent sa default model na may babala sa resulta ng tool)
- `thinking?` (opsyonal; i-o-override ang antas ng pag-iisip para sa sub-agent run)
- `runTimeoutSeconds?` (default `0`; kapag naka-set, ia-abort ang sub-agent run pagkalipas ng N segundo)
- `cleanup?` (`delete|keep`, default `keep`)

Allowlist:

- `agents.list[].subagents.allowAgents`: listahan ng mga agent id na maaaring i-target sa pamamagitan ng `agentId` (`["*"]` upang payagan ang alinman). Default: tanging ang agent na humihiling.

Discovery:

- Gamitin ang `agents_list` upang makita kung aling mga agent id ang kasalukuyang pinapayagan para sa `sessions_spawn`.

Auto-archive:

- Ang mga session ng sub-agent ay awtomatikong ina-archive pagkalipas ng `agents.defaults.subagents.archiveAfterMinutes` (default: 60).
- Ginagamit ng archive ang `sessions.delete` at pinapalitan ang pangalan ng transcript sa `*.deleted.<timestamp>` (parehong folder).
- `cleanup: "delete"` ay agad na nag-a-archive pagkatapos ng announce (pinananatili pa rin ang transcript sa pamamagitan ng rename).
- Ang auto-archive ay best-effort; nawawala ang mga nakabinbing timer kapag nag-restart ang gateway.
- Ang `runTimeoutSeconds` ay **hindi** nag-a-auto-archive; itinitigil lamang nito ang run. Mananatili ang session hanggang sa auto-archive.
- Parehong nalalapat ang auto-archive sa depth-1 at depth-2 na mga session.

## Nested Sub-Agents

Bilang default, hindi maaaring mag-spawn ang mga sub-agent ng sarili nilang sub-agent (`maxSpawnDepth: 1`). Maaari mong paganahin ang isang antas ng nesting sa pamamagitan ng pagtatakda ng `maxSpawnDepth: 2`, na nagpapahintulot sa **orchestrator pattern**: main → orchestrator sub-agent → worker sub-sub-agents.

### Paano Paganahin

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxSpawnDepth: 2, // payagan ang mga sub-agent na mag-spawn ng mga child (default: 1)
        maxChildrenPerAgent: 5, // max na aktibong child bawat agent session (default: 5)
        maxConcurrent: 8, // global concurrency lane cap (default: 8)
      },
    },
  },
}
```

### Mga Antas ng Depth

| Depth | Session key shape                            | Role                                          | Maaaring mag-spawn?            |
| ----- | -------------------------------------------- | --------------------------------------------- | ------------------------------ |
| 0     | `agent:<id>:main`                            | Pangunahing agent                             | Palagi                        |
| 1     | `agent:<id>:subagent:<uuid>`                 | Sub-agent (orchestrator kapag pinayagan ang depth 2) | Kung `maxSpawnDepth >= 2` |
| 2     | `agent:<id>:subagent:<uuid>:subagent:<uuid>` | Sub-sub-agent (leaf worker)                   | Hindi kailanman               |

### Announce chain

Umaakyat pabalik ang mga resulta sa chain:

1. Nagtatapos ang depth-2 worker → nag-a-announce sa parent nito (depth-1 orchestrator)
2. Tinatanggap ng depth-1 orchestrator ang announce, sini-synthesize ang mga resulta, nagtatapos → nag-a-announce sa main
3. Tinatanggap ng main agent ang announce at dinideliver sa user

Bawat antas ay nakakakita lamang ng mga announce mula sa direkta nitong mga child.

### Patakaran sa Tool ayon sa Depth

- **Depth 1 (orchestrator, kapag `maxSpawnDepth >= 2`)**: Nakakakuha ng `sessions_spawn`, `subagents`, `sessions_list`, `sessions_history` upang mapamahalaan ang mga child. Ang iba pang session/system tools ay nananatiling denied.
- **Depth 1 (leaf, kapag `maxSpawnDepth == 1`)**: Walang session tools (kasalukuyang default na behavior).
- **Depth 2 (leaf worker)**: Walang session tools — ang `sessions_spawn` ay laging denied sa depth 2. Hindi maaaring mag-spawn ng karagdagang child.

### Limitasyon ng Spawn bawat Agent

Bawat agent session (anumang depth) ay maaaring magkaroon ng hanggang `maxChildrenPerAgent` (default: 5) na aktibong child sa isang pagkakataon. Pinipigilan nito ang runaway fan-out mula sa iisang orchestrator.

### Cascade stop

Ang paghinto sa isang depth-1 orchestrator ay awtomatikong humihinto sa lahat ng depth-2 nitong mga child:

- Ang `/stop` sa main chat ay humihinto sa lahat ng depth-1 agent at nagca-cascade sa kanilang depth-2 na mga child.
- Ang `/subagents kill <id>` ay humihinto sa isang partikular na sub-agent at nagca-cascade sa mga child nito.
- Ang `/subagents kill all` ay humihinto sa lahat ng sub-agent para sa humihiling at nagca-cascade.

## Authentication

Ang auth ng sub-agent ay nireresolba ayon sa **agent id**, hindi ayon sa uri ng session:

- Ang sub-agent session key ay `agent:<agentId>:subagent:<uuid>`.
- Ang auth store ay nilo-load mula sa `agentDir` ng agent na iyon.
- Ang mga auth profile ng pangunahing agent ay isinasama bilang isang **fallback**; nangingibabaw ang mga profile ng agent kapag may conflict.

Tandaan: additive ang merge, kaya laging available ang mga pangunahing profile bilang fallback. Hindi pa suportado ang ganap na isolated auth bawat agent.

## Announce

Nag-uulat pabalik ang mga sub-agent sa pamamagitan ng announce step:

- Tumatakbo ang announce step sa loob ng session ng sub-agent (hindi sa session ng humihiling).
- Kung eksaktong `ANNOUNCE_SKIP` ang tugon ng sub-agent, walang ipo-post.
- Kung hindi, ipo-post ang announce reply sa chat channel ng humihiling sa pamamagitan ng follow-up na `agent` call (`deliver=true`).
- Pinapanatili ng mga announce reply ang thread/topic routing kapag available (Slack threads, Telegram topics, Matrix threads).
- Ang mga announce message ay ino-normalize sa isang stable na template:
  - `Status:` hinango mula sa kinalabasan ng run (`success`, `error`, `timeout`, o `unknown`).
  - `Result:` ang buod na nilalaman mula sa announce step (o `(not available)` kung wala).
  - `Notes:` mga detalye ng error at iba pang kapaki-pakinabang na konteksto.
- Ang `Status` ay hindi hinuhango mula sa output ng modelo; nanggagaling ito sa mga runtime outcome signal.

Kasama sa mga announce payload ang isang stats line sa dulo (kahit naka-wrap):

- Runtime (hal., `runtime 5m12s`)
- Paggamit ng token (input/output/total)
- Tinatayang gastos kapag naka-configure ang model pricing (`models.providers.*.models[].cost`)
- `sessionKey`, `sessionId`, at transcript path (upang ma-fetch ng main agent ang history sa pamamagitan ng `sessions_history` o inspeksyunin ang file sa disk)

## Tool Policy (mga tool ng sub-agent)

Bilang default, nakakakuha ang mga sub-agent ng **lahat ng tool maliban sa session tools** at system tools:

- `sessions_list`
- `sessions_history`
- `sessions_send`
- `sessions_spawn`

Kapag `maxSpawnDepth >= 2`, ang depth-1 orchestrator sub-agents ay dagdag na nakakakuha ng `sessions_spawn`, `subagents`, `sessions_list`, at `sessions_history` upang mapamahalaan ang kanilang mga child.

I-override sa pamamagitan ng config:

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

## Concurrency

Gumagamit ang mga sub-agent ng isang nakalaang in-process queue lane:

- Lane name: `subagent`
- Concurrency: `agents.defaults.subagents.maxConcurrent` (default `8`)

## Stopping

- Ang pagpapadala ng `/stop` sa chat ng humihiling ay nag-a-abort sa session ng humihiling at humihinto sa anumang aktibong sub-agent run na inilunsad mula rito, na nagca-cascade sa nested na mga child.
- Ang `/subagents kill <id>` ay humihinto sa isang partikular na sub-agent at nagca-cascade sa mga child nito.

## Limitations

- Ang sub-agent announce ay **best-effort**. Kapag nag-restart ang gateway, mawawala ang mga nakabinbing “announce back” na gawain.
- Ibinabahagi pa rin ng mga sub-agent ang parehong gateway process resources; ituring ang `maxConcurrent` bilang safety valve.
- Ang `sessions_spawn` ay laging non-blocking: agad itong nagbabalik ng `{ status: "accepted", runId, childSessionKey }`.
- Ang context ng sub-agent ay nag-i-inject lamang ng `AGENTS.md` + `TOOLS.md` (walang `SOUL.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, o `BOOTSTRAP.md`).
- Ang maximum nesting depth ay 5 (`maxSpawnDepth` range: 1–5). Inirerekomenda ang depth 2 para sa karamihan ng use case.
- Nililimitahan ng `maxChildrenPerAgent` ang aktibong mga child bawat session (default: 5, range: 1–20).