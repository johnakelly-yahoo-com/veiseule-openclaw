---
summary: "`~/.openclaw/openclaw.json` uchun barcha konfiguratsiya opsiyalari misollar bilan"
read_when:
  - Konfiguratsiya maydonlarini qo‘shish yoki o‘zgartirish
  - Keng tarqalgan konfiguratsiya andozalarini qidiryapsizmi
  - Muayyan konfiguratsiya bo‘limlariga o‘tish
title: "Konfiguratsiya"
---

# Konfiguratsiya 🔧

OpenClaw `~/.openclaw/openclaw.json` faylidan ixtiyoriy **JSON5** konfiguratsiyani o‘qiydi (izohlar va oxirgi vergullar ruxsat etiladi).

Agar fayl mavjud bo‘lmasa, OpenClaw xavfsiz sukut bo‘yicha sozlamalardan foydalanadi. Konfiguratsiya qo‘shish uchun keng tarqalgan sabablar:

- botni kim ishga tushira olishini cheklash (`channels.whatsapp.allowFrom`, `channels.telegram.allowFrom`, va hokazo)
- guruhlar uchun ruxsat ro‘yxatlari va eslatish xatti-harakatini boshqarish (`channels.whatsapp.groups`, `channels.telegram.groups`, `channels.discord.guilds`, `agents.list[].groupChat`)
- xabar prefikslarini moslashtirish (`messages`)

Mavjud barcha maydonlar uchun [to‘liq ma’lumotnoma](/gateway/configuration-reference)ni ko‘ring.

<Tip>
**Konfiguratsiyaga yangimisiz?** Interaktiv sozlash uchun `openclaw onboard` buyrug‘idan boshlang yoki to‘liq ko‘chirib‑qo‘yishga tayyor konfiguratsiyalar uchun [Configuration Examples](/gateway/configuration-examples) qo‘llanmasini ko‘ring.
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
  <Tab title="Interactive wizard">
    ```bash
    openclaw onboard       # full setup wizard
    openclaw configure     # config wizard
    ```
  
</Tab>
  <Tab title="CLI (one-liners)">
    ```bash
    openclaw config get agents.defaults.workspace
    openclaw config set agents.defaults.heartbeat.every "2h"
    openclaw config unset tools.web.search.apiKey
    ```
  
