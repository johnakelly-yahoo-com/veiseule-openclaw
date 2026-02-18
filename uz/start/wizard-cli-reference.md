---
title: "CLI Ishga tushirish bo‘yicha qo‘llanma"
sidebarTitle: "CLI ma’lumotnomasi"
---

# CLI Ishga tushirish bo‘yicha qo‘llanma

Ushbu sahifa `openclaw onboard` uchun to‘liq ma’lumotnomadir.
Qisqa qo‘llanma uchun [Onboarding Wizard (CLI)](/start/wizard) ga qarang.

## Wizard nima qiladi

Mahalliy rejim (standart) sizni quyidagilar bo‘yicha bosqichma-bosqich olib boradi:

- Model va autentifikatsiya sozlamalari (OpenAI Code obunasi OAuth, Anthropic API kaliti yoki sozlash tokeni, shuningdek MiniMax, GLM, Moonshot va AI Gateway variantlari)
- Ishchi makon joylashuvi va bootstrap fayllari
- Gateway sozlamalari (port, bind, auth, tailscale)
- Kanallar va provayderlar (Telegram, WhatsApp, Discord, Google Chat, Mattermost plagini, Signal)
- Daemon o‘rnatish (LaunchAgent yoki systemd foydalanuvchi birligi)
- Holatni tekshirish
- Skills sozlamasi

Masofaviy rejim ushbu qurilmani boshqa joydagi gateway’ga ulanish uchun sozlaydi.
U masofaviy xostda hech narsa o‘rnatmaydi yoki o‘zgartirmaydi.

## Mahalliy oqim tafsilotlari

<Steps>
  <Step title="Existing config detection">
    - Agar `~/.openclaw/openclaw.json` mavjud bo‘lsa, Keep, Modify yoki Reset’ni tanlang.
    - Sehrgarni qayta ishga tushirish hech narsani o‘chirmaydi, agar siz aniq Reset’ni tanlamasangiz (yoki `--reset` parametrini bermasangiz).
    - Agar konfiguratsiya noto‘g‘ri bo‘lsa yoki eski kalitlarni o‘z ichiga olsa, sehrgar davom etishdan oldin `openclaw doctor` ni ishga tushirishingizni so‘raydi.
    - Reset `trash` dan foydalanadi va quyidagi doiralarni taklif qiladi:
      - Faqat konfiguratsiya
      - Konfiguratsiya + hisob ma’lumotlari + sessiyalar
      - To‘liq reset (ishchi makonni ham olib tashlaydi)
  
