---
summary: "تفصیلی جائزہ: سیشن اسٹور + ٹرانسکرپٹس، لائف سائیکل، اور (خودکار) کمپیکشن کے اندرونی پہلو"
read_when:
  - آپ کو سیشن آئی ڈیز، transcript JSONL، یا sessions.json فیلڈز کی ڈیبگنگ کرنی ہو
  - آپ خودکار کمپیکشن کے رویے میں تبدیلی کر رہے ہوں یا “پری-کمپیکشن” ہاؤس کیپنگ شامل کر رہے ہوں
  - آپ میموری فلشز یا خاموش سسٹم ٹرنز نافذ کرنا چاہتے ہوں
title: "سیشن مینجمنٹ کی تفصیلی رہنمائی"
---

# سیشن مینجمنٹ اور کمپیکشن (تفصیلی جائزہ)

یہ دستاویز وضاحت کرتی ہے کہ OpenClaw سیشنز کو ابتدا سے انتہا تک کیسے منظم کرتا ہے:

- **Session routing** (ان باؤنڈ پیغامات کیسے `sessionKey` سے میپ ہوتے ہیں)
- **Session store** (`sessions.json`) اور یہ کیا ٹریک کرتا ہے
- **Transcript persistence** (`*.jsonl`) اور اس کی ساخت
- **Transcript hygiene** (رنز سے پہلے فراہم کنندہ کے لحاظ سے درستگیاں)
- **Context limits** (کانٹیکسٹ ونڈو بمقابلہ ٹریک کیے گئے ٹوکنز)
- **Compaction** (دستی + خودکار کمپیکشن) اور پری-کمپیکشن کام کہاں جوڑا جائے
- **Silent housekeeping** (مثلاً میموری لکھائیاں جو صارف کو نظر آنے والی آؤٹ پٹ پیدا نہ کریں)

اگر آپ پہلے اعلیٰ سطحی جائزہ چاہتے ہیں تو یہاں سے شروع کریں:

- [/concepts/session](/concepts/session)
- [/concepts/compaction](/concepts/compaction)
- [/concepts/session-pruning](/concepts/session-pruning)
- [/reference/transcript-hygiene](/reference/transcript-hygiene)

---

## بنیادی ماخذ: Gateway

OpenClaw کو ایک واحد **Gateway process** کے گرد ڈیزائن کیا گیا ہے جو سیشن اسٹیٹ کا مالک ہوتا ہے۔

- UIs (macOS ایپ، ویب Control UI، TUI) کو سیشن فہرستیں اور ٹوکن گنتیاں Gateway سے کوئری کرنی چاہئیں۔
- ریموٹ موڈ میں، سیشن فائلیں ریموٹ ہوسٹ پر ہوتی ہیں؛ “اپنی لوکل Mac فائلیں چیک کرنا” اس بات کی عکاسی نہیں کرے گا کہ Gateway کیا استعمال کر رہا ہے۔

---

## پائیداری کی دو سطحیں

OpenClaw سیشنز کو دو سطحوں میں محفوظ کرتا ہے:

1. **سیشن اسٹور (`sessions.json`)**
   - Key/value میپ: `sessionKey -> SessionEntry`
   - چھوٹا، قابلِ تغیر، ترمیم (یا اندراجات حذف) کے لیے محفوظ
   - سیشن میٹاڈیٹا ٹریک کرتا ہے (موجودہ سیشن آئی ڈی، آخری سرگرمی، ٹوگلز، ٹوکن کاؤنٹرز وغیرہ)

2. **Transcript (`<sessionId>.jsonl`)**
   - ضمیمہ-صرف ٹرانسکرپٹ جس میں درختی ساخت ہوتی ہے (اندراجات میں `id` + `parentId`)
   - اصل گفتگو + ٹول کالز + کمپیکشن خلاصے محفوظ کرتا ہے
   - آئندہ ٹرنز کے لیے ماڈل کانٹیکسٹ دوبارہ بنانے میں استعمال ہوتا ہے

---

## On-disk locations

ہر ایجنٹ کے لیے، گیٹ وے ہوسٹ پر:

- Store: `~/.openclaw/agents/<agentId>/sessions/sessions.json`
- Transcripts: `~/.openclaw/agents/<agentId>/sessions/<sessionId>.jsonl`
  - Telegram موضوعی سیشنز: `.../<sessionId>-topic-<threadId>.jsonl`

OpenClaw ان کو `src/config/sessions.ts` کے ذریعے ریزولو کرتا ہے۔

---

## Session keys (`sessionKey`)

