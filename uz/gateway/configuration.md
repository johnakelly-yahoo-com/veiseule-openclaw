---
title: "Konfiguratsiya"
---

# Konfiguratsiya

OpenClaw ixtiyoriy <Tooltip tip="JSON5 izohlar va oxirgi vergullarni qo‘llab-quvvatlaydi">**JSON5**</Tooltip> konfiguratsiyasini `~/.openclaw/openclaw.json` faylidan o‘qiydi.

Agar fayl mavjud bo‘lmasa, OpenClaw xavfsiz standart sozlamalardan foydalanadi. Konfiguratsiya qo‘shishning odatiy sabablari:

- Kanallarni ulash va botga kim xabar yubora olishini boshqarish
- Modellar, vositalar, sandbox yoki avtomatlashtirishni (cron, hooks) sozlash
- Sessiyalar, media, tarmoq yoki UI sozlamalarini moslashtirish

Barcha mavjud maydonlar uchun [to‘liq ma’lumotnoma](/gateway/configuration-reference) ga qarang.

<Tip>
**Konfiguratsiyaga yangimisiz?** Interaktiv sozlash uchun `openclaw onboard` ni ishga tushiring yoki to‘liq nusxa‑ko‘chirishga tayyor konfiguratsiyalar uchun [Configuration Examples](/gateway/configuration-examples) qo‘llanmasini ko‘ring.
</Tip>

## Minimal konfiguratsiya

```json5
// ~/.openclaw/openclaw.json
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

## Konfiguratsiyani tahrirlash

<Tabs>
  <Tab title="Interaktiv ustoz (wizard)">
    ```bash
    openclaw onboard       # to‘liq sozlash ustasi
    openclaw configure     # konfiguratsiya ustasi
    ```
  </Tab>
  <Tab title="CLI (bir qatorli buyruqlar)">
    ```bash
    openclaw config get agents.defaults.workspace
    openclaw config set agents.defaults.heartbeat.every "2h"
    openclaw config unset tools.web.search.apiKey
    ```
  </Tab>
  <Tab title="Control UI">
    [http://127.0.0.1:18789](http://127.0.0.1:18789) manzilini oching va **Config** yorlig‘idan foydalaning.  
    Control UI konfiguratsiya sxemasi asosida forma chizadi va zarurat bo‘lsa **Raw JSON** muharririni taqdim etadi.
  </Tab>
  <Tab title="To‘g‘ridan-to‘g‘ri tahrirlash">
    `~/.openclaw/openclaw.json` faylini bevosita tahrirlang. Gateway faylni kuzatadi va o‘zgarishlarni avtomatik qo‘llaydi (qarang: [hot reload](#config-hot-reload)).
  </Tab>
</Tabs>

## Qat’iy tekshiruv (Strict validation)

<Warning>
OpenClaw faqat sxemaga to‘liq mos keladigan konfiguratsiyalarni qabul qiladi. Noma’lum kalitlar, noto‘g‘ri turlar yoki yaroqsiz qiymatlar bo‘lsa, Gateway **ishga tushmaydi**. Ildiz darajadagi yagona istisno — `$schema` (string), JSON Schema metama’lumotlari uchun.
</Warning>

Tekshiruv muvaffaqiyatsiz bo‘lsa:

- Gateway ishga tushmaydi
- Faqat diagnostika buyruqlari ishlaydi (`openclaw doctor`, `openclaw logs`, `openclaw health`, `openclaw status`)
- Muammolarni ko‘rish uchun `openclaw doctor` ni ishga tushiring
- Tuzatishlarni qo‘llash uchun `openclaw doctor --fix` (yoki `--yes`) ni ishga tushiring

## Tez-tez bajariladigan vazifalar

<AccordionGroup>
  <Accordion title="Kanal sozlash (WhatsApp, Telegram, Discord va boshqalar)">
    Har bir kanal o‘z konfiguratsiya bo‘limiga ega: `channels.<provider>`. Sozlash bosqichlari uchun mos sahifani ko‘ring:

    - [WhatsApp](/channels/whatsapp) — `channels.whatsapp`
    - [Telegram](/channels/telegram) — `channels.telegram`
    - [Discord](/channels/discord) — `channels.discord`
    - [Slack](/channels/slack) — `channels.slack`
    - [Signal](/channels/signal) — `channels.signal`
    - [iMessage](/channels/imessage) — `channels.imessage`
    - [Google Chat](/channels/googlechat) — `channels.googlechat`
    - [Mattermost](/channels/mattermost) — `channels.mattermost`
    - [MS Teams](/channels/msteams) — `channels.msteams`

    Barcha kanallar bir xil DM siyosat naqshini qo‘llaydi:

    ```json5
    {
      channels: {
        telegram: {
          enabled: true,
          botToken: "123:abc",
          dmPolicy: "pairing",   // pairing | allowlist | open | disabled
          allowFrom: ["tg:123"], // only for allowlist/open
        },
      },
    }
    ```

  </Accordion>

  <Accordion title="Modelni tanlash va sozlash">
    Asosiy model va ixtiyoriy zaxira variantlarni belgilang:

    ```json5
    {
      agents: {
        defaults: {
          model: {
            primary: "anthropic/claude-sonnet-4-5",
            fallbacks: ["openai/gpt-5.2"],
          },
          models: {
            "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
            "openai/gpt-5.2": { alias: "GPT" },
          },
        },
      },
    }
    ```

    - `agents.defaults.models` model katalogini belgilaydi va `/model` uchun allowlist vazifasini bajaradi.
    - Model havolalari `provider/model` formatida bo‘ladi (masalan, `anthropic/claude-opus-4-6`).
    - Chatda modelni almashtirish uchun [Models CLI](/concepts/models), zaxira xatti-harakati uchun [Model Failover](/concepts/model-failover) sahifalariga qarang.
    - Maxsus/self-hosted provayderlar uchun [Custom providers](/gateway/configuration-reference#custom-providers-and-base-urls) ga qarang.

  </Accordion>

  <Accordion title="Botga kim xabar yubora olishini boshqarish">
    DM kirishi kanal bo‘yicha `dmPolicy` orqali boshqariladi:

    - `"pairing"` (standart): noma’lum yuboruvchilarga bir martalik juftlash kodi yuboriladi
    - `"allowlist"`: faqat `allowFrom` dagilar (yoki juftlanganlar)
    - `"open"`: barcha kiruvchi DMlarga ruxsat (talab qiladi `allowFrom: ["*"]`)
    - `"disabled"`: barcha DMlar e’tiborsiz qoldiriladi

    Guruhlar uchun `groupPolicy` + `groupAllowFrom` yoki kanalga xos allowlistlardan foydalaning.

    Batafsil ma’lumot uchun [to‘liq ma’lumotnoma](/gateway/configuration-reference#dm-and-group-access) ga qarang.

  </Accordion>
</AccordionGroup>

## Config hot reload

Gateway `~/.openclaw/openclaw.json` faylini kuzatadi va o‘zgarishlarni avtomatik qo‘llaydi — ko‘p sozlamalar uchun qo‘lda qayta ishga tushirish talab qilinmaydi.

### Qayta yuklash rejimlari

| Rejim                  | Xatti-harakat                                                                          |
| ---------------------- | --------------------------------------------------------------------------------------- |
| **`hybrid`** (standart) | Xavfsiz o‘zgarishlarni darhol qo‘llaydi. Muhimlari uchun avtomatik qayta ishga tushadi. |
| **`hot`**              | Faqat hot‑xavfsiz o‘zgarishlarni qo‘llaydi. Qayta ishga tushirish kerak bo‘lsa ogohlantiradi. |
| **`restart`**          | Har qanday konfiguratsiya o‘zgarishida Gateway’ni qayta ishga tushiradi.               |
| **`off`**              | Fayl kuzatuvini o‘chiradi. O‘zgarishlar keyingi qo‘lda restartda kuchga kiradi.       |

```json5
{
  gateway: {
    reload: { mode: "hybrid", debounceMs: 300 },
  },
}
```

## Environment variables

OpenClaw muhit o‘zgaruvchilarini ota‑jarayondan hamda quyidagilardan o‘qiydi:

- Joriy katalogdagi `.env` (mavjud bo‘lsa)
- `~/.openclaw/.env` (global fallback)

Hech biri mavjud env o‘zgaruvchilarini ustiga yozmaydi. Konfiguratsiyada ham inline env berishingiz mumkin:

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: { GROQ_API_KEY: "gsk-..." },
  },
}
```

