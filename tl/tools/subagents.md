---
summary: "Mga sub-agent: pag-spawn ng mga hiwalay na agent run na nag-aanunsyo ng mga resulta pabalik sa requester chat"
read_when:
  - Gusto mo ng background/parallel na trabaho gamit ang agent
  - Binabago mo ang sessions_spawn o patakaran ng sub-agent tool
title: "Mga Sub-Agent"
---

# Mga Sub-Agent

Ang mga sub-agent ay mga background agent run na inilulunsad mula sa isang umiiral na agent run. Tumatakbo sila sa sarili nilang session (`agent:<agentId>:subagent:<uuid>`) at, kapag natapos, **ina-anunsyo** ang kanilang resulta pabalik sa requester chat channel.

## Slash command

Gamitin ang `/subagents` upang suriin o kontrolin ang mga sub-agent run para sa **kasalukuyang session**:

- `/subagents list`
- `/subagents kill <id|#|all>`
- `/subagents log <id|#> [limit] [tools]`
- `/subagents info <id|#>`
- `/subagents send <id|#> <message>`

Ang pinakasimpleng paraan para gumamit ng mga sub-agent ay ang natural na paghingi sa iyong agent:

Mga pangunahing layunin:

- I-parallelize ang mga gawaing "research / long task / slow tool" nang hindi bina-block ang pangunahing run.
- Panatilihing hiwalay ang mga sub-agent bilang default (session separation + opsyonal na sandboxing).
- Panatilihing mahirap abusuhin ang tool surface: ang mga sub-agent ay **hindi** nakakakuha ng session tools bilang default.
- Suportahan ang nako-configure na nesting depth para sa mga orchestrator pattern.

Paalala sa gastos: bawat sub-agent ay may sarili nitong **konteksto** at paggamit ng token. Para sa mabibigat o paulit-ulit na
ga gawain, magtakda ng mas murang model para sa mga sub-agent at panatilihin ang iyong pangunahing agent sa mas mataas na kalidad na model.
Maaari mo itong i-configure sa pamamagitan ng `agents.defaults.subagents.model` o mga per-agent override.

## Tool

Gamitin ang `sessions_spawn`:

- Nagsisimula ng sub-agent run (`deliver: false`, global lane: `subagent`)
- Pagkatapos ay nagpapatakbo ng announce step at ipinapadala ang announce reply sa chat channel ng requester
- Default na model: minamana mula sa tumawag maliban kung itatakda mo ang `agents.defaults.subagents.model` (o per-agent na `agents.list[].subagents.model`); mananaig pa rin ang tahasang `sessions_spawn.model`.
- Default na thinking: minamana mula sa tumawag maliban kung itatakda mo ang `agents.defaults.subagents.thinking` (o per-agent na `agents.list[].subagents.thinking`); mananaig pa rin ang tahasang `sessions_spawn.thinking`.

Mga parameter ng tool:

- `task` (kinakailangan)
- `label?` (opsyonal)
- `agentId?` (opsyonal; mag-spawn sa ilalim ng ibang agent id kung pinapayagan)
- `model?` (opsyonal; ino-override ang model ng sub-agent; ang mga hindi valid na value ay lalaktawan at tatakbo ang sub-agent sa default na model na may babala sa tool result)
- `thinking?` (opsyonal; ino-override ang antas ng thinking para sa sub-agent run)
- `runTimeoutSeconds?` (default `0`; kapag itinakda, ihihinto ang sub-agent run pagkalipas ng N segundo)
- `cleanup?` (`delete|keep`, default `keep`)

Allowlist:

- `agents.list[].subagents.allowAgents`: listahan ng mga agent id na maaaring i-target sa pamamagitan ng `agentId` (`["*"]` para payagan ang alinman). Default: tanging ang requester agent lamang.

Discovery:

- Gamitin ang `agents_list` upang makita kung aling mga agent id ang kasalukuyang pinapayagan para sa `sessions_spawn`.

Auto-archive:

