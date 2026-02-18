---
title: "OpenProse"
---

# OpenProse

OpenProse — bu AI sessiyalarini boshqarish uchun mo‘ljallangan, ko‘chma va markdown-ga asoslangan ish jarayoni formatidir. OpenClaw ichida u OpenProse skill to‘plamini hamda `/prose` slash buyrug‘ini o‘rnatadigan plagin sifatida yetkaziladi. Dasturlar `.prose` fayllarida joylashadi va aniq boshqaruv oqimi bilan bir nechta sub-agentlarni ishga tushirishi mumkin.

Rasmiy sayt: [https://www.prose.md](https://www.prose.md)

## Nimalar qila oladi

- Aniq parallelizm bilan ko‘p agentli tadqiqot + sintez.
- Takrorlanadigan, tasdiqlashga xavfsiz ish jarayonlari (kod ko‘rib chiqish, hodisalarni saralash, kontent pipeline’lari).
- Qo‘llab-quvvatlanadigan agent muhitlarida ishga tushiriladigan qayta foydalaniladigan `.prose` dasturlari.

## O‘rnatish + yoqish

Paketga kiritilgan plaginlar sukut bo‘yicha o‘chiq bo‘ladi. OpenProse’ni yoqing:

```bash
openclaw plugins enable open-prose
```

Plaginni yoqqandan so‘ng Gateway’ni qayta ishga tushiring.

Dev/local ishchi nusxa uchun: `openclaw plugins install ./extensions/open-prose`

Tegishli hujjatlar: [Plugins](/tools/plugin), [Plugin manifest](/plugins/manifest), [Skills](/tools/skills).

## Slash buyrug‘i

OpenProse foydalanuvchi tomonidan chaqiriladigan skill buyrug‘i sifatida `/prose` ni ro‘yxatdan o‘tkazadi. U OpenProse VM ko‘rsatmalariga yo‘naltiradi va ichki jarayonda OpenClaw vositalaridan foydalanadi.

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

`/prose run <handle/slug>` manzili `https://p.prose.md/<handle>/<slug>` ga yechiladi.
To‘g‘ridan-to‘g‘ri URL’lar o‘zgartirilmasdan yuklab olinadi. Bunda `web_fetch` vositasi (yoki POST uchun `exec`) ishlatiladi.

## OpenClaw runtime moslashuvi

OpenProse dasturlari OpenClaw primitivlariga moslashtiriladi:

| OpenProse tushunchasi     | OpenClaw vositasi |
| ------------------------- | ----------------- |
| Sessiyani ishga tushirish / Task vositasi | `sessions_spawn` |
| Fayl o‘qish/yozish        | `read` / `write` |
| Vebdan yuklash            | `web_fetch`      |

Agar vositalar allowlist’i ushbu vositalarni bloklasa, OpenProse dasturlari ishlamaydi. Qarang: [Skills config](/tools/skills-config).

## Xavfsizlik + tasdiqlashlar

`.prose` fayllarini kod kabi qabul qiling. Ishga tushirishdan oldin ko‘rib chiqing. Yon ta’sirlarni boshqarish uchun OpenClaw vosita allowlist’lari va tasdiqlash bosqichlaridan foydalaning.

Deterministik va tasdiqlash bosqichlari bilan boshqariladigan ish jarayonlari uchun [Lobster](/tools/lobster) bilan taqqoslang.

