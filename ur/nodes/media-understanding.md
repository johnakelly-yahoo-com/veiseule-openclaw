---
summary: "فراہم کنندہ + CLI فال بیکس کے ساتھ ان باؤنڈ تصویر/آڈیو/ویڈیو کی سمجھ (اختیاری)"
read_when:
  - میڈیا سمجھ بوجھ کی ڈیزائننگ یا ری فیکٹرنگ
  - ان باؤنڈ آڈیو/ویڈیو/تصویر کی پری پروسیسنگ کی ٹیوننگ
title: "میڈیا سمجھ بوجھ"
---

# میڈیا سمجھ بوجھ (ان باؤنڈ) — 2026-01-17

OpenClaw جواب کے پائپ لائن چلنے سے پہلے **آنے والے میڈیا کا خلاصہ** (تصویر/آڈیو/ویڈیو) تیار کر سکتا ہے۔ یہ خودکار طور پر معلوم کر لیتا ہے کہ آیا مقامی ٹولز یا پرووائیڈر کیز دستیاب ہیں، اور اسے غیر فعال یا حسبِ ضرورت ترتیب دیا جا سکتا ہے۔ اگر سمجھنے کی سہولت بند ہو تو بھی ماڈلز کو حسبِ معمول اصل فائلیں/URLs موصول ہوتے رہتے ہیں۔

## مقاصد

- اختیاری: تیز تر روٹنگ اور بہتر کمانڈ پارسنگ کے لیے ان باؤنڈ میڈیا کو مختصر متن میں پہلے سے ہضم کرنا۔
- اصل میڈیا کی ماڈل تک ترسیل برقرار رکھنا (ہمیشہ)۔
- **فراہم کنندہ APIs** اور **CLI فال بیکس** کی معاونت۔
- ترتیب وار فال بیک کے ساتھ متعدد ماڈلز کی اجازت (غلطی/سائز/ٹائم آؤٹ)۔

## اعلیٰ سطحی رویہ

1. ان باؤنڈ اٹیچمنٹس جمع کریں (`MediaPaths`, `MediaUrls`, `MediaTypes`)۔
2. ہر فعال صلاحیت (تصویر/آڈیو/ویڈیو) کے لیے پالیسی کے مطابق اٹیچمنٹس منتخب کریں (بطورِ طے شدہ: **پہلا**)۔
3. پہلی اہل ماڈل انٹری منتخب کریں (سائز + صلاحیت + توثیق)۔
4. اگر کوئی ماڈل ناکام ہو یا میڈیا بہت بڑا ہو تو **اگلی انٹری پر فال بیک** کریں۔
5. کامیابی پر:
   - `Body`، `[Image]`، `[Audio]`، یا `[Video]` بلاک بن جاتا ہے۔
   - آڈیو `{{Transcript}}` سیٹ کرتا ہے؛ کمانڈ پارسنگ دستیاب ہونے پر کیپشن متن استعمال کرتی ہے،
     بصورتِ دیگر ٹرانسکرپٹ۔
   - کیپشنز بلاک کے اندر `User text:` کے طور پر محفوظ رہتی ہیں۔

اگر سمجھ بوجھ ناکام ہو یا غیر فعال ہو تو **جواب کا بہاؤ جاری رہتا ہے** اور اصل باڈی + اٹیچمنٹس استعمال ہوتی ہیں۔

## کنفیگ جائزہ

`tools.media` **مشترکہ ماڈلز** کے ساتھ فی‑صلاحیت اوور رائیڈز کی معاونت کرتا ہے:

- `tools.media.models`: مشترکہ ماڈل فہرست (گیٹنگ کے لیے `capabilities` استعمال کریں)۔
- `tools.media.image` / `tools.media.audio` / `tools.media.video`:
  - ڈیفالٹس (`prompt`, `maxChars`, `maxBytes`, `timeoutSeconds`, `language`)
  - فراہم کنندہ اوور رائیڈز (`baseUrl`, `headers`, `providerOptions`)
  - Deepgram آڈیو اختیارات بذریعہ `tools.media.audio.providerOptions.deepgram`
  - اختیاری **فی‑صلاحیت `models` فہرست** (مشترکہ ماڈلز سے پہلے ترجیح)
  - `attachments` پالیسی (`mode`, `maxAttachments`, `prefer`)
  - `scope` (چینل/chatType/session کلید کے ذریعے اختیاری گیٹنگ)
