---
summary: "Models CLI: ro‘yxat, sozlash, aliaslar, fallbacklar, skanerlash, holat"
read_when:
  - Models CLI (models list/set/scan/aliases/fallbacks) qo‘shilayotganda yoki o‘zgartirilayotganda
  - Model fallback xatti‑harakati yoki tanlash UX o‘zgartirilayotganda
  - Model skanerlash probelari (tools/images) yangilanayotganda
title: "Modellar CLI"
---

# Modellar CLI

See [/concepts/model-failover](/concepts/model-failover) for auth profile
rotation, cooldowns, and how that interacts with fallbacks.
Quick provider overview + examples: [/concepts/model-providers](/concepts/model-providers).

## Model tanlash qanday ishlaydi

OpenClaw modelllarni quyidagi tartibda tanlaydi:

1. **Asosiy** model (`agents.defaults.model.primary` yoki `agents.defaults.model`).
2. `agents.defaults.model.fallbacks` ichidagi **fallback** modelllar (tartib bo‘yicha).
3. **Provayder auth failover** keyingi modelga o‘tishdan oldin provayder ichida amalga oshadi.

Bog‘liq:

- `agents.defaults.models` — OpenClaw foydalanishi mumkin bo‘lgan modelllar ro‘yxati/katalogi (va aliaslar).
- `agents.defaults.imageModel` — **faqat** asosiy model rasm qabul qila olmaganda ishlatiladi.
- Har bir agent uchun standart sozlamalar `agents.list[].model` orqali `agents.defaults.model` ni override qilishi mumkin (bindinglar bilan birga; qarang [/concepts/multi-agent](/concepts/multi-agent)).

## Tezkor model tanlovlari (subyektiv)

- **GLM**: kod yozish/tool chaqirish uchun biroz yaxshiroq.
- **MiniMax**: yozish uslubi va umumiy “vibe” uchun yaxshiroq.

## Setup wizard (tavsiya etiladi)

Agar konfiguratsiyani qo‘lda tahrirlashni istamasangiz, onboarding wizard’ni ishga tushiring:

```bash
openclaw onboard
```

U keng tarqalgan provayderlar uchun model + auth sozlashni bajaradi, jumladan **OpenAI Code (Codex)
subscription** (OAuth) va **Anthropic** (API key tavsiya etiladi; `claude
setup-token` ham qo‘llab‑quvvatlanadi).

## Config kalitlari (umumiy ko‘rinish)

- `agents.defaults.model.primary` va `agents.defaults.model.fallbacks`
- `agents.defaults.imageModel.primary` va `agents.defaults.imageModel.fallbacks`
- `agents.defaults.models` (allowlist + aliaslar + provayder parametrlari)
- `models.providers` (`models.json` ga yoziladigan maxsus provayderlar)

Model refs are normalized to lowercase. Provider aliases like `z.ai/*` normalize
to `zai/*`.