<Accordion title="Shell env import (ixtiyoriy)">
  Agar yoqilgan bo‘lsa va kerakli kalitlar hali o‘rnatilmagan bo‘lsa, OpenClaw login shell’ni ishga tushirib, faqat yetishmayotgan kalitlarni import qiladi:

```json5
{
  env: {
    shellEnv: { enabled: true, timeoutMs: 15000 },
  },
}
```

Env ekvivalenti: `OPENCLAW_LOAD_SHELL_ENV=1`
</Accordion>

<Accordion title="Konfiguratsiyada env o‘zgaruvchilarni ishlatish">
  Istalgan string qiymatda `${VAR_NAME}` sintaksisidan foydalaning:

```json5
{
  gateway: { auth: { token: "${OPENCLAW_GATEWAY_TOKEN}" } },
  models: { providers: { custom: { apiKey: "${CUSTOM_API_KEY}" } } },
}
```

Qoidalar:

- Faqat katta harfli nomlar mos keladi: `[A-Z_][A-Z0-9_]*`
- Yo‘q yoki bo‘sh qiymatlar yuklash vaqtida xatolik beradi
- Literal chiqarish uchun `$${VAR}` dan foydalaning
- `$include` fayllar ichida ham ishlaydi
- Inline birlashtirish: `"${BASE}/v1"` → `"https://api.example.com/v1"`

</Accordion>

## To‘liq ma’lumotnoma

Barcha maydonlar bo‘yicha batafsil hujjat uchun **[Configuration Reference](/gateway/configuration-reference)** sahifasiga qarang.

---

_Bog‘liq: [Configuration Examples](/gateway/configuration-examples) · [Configuration Reference](/gateway/configuration-reference) · [Doctor](/gateway/doctor)_