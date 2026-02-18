---
title: "عمومی سوالات"
---

# عمومی سوالات

حقیقی دنیا کے سیٹ اپس (local dev، VPS، multi-agent، OAuth/API keys، model failover) کے لیے فوری جوابات اور گہری ٹربل شوٹنگ۔ رن ٹائم ڈائیگناسٹکس کے لیے، [Troubleshooting](/gateway/troubleshooting) دیکھیں۔ مکمل کنفیگریشن ریفرنس کے لیے، [Configuration](/gateway/configuration) دیکھیں۔

## فہرستِ مضامین

- [فوری آغاز اور پہلی بار سیٹ اپ]
  - [میں پھنس گیا ہوں—سب سے تیز طریقہ کیا ہے؟](#im-stuck-whats-the-fastest-way-to-get-unstuck)
  - [OpenClaw کو انسٹال اور سیٹ اپ کرنے کا تجویز کردہ طریقہ کیا ہے؟](#whats-the-recommended-way-to-install-and-set-up-openclaw)
  - [آن بورڈنگ کے بعد ڈیش بورڈ کیسے کھولوں؟](#how-do-i-open-the-dashboard-after-onboarding)
  - [لوکل ہوسٹ بمقابلہ ریموٹ پر ڈیش بورڈ کی تصدیق (ٹوکن) کیسے کروں؟](#how-do-i-authenticate-the-dashboard-token-on-localhost-vs-remote)
  - [مجھے کون سا رن ٹائم درکار ہے؟](#what-runtime-do-i-need)
  - [کیا یہ Raspberry Pi پر چلتا ہے؟](#does-it-run-on-raspberry-pi)
  - [Raspberry Pi انسٹال کے لیے کوئی مشورے؟](#any-tips-for-raspberry-pi-installs)
  - [یہ "wake up my friend" پر اٹکا ہوا ہے / آن بورڈنگ شروع نہیں ہو رہی۔ اب کیا کریں؟](#it-is-stuck-on-wake-up-my-friend-onboarding-will-not-hatch-what-now)
  - [کیا میں آن بورڈنگ دوبارہ کیے بغیر اپنا سیٹ اپ نئی مشین (Mac mini) پر منتقل کر سکتا ہوں؟](#can-i-migrate-my-setup-to-a-new-machine-mac-mini-without-redoing-onboarding)
  - [تازہ ترین ورژن میں نیا کیا ہے، کہاں دیکھوں؟](#where-do-i-see-what-is-new-in-the-latest-version)
  - [میں docs.openclaw.ai تک رسائی حاصل نہیں کر پا رہا (SSL خرابی)۔ اب کیا کریں؟](#i-cant-access-docsopenclawai-ssl-error-what-now)
  - [stable اور beta میں کیا فرق ہے؟](#whats-the-difference-between-stable-and-beta)
  - [beta ورژن کیسے انسٹال کروں، اور beta اور dev میں کیا فرق ہے؟](#how-do-i-install-the-beta-version-and-whats-the-difference-between-beta-and-dev)
  - [میں تازہ ترین بِٹس کیسے آزماؤں؟](#how-do-i-try-the-latest-bits)
  - [انسٹال اور آن بورڈنگ میں عموماً کتنا وقت لگتا ہے؟](#how-long-does-install-and-onboarding-usually-take)
  - [انسٹالر اٹک گیا ہے؟ مجھے مزید فیڈبیک کیسے ملے گا؟](#installer-stuck-how-do-i-get-more-feedback)
  - [Windows انسٹال میں git نہیں ملا یا openclaw پہچانا نہیں گیا](#windows-install-says-git-not-found-or-openclaw-not-recognized)
  - [دستاویزات نے میرا سوال حل نہیں کیا—بہتر جواب کیسے حاصل کروں؟](#the-docs-didnt-answer-my-question-how-do-i-get-a-better-answer)
  - [Linux پر OpenClaw کیسے انسٹال کروں؟](#how-do-i-install-openclaw-on-linux)
  - [VPS پر OpenClaw کیسے انسٹال کروں؟](#how-do-i-install-openclaw-on-a-vps)
  - [کلاؤڈ/VPS انسٹال گائیڈز کہاں ہیں؟](#where-are-the-cloudvps-install-guides)
  - [کیا میں OpenClaw سے خود کو اپ ڈیٹ کروانے کو کہہ سکتا ہوں؟](#can-i-ask-openclaw-to-update-itself)
  - [آن بورڈنگ وزارڈ اصل میں کیا کرتا ہے؟](#what-does-the-onboarding-wizard-actually-do)
  - [کیا اسے چلانے کے لیے Claude یا OpenAI سبسکرپشن درکار ہے؟](#do-i-need-a-claude-or-openai-subscription-to-run-this)
  - [کیا میں API کلید کے بغیر Claude Max سبسکرپشن استعمال کر سکتا ہوں؟](#can-i-use-claude-max-subscription-without-an-api-key)
  - [Anthropic "setup-token" تصدیق کیسے کام کرتی ہے؟](#how-does-anthropic-setuptoken-auth-work)
  - [Anthropic setup-token کہاں ملتا ہے؟](#where-do-i-find-an-anthropic-setuptoken)
  - [کیا آپ Claude سبسکرپشن تصدیق (Claude Pro یا Max) کو سپورٹ کرتے ہیں؟](#do-you-support-claude-subscription-auth-claude-pro-or-max)
  - [Anthropic سے `HTTP 429: rate_limit_error` کیوں دکھ رہا ہے؟](#why-am-i-seeing-http-429-ratelimiterror-from-anthropic)
  - [کیا AWS Bedrock سپورٹڈ ہے؟](#is-aws-bedrock-supported)
  - [Codex تصدیق کیسے کام کرتی ہے؟](#how-does-codex-auth-work)
  - [کیا آپ OpenAI سبسکرپشن تصدیق (Codex OAuth) سپورٹ کرتے ہیں؟](#do-you-support-openai-subscription-auth-codex-oauth)
  - [Gemini CLI OAuth کیسے سیٹ اپ کروں؟](#how-do-i-set-up-gemini-cli-oauth)
  - [کیا عام چیٹس کے لیے لوکل ماڈل ٹھیک ہے؟](#is-a-local-model-ok-for-casual-chats)
  - [میں ہوسٹڈ ماڈل ٹریفک کو کسی مخصوص ریجن میں کیسے رکھوں؟](#how-do-i-keep-hosted-model-traffic-in-a-specific-region)
  - [کیا مجھے اسے انسٹال کرنے کے لیے Mac Mini خریدنا ہوگا؟](#do-i-have-to-buy-a-mac-mini-to-install-this)
  - [iMessage سپورٹ کے لیے Mac mini درکار ہے؟](#do-i-need-a-mac-mini-for-imessage-support)
  - [اگر میں OpenClaw چلانے کے لیے Mac mini خریدوں، تو کیا میں اسے اپنے MacBook Pro سے جوڑ سکتا ہوں؟](#if-i-buy-a-mac-mini-to-run-openclaw-can-i-connect-it-to-my-macbook-pro)
  - [کیا میں Bun استعمال کر سکتا ہوں؟](#can-i-use-bun)
  - [Telegram: `allowFrom` میں کیا ڈالنا ہے؟](#telegram-what-goes-in-allowfrom)
  - [کیا ایک WhatsApp نمبر کو مختلف OpenClaw انسٹینسز کے ساتھ کئی لوگ استعمال کر سکتے ہیں؟](#can-multiple-people-use-one-whatsapp-number-with-different-openclaw-instances)
  - [کیا میں "فاسٹ چیٹ" ایجنٹ اور "Opus برائے کوڈنگ" ایجنٹ چلا سکتا ہوں؟](#can-i-run-a-fast-chat-agent-and-an-opus-for-coding-agent)
  - [کیا Homebrew Linux پر کام کرتا ہے؟](#does-homebrew-work-on-linux)
  - [hackable (git) انسٹال اور npm انسٹال میں کیا فرق ہے؟](#whats-the-difference-between-the-hackable-git-install-and-npm-install)
  - [کیا میں بعد میں npm اور git انسٹال کے درمیان سوئچ کر سکتا ہوں؟](#can-i-switch-between-npm-and-git-installs-later)
  - [کیا مجھے Gateway لیپ ٹاپ پر چلانا چاہیے یا VPS پر؟](#should-i-run-the-gateway-on-my-laptop-or-a-vps)
  - [OpenClaw کو ڈیڈیکیٹڈ مشین پر چلانا کتنا اہم ہے؟](#how-important-is-it-to-run-openclaw-on-a-dedicated-machine)
  - [کم از کم VPS ضروریات اور تجویز کردہ OS کیا ہے؟](#what-are-the-minimum-vps-requirements-and-recommended-os)
  - [کیا میں OpenClaw کو VM میں چلا سکتا ہوں اور کیا ضروریات ہیں؟](#can-i-run-openclaw-in-a-vm-and-what-are-the-requirements)
- [OpenClaw کیا ہے؟](#what-is-openclaw)
  - [OpenClaw کیا ہے، ایک پیراگراف میں؟](#what-is-openclaw-in-one-paragraph)
  - [اس کی ویلیو پروپوزیشن کیا ہے؟](#whats-the-value-proposition)
  - [میں نے ابھی اسے سیٹ اپ کیا ہے، مجھے سب سے پہلے کیا کرنا چاہیے؟](#i-just-set-it-up-what-should-i-do-first)
  - [OpenClaw کے روزمرہ استعمال کے پانچ اہم ترین کیسز کون سے ہیں؟](#what-are-the-top-five-everyday-use-cases-for-openclaw)
  - [کیا OpenClaw کسی SaaS کے لیے لیڈ جنریشن، آؤٹ ریچ، اشتہارات اور بلاگز میں مدد کر سکتا ہے؟](#can-openclaw-help-with-lead-gen-outreach-ads-and-blogs-for-a-saas)
  - [ویب ڈویلپمنٹ کے لیے Claude Code کے مقابلے میں کیا فوائد ہیں؟](#what-are-the-advantages-vs-claude-code-for-web-development)
- [Skills and automation](#skills-and-automation)
- [Sandboxing and memory](#sandboxing-and-memory)
- [Where things live on disk](#where-things-live-on-disk)
- [Config basics](#config-basics)
- [Remote gateways and nodes](#remote-gateways-and-nodes)
- [Env vars and .env loading](#env-vars-and-env-loading)
- [Sessions and multiple chats](#sessions-and-multiple-chats)
- [Models: defaults, selection, aliases, switching](#models-defaults-selection-aliases-switching)
- [Model failover and "All models failed"](#model-failover-and-all-models-failed)
- [Auth profiles: what they are and how to manage them](#auth-profiles-what-they-are-and-how-to-manage-them)
- [Gateway: ports, "already running", and remote mode](#gateway-ports-already-running-and-remote-mode)
- [Logging and debugging](#logging-and-debugging)
- [Media and attachments](#media-and-attachments)
- [Security and access control](#security-and-access-control)
- [Chat commands, aborting tasks, and "it won't stop"](#chat-commands-aborting-tasks-and-it-wont-stop)

## اگر کچھ خراب ہو تو پہلے 60 سیکنڈز

1. **فوری اسٹیٹس (پہلا چیک)**

   ```bash
   openclaw status
   ```

   Fast local summary: OS + update, gateway/service reachability, agents/sessions, provider config + runtime issues (when gateway is reachable).

2. **چسپاں کیا جا سکنے والی رپورٹ (شیئر کرنے کے لیے محفوظ)**

   ```bash
   openclaw status --all
   ```

   Read-only diagnosis with log tail (tokens redacted).

3. **Daemon + port state**

   ```bash
   openclaw gateway status
   ```

   Shows supervisor runtime vs RPC reachability, the probe target URL, and which config the service likely used.

4. **Deep probes**

   ```bash
   openclaw status --deep
   ```

   Runs gateway health checks + provider probes (requires a reachable gateway). See [Health](/gateway/health).

5. **Tail the latest log**

   ```bash
   openclaw logs --follow
   ```

   If RPC is down, fall back to:

   ```bash
   tail -f "$(ls -t /tmp/openclaw/openclaw-*.log | head -1)"
   ```

   File logs are separate from service logs; see [Logging](/logging) and [Troubleshooting](/gateway/troubleshooting).

6. **Run the doctor (repairs)**

   ```bash
   openclaw doctor
   ```

   کنفیگ/اسٹیٹ کی مرمت یا مائیگریٹ کرتا ہے + ہیلتھ چیکس چلاتا ہے۔ See [Doctor](/gateway/doctor).

7. **Gateway snapshot**

   ```bash
   openclaw health --json
   openclaw health --verbose   # shows the target URL + config path on errors
   ```

   Asks the running gateway for a full snapshot (WS-only). See [Health](/gateway/health).

## فوری آغاز اور پہلی بار سیٹ اپ

### Im stuck whats the fastest way to get unstuck

اپنی مشین کو **دیکھ سکنے والا** ایک لوکل AI ایجنٹ استعمال کریں۔ یہ Discord میں پوچھنے سے کہیں زیادہ مؤثر ہے، کیونکہ زیادہ تر "میں پھنس گیا ہوں" کیسز **لوکل کنفیگ یا ماحول کے مسائل** ہوتے ہیں جنہیں ریموٹ مددگار براہِ راست نہیں دیکھ سکتے۔

- **Claude Code**: https://www.anthropic.com/claude-code/
- **OpenAI Codex**: https://openai.com/codex/

یہ ٹولز ریپو پڑھ سکتے ہیں، کمانڈز چلا سکتے ہیں، لاگز دیکھ سکتے ہیں، اور آپ کی مشین لیول سیٹ اپ (PATH، سروسز، پرمیشنز، auth فائلز) درست کرنے میں مدد کر سکتے ہیں۔ انہیں مکمل سورس چیک آؤٹ دیں بذریعہ hackable (git) انسٹال:

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method git
```

یہ OpenClaw کو **git checkout سے** انسٹال کرتا ہے، تاکہ ایجنٹ کوڈ + ڈاکس پڑھ سکے اور آپ کے چل رہے عین ورژن پر دلیل کر سکے۔ بعد میں آپ انسٹالر کو بغیر `--install-method git` کے دوبارہ چلا کر stable پر واپس جا سکتے ہیں۔

ٹِپ: ایجنٹ سے کہیں کہ وہ مرحلہ وار پلان بنائے اور نگرانی کرے، پھر صرف ضروری کمانڈز چلائے۔ اس سے تبدیلیاں کم اور آڈٹ کرنا آسان رہتا ہے۔

اگر آپ کو کوئی حقیقی بگ یا فکس ملے تو براہِ کرم GitHub پر issue فائل کریں یا PR بھیجیں:  
https://github.com/openclaw/openclaw/issues  
https://github.com/openclaw/openclaw/pulls

مدد مانگتے وقت ان کمانڈز کے آؤٹ پٹ شیئر کریں:

```bash
openclaw status
openclaw models status
openclaw doctor
```

یہ کیا کرتے ہیں:

- `openclaw status`: گیٹ وے/ایجنٹ صحت + بنیادی کنفیگ کا فوری اسنیپ شاٹ۔
- `openclaw models status`: پرووائیڈر auth + ماڈل دستیابی چیک کرتا ہے۔
- `openclaw doctor`: عام کنفیگ/اسٹیٹ مسائل کی توثیق اور مرمت کرتا ہے۔

دیگر مفید CLI چیکس: `openclaw status --all`, `openclaw logs --follow`, `openclaw gateway status`, `openclaw health --verbose`.

Quick debug loop: [First 60 seconds if something's broken](#اگر-کچھ-خراب-ہو-تو-پہلے-60-سیکنڈز).  
Install docs: [Install](/install), [Installer flags](/install/installer), [Updating](/install/updating).

---

## اسکرین شاٹ/چیٹ لاگ میں موجود عین سوال کا جواب دیں

**سوال:** "Anthropic کے لیے API key کے ساتھ ڈیفالٹ ماڈل کون سا ہے؟"

**جواب:** OpenClaw میں، اسناد (credentials) اور ماڈل کا انتخاب الگ ہوتے ہیں۔ `ANTHROPIC_API_KEY` سیٹ کرنا (یا Anthropic API key کو auth profiles میں محفوظ کرنا) توثیق کو فعال کرتا ہے، لیکن اصل ڈیفالٹ ماڈل وہی ہوتا ہے جو آپ `agents.defaults.model.primary` میں کنفیگر کرتے ہیں (مثال کے طور پر `anthropic/claude-sonnet-4-5` یا `anthropic/claude-opus-4-6`)۔ اگر آپ کو `No credentials found for profile "anthropic:default"` نظر آئے، تو اس کا مطلب ہے کہ گیٹ وے کو چلنے والے ایجنٹ کے لیے متوقع `auth-profiles.json` میں Anthropic کی اسناد نہیں مل سکیں۔

---

اب بھی پھنسے ہوئے ہیں؟ [Discord](https://discord.com/invite/clawd) میں پوچھیں یا [GitHub discussion](https://github.com/openclaw/openclaw/discussions) کھولیں۔