- Ang mga sub-agent session ay awtomatikong ina-archive pagkatapos ng `agents.defaults.subagents.archiveAfterMinutes` (default: 60).
- Ang pag-archive ay gumagamit ng `sessions.delete` at pinapalitan ang pangalan ng transcript sa `*.deleted.<timestamp>` (parehong folder).
- `cleanup: "delete"` ay nag-a-archive kaagad pagkatapos ng announce (pinananatili pa rin ang transcript sa pamamagitan ng pagpapalit ng pangalan).
- Ang auto-archive ay best-effort; ang mga nakabinbing timer ay mawawala kapag nag-restart ang gateway.
- Ang `runTimeoutSeconds` ay **hindi** nag-a-auto-archive; itinitigil lamang nito ang run. Mananatili ang session hanggang sa auto-archive.
- Ang auto-archive ay pantay na nalalapat sa depth-1 at depth-2 na mga session.

## Nested Sub-Agents

Bilang default, ang mga sub-agent ay hindi maaaring mag-spawn ng sarili nilang sub-agent (`maxSpawnDepth: 1`). Maaari mong paganahin ang isang antas ng nesting sa pamamagitan ng pagtatakda ng `maxSpawnDepth: 2`, na nagpapahintulot sa **orchestrator pattern**: main → orchestrator sub-agent → worker sub-sub-agents.

### Paano paganahin

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

### Mga antas ng depth

| Depth | Hugis ng session key                         | Tungkulin                                                                 | Maaaring mag-spawn?               |
| ----- | -------------------------------------------- | ------------------------------------------------------------------------- | --------------------------------- |
| 0     | `agent:&lt;id&gt;:main`                            | Main agent                                                                | Palagi                            |
| 1     | `agent:&lt;id&gt;:subagent:<uuid>`                 | Sub-agent (orchestrator kapag pinapayagan ang depth 2) | Tanging kung `maxSpawnDepth >= 2` |
| 2     | `agent:&lt;id&gt;:subagent:<uuid>:subagent:<uuid>` | Sub-sub-agent (leaf worker)                            | Hindi kailanman                   |

### I-anunsyo ang chain

Gumagamit ang mga sub-agent ng isang nakalaang queue lane (`subagent`) na hiwalay sa pangunahing agent queue, kaya hindi hinaharangan ng mga run ng sub-agent ang mga papasok na sagot.

1. Tinatapos ng depth-2 worker → nag-a-anunsyo sa parent nito (depth-1 orchestrator)
2. Tinatanggap ng depth-1 orchestrator ang anunsyo, pinagsasama ang mga resulta, tinatapos → nag-a-anunsyo sa main
3. Tinatanggap ng main agent ang anunsyo at inihahatid ito sa user

Ang mga session ng sub-agent ay awtomatikong ina-archive pagkatapos ng panahong maaaring i-configure:

### Patakaran sa tool ayon sa depth

- **Depth 1 (orchestrator, kapag `maxSpawnDepth >= 2`)**: Nakakatanggap ng `sessions_spawn`, `subagents`, `sessions_list`, `sessions_history` upang mapamahalaan nito ang mga anak. Ang ibang session/system tools ay nananatiling hindi pinapayagan.
- **Depth 1 (leaf, kapag `maxSpawnDepth == 1`)**: Walang session tools (kasalukuyang default na asal).
- **Depth 2 (leaf worker)**: Walang session tools — ang `sessions_spawn` ay laging hindi pinapayagan sa depth 2. Hindi maaaring mag-spawn ng karagdagang mga anak.

### Ang `sessions_spawn` Tool

Ang bawat agent session (sa anumang depth) ay maaaring magkaroon ng hanggang `maxChildrenPerAgent` (default: 5) aktibong mga anak sa isang oras. Pinipigilan nito ang hindi makontrol na pagdami mula sa iisang orchestrator.

### Parameters

Ang paghinto sa isang depth-1 orchestrator ay awtomatikong hihinto sa lahat ng depth-2 nitong mga anak:

- Ang `/stop` sa main chat ay hihinto sa lahat ng depth-1 agents at magka-cascade sa kanilang mga depth-2 na anak.
- Ang `/subagents kill <id>` ay hihinto sa isang partikular na sub-agent at magka-cascade sa mga anak nito.
- Ang `/subagents kill all` ay hihinto sa lahat ng sub-agents para sa requester at magka-cascade.

## Authentication

Ang auth ng sub-agent ay nireresolba ayon sa **agent id**, hindi ayon sa uri ng session:

- Ang sub-agent session key ay `agent:<agentId>:subagent:<uuid>`.
- Ang auth store ay nilo-load mula sa `agentDir` ng agent na iyon.
- Ang auth profiles ng main agent ay pinagsasama bilang **fallback**; ang mga profile ng agent ang mananaig kapag may conflict.

Tandaan: additive ang merge, kaya laging available ang main profiles bilang mga fallback. Hindi pa suportado ang ganap na hiwalay na auth para sa bawat agent.

## Anunsyo

Nag-uulat pabalik ang mga sub-agent sa pamamagitan ng hakbang ng anunsyo:

- Ang hakbang ng anunsyo ay tumatakbo sa loob ng sub-agent session (hindi sa requester session).
- Kung ang sub-agent ay eksaktong tumugon ng `ANNOUNCE_SKIP`, walang ipo-post.
- Kung hindi, ang tugon ng anunsyo ay ipo-post sa requester chat channel sa pamamagitan ng follow-up na `agent` call (`deliver=true`).
- Pinapanatili ng mga announce reply ang thread/topic routing kapag available (Slack threads, Telegram topics, Matrix threads).
- Ang mga mensahe ng anunsyo ay ginagawang pare-pareho ayon sa isang matatag na template:
  - `Status:` hinango mula sa kinalabasan ng run (`success`, `error`, `timeout`, o `unknown`).
  - `Result:` ang buod na nilalaman mula sa hakbang ng anunsyo (o `(not available)` kung wala).
  - `Notes:` mga detalye ng error at iba pang kapaki-pakinabang na konteksto.
- Ang `Status` ay hindi hinuhula mula sa output ng model; ito ay nagmumula sa mga runtime outcome signal.

Ang mga payload ng anunsyo ay may kasamang stats line sa dulo (kahit naka-wrap):

- Runtime (hal., `runtime 5m12s`)
- Paggamit ng token (input/output/kabuuan)
- Tinatayang gastos kapag naka-configure ang pagpepresyo ng model (`models.providers.*.models[].cost`)
- `sessionKey`, `sessionId`, at transcript path (para makuha ng main agent ang history sa pamamagitan ng `sessions_history` o suriin ang file sa disk)

## Patakaran sa Tool (mga tool ng sub-agent)

Bilang default, ang mga sub-agent ay nakakakuha ng **lahat ng tool maliban sa session tools** at system tools:

- `sessions_list`
- `sessions_history`
- `sessions_send`
- `sessions_spawn`

Kapag `maxSpawnDepth >= 2`, ang mga depth-1 orchestrator sub-agent ay tumatanggap din ng `sessions_spawn`, `subagents`, `sessions_list`, at `sessions_history` upang mapamahalaan nila ang kanilang mga anak.

I-override sa pamamagitan ng config:

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

## Concurrency

Ang mga sub-agent ay gumagamit ng hiwalay na in-process queue lane:

- Pangalan ng lane: `subagent`
- Concurrency: `agents.defaults.subagents.maxConcurrent` (default `8`)

## Paghinto

- Ang pagpapadala ng `/stop` sa requester chat ay mag-aabort sa requester session at hihinto sa anumang aktibong sub-agent runs na inilunsad mula rito, kabilang ang mga nested na anak.
- Ang `/subagents kill <id>` ay hihinto sa isang partikular na sub-agent at pati na rin sa mga anak nito.

## Mga Limitasyon

- Ang pag-aanunsyo ng sub-agent ay **best-effort**. Kapag nag-restart ang gateway, mawawala ang mga nakabinbing "announce back" na gawain.
- Ang mga sub-agent ay gumagamit pa rin ng parehong resources ng gateway process; ituring ang `maxConcurrent` bilang safety valve.
- Ang `sessions_spawn` ay palaging non-blocking: agad nitong ibinabalik ang `{ status: "accepted", runId, childSessionKey }`.
- Ang context ng sub-agent ay nag-i-inject lamang ng `AGENTS.md` + `TOOLS.md` (walang `SOUL.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, o `BOOTSTRAP.md`).
- Ang pinakamataas na nesting depth ay 5 (`maxSpawnDepth` range: 1–5). Inirerekomenda ang depth 2 para sa karamihan ng mga paggamit.
- Nililimitahan ng `maxChildrenPerAgent` ang mga aktibong anak bawat session (default: 5, range: 1–20).