ایک `sessionKey` اس بات کی شناخت کرتا ہے کہ آپ _کس گفتگو کے بکٹ_ میں ہیں (روٹنگ + آئسولیشن)۔

عام پیٹرنز:

- مین/براہِ راست چیٹ (ہر ایجنٹ کے لیے): `agent:<agentId>:<mainKey>` (ڈیفالٹ `main`)
- گروپ: `agent:<agentId>:<channel>:group:<id>`
- روم/چینل (Discord/Slack): `agent:<agentId>:<channel>:channel:<id>` یا `...:room:<id>`
- Cron: `cron:<job.id>`
- Webhook: `hook:<uuid>` (جب تک اوور رائیڈ نہ کیا جائے)

اصلی قواعد [/concepts/session](/concepts/session) میں دستاویزی ہیں۔

---

## Session ids (`sessionId`)

ہر `sessionKey` ایک موجودہ `sessionId` کی طرف اشارہ کرتا ہے (وہ ٹرانسکرپٹ فائل جو گفتگو کو جاری رکھتی ہے)۔

عمومی اصول:

- **Reset** (`/new`, `/reset`) اس `sessionKey` کے لیے نیا `sessionId` بناتا ہے۔
- **Daily reset** (ڈیفالٹ گیٹ وے ہوسٹ کے مقامی وقت کے مطابق صبح 4:00 بجے) ری سیٹ باؤنڈری کے بعد آنے والے اگلے پیغام پر نیا `sessionId` بناتا ہے۔
- **Idle expiry** (`session.reset.idleMinutes` or legacy `session.idleMinutes`) creates a new `sessionId` when a message arrives after the idle window. When daily + idle are both configured, whichever expires first wins.

نفاذی تفصیل: یہ فیصلہ `src/auto-reply/reply/session.ts` میں `initSessionState()` کے اندر ہوتا ہے۔

---

## Session store schema (`sessions.json`)

اسٹور کی ویلیو ٹائپ `src/config/sessions.ts` میں `SessionEntry` ہے۔

اہم فیلڈز (مکمل فہرست نہیں):

- `sessionId`: موجودہ ٹرانسکرپٹ آئی ڈی (فائل نیم اسی سے اخذ ہوتا ہے جب تک `sessionFile` سیٹ نہ ہو)
- `updatedAt`: آخری سرگرمی کا ٹائم اسٹیمپ
- `sessionFile`: اختیاری واضح ٹرانسکرپٹ پاتھ اوور رائیڈ
- `chatType`: `direct | group | room` (UIs اور بھیجنے کی پالیسی میں مدد دیتا ہے)
- `provider`, `subject`, `room`, `space`, `displayName`: گروپ/چینل لیبلنگ کے لیے میٹاڈیٹا
- Toggles:
  - `thinkingLevel`, `verboseLevel`, `reasoningLevel`, `elevatedLevel`
  - `sendPolicy` (ہر سیشن کے لیے اوور رائیڈ)
- Model selection:
  - `providerOverride`, `modelOverride`, `authProfileOverride`
- Token counters (بہترین کوشش / فراہم کنندہ پر منحصر):
  - `inputTokens`, `outputTokens`, `totalTokens`, `contextTokens`
- `compactionCount`: اس سیشن کی کے لیے خودکار کمپیکشن کتنی بار مکمل ہوئی
- `memoryFlushAt`: آخری پری-کمپیکشن میموری فلش کا ٹائم اسٹیمپ
- `memoryFlushCompactionCount`: آخری فلش کے وقت کمپیکشن کاؤنٹ

اسٹور میں ترمیم محفوظ ہے، مگر اختیار Gateway کے پاس ہے: سیشنز کے چلنے کے دوران یہ اندراجات کو دوبارہ لکھ یا ریہائیڈریٹ کر سکتا ہے۔

---

## Transcript structure (`*.jsonl`)

ٹرانسکرپٹس کو `@mariozechner/pi-coding-agent` کے `SessionManager` کے ذریعے منظم کیا جاتا ہے۔

فائل JSONL ہے:

- پہلی لائن: سیشن ہیڈر (`type: "session"`، جس میں `id`, `cwd`, `timestamp`, اختیاری `parentSession` شامل ہیں)
- اس کے بعد: سیشن اندراجات جن میں `id` + `parentId` (درخت)

نمایاں اندراجی اقسام:

