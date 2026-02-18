---
title: "CLI آن بورڈنگ حوالہ"
sidebarTitle: "CLI حوالہ"
---

# CLI آن بورڈنگ حوالہ

یہ صفحہ `openclaw onboard` کے لیے مکمل حوالہ ہے۔
مختصر رہنما کے لیے، دیکھیں [Onboarding Wizard (CLI)](/start/wizard).

## وزارڈ کیا کرتا ہے

لوکل موڈ (بطورِ طے شدہ) آپ کو درج ذیل مراحل سے گزارتا ہے:

- ماڈل اور تصدیق سیٹ اپ (OpenAI Code سبسکرپشن OAuth، Anthropic API کلید یا سیٹ اپ ٹوکن، نیز MiniMax، GLM، Moonshot، اور AI Gateway کے اختیارات)
- ورک اسپیس کا مقام اور بوٹ اسٹرَیپ فائلیں
- Gateway سیٹنگز (پورٹ، بائنڈ، تصدیق، tailscale)
- چینلز اور فراہم کنندگان (Telegram، WhatsApp، Discord، Google Chat، Mattermost پلگ اِن، Signal)
- ڈیمَن انسٹال (LaunchAgent یا systemd یوزر یونٹ)
- ہیلتھ چیک
- Skills سیٹ اپ

ریموٹ موڈ اس مشین کو کہیں اور موجود gateway سے منسلک کرنے کے لیے کنفیگر کرتا ہے۔
یہ ریموٹ ہوسٹ پر کچھ بھی انسٹال یا ترمیم نہیں کرتا۔

## لوکل فلو کی تفصیلات