</Step>
  <Step title="Model and auth">
    - To‘liq variantlar matritsasi [Autentifikatsiya va model variantlari](#auth-and-model-options) bo‘limida keltirilgan.
  
</Step>
  <Step title="Workspace">
    - Sukut bo‘yicha `~/.openclaw/workspace` (sozlanishi mumkin).
    - Birinchi ishga tushirish bootstrap jarayoni uchun zarur bo‘lgan workspace fayllarini yaratadi.
    - Ishchi makon tuzilishi: [Agent workspace](/concepts/agent-workspace).
  
</Step>
  <Step title="Gateway">
    - Port, bind, auth rejimi va tailscale ochiqligi bo‘yicha so‘rovlar beradi.
    - Tavsiya etiladi: loopback uchun ham token autentifikatsiyasini yoqilgan holda saqlang, shunda mahalliy WS mijozlari autentifikatsiyadan o‘tishi shart bo‘ladi.
    - Auth’ni faqat barcha mahalliy jarayonlarga to‘liq ishonsangizgina o‘chiring.
    - Non-loopback bind’lar baribir auth talab qiladi.
  
</Step>
  <Step title="Channels">
    - [WhatsApp](/channels/whatsapp): ixtiyoriy QR login
    - [Telegram](/channels/telegram): bot token
    - [Discord](/channels/discord): bot token
    - [Google Chat](/channels/googlechat): service account JSON + webhook audience
    - [Mattermost](/channels/mattermost) plagini: bot token + base URL
    - [Signal](/channels/signal): ixtiyoriy `signal-cli` o‘rnatish + akkaunt konfiguratsiyasi
    - [BlueBubbles](/channels/bluebubbles): iMessage uchun tavsiya etiladi; server URL + parol + webhook
    - [iMessage](/channels/imessage): legacy `imsg` CLI yo‘li + DB kirish
    - DM xavfsizligi: standart — pairing. Birinchi DM kod yuboradi; tasdiqlash:
      `openclaw pairing approve <channel> <code>` yoki allowlist’dan foydalaning.
  
</Step>
  <Step title="Daemon install">
    - macOS: LaunchAgent
      - Tizimga kirilgan foydalanuvchi sessiyasini talab qiladi; headless holat uchun maxsus LaunchDaemon’dan foydalaning (taqdim etilmaydi).
    - Linux va Windows (WSL2 orqali): systemd user unit
      - Sehrgar `loginctl enable-linger <user>` ni ishga tushirishga urinadi, shunda gateway logout’dan keyin ham ishlashda davom etadi.
      - Sudo so‘rashi mumkin (`/var/lib/systemd/linger` yozadi); avval sudo’siz urinib ko‘radi.
    - Runtime tanlovi: Node (tavsiya etiladi; WhatsApp va Telegram uchun talab qilinadi). Bun tavsiya etilmaydi.
  
</Step>
  <Step title="Health check">
    - Gateway’ni (kerak bo‘lsa) ishga tushiradi va `openclaw health` ni bajaradi.
    - `openclaw status --deep` status chiqishiga gateway health probe’larini qo‘shadi.
  
</Step>
  <Step title="Skills">
    - Mavjud skills’larni o‘qiydi va talablarni tekshiradi.
    - Node manager tanlash imkonini beradi: npm yoki pnpm (bun tavsiya etilmaydi).
    - Ixtiyoriy bog‘liqliklarni o‘rnatadi (ba’zilari macOS’da Homebrew’dan foydalanadi).
  
</Step>
  <Step title="Finish">
    - Xulosa va keyingi qadamlar, jumladan iOS, Android va macOS ilova variantlari.
  
</Step>
</Steps>

<Note>
Agar GUI aniqlanmasa, sehrgar brauzer ochish o‘rniga Control UI uchun SSH port-forward ko‘rsatmalarini chiqaradi.
Agar Control UI asset’lari mavjud bo‘lmasa, sehrgar ularni build qilishga urinadi; fallback — `pnpm ui:build` (UI deps avtomatik o‘rnatiladi).
</Note>

## Masofaviy rejim tafsilotlari

Masofaviy rejim ushbu qurilmani boshqa joydagi gateway’ga ulanish uchun sozlaydi.

<Info>
Masofaviy rejim masofaviy xostda hech narsa o‘rnatmaydi yoki o‘zgartirmaydi.
</Info>

Siz sozlaysiz:

- Masofaviy gateway URL’i (`ws://...`)
- Agar masofaviy gateway auth talab qilsa, token (tavsiya etiladi)

<Note>
- Agar gateway faqat loopback bo‘lsa, SSH tunneling yoki tailnet’dan foydalaning.
- Aniqlash bo‘yicha maslahatlar:
  - macOS: Bonjour (`dns-sd`)
  - Linux: Avahi (`avahi-browse`)
</Note>

## Autentifikatsiya va model variantlari

<AccordionGroup>
  <Accordion title="Anthropic API key (recommended)">
    Agar mavjud bo‘lsa `ANTHROPIC_API_KEY` dan foydalanadi yoki kalitni so‘raydi, so‘ng uni daemon foydalanishi uchun saqlaydi.
  
</Accordion>
  <Accordion title="Anthropic OAuth (Claude Code CLI)">
    - macOS: Keychain’dagi "Claude Code-credentials" elementini tekshiradi
    - Linux va Windows: agar mavjud bo‘lsa `~/.claude/.credentials.json` dan foydalanadi

    macOS’da "Always Allow" ni tanlang, shunda launchd ishga tushirishlari bloklanmaydi.

  
</Accordion>
  <Accordion title="Anthropic token (setup-token paste)">
    `claude setup-token` buyrug‘ini istalgan mashinada ishga tushiring, so‘ng tokenni joylashtiring.
    Nom berishingiz mumkin; bo‘sh qoldirilsa, standart qiymat ishlatiladi.
  
</Accordion>
  <Accordion title="OpenAI Code subscription (Codex CLI reuse)">
    Agar `~/.codex/auth.json` mavjud bo‘lsa, wizard undan qayta foydalanishi mumkin.
  
</Accordion>
  <Accordion title="OpenAI Code subscription (OAuth)">
    Brauzer oqimi; `code#state` ni joylashtiring.

    Agar model o‘rnatilmagan yoki `openai/*` bo‘lsa, `agents.defaults.model` ni `openai-codex/gpt-5.3-codex` ga o‘rnatadi.

  
</Accordion>
  <Accordion title="OpenAI API key">
    Agar mavjud bo‘lsa `OPENAI_API_KEY` dan foydalanadi yoki kalitni so‘raydi, so‘ng launchd o‘qishi uchun
    uni `~/.openclaw/.env` ga saqlaydi.

    Agar model o‘rnatilmagan, `openai/*` yoki `openai-codex/*` bo‘lsa, `agents.defaults.model` ni `openai/gpt-5.1-codex` ga o‘rnatadi.

  
</Accordion>
  <Accordion title="xAI (Grok) API key">
    `XAI_API_KEY` ni so‘raydi va xAI’ni model provayderi sifatida sozlaydi.
  
</Accordion>
  <Accordion title="OpenCode Zen">
    `OPENCODE_API_KEY` (yoki `OPENCODE_ZEN_API_KEY`) ni so‘raydi.
    Sozlash URL’i: [opencode.ai/auth](https://opencode.ai/auth).
  
</Accordion>
  <Accordion title="API key (generic)">
    Kalitni siz uchun saqlaydi.
  
</Accordion>
  <Accordion title="Vercel AI Gateway">
    `AI_GATEWAY_API_KEY` ni so‘raydi.
    Batafsil: [Vercel AI Gateway](/providers/vercel-ai-gateway).
  
</Accordion>
  <Accordion title="Cloudflare AI Gateway">
    Hisob ID’si, gateway ID’si va `CLOUDFLARE_AI_GATEWAY_API_KEY` ni so‘raydi.
    Batafsil: [Cloudflare AI Gateway](/providers/cloudflare-ai-gateway).
  
</Accordion>
  <Accordion title="MiniMax M2.1">
    Konfiguratsiya avtomatik yoziladi.
    Batafsil: [MiniMax](/providers/minimax).
  
</Accordion>
  <Accordion title="Synthetic (Anthropic-compatible)">
    `SYNTHETIC_API_KEY` ni so‘raydi.
    Batafsil: [Synthetic](/providers/synthetic).
  
</Accordion>
  <Accordion title="Moonshot and Kimi Coding">
    Moonshot (Kimi K2) va Kimi Coding konfiguratsiyalari avtomatik yoziladi.
    Batafsil: [Moonshot AI (Kimi + Kimi Coding)](/providers/moonshot).
  
</Accordion>
  <Accordion title="Custom provider">
    OpenAI-compatible va Anthropic-compatible endpoint’lar bilan ishlaydi.

    Non-interactive flag’lar:
    - `--auth-choice custom-api-key`
    - `--custom-base-url`
    - `--custom-model-id`
    - `--custom-api-key` (ixtiyoriy; `CUSTOM_API_KEY` ga fallback qiladi)
    - `--custom-provider-id` (ixtiyoriy)
    - `--custom-compatibility <openai|anthropic>` (ixtiyoriy; standart `openai`)

  
</Accordion>
  <Accordion title="Skip">
    Autentifikatsiyani sozlanmagan holda qoldiradi.
  
</Accordion>
</AccordionGroup>

Model xulqi:

- Aniqlangan variantlardan standart modelni tanlang yoki provayder va modelni qo‘lda kiriting.
- Wizard model tekshiruvini o‘tkazadi va sozlangan model noma’lum bo‘lsa yoki auth yetishmasa, ogohlantiradi.

Kirish ma’lumotlari va profil yo‘llari:

- OAuth kirish ma’lumotlari: `~/.openclaw/credentials/oauth.json`
- Auth profillari (API kalitlar + OAuth): `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`

<Note>
Brauzersiz va serverlar uchun maslahat: OAuth’ni brauzerli mashinada yakunlang, so‘ng
`~/.openclaw/credentials/oauth.json` (yoki `$OPENCLAW_STATE_DIR/credentials/oauth.json`)
ni gateway xostiga ko‘chiring.
</Note>

## Chiqishlar va ichki tuzilma

`~/.openclaw/openclaw.json` dagi odatiy maydonlar:

- `agents.defaults.workspace`
- `agents.defaults.model` / `models.providers` (agar Minimax tanlansa)
- `gateway.*` (rejim, bind, auth, tailscale)
- `channels.telegram.botToken`, `channels.discord.token`, `channels.signal.*`, `channels.imessage.*`
- So‘rovlar davomida rozilik bersangiz, kanal allowlist’lari (Slack, Discord, Matrix, Microsoft Teams) (nomlar imkon bo‘lsa ID’larga moslashtiriladi)
- `skills.install.nodeManager`
- `wizard.lastRunAt`
- `wizard.lastRunVersion`
- `wizard.lastRunCommit`
- `wizard.lastRunCommand`
- `wizard.lastRunMode`

`openclaw agents add` `agents.list[]` va ixtiyoriy `bindings` ni yozadi.

WhatsApp kirish ma’lumotlari `~/.openclaw/credentials/whatsapp/<accountId>/` ostida saqlanadi.
Sessiyalar `~/.openclaw/agents/<agentId>/sessions/` ostida saqlanadi.

<Note>
Ba’zi kanallar plagin sifatida yetkazib beriladi. Onboarding jarayonida tanlanganda, wizard
kanal konfiguratsiyasidan oldin plaginni (npm yoki lokal yo‘l) o‘rnatishni taklif qiladi.
</Note>

Gateway wizard RPC:

- `wizard.start`
- `wizard.next`
- `wizard.cancel`
- `wizard.status`

Mijozlar (macOS ilovasi va Control UI) onboarding mantiqini qayta amalga oshirmasdan qadamlarni ko‘rsatishi mumkin.

Signal sozlash xatti-harakati:

- Mos reliz assetini yuklab oladi
- Uni `~/.openclaw/tools/signal-cli/<version>/` ostida saqlaydi
- Konfiguratsiyada `channels.signal.cliPath` ni yozadi
- JVM build’lar Java 21 ni talab qiladi
- Mavjud bo‘lganda native build’lar ishlatiladi
- Windows WSL2’dan foydalanadi va WSL ichida Linux signal-cli oqimiga amal qiladi

## Bog‘liq hujjatlar

- Onboarding hub: [Onboarding Wizard (CLI)](/start/wizard)
- Avtomatlashtirish va skriptlar: [CLI Automation](/start/wizard-cli-automation)
- Buyruqlar ma’lumotnomasi: [`openclaw onboard`](/cli/onboard)

