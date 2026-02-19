---
summary: "OpenProse: OpenClaw ichida .prose ish jarayonlari, slash buyruqlar va holat"
read_when:
  - Siz .prose ish jarayonlarini ishga tushirmoqchi yoki yozmoqchisiz
  - Siz OpenProse plaginini yoqmoqchisiz
  - Sizga holat saqlash mexanizmini tushunish kerak
title: "OpenProse"
---

# OpenProse

OpenProse is a portable, markdown-first workflow format for orchestrating AI sessions. In OpenClaw it ships as a plugin that installs an OpenProse skill pack plus a `/prose` slash command. Programs live in `.prose` files and can spawn multiple sub-agents with explicit control flow.

Rasmiy sayt: [https://www.prose.md](https://www.prose.md)

## Nimalar qila oladi

- Aniq parallelizm bilan ko‘p agentli tadqiqot + sintez.
- Takrorlanadigan, tasdiqlashga xavfsiz ish jarayonlari (kod ko‘rib chiqish, hodisalarni saralash, kontent pipeline’lari).
- Qo‘llab-quvvatlanadigan agent muhitlarida ishga tushiriladigan qayta foydalaniladigan `.prose` dasturlari.

## O‘rnatish + yoqish

Bundled plugins are disabled by default. Enable OpenProse:

```bash
openclaw plugins enable open-prose
```

Plaginni yoqqandan so‘ng Gateway’ni qayta ishga tushiring.

Dev/local ishchi nusxa uchun: `openclaw plugins install ./extensions/open-prose`

Tegishli hujjatlar: [Plugins](/tools/plugin), [Plugin manifest](/plugins/manifest), [Skills](/tools/skills).

## Slash buyrug‘i

OpenProse registers `/prose` as a user-invocable skill command. It routes to the OpenProse VM instructions and uses OpenClaw tools under the hood.

Keng tarqalgan buyruqlar:

```
/prose help
/prose run <file.prose>
/prose run <handle/slug>
/prose run <https://example.com/file.prose>
/prose compile <file.prose>
/prose examples
/prose update
```

## Misol: oddiy `.prose` fayli

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

## Fayl joylashuvlari

OpenProse ish maydoningizda holatni `.prose/` papkasi ostida saqlaydi:

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

Foydalanuvchi darajasidagi doimiy agentlar quyidagi joyda saqlanadi:

```
~/.prose/agents/
```

## Holat rejimlari

OpenProse bir nechta holat backend’larini qo‘llab-quvvatlaydi:

- **filesystem** (sukut bo‘yicha): `.prose/runs/...`
- **in-context**: vaqtinchalik, kichik dasturlar uchun
- **sqlite** (eksperimental): `sqlite3` binary talab qilinadi
- **postgres** (eksperimental): `psql` va ulanish satri talab qilinadi

Eslatmalar:

- sqlite/postgres ixtiyoriy va eksperimental hisoblanadi.
- postgres hisob ma’lumotlari subagent loglariga uzatiladi; alohida va minimal huquqli DB’dan foydalaning.

## Masofaviy dasturlar

`/prose run <handle/slug>` `https://p.prose.md/<handle>/<slug>` manziliga yo‘naltiriladi.
To‘g‘ridan-to‘g‘ri URL’lar qanday bo‘lsa, shundayicha olinadi. Bu `web_fetch` vositasidan foydalanadi (yoki POST uchun `exec`).

## OpenClaw runtime moslashuvi

OpenProse dasturlari OpenClaw primitivlariga moslashtiriladi:

| OpenProse tushunchasi                     | OpenClaw vositasi |
| ----------------------------------------- | ----------------- |
| Sessiyani ishga tushirish / Task vositasi | `sessions_spawn`  |
| Fayl o‘qish/yozish                        | `read` / `write`  |
| Vebdan yuklash                            | `web_fetch`       |

Agar vositalar allowlist’ingiz bu vositalarni bloklasa, OpenProse dasturlari ishlamaydi. [Skills config](/tools/skills-config) sahifasiga qarang.

## Xavfsizlik + tasdiqlashlar

`.prose` fayllarini kod kabi ko‘ring. Ishga tushirishdan oldin ko‘rib chiqing. Yon ta’sirlarni boshqarish uchun OpenClaw vositalari allowlist’lari va tasdiqlash darvozalaridan foydalaning.

Deterministik va tasdiqlash bosqichlari bilan boshqariladigan ish jarayonlari uchun [Lobster](/tools/lobster) bilan taqqoslang.