<Steps>
  <Step title="Existing config detection">
    - اگر `~/.openclaw/openclaw.json` موجود ہو تو Keep، Modify، یا Reset کا انتخاب کریں۔
    - وزرڈ کو دوبارہ چلانے سے کچھ بھی حذف نہیں ہوتا جب تک کہ آپ واضح طور پر Reset منتخب نہ کریں (یا `--reset` پاس نہ کریں)۔
    - اگر کنفیگ غلط ہو یا اس میں پرانی keys شامل ہوں تو وزرڈ رک جاتا ہے اور آپ سے کہتا ہے کہ آگے بڑھنے سے پہلے `openclaw doctor` چلائیں۔
    - Reset `trash` استعمال کرتا ہے اور درج ذیل اسکوپس پیش کرتا ہے:
      - صرف کنفیگ
      - کنفیگ + اسناد + سیشنز
      - مکمل Reset (ورک اسپیس بھی ہٹا دیتا ہے)
  </Step>
  <Step title="Model and auth">
    - مکمل اختیارات کی فہرست [Auth and model options](#auth-and-model-options) میں ہے۔
  </Step>
  <Step title="Workspace">
    - ڈیفالٹ `~/.openclaw/workspace` (قابلِ ترتیب)۔
    - پہلی بار چلانے کے bootstrap ritual کے لیے درکار ورک اسپیس فائلیں تیار کرتا ہے۔
    - ورک اسپیس لے آؤٹ: [Agent workspace](/concepts/agent-workspace).
  </Step>
  <Step title="Gateway">
    - پورٹ، بائنڈ، auth موڈ، اور tailscale ایکسپوژر کے لیے پرامپٹ کرتا ہے۔
    - تجویز کردہ: لوپ بیک کے لیے بھی token auth فعال رکھیں تاکہ لوکل WS کلائنٹس کو تصدیق کرنی پڑے۔
    - auth صرف اسی صورت غیر فعال کریں جب آپ ہر لوکل پراسیس پر مکمل اعتماد رکھتے ہوں۔
    - نان-لوپ بیک بائنڈز میں بھی auth درکار ہوتا ہے۔
  </Step>
  <Step title="Channels">
    - [WhatsApp](/channels/whatsapp): اختیاری QR لاگ اِن
    - [Telegram](/channels/telegram): بوٹ ٹوکن
    - [Discord](/channels/discord): بوٹ ٹوکن
    - [Google Chat](/channels/googlechat): سروس اکاؤنٹ JSON + ویب ہُک آڈیئنس
    - [Mattermost](/channels/mattermost) پلگ اِن: بوٹ ٹوکن + بیس URL
    - [Signal](/channels/signal): اختیاری `signal-cli` انسٹال + اکاؤنٹ کنفیگ
    - [BlueBubbles](/channels/bluebubbles): iMessage کے لیے تجویز کردہ؛ سرور URL + پاس ورڈ + ویب ہُک
    - [iMessage](/channels/imessage): لیگیسی `imsg` CLI پاتھ + DB رسائی
    - DM سیکیورٹی: بطورِ طے شدہ pairing۔ پہلا DM ایک کوڈ بھیجتا ہے؛ منظوری دیں
      `openclaw pairing approve <channel> <code>` کے ذریعے یا allowlists استعمال کریں۔
  </Step>
  <Step title="Daemon install">
    - macOS: LaunchAgent
      - لاگ اِن شدہ یوزر سیشن درکار ہے؛ ہیڈلیس کے لیے کسٹم LaunchDaemon استعمال کریں (فراہم نہیں کیا جاتا)۔
    - Linux اور Windows بذریعہ WSL2: systemd یوزر یونٹ
      - وزرڈ `loginctl enable-linger <user>` کی کوشش کرتا ہے تاکہ لاگ آؤٹ کے بعد بھی gateway چلتا رہے۔
      - sudo کے لیے پرامپٹ آ سکتا ہے ( `/var/lib/systemd/linger` لکھتا ہے )؛ پہلے بغیر sudo کوشش کرتا ہے۔
    - رن ٹائم انتخاب: Node (تجویز کردہ؛ WhatsApp اور Telegram کے لیے درکار)۔ Bun تجویز نہیں کیا جاتا۔
  </Step>
  <Step title="Health check">
    - gateway شروع کرتا ہے (اگر ضرورت ہو) اور `openclaw health` چلاتا ہے۔
    - `openclaw status --deep` اسٹیٹس آؤٹ پٹ میں gateway ہیلتھ پروبز شامل کرتا ہے۔
  </Step>
  <Step title="Skills">
    - دستیاب skills پڑھتا ہے اور تقاضے چیک کرتا ہے۔
    - node manager منتخب کرنے دیتا ہے: npm یا pnpm (bun تجویز نہیں کیا جاتا)۔
    - اختیاری dependencies انسٹال کرتا ہے (کچھ macOS پر Homebrew استعمال کرتی ہیں)۔
  </Step>
  <Step title="Finish">
    - خلاصہ اور اگلے مراحل، بشمول iOS، Android، اور macOS ایپ کے اختیارات۔
  </Step>
</Steps>

<Note>
اگر کوئی GUI دریافت نہ ہو تو وزرڈ براؤزر کھولنے کے بجائے Control UI کے لیے SSH پورٹ-فارورڈ ہدایات پرنٹ کرتا ہے۔
اگر Control UI کے assets موجود نہ ہوں تو وزرڈ انہیں build کرنے کی کوشش کرتا ہے؛ متبادل `pnpm ui:build` ہے (UI deps خودکار انسٹال کرتا ہے)۔
</Note>

## ریموٹ موڈ کی تفصیلات

ریموٹ موڈ اس مشین کو کہیں اور موجود gateway سے منسلک کرنے کے لیے کنفیگر کرتا ہے۔

<Info>
ریموٹ موڈ ریموٹ ہوسٹ پر کچھ بھی انسٹال یا ترمیم نہیں کرتا۔
</Info>

جو آپ سیٹ کرتے ہیں:

- ریموٹ gateway URL (`ws://...`)
- اگر ریموٹ gateway تصدیق درکار کرے تو ٹوکن (سفارش کردہ)

<Note>
- اگر gateway صرف loopback ہو تو SSH tunneling یا tailnet استعمال کریں۔
- دریافت کے اشارے:
  - macOS: Bonjour (`dns-sd`)
  - Linux: Avahi (`avahi-browse`)
</Note>

## تصدیق اور ماڈل کے اختیارات

<AccordionGroup>
  <Accordion title="Anthropic API key (recommended)">
    اگر موجود ہو تو `ANTHROPIC_API_KEY` استعمال کرتا ہے یا کلید کے لیے پرامپٹ کرتا ہے، پھر ڈیمَن کے استعمال کے لیے محفوظ کرتا ہے۔
  </Accordion>
  <Accordion title="Anthropic OAuth (Claude Code CLI)">
    - macOS: Keychain آئٹم "Claude Code-credentials" چیک کرتا ہے
    - Linux اور Windows: اگر موجود ہو تو `~/.claude/.credentials.json` دوبارہ استعمال کرتا ہے

    macOS پر "Always Allow" منتخب کریں تاکہ launchd اسٹارٹس بلاک نہ ہوں۔
  </Accordion>
  <Accordion title="Anthropic token (setup-token paste)">
    کسی بھی مشین پر `claude setup-token` چلائیں، پھر ٹوکن پیسٹ کریں۔
    آپ اسے نام دے سکتے ہیں؛ خالی چھوڑنے پر ڈیفالٹ استعمال ہوگا۔
  </Accordion>
  <Accordion title="OpenAI Code subscription (Codex CLI reuse)">
    اگر `~/.codex/auth.json` موجود ہو تو وزرڈ اسے دوبارہ استعمال کر سکتا ہے۔
  </Accordion>
  <Accordion title="OpenAI Code subscription (OAuth)">
    براؤزر فلو؛ `code#state` پیسٹ کریں۔

    جب ماڈل غیر سیٹ ہو یا `openai/*` ہو تو `agents.defaults.model` کو `openai-codex/gpt-5.3-codex` پر سیٹ کرتا ہے۔
  </Accordion>
  <Accordion title="OpenAI API key">
    اگر موجود ہو تو `OPENAI_API_KEY` استعمال کرتا ہے یا کلید کے لیے پرامپٹ کرتا ہے، پھر اسے
    `~/.openclaw/.env` میں محفوظ کرتا ہے تاکہ launchd اسے پڑھ سکے۔

    جب ماڈل غیر سیٹ ہو، `openai/*` ہو، یا `openai-codex/*` ہو تو `agents.defaults.model` کو `openai/gpt-5.1-codex` پر سیٹ کرتا ہے۔
  </Accordion>
  <Accordion title="xAI (Grok) API key">
    `XAI_API_KEY` کے لیے پرامپٹ کرتا ہے اور xAI کو ماڈل فراہم کنندہ کے طور پر کنفیگر کرتا ہے۔
  </Accordion>
  <Accordion title="OpenCode Zen">
    `OPENCODE_API_KEY` (یا `OPENCODE_ZEN_API_KEY`) کے لیے پرامپٹ کرتا ہے۔
    سیٹ اپ URL: [opencode.ai/auth](https://opencode.ai/auth).
  </Accordion>
  <Accordion title="API key (generic)">
    کلید آپ کے لیے محفوظ کرتا ہے۔
  </Accordion>
  <Accordion title="Vercel AI Gateway">
    `AI_GATEWAY_API_KEY` کے لیے پرامپٹ کرتا ہے۔
    مزید تفصیل: [Vercel AI Gateway](/providers/vercel-ai-gateway).
  </Accordion>
  <Accordion title="Cloudflare AI Gateway">
    اکاؤنٹ ID، gateway ID، اور `CLOUDFLARE_AI_GATEWAY_API_KEY` کے لیے پرامپٹ کرتا ہے۔
    مزید تفصیل: [Cloudflare AI Gateway](/providers/cloudflare-ai-gateway).
  </Accordion>
  <Accordion title="MiniMax M2.1">
    کنفیگ خودکار طور پر لکھا جاتا ہے۔
    مزید تفصیل: [MiniMax](/providers/minimax).
  </Accordion>
  <Accordion title="Synthetic (Anthropic-compatible)">
    `SYNTHETIC_API_KEY` کے لیے پرامپٹ کرتا ہے۔
    مزید تفصیل: [Synthetic](/providers/synthetic).
  </Accordion>
  <Accordion title="Moonshot and Kimi Coding">
    Moonshot (Kimi K2) اور Kimi Coding کی کنفیگز خودکار طور پر لکھی جاتی ہیں۔
    مزید تفصیل: [Moonshot AI (Kimi + Kimi Coding)](/providers/moonshot).
  </Accordion>
  <Accordion title="Custom provider">
    OpenAI-compatible اور Anthropic-compatible endpoints کے ساتھ کام کرتا ہے۔

    Non-interactive flags:
    - `--auth-choice custom-api-key`
    - `--custom-base-url`
    - `--custom-model-id`
    - `--custom-api-key` (اختیاری؛ بصورت دیگر `CUSTOM_API_KEY` استعمال ہوگا)
    - `--custom-provider-id` (اختیاری)
    - `--custom-compatibility <openai|anthropic>` (اختیاری؛ ڈیفالٹ `openai`)
  </Accordion>
  <Accordion title="Skip">
    تصدیق کو غیر کنفیگرڈ چھوڑ دیتا ہے۔
  </Accordion>
</AccordionGroup>

ماڈل کا رویّہ:

- شناخت شدہ اختیارات میں سے ڈیفالٹ ماڈل منتخب کریں، یا فراہم کنندہ اور ماڈل دستی طور پر درج کریں۔
- وزارڈ ماڈل چیک چلاتا ہے اور اگر کنفیگرڈ ماڈل نامعلوم ہو یا تصدیق غائب ہو تو انتباہ دیتا ہے۔

اسناد اور پروفائل پاتھس:

- OAuth اسناد: `~/.openclaw/credentials/oauth.json`
- تصدیقی پروفائلز (API کلیدیں + OAuth): `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`

<Note>
ہیڈلیس اور سرور ٹِپ: براؤزر والی مشین پر OAuth مکمل کریں، پھر
`~/.openclaw/credentials/oauth.json` (یا `$OPENCLAW_STATE_DIR/credentials/oauth.json`)
کو gateway ہوسٹ پر کاپی کریں۔
</Note>

## آؤٹ پٹس اور اندرونی پہلو

`~/.openclaw/openclaw.json` میں عام فیلڈز:

- `agents.defaults.workspace`
- `agents.defaults.model` / `models.providers` (اگر Minimax منتخب کیا گیا ہو)
- `gateway.*` (موڈ، بائنڈ، تصدیق، tailscale)
- `channels.telegram.botToken`, `channels.discord.token`, `channels.signal.*`, `channels.imessage.*`
- چینل allowlists (Slack، Discord، Matrix، Microsoft Teams) جب آپ پرامپٹس کے دوران آپٹ اِن کریں (نام جہاں ممکن ہو IDs میں ریزولو ہو جاتے ہیں)
- `skills.install.nodeManager`
- `wizard.lastRunAt`
- `wizard.lastRunVersion`
- `wizard.lastRunCommit`
- `wizard.lastRunCommand`
- `wizard.lastRunMode`

`openclaw agents add`، `agents.list[]` اور اختیاری `bindings` لکھتا ہے۔

WhatsApp credentials `~/.openclaw/credentials/whatsapp/<accountId>/` کے تحت محفوظ ہوتے ہیں۔
Sessions `~/.openclaw/agents/<agentId>/sessions/` کے تحت محفوظ ہوتے ہیں۔

<Note>
کچھ چینلز پلگ اِنز کے طور پر فراہم کیے جاتے ہیں۔ آن بورڈنگ کے دوران منتخب کرنے پر، وزرڈ
چینل کنفیگریشن سے پہلے پلگ اِن (npm یا لوکل پاتھ) انسٹال کرنے کے لیے پرامپٹ کرتا ہے۔
</Note>

Gateway وزارڈ RPC:

- `wizard.start`
- `wizard.next`
- `wizard.cancel`
- `wizard.status`

کلائنٹس (macOS ایپ اور Control UI) آن بورڈنگ لاجک دوبارہ نافذ کیے بغیر مراحل رینڈر کر سکتے ہیں۔

Signal سیٹ اپ کا رویّہ:

- مناسب ریلیز اثاثہ ڈاؤن لوڈ کرتا ہے
- اسے `~/.openclaw/tools/signal-cli/<version>/` کے تحت محفوظ کرتا ہے
- کنفیگ میں `channels.signal.cliPath` لکھتا ہے
- JVM بلڈز کے لیے Java 21 درکار ہے
- جہاں دستیاب ہوں، نیٹو بلڈز استعمال کیے جاتے ہیں
- Windows WSL2 استعمال کرتا ہے اور WSL کے اندر Linux signal-cli فلو کی پیروی کرتا ہے

## متعلقہ دستاویزات

- آن بورڈنگ ہب: [Onboarding Wizard (CLI)](/start/wizard)
- آٹومیشن اور اسکرپٹس: [CLI Automation](/start/wizard-cli-automation)
- کمانڈ حوالہ: [`openclaw onboard`](/cli/onboard)
