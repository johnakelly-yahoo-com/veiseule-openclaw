---
summary: "Kung ano ang nilalaman ng OpenClaw system prompt at kung paano ito binubuo"
read_when:
  - Pag-eedit ng text ng system prompt, listahan ng tools, o mga seksyon ng oras/heartbeat
  - Pagbabago ng workspace bootstrap o behavior ng skills injection
title: "Prompt ng Sistema"
---

# Prompt ng Sistema

Bumubuo ang OpenClaw ng isang custom system prompt para sa bawat agent run. Ang prompt ay **pag-aari ng OpenClaw** at hindi gumagamit ng default prompt ng pi-coding-agent.

Ang prompt ay binubuo ng OpenClaw at ini-inject sa bawat agent run.

## Estruktura

Ang prompt ay sinadyang compact at gumagamit ng mga fixed na seksyon:

- **Tooling**: kasalukuyang listahan ng tools + maiikling paglalarawan.
- **Safety**: maikling paalala ng guardrail para iwasan ang power-seeking behavior o pag-bypass ng oversight.
- **Skills** (kapag available): nagsasabi sa model kung paano i-load ang mga instruksyon ng skill on demand.
- **OpenClaw Self-Update**: kung paano patakbuhin ang `config.apply` at `update.run`.
- **Workspace**: gumaganang direktoryo (`agents.defaults.workspace`).
- **Documentation**: lokal na path papunta sa OpenClaw docs (repo o npm package) at kung kailan ito babasahin.
- **Workspace Files (injected)**: nagpapahiwatig na ang mga bootstrap file ay kasama sa ibaba.
- **Sandbox** (kapag naka-enable): nagpapahiwatig ng sandboxed runtime, mga sandbox path, at kung available ang elevated exec.
- **Current Date & Time**: oras na lokal sa user, timezone, at time format.
- **Reply Tags**: opsyonal na reply tag syntax para sa mga suportadong provider.
- **Heartbeats**: heartbeat prompt at ack behavior.
- **Runtime**: host, OS, node, model, repo root (kapag na-detect), thinking level (isang linya).
- **Reasoning**: kasalukuyang antas ng visibility + hint para sa /reasoning toggle.

Safety guardrails in the system prompt are advisory. They guide model behavior but do not enforce policy. Use tool policy, exec approvals, sandboxing, and channel allowlists for hard enforcement; operators can disable these by design.

## Mga mode ng Prompt

OpenClaw can render smaller system prompts for sub-agents. The runtime sets a
`promptMode` for each run (not a user-facing config):

- `full` (default): kasama ang lahat ng seksyon sa itaas.
- `minimal`: used for sub-agents; omits **Skills**, **Memory Recall**, **OpenClaw
  Self-Update**, **Model Aliases**, **User Identity**, **Reply Tags**,
  **Messaging**, **Silent Replies**, and **Heartbeats**. Tooling, **Safety**,
  Workspace, Sandbox, Current Date & Time (when known), Runtime, and injected
  context stay available.
- `none`: ibinabalik lamang ang base identity line.

Kapag `promptMode=minimal`, ang mga extra na injected prompt ay nilalabel bilang **Subagent
Context** sa halip na **Group Chat Context**.

## Pag-inject ng bootstrap sa Workspace

Ang mga bootstrap file ay tine-trim at idinadagdag sa ilalim ng **Project Context** para makita ng model ang identity at profile context nang hindi na nangangailangan ng tahasang pagbasa:

- `AGENTS.md`
- `SOUL.md`
- `TOOLS.md`
- `IDENTITY.md`
- `USER.md`
- `HEARTBEAT.md`
- `BOOTSTRAP.md` (para lamang sa mga bagong-bagong workspace)
- `MEMORY.md` at/o `memory.md` (kapag naroroon sa workspace; alinman o pareho ay maaaring i-inject)

Lahat ng mga file na ito ay **ini-inject sa context window** sa bawat turn, na
nangangahulugang kumokonsumo sila ng mga token. Panatilihing maikli ang mga ito — lalo na ang `MEMORY.md`, na maaaring
lumaki sa paglipas ng panahon at magdulot ng hindi inaasahang mataas na paggamit ng context at mas madalas na
compaction.

> **Tandaan:** Ang mga `memory/*.md` na pang-araw-araw na file ay **hindi** awtomatikong ini-inject. Ina-access ang mga ito kapag kailangan sa pamamagitan ng mga tool na `memory_search` at `memory_get`, kaya hindi sila nababawas sa context window maliban kung tahasang babasahin sila ng model.

Large files are truncated with a marker. The max per-file size is controlled by
`agents.defaults.bootstrapMaxChars` (default: 20000). Ang kabuuang na-inject na bootstrap content sa lahat ng file ay may limitasyon na itinakda ng `agents.defaults.bootstrapTotalMaxChars` (default: 24000). Ang mga nawawalang file ay nag-iinject ng maikling marker na nagsasaad na nawawala ang file.

Ang mga sub-agent session ay nag-iinject lamang ng `AGENTS.md` at `TOOLS.md` (ang ibang bootstrap file ay sini-filter upang mapanatiling maliit ang context ng sub-agent).

Maaaring saluhin ng mga internal hook ang hakbang na ito sa pamamagitan ng `agent:bootstrap` para baguhin o palitan
ang mga injected na bootstrap file (halimbawa, pagpapalit ng `SOUL.md` ng alternatibong persona).

To inspect how much each injected file contributes (raw vs injected, truncation, plus tool schema overhead), use `/context list` or `/context detail`. See [Context](/concepts/context).

## Time handling

The system prompt includes a dedicated **Current Date & Time** section when the
user timezone is known. To keep the prompt cache-stable, it now only includes
the **time zone** (no dynamic clock or time format).

Tingnan ang [Date & Time](/date-time) para sa kumpletong detalye ng behavior.

I-configure gamit ang:

- `agents.defaults.userTimezone`
- `agents.defaults.timeFormat` (`auto` | `12` | `24`)

Tingnan ang [Date & Time](/date-time) para sa kumpletong detalye ng behavior.

## Skills

When eligible skills exist, OpenClaw injects a compact **available skills list**
(`formatSkillsForPrompt`) that includes the **file path** for each skill. The
prompt instructs the model to use `read` to load the SKILL.md at the listed
location (workspace, managed, or bundled). If no skills are eligible, the
Skills section is omitted.

```
<available_skills>
  <skill>
    <name>...</name>
    <description>...</description>
    <location>...</location>
  </skill>
</available_skills>
```

Pinananatiling maliit nito ang base prompt habang pinapagana pa rin ang target na paggamit ng skill.

## Documentation

When available, the system prompt includes a **Documentation** section that points to the
local OpenClaw docs directory (either `docs/` in the repo workspace or the bundled npm
package docs) and also notes the public mirror, source repo, community Discord, and
ClawHub ([https://clawhub.com](https://clawhub.com)) for skills discovery. The prompt instructs the model to consult local docs first
for OpenClaw behavior, commands, configuration, or architecture, and to run
`openclaw status` itself when possible (asking the user only when it lacks access).
