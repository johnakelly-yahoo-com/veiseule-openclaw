---
title: "Onboarding Wizard ma’lumotnomasi"
sidebarTitle: "Wizard ma’lumotnomasi"
---

# Onboarding Wizard ma’lumotnomasi

Bu `openclaw onboard` CLI wizard uchun to‘liq ma’lumotnoma.
Yuqori darajadagi umumiy ko‘rinish uchun [Onboarding Wizard](/start/wizard) sahifasiga qarang.

## Jarayon tafsilotlari (local rejim)

<Steps>
  <Step title="Mavjud konfiguratsiyani aniqlash">
    - Agar `~/.openclaw/openclaw.json` mavjud bo‘lsa, **Keep / Modify / Reset** dan birini tanlang.
    - Wizard’ni qayta ishga tushirish **Reset** ni aniq tanlamaguningizcha
      (yoki `--reset` uzatmaguningizcha) hech narsani o‘chirmaydi.
    - Agar konfiguratsiya yaroqsiz bo‘lsa yoki eski (legacy) kalitlarni o‘z ichiga olsa, wizard to‘xtaydi va
      davom etishdan oldin `openclaw doctor` ni ishga tushirishingizni so‘raydi.
    - Reset `trash` dan foydalanadi (`rm` emas) va quyidagi qamrovlarni taklif qiladi:
      - Faqat konfiguratsiya
      - Konfiguratsiya + credentiallar + sessiyalar
      - To‘liq reset (workspace ham o‘chiriladi)
  