- `tools.media.concurrency`: زیادہ سے زیادہ ہم وقت صلاحیتی رنز (بطورِ طے شدہ **2**)۔

```json5
{
  tools: {
    media: {
      models: [
        /* shared list */
      ],
      image: {
        /* optional overrides */
      },
      audio: {
        /* optional overrides */
      },
      video: {
        /* optional overrides */
      },
    },
  },
}
```

### ماڈل انٹریز

ہر `models[]` انٹری **فراہم کنندہ** یا **CLI** ہو سکتی ہے:

```json5
{
  type: "provider", // default if omitted
  provider: "openai",
  model: "gpt-5.2",
  prompt: "Describe the image in <= 500 chars.",
  maxChars: 500,
  maxBytes: 10485760,
  timeoutSeconds: 60,
  capabilities: ["image"], // optional, used for multi‑modal entries
  profile: "vision-profile",
  preferredProfile: "vision-fallback",
}
```

```json5
{
  type: "cli",
  command: "gemini",
  args: [
    "-m",
    "gemini-3-flash",
    "--allowed-tools",
    "read_file",
    "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
  ],
  maxChars: 500,
  maxBytes: 52428800,
  timeoutSeconds: 120,
  capabilities: ["video", "image"],
}
```

CLI ٹیمپلیٹس یہ بھی استعمال کر سکتے ہیں:

- `{{MediaDir}}` (میڈیا فائل پر مشتمل ڈائریکٹری)
- `{{OutputDir}}` (اس رن کے لیے بنائی گئی اسکریچ ڈائریکٹری)
- `{{OutputBase}}` (اسکریچ فائل کی بنیادی راہ، بغیر ایکسٹینشن)

## ڈیفالٹس اور حدود

سفارش کردہ ڈیفالٹس:

- `maxChars`: تصویر/ویڈیو کے لیے **500** (مختصر، کمانڈ‑دوستانہ)
- `maxChars`: آڈیو کے لیے **غیر متعین** (مکمل ٹرانسکرپٹ جب تک آپ حد مقرر نہ کریں)
- `maxBytes`:
  - تصویر: **10MB**
  - آڈیو: **20MB**
  - ویڈیو: **50MB**

قواعد:

- اگر میڈیا `maxBytes` سے تجاوز کرے تو وہ ماڈل چھوڑ دیا جاتا ہے اور **اگلا ماڈل آزمایا جاتا ہے**۔
- اگر ماڈل `maxChars` سے زیادہ واپس کرے تو آؤٹ پٹ تراش دی جاتی ہے۔
- `prompt` defaults to simple “Describe the {media}.” plus the `maxChars` guidance (image/video only).
- اگر `<capability>.enabled: true` ہو مگر کوئی ماڈلز کنفیگر نہ ہوں تو OpenClaw
  **فعال جواب ماڈل** آزما لیتا ہے جب اس کا فراہم کنندہ صلاحیت کی معاونت کرتا ہو۔

### میڈیا سمجھ بوجھ کی خودکار شناخت (بطورِ طے شدہ)

اگر `tools.media.<capability>.enabled` کو `false` پر **سیٹ نہیں** کیا گیا اور آپ نے نہیں
configured models, OpenClaw auto-detects in this order and **stops at the first
working option**:

1. **مقامی CLIs** (صرف آڈیو؛ اگر انسٹال ہوں)
   - `sherpa-onnx-offline` (درکار: `SHERPA_ONNX_MODEL_DIR` بمعہ encoder/decoder/joiner/tokens)
   - `whisper-cli` (`whisper-cpp`; `WHISPER_CPP_MODEL` یا بنڈلڈ tiny ماڈل استعمال کرتا ہے)
   - `whisper` (Python CLI؛ ماڈلز خودکار طور پر ڈاؤن لوڈ کرتا ہے)