Provayder konfiguratsiyasi misollari (OpenCode Zen bilan birga) quyida:
[/gateway/configuration](/gateway/configuration#opencode-zen-multi-model-proxy).

## “Model is not allowed” (va nega javob to‘xtab qoladi)

If `agents.defaults.models` is set, it becomes the **allowlist** for `/model` and for
session overrides. When a user selects a model that isn’t in that allowlist,
OpenClaw returns:

```
Model "provider/model" is not allowed. Use /model to list available models.
```

This happens **before** a normal reply is generated, so the message can feel
like it “didn’t respond.” The fix is to either:

- Modelni `agents.defaults.models` ga qo‘shing, yoki
- Allowlist’ni tozalang (`agents.defaults.models` ni olib tashlang), yoki
- `/model list` dan model tanlang.

Allowlist konfiguratsiyasi misoli:

```json5
{
  agent: {
    model: { primary: "anthropic/claude-sonnet-4-5" },
    models: {
      "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
      "anthropic/claude-opus-4-6": { alias: "Opus" },
    },
  },
}
```

## Chat ichida modelni almashtirish (`/model`)

Sessiyani qayta ishga tushirmasdan joriy sessiya uchun modelni almashtirishingiz mumkin:

```
/model
/model list
/model 3
/model openai/gpt-5.2
/model status
```

Eslatmalar:

- `/model` (va `/model list`) — ixcham, raqamlangan tanlash ro‘yxati (model oilasi + mavjud provayderlar).
- `/model <#>` — shu ro‘yxatdan tanlaydi.
- `/model status` — batafsil ko‘rinish (auth nomzodlari va, sozlangan bo‘lsa, provayder endpoint `baseUrl` + `api` rejimi).
- Model refs are parsed by splitting on the **first** `/`. Use `provider/model` when typing `/model <ref>`.
- Agar model ID’ning o‘zida `/` bo‘lsa (OpenRouter uslubida), provayder prefiksini kiritish shart (masalan: `/model openrouter/moonshotai/kimi-k2`).
- Agar provayder ko‘rsatilmasa, OpenClaw kiritmani alias yoki **default provayder** uchun model sifatida talqin qiladi (faqat model ID’da `/` bo‘lmasa ishlaydi).

To‘liq buyruq xatti‑harakati/konfiguratsiyasi: [Slash commands](/tools/slash-commands).

## CLI buyruqlari

```bash
openclaw models list
openclaw models status
openclaw models set <provider/model>
openclaw models set-image <provider/model>

openclaw models aliases list
openclaw models aliases add <alias> <provider/model>
openclaw models aliases remove <alias>

openclaw models fallbacks list
openclaw models fallbacks add <provider/model>
openclaw models fallbacks remove <provider/model>
openclaw models fallbacks clear

openclaw models image-fallbacks list
openclaw models image-fallbacks add <provider/model>
openclaw models image-fallbacks remove <provider/model>
openclaw models image-fallbacks clear
```

`openclaw models` (subbuyruqsiz) — `models status` uchun qisqa yo‘l.

### `models list`

Shows configured models by default. Useful flags:

- `--all`: to‘liq katalog
- `--local`: faqat lokal provayderlar
- `--provider <name>`: provayder bo‘yicha filtrlash
- `--plain`: har qatorda bitta model
- `--json`: mashina o‘qiy oladigan chiqish

### `models status`

Sozlangan provayderlar uchun aniqlangan asosiy model, zaxira modellar, rasm modeli va autentifikatsiya sharhini ko‘rsatadi. Shuningdek, autentifikatsiya omborida topilgan profillar uchun OAuth amal qilish muddati holatini ko‘rsatadi
(standart bo‘yicha 24 soat ichida ogohlantiradi). `--plain` faqat aniqlangan
asosiy modelni chiqaradi.
OAuth holati har doim ko‘rsatiladi (va `--json` chiqishiga kiritiladi). Agar sozlangan
provayderda hisob ma’lumotlari bo‘lmasa, `models status` **Missing auth** bo‘limini chiqaradi.
JSON tarkibiga `auth.oauth` (ogohlantirish oynasi + profillar) va `auth.providers`
(har bir provayder bo‘yicha samarali autentifikatsiya) kiradi.
Avtomatlashtirish uchun `--check` dan foydalaning (yo‘qolgan/muddati o‘tgan bo‘lsa `1`, muddati yaqinlashayotgan bo‘lsa `2` bilan chiqadi).

Anthropic uchun afzal auth — Claude Code CLI setup-token (istalgan joyda ishga tushiring; kerak bo‘lsa gateway host’da joylashtiring):

```bash
claude setup-token
openclaw models status
```

## Skanerlash (OpenRouter bepul modellari)

`openclaw models scan` OpenRouter’ning **bepul model katalogi** ni tekshiradi va
ixtiyoriy ravishda modellarning tool va image qo‘llab‑quvvatlashini probelashi mumkin.

Asosiy flag’lar:

- `--no-probe`: live probe’larni o‘tkazib yuboradi (faqat metadata)
- `--min-params <b>`: minimal parametr hajmi (milliardlarda)
- `--max-age-days <days>`: eski modelllarni o‘tkazib yuboradi
- `--provider <name>`: provayder prefiksi bo‘yicha filtr
- `--max-candidates <n>`: fallback ro‘yxati hajmi
- `--set-default`: `agents.defaults.model.primary` ni birinchi tanlovga o‘rnatadi
- `--set-image`: `agents.defaults.imageModel.primary` ni birinchi image tanlovga o‘rnatadi

Tekshiruv uchun OpenRouter API kaliti talab qilinadi (auth profillaridan yoki
`OPENROUTER_API_KEY`). Kalitsiz, faqat nomzodlarni ro‘yxatlash uchun `--no-probe` dan foydalaning.

Skan natijalari quyidagilar bo‘yicha reytinglanadi:

1. Image qo‘llab‑quvvatlashi
2. Tool kechikishi (latency)
3. Kontekst hajmi
4. Parametrlar soni

Input

- OpenRouter `/models` ro‘yxati (filter `:free`)
- Auth profillardan yoki `OPENROUTER_API_KEY` dan OpenRouter API key talab qilinadi (qarang [/environment](/help/environment))
- Ixtiyoriy filtrlar: `--max-age-days`, `--min-params`, `--provider`, `--max-candidates`
- Probe boshqaruvi: `--timeout`, `--concurrency`

TTY da ishga tushirilganda, zaxiralarni interaktiv tarzda tanlashingiz mumkin. Interaktiv bo‘lmagan
rejimda, standartlarni qabul qilish uchun `--yes` ni bering.

## Models registry (`models.json`)

`models.providers` dagi maxsus provayderlar agent katalogi ostidagi `models.json` ga yoziladi
(standart `~/.openclaw/agents/<agentId>/models.json`). Bu fayl
`models.mode` `replace` ga o‘rnatilmaguncha standart bo‘yicha birlashtiriladi.