</Step>
  <Step title="Model/Auth">
    - **Anthropic API key (tavsiya etiladi)**: agar mavjud bo‘lsa `ANTHROPIC_API_KEY` dan foydalanadi yoki kalitni so‘raydi, so‘ngra uni daemon foydalanishi uchun saqlaydi.
    - **Anthropic OAuth (Claude Code CLI)**: macOS’da wizard Keychain’dagi "Claude Code-credentials" elementini tekshiradi ("Always Allow" ni tanlang, shunda launchd ishga tushganda bloklanmaydi); Linux/Windows’da mavjud bo‘lsa `~/.claude/.credentials.json` dan foydalanadi.
    - **Anthropic token (setup-token ni joylashtirish)**: istalgan mashinada `claude setup-token` ni ishga tushiring, so‘ng tokenni joylashtiring (unga nom berishingiz mumkin; bo‘sh qoldirilsa = default).
    - **OpenAI Code (Codex) subscription (Codex CLI)**: agar `~/.codex/auth.json` mavjud bo‘lsa, wizard undan foydalanishi mumkin.
    - **OpenAI Code (Codex) subscription (OAuth)**: brauzer jarayoni; `code#state` ni joylashtiring.
      - Agar model o‘rnatilmagan bo‘lsa yoki `openai/*` bo‘lsa, `agents.defaults.model` ni `openai-codex/gpt-5.2` ga o‘rnatadi.
    - **OpenAI API key**: agar mavjud bo‘lsa `OPENAI_API_KEY` dan foydalanadi yoki kalitni so‘raydi, so‘ng launchd o‘qishi uchun uni `~/.openclaw/.env` ga saqlaydi.
    - **xAI (Grok) API key**: `XAI_API_KEY` ni so‘raydi va xAI’ni model provayder sifatida sozlaydi.
    - **OpenCode Zen (multi-model proxy)**: `OPENCODE_API_KEY` (yoki `OPENCODE_ZEN_API_KEY`, uni https://opencode.ai/auth dan oling) ni so‘raydi.
    - **API key**: kalitni siz uchun saqlaydi.
    - **Vercel AI Gateway (multi-model proxy)**: `AI_GATEWAY_API_KEY` ni so‘raydi.
    - Batafsil: [Vercel AI Gateway](/providers/vercel-ai-gateway)
    - **Cloudflare AI Gateway**: Account ID, Gateway ID va `CLOUDFLARE_AI_GATEWAY_API_KEY` ni so‘raydi.
    - Batafsil: [Cloudflare AI Gateway](/providers/cloudflare-ai-gateway)
    - **MiniMax M2.1**: konfiguratsiya avtomatik yoziladi.
    - Batafsil: [MiniMax](/providers/minimax)
    - **Synthetic (Anthropic-compatible)**: `SYNTHETIC_API_KEY` ni so‘raydi.
    - Batafsil: [Synthetic](/providers/synthetic)
    - **Moonshot (Kimi K2)**: konfiguratsiya avtomatik yoziladi.
    - **Kimi Coding**: konfiguratsiya avtomatik yoziladi.
    - Batafsil: [Moonshot AI (Kimi + Kimi Coding)](/providers/moonshot)
    - **Skip**: hozircha auth sozlanmaydi.
    - Aniqlangan variantlardan default modelni tanlang (yoki provider/model ni qo‘lda kiriting).
    - Wizard modelni tekshiradi va agar sozlangan model noma’lum bo‘lsa yoki auth yetishmasa, ogohlantiradi.
    - OAuth credentiallari `~/.openclaw/credentials/oauth.json` da saqlanadi; auth profillari `~/.openclaw/agents/<agentId>/agent/auth-profiles.json` da saqlanadi (API keylar + OAuth).
    - Batafsil: [/concepts/oauth](/concepts/oauth)
    <Note>
    Headless/server uchun maslahat: OAuth jarayonini brauzerli mashinada yakunlang, so‘ng
    `~/.openclaw/credentials/oauth.json` (yoki `$OPENCLAW_STATE_DIR/credentials/oauth.json`) faylini
    gateway hostiga ko‘chiring.
    
</Note>
  
</Step>
  <Step title="Workspace">
    - Standart `~/.openclaw/workspace` (o‘zgartirilishi mumkin).
    - Agent bootstrap jarayoni uchun zarur workspace fayllarini yaratadi.
    - To‘liq workspace tuzilmasi + zaxira qo‘llanmasi: [Agent workspace](/concepts/agent-workspace)
  
</Step>
  <Step title="Gateway">
    - Port, bind, auth rejimi, tailscale orqali ochish.
    - Auth bo‘yicha tavsiya: hatto loopback uchun ham **Token** ni saqlang, shunda mahalliy WS mijozlari autentifikatsiyadan o‘tadi.
    - Faqat barcha mahalliy jarayonlarga to‘liq ishonchingiz bo‘lsa auth’ni o‘chiring.
    - Non‑loopback bind’lar ham auth talab qiladi.
  
</Step>
  <Step title="Channels">
    - [WhatsApp](/channels/whatsapp): ixtiyoriy QR login.
    - [Telegram](/channels/telegram): bot tokeni.
    - [Discord](/channels/discord): bot tokeni.
    - [Google Chat](/channels/googlechat): xizmat hisobi JSON + webhook auditoriysi.
    - [Mattermost](/channels/mattermost) (plagin): bot token + asosiy URL.
    - [Signal](/channels/signal): ixtiyoriy `signal-cli` o‘rnatish + account konfiguratsiyasi.
    - [BlueBubbles](/channels/bluebubbles): **iMessage uchun tavsiya etiladi**; server URL + parol + webhook.
    - [iMessage](/channels/imessage): eski `imsg` CLI yo‘li + DB kirishi.
    - DM xavfsizligi: standart holatda pairing. Birinchi DM kod yuboradi; `openclaw pairing approve <channel> <code>` orqali tasdiqlang yoki allowlist’dan foydalaning.
  
</Step>
  <Step title="Daemon o‘rnatish">
    - macOS: LaunchAgent
      - Tizimga kirilgan foydalanuvchi sessiyasi talab qilinadi; headless uchun maxsus LaunchDaemon’dan foydalaning (taqdim etilmaydi).
    - Linux (va Windows WSL2 orqali): systemd user unit
      - Gateway logout’dan keyin ham ishlashi uchun wizard `loginctl enable-linger <user>` ni yoqishga harakat qiladi.
      - sudo so‘rashi mumkin (`/var/lib/systemd/linger` ga yozadi); avval sudo’siz urinib ko‘radi.
    - **Runtime tanlash:** Node (tavsiya etiladi; WhatsApp/Telegram uchun talab qilinadi). Bun **tavsiya etilmaydi**.
  
</Step>
  <Step title="Health check">
    - Gateway’ni ishga tushiradi (agar kerak bo‘lsa) va `openclaw health` ni ishga tushiradi.
    - Maslahat: `openclaw status --deep` status chiqishiga gateway health tekshiruvlarini qo‘shadi (gateway mavjud bo‘lishi kerak).
  
</Step>
  <Step title="Skills (tavsiya etiladi)">
    - Mavjud skills’larni o‘qiydi va talablarni tekshiradi.
    - Node manager tanlash imkonini beradi: **npm / pnpm** (bun tavsiya etilmaydi).
    - Ixtiyoriy bog‘liqliklarni o‘rnatadi (ba’zilari macOS’da Homebrew’dan foydalanadi).
  
</Step>
  <Step title="Yakunlash">
    - Xulosa + keyingi qadamlar, jumladan qo‘shimcha funksiyalar uchun iOS/Android/macOS ilovalari.
  
</Step>
</Steps>

<Note>
Agar GUI aniqlanmasa, wizard brauzerni ochish o‘rniga Control UI uchun SSH port-forward ko‘rsatmalarini chiqaradi.
Agar Control UI assetlari mavjud bo‘lmasa, wizard ularni build qilishga harakat qiladi; fallback — `pnpm ui:build` (UI bog‘liqliklarini avtomatik o‘rnatadi).
</Note>

## Non-interactive rejim

Onboarding’ni avtomatlashtirish yoki skriptlash uchun `--non-interactive` dan foydalaning:

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice apiKey \
  --anthropic-api-key "$ANTHROPIC_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback \
  --install-daemon \
  --daemon-runtime node \
  --skip-skills
```

Mashina o‘qiy oladigan xulosa uchun `--json` ni qo‘shing.

<Note>
`--json` **non-interactive** rejimni avtomatik yoqmaydi. Skriptlar uchun `--non-interactive` (va `--workspace`) dan foydalaning.
</Note>

<AccordionGroup>
  <Accordion title="Gemini misoli">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice gemini-api-key \
      --gemini-api-key "$GEMINI_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  
</Accordion>
  <Accordion title="Z.AI misoli">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice zai-api-key \
      --zai-api-key "$ZAI_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  
</Accordion>
  <Accordion title="Vercel AI Gateway misoli">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice ai-gateway-api-key \
      --ai-gateway-api-key "$AI_GATEWAY_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  
</Accordion>
  <Accordion title="Cloudflare AI Gateway misoli">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice cloudflare-ai-gateway-api-key \
      --cloudflare-ai-gateway-account-id "your-account-id" \
      --cloudflare-ai-gateway-gateway-id "your-gateway-id" \
      --cloudflare-ai-gateway-api-key "$CLOUDFLARE_AI_GATEWAY_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  
</Accordion>
  <Accordion title="Moonshot misoli">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice moonshot-api-key \
      --moonshot-api-key "$MOONSHOT_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  
</Accordion>
  <Accordion title="Synthetic misoli">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice synthetic-api-key \
      --synthetic-api-key "$SYNTHETIC_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  
</Accordion>
  <Accordion title="OpenCode Zen misoli">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice opencode-zen \
      --opencode-zen-api-key "$OPENCODE_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  
</Accordion>
</AccordionGroup>

### Agent qo‘shish (non-interactive)

```bash
openclaw agents add work \
  --workspace ~/.openclaw/workspace-work \
  --model openai/gpt-5.2 \
  --bind whatsapp:biz \
  --non-interactive \
  --json
```

## Gateway sozlash ustasi RPC

Gateway wizard jarayonini RPC orqali taqdim etadi (`wizard.start`, `wizard.next`, `wizard.cancel`, `wizard.status`).
Mijozlar (macOS ilovasi, Control UI) onboarding logikasini qayta yozmasdan qadamlarni ko‘rsatishi mumkin.

## Signal sozlamasi (signal-cli)

Wizard `signal-cli` ni GitHub release’laridan o‘rnatishi mumkin:

- Mos release assetini yuklab oladi.
- Uni `~/.openclaw/tools/signal-cli/<version>/` ichiga saqlaydi.
- Konfiguratsiyaga `channels.signal.cliPath` ni yozadi.

Eslatmalar:

- JVM build’lari uchun **Java 21** talab qilinadi.
- Mavjud bo‘lsa, native build’lardan foydalaniladi.
- Windows WSL2’dan foydalanadi; signal-cli o‘rnatilishi WSL ichida Linux jarayoni bo‘yicha amalga oshiriladi.

## Wizard nima yozadi

`~/.openclaw/openclaw.json` ichidagi odatiy maydonlar:

- `agents.defaults.workspace`
- `agents.defaults.model` / `models.providers` (agar Minimax tanlansa)
- `gateway.*` (mode, bind, auth, tailscale)
- `channels.telegram.botToken`, `channels.discord.token`, `channels.signal.*`, `channels.imessage.*`
- Promptlar vaqtida rozi bo‘lsangiz, channel allowlist’lari (Slack/Discord/Matrix/Microsoft Teams) (imkon bo‘lsa nomlar ID’ga aylantiriladi).
- `skills.install.nodeManager`
- `wizard.lastRunAt`
- `wizard.lastRunVersion`
- `wizard.lastRunCommit`
- `wizard.lastRunCommand`
- `wizard.lastRunMode`

`openclaw agents add` `agents.list[]` va ixtiyoriy `bindings` ni yozadi.

WhatsApp credentiallari `~/.openclaw/credentials/whatsapp/<accountId>/` ostida saqlanadi.
Sessiyalar `~/.openclaw/agents/<agentId>/sessions/` ostida saqlanadi.

Ba’zi channel’lar plugin sifatida yetkaziladi. Onboarding vaqtida ulardan birini tanlasangiz, wizard
uni sozlashdan oldin o‘rnatishni (npm yoki lokal yo‘l orqali) so‘raydi.

## Tegishli hujjatlar

- Wizard umumiy ko‘rinishi: [Onboarding Wizard](/start/wizard)
- macOS ilovasi onboarding: [Onboarding](/start/onboarding)
- Konfiguratsiya ma’lumotnomasi: [Gateway configuration](/gateway/configuration)
- Provayderlar: [WhatsApp](/channels/whatsapp), [Telegram](/channels/telegram), [Discord](/channels/discord), [Google Chat](/channels/googlechat), [Signal](/channels/signal), [BlueBubbles](/channels/bluebubbles) (iMessage), [iMessage](/channels/imessage) (eski)
- Skills: [Skills](/tools/skills), [Skills config](/tools/skills-config)