</Tab>
  <Tab title="Control UI">
    [http://127.0.0.1:18789](http://127.0.0.1:18789) manzilini oching va **Config** yorlig‘idan foydalaning.
    Control UI konfiguratsiya sxemasidan forma yaratadi va zarurat uchun **Raw JSON** muharririni ham taqdim etadi.
  
</Tab>
  <Tab title="Direct edit">
    `~/.openclaw/openclaw.json` faylini to‘g‘ridan-to‘g‘ri tahrirlang. Gateway faylni kuzatadi va o‘zgarishlarni avtomatik qo‘llaydi (qarang: [hot reload](#config-hot-reload)).
  
</Tab>
</Tabs>

## Sxema + UI ko‘rsatmalari

<Warning>
Har bir agent uchun identifikatsiyani o‘rnating (`agents.list[].identity`). Noma’lum kalitlar, noto‘g‘ri turdagi qiymatlar yoki yaroqsiz qiymatlar sababli Gateway **ishga tushishni rad etadi**. Ildiz darajadagi yagona istisno — `$schema` (string), shunda muharrirlar JSON Schema metama’lumotlarini biriktira oladi.
</Warning>

Kanal plaginlari va kengaytmalari o‘z konfiguratsiyasi uchun sxema va UI ko‘rsatmalarini ro‘yxatdan o‘tkazishi mumkin, shunda kanal sozlamalari ilovalar bo‘ylab qattiq kodlangan formalarsiz sxema asosida qoladi.

- Gateway ishga tushmaydi
- Faqat diagnostika buyruqlari ishlaydi (`openclaw doctor`, `openclaw logs`, `openclaw health`, `openclaw status`)
- Aniq muammolarni ko‘rish uchun `openclaw doctor` ni ishga tushiring
- Tuzatishlarni qo‘llash uchun `openclaw doctor --fix` (yoki `--yes`) ni ishga tushiring

## Qo‘llash + qayta ishga tushirish (RPC)

<AccordionGroup>
  <Accordion title="Set up a channel (WhatsApp, Telegram, Discord, etc.)">
    Har bir kanalning o‘z konfiguratsiya bo‘limi `channels.` ostida mavjud.<provider>`commands.bashForegroundMs` bash fon rejimiga o‘tishdan oldin qancha kutishini boshqaradi. Sozlash bosqichlari uchun tegishli kanal sahifasini ko‘ring:

    ````
    - [WhatsApp](/channels/whatsapp) — `channels.whatsapp`
    - [Telegram](/channels/telegram) — `channels.telegram`
    - [Discord](/channels/discord) — `channels.discord`
    - [Slack](/channels/slack) — `channels.slack`
    - [Signal](/channels/signal) — `channels.signal`
    - [iMessage](/channels/imessage) — `channels.imessage`
    - [Google Chat](/channels/googlechat) — `channels.googlechat`
    - [Mattermost](/channels/mattermost) — `channels.mattermost`
    - [MS Teams](/channels/msteams) — `channels.msteams`
    
    Barcha kanallar bir xil DM siyosati andozasidan foydalanadi:
    
    ```json5
    {
      channels: {
        telegram: {
          enabled: true,
          botToken: "123:abc",
          dmPolicy: "pairing",   // pairing | allowlist | open | disabled
          allowFrom: ["tg:123"], // faqat allowlist/open uchun
        },
      },
    }
    ```
    ````

  
</Accordion>

  <Accordion title="Choose and configure models">
    Asosiy modelni va ixtiyoriy zaxira variantlarini sozlang:

    ````
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
    - Model havolalari `provider/model` formatidan foydalanadi (masalan, `anthropic/claude-opus-4-6`).
    - Chatda modelni almashtirish uchun [Models CLI](/concepts/models) va autentifikatsiya rotatsiyasi hamda fallback xatti-harakati uchun [Model Failover](/concepts/model-failover) sahifalarini ko‘ring.
    - Maxsus/self-hosted provayderlar uchun ma’lumotnomadagi [Custom providers](/gateway/configuration-reference#custom-providers-and-base-urls) bo‘limiga qarang.
    ````

  
</Accordion>

  <Accordion title="Control who can message the bot">
    DM kirishi har bir kanal uchun `dmPolicy` orqali boshqariladi:

    ```
    - `"pairing"` (sukut bo‘yicha): noma’lum jo‘natuvchilarga tasdiqlash uchun bir martalik pairing kodi beriladi
    - `"allowlist"`: faqat `allowFrom` dagi (yoki pairing orqali qo‘shilgan) jo‘natuvchilar
    - `"open"`: barcha kiruvchi DMlarga ruxsat (buning uchun `allowFrom: ["*"]` talab qilinadi)
    - `"disabled"`: barcha DMlar e’tiborsiz qoldiriladi
    
    Guruhlar uchun `groupPolicy` + `groupAllowFrom` yoki kanalga xos allowlistlardan foydalaning.
    
    Har bir kanal bo‘yicha tafsilotlar uchun [to‘liq ma’lumotnoma](/gateway/configuration-reference#dm-and-group-access)ni ko‘ring.
    ```

  
</Accordion>

  <Accordion title="Set up group chat mention gating">
    Guruh xabarlari sukut bo‘yicha **mention talab qiladi**. Har bir agent uchun andozalarni sozlang:

    ````
    ```json5
    {
      agents: {
        list: [
          {
            id: "main",
            groupChat: {
              mentionPatterns: ["@openclaw", "openclaw"],
            },
          },
        ],
      },
      channels: {
        whatsapp: {
          groups: { "*": { requireMention: true } },
        },
      },
    }
    ```
    
    - **Metadata mentionlar**: platformaning o‘ziga xos @-mentionlari (WhatsApp tap-to-mention, Telegram @bot va boshqalar)
    - **Matn andozalari**: `mentionPatterns` ichidagi regex andozalar
    - Har bir kanal bo‘yicha override’lar va self-chat rejimi uchun [to‘liq ma’lumotnoma](/gateway/configuration-reference#group-chat-mention-gating)ni ko‘ring.
    ````

  
</Accordion>

  <Accordion title="Configure sessions and resets">
    Sessiyalar suhbat uzluksizligi va izolyatsiyasini boshqaradi:

    ````
    ```json5
    {
      session: {
        dmScope: "per-channel-peer",  // ko‘p foydalanuvchi uchun tavsiya etiladi
        reset: {
          mode: "daily",
          atHour: 4,
          idleMinutes: 120,
        },
      },
    }
    ```
    
    - `dmScope`: `main` (umumiy) | `per-peer` | `per-channel-peer` | `per-account-channel-peer`
    - Qamrov, identifikatsiya bog‘lanishlari va yuborish siyosati uchun [Session Management](/concepts/session)ni ko‘ring.
    - Barcha maydonlar uchun [to‘liq ma’lumotnoma](/gateway/configuration-reference#session)ni ko‘ring.
    ````

  
</Accordion>

  <Accordion title="Enable sandboxing">
    Agent sessiyalarini izolyatsiyalangan Docker konteynerlarida ishga tushiring:

    ```
    scripts/sandbox-setup.sh
    ```

  
</Accordion>

  <Accordion title="Set up heartbeat (periodic check-ins)">
    ```json5
    {
      agents: {
        defaults: {
          heartbeat: {
            every: "30m",
            target: "last",
          },
        },
      },
    }
    ```
  

    ```
    {
      agents: {
        defaults: { workspace: "~/.openclaw/workspace" },
        list: [
          {
            id: "main",
            groupChat: { mentionPatterns: ["@openclaw", "reisponde"] },
          },
        ],
      },
      channels: {
        whatsapp: {
          // Allowlist is DMs only; including your own number enables self-chat mode.
          allowFrom: ["+15555550123"],
          groups: { "*": { requireMention: true } },
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="Configure cron jobs">```json5
{
  cron: {
    enabled: true,
    maxConcurrentRuns: 2,
    sessionRetention: "24h",
  },
}
```

    ```
    See [Cron jobs](/automation/cron-jobs) for the feature overview and CLI examples.
    ```

  
</Accordion>

  <Accordion title="Set up webhooks (hooks)">Gateway’da HTTP webhook endpoint’larini yoqing:

    ```
    // ~/.openclaw/agents.json5
    {
      defaults: { sandbox: { mode: "all", scope: "session" } },
      list: [{ id: "main", workspace: "~/.openclaw/workspace" }],
    }
    ```

  
</Accordion>

  <Accordion title="Configure multi-agent routing">Alohida ish maydonlari va sessiyalarga ega bir nechta izolyatsiyalangan agentlarni ishga tushiring:

    ```
    // Sibling keys override included values
    {
      $include: "./base.json5", // { a: 1, b: 2 }
      b: 99, // Result: { a: 1, b: 99 }
    }
    ```

  
</Accordion>

  <Accordion title="Split config into multiple files ($include)">Katta konfiguratsiyalarni tartibga solish uchun `$include` dan foydalaning:

    ```
    // clients/mueller.json5
    {
      agents: { $include: "./mueller/agents.json5" },
      broadcast: { $include: "./mueller/broadcast.json5" },
    }
    ```

  
</Accordion>
</AccordionGroup>

## Konfiguratsiyani hot reload qilish

Gateway `~/.openclaw/openclaw.json` faylini kuzatadi va o‘zgarishlarni avtomatik qo‘llaydi — ko‘pchilik sozlamalar uchun qo‘lda qayta ishga tushirish talab etilmaydi.

### Error handling

| Rejim                                      | Xulq-atvor                                                                                                                                                                   |
| ------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`hybrid`** (standart) | Xavfsiz o‘zgarishlarni darhol hot qo‘llaydi. Muhim o‘zgarishlar uchun avtomatik ravishda qayta ishga tushiradi.                              |
| **`hot`**                                  | Faqat xavfsiz o‘zgarishlarni hot qo‘llaydi. Qayta ishga tushirish kerak bo‘lganda ogohlantirish yozuvini chiqaradi — uni o‘zingiz bajarasiz. |
| **`restart`**                              | Har qanday konfiguratsiya o‘zgarishida, xavfsiz yoki yo‘qligidan qat’i nazar, Gateway’ni qayta ishga tushiradi.                                              |
| **`off`**                                  | Fayl kuzatuvini o‘chiradi. O‘zgarishlar keyingi qo‘lda qayta ishga tushirishda kuchga kiradi.                                                |

```json5
{
  gateway: {
    reload: { mode: "hybrid", debounceMs: 300 },
  },
}
```

### Qaysi o‘zgarishlar hot qo‘llanadi va qaysilari qayta ishga tushirishni talab qiladi

Ko‘pchilik maydonlar uzilishsiz hot qo‘llanadi. `hybrid` rejimida qayta ishga tushirish talab qiladigan o‘zgarishlar avtomatik tarzda bajariladi.

| Kategoriya             | Maydonlar                                                                              | Qayta ishga tushirish kerakmi? |
| ---------------------- | -------------------------------------------------------------------------------------- | ------------------------------ |
| Kanallar               | `channels.*`, `web` (WhatsApp) — barcha ichki va kengaytma kanallar | Yo‘q                           |
| Agent va modellar      | `agent`, `agents`, `models`, `routing`                                                 | Yo‘q                           |
| Avtomatlashtirish      | `hooks`, `cron`, `agent.heartbeat`                                                     | Yo‘q                           |
| Sessiyalar va xabarlar | `session`, `messages`                                                                  | Yo‘q                           |
| Asboblar va media      | `tools`, `browser`, `skills`, `audio`, `talk`                                          | Yo‘q                           |
| UI va boshqalar        | `ui`, `logging`, `identity`, `bindings`                                                | Yo‘q                           |
| Gateway serveri        | `gateway.*` (port, bind, auth, tailscale, TLS, HTTP)                | **Ha**                         |
| Infratuzilma           | `discovery`, `canvasHost`, `plugins`                                                   | **Ha**                         |

<Note>
`gateway.reload` va `gateway.remote` istisno hisoblanadi — ularni o‘zgartirish qayta ishga tushirishni **ishga tushirmaydi**.
</Note>

## Env vars + `.env`

<AccordionGroup>
  <Accordion title="config.apply (full replace)">
    To‘liq konfiguratsiyani tekshiradi + yozadi va Gateway’ni bir bosqichda qayta ishga tushiradi.


    ````
    <Warning>
    `config.apply` **butun konfiguratsiyani** almashtiradi. Qisman yangilash uchun `config.patch` dan yoki alohida kalitlar uchun `openclaw config set` dan foydalaning.
    
</Warning>
    
    Params:
    
    - `raw` (string) — butun konfiguratsiya uchun JSON5 yuklama
    - `baseHash` (ixtiyoriy) — `config.get` dan olingan konfiguratsiya xeshi (konfiguratsiya mavjud bo‘lsa majburiy)
    - `sessionKey` (ixtiyoriy) — qayta ishga tushirilgandan keyingi uyg‘otish pingi uchun sessiya kaliti
    - `note` (ixtiyoriy) — qayta ishga tushirish belgisi uchun izoh
    - `restartDelayMs` (ixtiyoriy) — qayta ishga tushirishdan oldingi kechikish (standart 2000)
    
    ```bash
    openclaw gateway call config.get --params '{}'  # payload.hash ni oling
    openclaw gateway call config.apply --params '{
      "raw": "{ agents: { defaults: { workspace: \"~/.openclaw/workspace\" } } }",
      "baseHash": "<hash>",
      "sessionKey": "agent:main:whatsapp:dm:+15555550123"
    }'
    ```
    ````

  
</Accordion>

  <Accordion title="config.patch (partial update)">
    Qisman yangilanishni mavjud konfiguratsiyaga birlashtiradi (JSON merge patch semantikasi):


    ````
    - Obyektlar rekursiv tarzda birlashtiriladi
    - `null` kalitni o‘chiradi
    - Massivlar to‘liq almashtiriladi
    
    Params:
    
    - `raw` (string) — faqat o‘zgartiriladigan kalitlar bilan JSON5
    - `baseHash` (majburiy) — `config.get` dan olingan konfiguratsiya xeshi
    - `sessionKey`, `note`, `restartDelayMs` — `config.apply` bilan bir xil
    
    ```bash
    openclaw gateway call config.patch --params '{
      "raw": "{ channels: { telegram: { groups: { \"*\": { requireMention: false } } } } }",
      "baseHash": "<hash>"
    }'
    ```
    ````

  
</Accordion>
</AccordionGroup>

## Muhit o‘zgaruvchilari

OpenClaw muhit o‘zgaruvchilarini ota-jarayondan, shuningdek quyidagilardan o‘qiydi:

- `.env` from the current working directory (if present)
- `~/.openclaw/.env` (global zaxira)

Hech biri mavjud muhit o‘zgaruvchilarini ustiga yozmaydi. Shuningdek, konfiguratsiyada inline muhit o‘zgaruvchilarini ham sozlashingiz mumkin:

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: { GROQ_API_KEY: "gsk-..." },
  },
}
```

<Accordion title="Shell env import (optional)">
  Agar yoqilgan bo‘lsa va kutilgan kalitlar o‘rnatilmagan bo‘lsa, OpenClaw login shell’ingizni ishga tushiradi va faqat yetishmayotgan kalitlarni import qiladi:


```json5
{
  env: {
    shellEnv: { enabled: true, timeoutMs: 15000 },
  },
}
```

Muhit o‘zgaruvchisi ekvivalenti: `OPENCLAW_LOAD_SHELL_ENV=1` 
</Accordion>

<Accordion title="Env var substitution in config values">
  Istalgan konfiguratsiya satr qiymatida muhit o‘zgaruvchilariga `${VAR_NAME}` orqali murojaat qiling:


```json5
{
  gateway: { auth: { token: "${OPENCLAW_GATEWAY_TOKEN}" } },
  models: { providers: { custom: { apiKey: "${CUSTOM_API_KEY}" } } },
}
```

Qoidalar:

- Faqat katta harflardagi nomlar mos keladi: `[A-Z_][A-Z0-9_]*`
- Mavjud bo‘lmagan/bo‘sh o‘zgaruvchilar yuklash vaqtida xatolik chiqaradi
- Literal chiqish uchun `$${VAR}` dan foydalaning
- `$include` fayllari ichida ham ishlaydi
- Inline almashtirish: `"${BASE}/v1"` → `"https://api.example.com/v1"`

</Accordion>

To‘liq ustuvorlik va manbalar uchun [Environment](/help/environment) ga qarang.

## To‘liq ma’lumotnoma

To‘liq maydonma-maydon ma’lumot uchun **[Configuration Reference](/gateway/configuration-reference)** ga qarang.

---

Legacy OAuth imports:

