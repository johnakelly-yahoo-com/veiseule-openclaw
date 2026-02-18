---
title: "OpenProse"
---

# OpenProse

Ang OpenProse ay isang portable, markdown-first na format ng workflow para sa pag-oorkestra ng mga AI session. Sa OpenClaw, ito ay ipinapadala bilang isang plugin na nag-i-install ng OpenProse skill pack kasama ang isang `/prose` slash command. Ang mga programa ay nasa `.prose` files at maaaring mag-spawn ng maraming sub-agent na may malinaw na control flow.

Opisyal na site: [https://www.prose.md](https://www.prose.md)

## Ano ang kaya nitong gawin

- Multi-agent na pananaliksik + synthesis na may malinaw na parallelism.
- Mga nauulit at approval-safe na workflow (code review, incident triage, content pipelines).
- Mga reusable na `.prose` program na maaari mong patakbuhin sa mga suportadong agent runtime.

## I-install + i-enable

Bundled plugins are disabled by default. Paganahin ang OpenProse:

```bash
openclaw plugins enable open-prose
```

I-restart ang Gateway pagkatapos i-enable ang plugin.

Dev/local checkout: `openclaw plugins install ./extensions/open-prose`

Kaugnay na docs: [Plugins](/tools/plugin), [Plugin manifest](/plugins/manifest), [Skills](/tools/skills).

## Slash command

OpenProse registers `/prose` as a user-invocable skill command. Ito ay niruruta sa mga instruction ng OpenProse VM at gumagamit ng mga OpenClaw tool sa likod ng mga eksena.

Mga karaniwang command:

```
/prose help
/prose run <file.prose>
/prose run <handle/slug>
/prose run <https://example.com/file.prose>
/prose compile <file.prose>
/prose examples
/prose update
```

## Halimbawa: isang simpleng `.prose` file

```prose
# Research + synthesis with two agents running in parallel.

input topic: "What should we research?"

agent researcher:
  model: sonnet
  prompt: "You research thoroughly and cite sources."

agent writer:
  model: opus
  prompt: "You write a concise summary."

parallel:
  findings = session: researcher
    prompt: "Research {topic}."
  draft = session: writer
    prompt: "Summarize {topic}."

session "Merge the findings + draft into a final answer."
context: { findings, draft }
```

## Mga lokasyon ng file

Pinapanatili ng OpenProse ang state sa ilalim ng `.prose/` sa iyong workspace:

```
.prose/
├── .env
├── runs/
│   └── {YYYYMMDD}-{HHMMSS}-{random}/
│       ├── program.prose
│       ├── state.md
│       ├── bindings/
│       └── agents/
└── agents/
```

Ang mga user-level na persistent agent ay matatagpuan sa:

```
~/.prose/agents/
```

## Mga mode ng state

Sinusuportahan ng OpenProse ang maraming state backend:

- **filesystem** (default): `.prose/runs/...`
- **in-context**: pansamantala, para sa maliliit na programa
- **sqlite** (experimental): nangangailangan ng `sqlite3` binary
- **postgres** (experimental): nangangailangan ng `psql` at isang connection string

Mga tala:

- Ang sqlite/postgres ay opt-in at experimental.
- Dumadaloy ang postgres credentials papunta sa mga log ng subagent; gumamit ng dedikado at least-privileged na DB.

## Mga remote na programa

`/prose run <handle/slug>` ay nireresolba sa `https://p.prose.md/<handle>/<slug>`.
Ang mga direktang URL ay kinukuha kung ano ang eksakto. Ginagamit nito ang `web_fetch` tool (o `exec` para sa POST).

## OpenClaw runtime mapping

Ang mga OpenProse program ay mina-map sa mga OpenClaw primitive:

| Konsepto ng OpenProse         | Tool ng OpenClaw    |
| ------------------------- | ---------------- |
| Mag-spawn ng session / Task tool | `sessions_spawn` |
| File read/write           | `read` / `write` |
| Web fetch                 | `web_fetch`      |

Kung hinaharangan ng iyong tool allowlist ang mga tool na ito, mabibigo ang mga programang OpenProse. Tingnan ang [Skills config](/tools/skills-config).

## Seguridad + mga pag-apruba

Tratuhin ang mga `.prose` file na parang code. Suriin bago patakbuhin. Use OpenClaw tool allowlists and approval gates to control side effects.

Para sa deterministiko at approval-gated na mga workflow, ihambing sa [Lobster](/tools/lobster).