- `message`: صارف/اسسٹنٹ/ٹول رزلٹ پیغامات
- `custom_message`: ایکسٹینشن کے شامل کردہ پیغامات جو ماڈل کانٹیکسٹ میں _داخل ہوتے ہیں_ (UI سے چھپائے جا سکتے ہیں)
- `custom`: ایکسٹینشن اسٹیٹ جو ماڈل کانٹیکسٹ میں _داخل نہیں ہوتی_
- `compaction`: محفوظ شدہ کمپیکشن خلاصہ جس میں `firstKeptEntryId` اور `tokensBefore`
- `branch_summary`: درختی برانچ پر نیویگیٹ کرتے وقت محفوظ شدہ خلاصہ

OpenClaw جان بوجھ کر ٹرانسکرپٹس کو “درست” نہیں کرتا؛ Gateway انہیں پڑھنے/لکھنے کے لیے `SessionManager` استعمال کرتا ہے۔

---

## Context windows vs tracked tokens

دو مختلف تصورات اہم ہیں:

1. **Model context window**: ہر ماڈل کے لیے سخت حد (وہ ٹوکنز جو ماڈل کو نظر آتے ہیں)
2. **Session store counters**: گھومتی ہوئی شماریات جو `sessions.json` میں لکھی جاتی ہیں (/status اور ڈیش بورڈز کے لیے استعمال)

اگر آپ حدود ٹیون کر رہے ہیں:

- کانٹیکسٹ ونڈو ماڈل کیٹلاگ سے آتی ہے (اور کنفیگ کے ذریعے اوور رائیڈ کی جا سکتی ہے)۔
- اسٹور میں `contextTokens` رن ٹائم تخمینہ/رپورٹنگ ویلیو ہے؛ اسے سخت ضمانت نہ سمجھیں۔

مزید کے لیے دیکھیں [/token-use](/reference/token-use)۔

---

## Compaction: what it is

کمپیکشن پرانی گفتگو کو ٹرانسکرپٹ میں محفوظ شدہ `compaction` اندراج میں سمیٹ دیتی ہے اور حالیہ پیغامات کو برقرار رکھتی ہے۔

کمپیکشن کے بعد، آئندہ ٹرنز کو نظر آتا ہے:

- کمپیکشن خلاصہ
- `firstKeptEntryId` کے بعد کے پیغامات

Compaction is **persistent** (unlike session pruning). See [/concepts/session-pruning](/concepts/session-pruning).

---

## When auto-compaction happens (Pi runtime)

ایمبیڈڈ Pi ایجنٹ میں، خودکار کمپیکشن دو صورتوں میں متحرک ہوتی ہے:

1. **Overflow recovery**: ماڈل کانٹیکسٹ اوورفلو ایرر واپس کرتا ہے → کمپیکٹ → دوبارہ کوشش۔
2. **Threshold maintenance**: کامیاب ٹرن کے بعد، جب:

`contextTokens > contextWindow - reserveTokens`

جہاں:

- `contextWindow` ماڈل کی کانٹیکسٹ ونڈو ہے
- `reserveTokens` پرامپٹس + اگلی ماڈل آؤٹ پٹ کے لیے محفوظ ہیڈ روم ہے

یہ Pi رن ٹائم کی معنویت ہے (OpenClaw ایونٹس استعمال کرتا ہے، مگر کمپیکشن کا وقت Pi طے کرتا ہے)۔

---

## Compaction settings (`reserveTokens`, `keepRecentTokens`)

Pi کی کمپیکشن سیٹنگز Pi سیٹنگز میں ہوتی ہیں:

```json5
{
  compaction: {
    enabled: true,
    reserveTokens: 16384,
    keepRecentTokens: 20000,
  },
}
```

OpenClaw ایمبیڈڈ رنز کے لیے سکیورٹی فلور بھی نافذ کرتا ہے:

- اگر `compaction.reserveTokens < reserveTokensFloor` ہو تو OpenClaw اسے بڑھا دیتا ہے۔
- ڈیفالٹ فلور `20000` ٹوکنز ہے۔
- فلور کو غیر فعال کرنے کے لیے `agents.defaults.compaction.reserveTokensFloor: 0` سیٹ کریں۔
- اگر یہ پہلے ہی زیادہ ہو تو OpenClaw اسے تبدیل نہیں کرتا۔

وجہ: کمپیکشن ناگزیر ہونے سے پہلے کثیر-ٹرن “ہاؤس کیپنگ” (جیسے میموری لکھائیاں) کے لیے کافی ہیڈ روم چھوڑنا۔

نفاذ: `src/agents/pi-settings.ts` میں `ensurePiCompactionReserveTokens()`
(`src/agents/pi-embedded-runner.ts` سے کال ہوتا ہے)۔

---

## User-visible surfaces

آپ کمپیکشن اور سیشن اسٹیٹ کو یہاں سے دیکھ سکتے ہیں:

- `/status` (کسی بھی چیٹ سیشن میں)
- `openclaw status` (CLI)
- `openclaw sessions` / `sessions --json`
- Verbose موڈ: `🧹 Auto-compaction complete` + کمپیکشن کاؤنٹ

---

## Silent housekeeping (`NO_REPLY`)

OpenClaw پس منظر کے کاموں کے لیے “خاموش” ٹرنز کی حمایت کرتا ہے جہاں صارف کو درمیانی آؤٹ پٹ نظر نہیں آنی چاہیے۔

رواج:

- اسسٹنٹ اپنی آؤٹ پٹ کا آغاز `NO_REPLY` سے کرتا ہے تاکہ “صارف کو جواب نہ بھیجا جائے” کی نشاندہی ہو۔
- OpenClaw ڈیلیوری لیئر میں اسے ہٹا/دبا دیتا ہے۔

`2026.1.10` کے بعد سے، OpenClaw **ڈرافٹ/ٹائپنگ اسٹریمنگ** کو بھی دبا دیتا ہے جب جزوی چنک `NO_REPLY` سے شروع ہو، تاکہ خاموش آپریشنز دورانِ ٹرن جزوی آؤٹ پٹ لیک نہ کریں۔

---

## Pre-compaction “memory flush” (implemented)

مقصد: خودکار کمپیکشن ہونے سے پہلے ایک خاموش ایجنٹک ٹرن چلانا جو پائیدار
اسٹیٹ کو ڈسک پر لکھ دے (مثلاً ایجنٹ ورک اسپیس میں `memory/YYYY-MM-DD.md`) تاکہ کمپیکشن
اہم کانٹیکسٹ کو مٹا نہ سکے۔

OpenClaw **پری-تھریش ہولڈ فلش** طریقہ استعمال کرتا ہے:

1. سیشن کانٹیکسٹ کے استعمال کی نگرانی کریں۔
2. جب یہ “سافٹ تھریش ہولڈ” (Pi کی کمپیکشن تھریش ہولڈ سے کم) عبور کرے تو ایجنٹ کو ایک خاموش
   “ابھی میموری لکھیں” ہدایت چلائیں۔
3. `NO_REPLY` استعمال کریں تاکہ صارف کو کچھ نظر نہ آئے۔

کنفیگ (`agents.defaults.compaction.memoryFlush`):

- `enabled` (ڈیفالٹ: `true`)
- `softThresholdTokens` (ڈیفالٹ: `4000`)
- `prompt` (فلش ٹرن کے لیے صارف پیغام)
- `systemPrompt` (فلش ٹرن کے لیے شامل کیا جانے والا اضافی سسٹم پرامپٹ)

نوٹس:

- ڈیفالٹ پرامپٹ/سسٹم پرامپٹ میں ڈیلیوری دبانے کے لیے `NO_REPLY` ہنٹ شامل ہے۔
- فلش ہر کمپیکشن سائیکل میں ایک بار چلتا ہے (`sessions.json` میں ٹریک کیا جاتا ہے)۔
- فلش صرف ایمبیڈڈ Pi سیشنز کے لیے چلتا ہے (CLI بیک اینڈز اسے اسکیپ کرتے ہیں)۔
- جب سیشن ورک اسپیس ریڈ اونلی ہو (`workspaceAccess: "ro"` یا `"none"`) تو فلش اسکیپ ہو جاتا ہے۔
- ورک اسپیس فائل لے آؤٹ اور لکھنے کے پیٹرنز کے لیے [Memory](/concepts/memory) دیکھیں۔

Pi ایکسٹینشن API میں `session_before_compact` ہُک بھی فراہم کرتا ہے، مگر OpenClaw کی
فلش لاجک فی الحال Gateway سائیڈ پر رہتی ہے۔

---

## Troubleshooting checklist

- Session key wrong? Start with [/concepts/session](/concepts/session) and confirm the `sessionKey` in `/status`.
- Store vs transcript mismatch? Confirm the Gateway host and the store path from `openclaw status`.
- Compaction spam? Check:
  - ماڈل کانٹیکسٹ ونڈو (بہت چھوٹی)
  - کمپیکشن سیٹنگز (ماڈل ونڈو کے لیے `reserveTokens` بہت زیادہ ہو تو جلد کمپیکشن ہو سکتی ہے)
  - ٹول-رزلٹ پھولاؤ: سیشن پروننگ کو فعال/ٹیون کریں
- Silent turns leaking? Confirm the reply starts with `NO_REPLY` (exact token) and you’re on a build that includes the streaming suppression fix.