2. **Gemini CLI** (`gemini`) بذریعہ `read_many_files`
3. **فراہم کنندہ کی کلیدیں**
   - آڈیو: OpenAI → Groq → Deepgram → Google
   - تصویر: OpenAI → Anthropic → Google → MiniMax
   - ویڈیو: Google

خودکار شناخت غیر فعال کرنے کے لیے سیٹ کریں:

```json5
{
  tools: {
    media: {
      audio: {
        enabled: false,
      },
    },
  },
}
```

نوٹ: بائنری کی شناخت macOS/Linux/Windows پر بہترین کوشش کی بنیاد پر ہوتی ہے؛ یقینی بنائیں کہ CLI `PATH` پر موجود ہو (ہم `~` کو توسیع دیتے ہیں)، یا مکمل کمانڈ پاتھ کے ساتھ ایک واضح CLI ماڈل سیٹ کریں۔

## صلاحیتیں (اختیاری)

اگر آپ `capabilities` سیٹ کرتے ہیں تو یہ اندراج صرف انہی میڈیا اقسام کے لیے چلے گا۔ مشترکہ کے لیے
lists, OpenClaw can infer defaults:

- `openai`, `anthropic`, `minimax`: **تصویر**
- `google` (Gemini API): **تصویر + آڈیو + ویڈیو**
- `groq`: **آڈیو**
- `deepgram`: **آڈیو**

CLI اندراجات کے لیے، غیر متوقع مماثلتوں سے بچنے کے لیے **`capabilities` کو واضح طور پر سیٹ کریں**۔
If you omit `capabilities`, the entry is eligible for the list it appears in.

## فراہم کنندہ سپورٹ میٹرکس (OpenClaw انٹیگریشنز)

| صلاحیت | فراہم کنندہ انٹیگریشن                             | نوٹس                                                                 |
| ------ | ------------------------------------------------- | -------------------------------------------------------------------- |
| تصویر  | OpenAI / Anthropic / Google / دیگر بذریعہ `pi-ai` | رجسٹری میں کوئی بھی تصویر‑قابل ماڈل کام کرتا ہے۔                     |
| آڈیو   | OpenAI، Groq، Deepgram، Google                    | فراہم کنندہ ٹرانسکرپشن (Whisper/Deepgram/Gemini)۔ |
| ویڈیو  | Google (Gemini API)            | فراہم کنندہ ویڈیو سمجھ بوجھ۔                                         |

## سفارش کردہ فراہم کنندگان

**تصویر**

- اگر آپ کا فعال ماڈل تصاویر کی معاونت کرتا ہو تو اسی کو ترجیح دیں۔
- اچھے ڈیفالٹس: `openai/gpt-5.2`, `anthropic/claude-opus-4-6`, `google/gemini-3-pro-preview`۔

**آڈیو**

- `openai/gpt-4o-mini-transcribe`, `groq/whisper-large-v3-turbo`, یا `deepgram/nova-3`۔
- CLI فال بیک: `whisper-cli` (whisper-cpp) یا `whisper`۔
- Deepgram سیٹ اپ: [Deepgram (audio transcription)](/providers/deepgram)۔

**ویڈیو**

- `google/gemini-3-flash-preview` (تیز)، `google/gemini-3-pro-preview` (زیادہ بھرپور)۔
- CLI فال بیک: `gemini` CLI (ویڈیو/آڈیو پر `read_file` کی معاونت کرتا ہے)۔

## اٹیچمنٹ پالیسی

فی‑صلاحیت `attachments` کنٹرول کرتا ہے کہ کون سی اٹیچمنٹس پروسیس ہوں:

- `mode`: `first` (بطورِ طے شدہ) یا `all`
- `maxAttachments`: پروسیس کی جانے والی تعداد کی حد (بطورِ طے شدہ **1**)
- `prefer`: `first`, `last`, `path`, `url`

جب `mode: "all"` ہو تو آؤٹ پٹس کو `[Image 1/2]`, `[Audio 2/2]` وغیرہ کے طور پر لیبل کیا جاتا ہے۔

## کنفیگ مثالیں

### 1. مشترکہ ماڈلز فہرست + اوور رائیڈز

```json5
{
  tools: {
    media: {
      models: [
        { provider: "openai", model: "gpt-5.2", capabilities: ["image"] },
        {
          provider: "google",
          model: "gemini-3-flash-preview",
          capabilities: ["image", "audio", "video"],
        },
        {
          type: "cli",
          command: "gemini",
          args: [
            "-m",
            "gemini-3-flash",
            "--allowed-tools",
            "read_file",
            "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
          ],
          capabilities: ["image", "video"],
        },
      ],
      audio: {
        attachments: { mode: "all", maxAttachments: 2 },
      },
      video: {
        maxChars: 500,
      },
    },
  },
}
```

### 2. صرف آڈیو + ویڈیو (تصویر بند)

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          {
            type: "cli",
            command: "whisper",
            args: ["--model", "base", "{{MediaPath}}"],
          },
        ],
      },
      video: {
        enabled: true,
        maxChars: 500,
        models: [
          { provider: "google", model: "gemini-3-flash-preview" },
          {
            type: "cli",
            command: "gemini",
            args: [
              "-m",
              "gemini-3-flash",
              "--allowed-tools",
              "read_file",
              "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
            ],
          },
        ],
      },
    },
  },
}
```

### 3. اختیاری تصویر سمجھ بوجھ

```json5
{
  tools: {
    media: {
      image: {
        enabled: true,
        maxBytes: 10485760,
        maxChars: 500,
        models: [
          { provider: "openai", model: "gpt-5.2" },
          { provider: "anthropic", model: "claude-opus-4-6" },
          {
            type: "cli",
            command: "gemini",
            args: [
              "-m",
              "gemini-3-flash",
              "--allowed-tools",
              "read_file",
              "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
            ],
          },
        ],
      },
    },
  },
}
```

### 4. ملٹی‑موڈل واحد انٹری (واضح صلاحیتیں)

```json5
{
  tools: {
    media: {
      image: {
        models: [
          {
            provider: "google",
            model: "gemini-3-pro-preview",
            capabilities: ["image", "video", "audio"],
          },
        ],
      },
      audio: {
        models: [
          {
            provider: "google",
            model: "gemini-3-pro-preview",
            capabilities: ["image", "video", "audio"],
          },
        ],
      },
      video: {
        models: [
          {
            provider: "google",
            model: "gemini-3-pro-preview",
            capabilities: ["image", "video", "audio"],
          },
        ],
      },
    },
  },
}
```

## اسٹیٹس آؤٹ پٹ

جب میڈیا سمجھ بوجھ چلتی ہے تو `/status` میں ایک مختصر خلاصہ لائن شامل ہوتی ہے:

```
📎 Media: image ok (openai/gpt-5.2) · audio skipped (maxBytes)
```

یہ فی‑صلاحیت نتائج اور جہاں قابلِ اطلاق ہو منتخب فراہم کنندہ/ماڈل دکھاتا ہے۔

## نوٹس

- سمجھنا **بہترین کوشش** کی بنیاد پر ہوتا ہے۔ غلطیاں جوابات کو نہیں روکتیں۔
- سمجھ بوجھ غیر فعال ہونے پر بھی اٹیچمنٹس ماڈلز کو منتقل کی جاتی ہیں۔
- جہاں سمجھ بوجھ چلتی ہے اسے محدود کرنے کے لیے `scope` استعمال کریں (مثلاً صرف DMs)۔

## متعلقہ دستاویزات

- [Configuration](/gateway/configuration)
- [Image & Media Support](/nodes/images)
