---
summary: "Gateway سروس، لائف سائیکل، اور آپریشنز کے لیے رن بُک"
read_when:
  - Gateway پروسیس کو چلانے یا ڈیبگ کرنے کے دوران
title: "Gateway رن بُک"
---

# Gateway سروس رن بُک

آخری تازہ کاری: 2025-12-09

<CardGroup cols={2}>
  <Card title="Deep troubleshooting" icon="siren" href="/gateway/troubleshooting">
    علامت کی بنیاد پر تشخیص، درست کمانڈ مراحل اور لاگ سِگنیچرز کے ساتھ۔
  
</Card>
  <Card title="Configuration" icon="sliders" href="/gateway/configuration">
    ٹاسک پر مبنی سیٹ اپ گائیڈ + مکمل کنفیگریشن ریفرنس۔
  
</Card>
</CardGroup>

## 5 منٹ میں مقامی آغاز

<Steps>
  <Step title="Start the Gateway">

```bash
openclaw gateway --port 18789
# for full debug/trace logs in stdio:
openclaw gateway --port 18789 --verbose
# if the port is busy, terminate listeners then start:
openclaw gateway --force
# dev loop (auto-reload on TS changes):
pnpm gateway:watch
```

  
</Step>

  <Step title="Verify service health">

```bash
openclaw gateway status
openclaw status
openclaw logs --follow
```

صحتمند بنیادی حالت: `Runtime: running` اور `RPC probe: ok`۔

  
</Step>

  <Step title="Validate channel readiness">

```bash
openclaw channels status --probe
```

  
</Step>
</Steps>

<Note>
Gateway کنفیگ ری لوڈ فعال کنفیگ فائل کے راستے کو مانیٹر کرتا ہے (جو پروفائل/اسٹیٹ ڈیفالٹس سے حل کیا جاتا ہے، یا جب `OPENCLAW_CONFIG_PATH` سیٹ ہو)۔
ڈیفالٹ موڈ `gateway.reload.mode="hybrid"` ہے۔
</Note>

## رن ٹائم ماڈل

- روٹنگ، کنٹرول پلین اور چینل کنکشنز کے لیے ایک ہمیشہ فعال پراسیس۔
- درج ذیل کے لیے ایک سنگل ملٹی پلیکسڈ پورٹ:
  - WebSocket کنٹرول/RPC
  - HTTP APIs (OpenAI-compatible، Responses، tools invoke)
  - کنٹرول UI اور ہُکس
- ڈیفالٹ بائنڈ موڈ: `loopback`۔
- ڈیفالٹ کے طور پر Auth لازمی ہے (`gateway.auth.token` / `gateway.auth.password`، یا `OPENCLAW_GATEWAY_TOKEN` / `OPENCLAW_GATEWAY_PASSWORD`)۔

### ڈیو پروفائل (`--dev`)

| سیٹنگ        | حل کی ترتیب                                                   |
| ------------ | ------------------------------------------------------------- |
| Gateway پورٹ | `--port` → `OPENCLAW_GATEWAY_PORT` → `gateway.port` → `18789` |
| بائنڈ موڈ    | CLI/override → `gateway.bind` → `loopback`                    |

### ہاٹ ری لوڈ موڈز

| `gateway.reload.mode`                | رویہ                                                                 |
| ------------------------------------ | -------------------------------------------------------------------- |
| `off`                                | کوئی کنفیگ ری لوڈ نہیں                                               |
| `hot`                                | صرف ہاٹ-سیف تبدیلیاں لاگو کریں                                       |
| `restart`                            | ری لوڈ درکار تبدیلیوں پر ری اسٹارٹ کریں                              |
| `hybrid` (ڈیفالٹ) | جہاں محفوظ ہو وہاں ہاٹ اپلائی کریں، اور جہاں ضروری ہو ری اسٹارٹ کریں |

## آپریٹر کمانڈ سیٹ

```bash
openclaw gateway status
openclaw gateway status --deep
openclaw gateway status --json
openclaw gateway install
openclaw gateway restart
openclaw gateway stop
openclaw logs --follow
openclaw doctor
```

## ریموٹ رسائی

ترجیحی: Tailscale/VPN۔
متبادل طریقہ: SSH ٹنل۔

```bash
ssh -N -L 18789:127.0.0.1:18789 user@host
```

ہر پروفائل کے لیے سروس انسٹال:

<Warning>
اگر gateway auth کنفیگر کیا گیا ہو تو کلائنٹس کو SSH ٹنلز کے ذریعے بھی auth (`token`/`password`) بھیجنا لازمی ہے۔
</Warning>

مثال:

## نگرانی اور سروس لائف سائیکل

پروڈکشن جیسی قابلِ اعتماد کارکردگی کے لیے supervised رنز استعمال کریں۔

<Tabs>
  <Tab title="macOS (launchd)">

```bash
openclaw gateway install
openclaw gateway status
openclaw gateway restart
openclaw gateway stop
```

LaunchAgent لیبلز `ai.openclaw.gateway` (ڈیفالٹ) یا `ai.openclaw.<profile>` ہوتے ہیں`(نامزد پروفائل)۔`openclaw doctor\` سروس کنفیگریشن میں تبدیلیوں (config drift) کا جائزہ لے کر انہیں درست کرتا ہے۔

  
</Tab>

  <Tab title="Linux (systemd user)">

```bash
openclaw gateway install
systemctl --user enable --now openclaw-gateway[-<profile>].service
openclaw gateway status
```

لاگ آؤٹ کے بعد بھی برقرار رکھنے کے لیے lingering فعال کریں:

```bash
sudo loginctl enable-linger <user>
```

  
</Tab>

  <Tab title="Linux (system service)">

متعدد صارفین یا ہمیشہ آن رہنے والے ہوسٹس کے لیے system unit استعمال کریں۔

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now openclaw-gateway[-<profile>].service
```

  
</Tab>
</Tabs>

## ایک ہی ہوسٹ پر متعدد gateways

زیادہ تر سیٹ اپس میں **ایک** Gateway چلانا چاہیے۔
متعدد صرف سخت علیحدگی یا redundancy کے لیے استعمال کریں (مثلاً ریسکیو پروفائل)۔

ہر انسٹینس کے لیے چیک لسٹ:

- منفرد `gateway.port`
- منفرد `OPENCLAW_CONFIG_PATH`
- منفرد `OPENCLAW_STATE_DIR`
- منفرد `agents.defaults.workspace`

مثال:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json OPENCLAW_STATE_DIR=~/.openclaw-a openclaw gateway --port 19001
OPENCLAW_CONFIG_PATH=~/.openclaw/b.json OPENCLAW_STATE_DIR=~/.openclaw-b openclaw gateway --port 19002
```

دیکھیں: [Multiple gateways](/gateway/multiple-gateways)।

### Gateway سروس مینجمنٹ (CLI)

```bash
openclaw --dev setup
openclaw --dev gateway --allow-unconfigured
openclaw --dev status
```

ڈیفالٹس میں علیحدہ state/config اور بنیادی gateway پورٹ `19001` شامل ہیں۔

## پروٹوکول فوری حوالہ (آپریٹر کا نقطۂ نظر)

- `gateway status` بطورِ طے شدہ سروس کے ریزولوڈ پورٹ/کنفیگ کا استعمال کرتے ہوئے Gateway RPC کو پروب کرتا ہے ( `--url` کے ساتھ اوور رائیڈ کریں)۔
- `gateway status --deep` سسٹم-لیول اسکینز (LaunchDaemons/system units) شامل کرتا ہے۔
- `gateway status --no-probe` RPC پروب چھوڑ دیتا ہے (جب نیٹ ورکنگ ڈاؤن ہو تو مفید)۔
- `gateway status --json` اسکرپٹس کے لیے مستحکم ہے۔

بنڈلڈ mac ایپ:

1. فوری طور پر قبول شدہ تصدیق (`status:"accepted"`)
2. اسے صاف طور پر روکنے کے لیے `openclaw gateway stop` استعمال کریں (یا `launchctl bootout gui/$UID/bot.molt.gateway`)۔

مکمل پروٹوکول دستاویزات دیکھیں: [Gateway Protocol](/gateway/protocol)।

## آپریشنل چیکس

### فعالیت (Liveness)

- WS کھولیں اور `connect` بھیجیں۔
- snapshot کے ساتھ `hello-ok` جواب کی توقع کریں۔

### تیاری (Readiness)

```bash
sudo loginctl enable-linger youruser
```

### گیپ ریکوری

Events are not replayed. sequence میں خلاء کی صورت میں جاری رکھنے سے پہلے state (`health`, `system-presence`) ریفریش کریں۔

## عام خرابی کی نشانیاں

| نشانی                                                          | ممکنہ مسئلہ                                       |
| -------------------------------------------------------------- | ------------------------------------------------- |
| `refusing to bind gateway ... without auth`                    | token/password کے بغیر non-loopback بائنڈ         |
| `another gateway instance is already listening` / `EADDRINUSE` | پورٹ کا تصادم                                     |
| `Gateway start blocked: set gateway.mode=local`                | کنفیگ کو ریموٹ موڈ پر سیٹ کر دیا گیا ہے           |
| کنیکٹ کے دوران `unauthorized`                                  | کلائنٹ اور gateway کے درمیان توثیق میں عدم مطابقت |

مکمل تشخیصی مراحل کے لیے، [Gateway Troubleshooting](/gateway/troubleshooting) استعمال کریں۔

## حفاظتی ضمانتیں

- جب Gateway دستیاب نہ ہو تو Gateway پروٹوکول کلائنٹس فوری طور پر ناکام ہو جاتے ہیں (کوئی خودکار direct-channel fallback نہیں ہوتا)۔
- غیر معتبر یا non-connect ابتدائی فریمز مسترد کر کے کنکشن بند کر دیا جاتا ہے۔
- مناسب بندش کے دوران socket بند ہونے سے پہلے `shutdown` ایونٹ جاری کیا جاتا ہے۔

---

متعلقہ:

- بطورِ طے شدہ فی ہوسٹ ایک Gateway فرض کریں؛ اگر متعدد پروفائلز چلائیں تو پورٹس/اسٹیٹ الگ رکھیں اور درست انسٹینس کو ہدف بنائیں۔
- براہِ راست Baileys کنکشنز پر کوئی فال بیک نہیں؛ اگر Gateway ڈاؤن ہو تو بھیجنا فوراً ناکام ہو جاتا ہے۔
- نان-کنیکٹ ابتدائی فریمز یا خراب JSON مسترد کر دیے جاتے ہیں اور ساکٹ بند کر دی جاتی ہے۔
- گِریس فل شٹ ڈاؤن: بند کرنے سے پہلے `shutdown` ایونٹ ایمٹ کریں؛ کلائنٹس کو کلوز + ری کنیکٹ ہینڈل کرنا چاہیے۔
- [Doctor](/gateway/doctor)
- [Authentication](/gateway/authentication)
