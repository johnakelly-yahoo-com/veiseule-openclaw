---
summary: "یہ وضاحت کرتا ہے کہ آنے والی آڈیو/وائس نوٹس کیسے ڈاؤن لوڈ کی جاتی ہیں، ٹرانسکرائب ہوتی ہیں، اور جوابات میں شامل کی جاتی ہیں"
read_when:
  - آڈیو ٹرانسکرپشن یا میڈیا ہینڈلنگ میں تبدیلی کرتے وقت
title: "آڈیو اور وائس نوٹس"
---

# آڈیو / وائس نوٹس — 2026-01-17

## کیا کام کرتا ہے

- **میڈیا سمجھ (آڈیو)**: اگر آڈیو سمجھ فعال ہو (یا خودکار طور پر معلوم ہو جائے)، تو OpenClaw:
  1. پہلی آڈیو اٹیچمنٹ (لوکل پاتھ یا URL) تلاش کرتا ہے اور ضرورت ہو تو ڈاؤن لوڈ کرتا ہے۔
  2. ہر ماڈل اندراج کو بھیجنے سے پہلے `maxBytes` نافذ کرتا ہے۔
  3. ترتیب کے مطابق پہلے اہل ماڈل اندراج (provider یا CLI) کو چلاتا ہے۔
  4. اگر ناکام ہو جائے یا چھوڑ دے (سائز/ٹائم آؤٹ)، تو اگلے اندراج کی کوشش کرتا ہے۔
  5. کامیابی پر، `Body` کو ایک `[Audio]` بلاک سے بدل دیتا ہے اور `{{Transcript}}` سیٹ کرتا ہے۔
- **کمانڈ پارسنگ**: جب ٹرانسکرپشن کامیاب ہو جائے، تو `CommandBody`/`RawBody` کو ٹرانسکرپٹ پر سیٹ کیا جاتا ہے تاکہ سلیش کمانڈز بدستور کام کریں۔
- **تفصیلی لاگنگ**: `--verbose` میں، ہم لاگ کرتے ہیں کہ کب ٹرانسکرپشن چلتی ہے اور کب وہ باڈی کو تبدیل کرتی ہے۔

## خودکار تشخیص (بطورِ طے شدہ)

اگر آپ **ماڈلز کنفیگر نہیں کرتے** اور `tools.media.audio.enabled` کو `false` پر **سیٹ نہیں** کیا گیا،
تو OpenClaw اس ترتیب سے خودکار طور پر تشخیص کرتا ہے اور پہلی کام کرنے والی آپشن پر رک جاتا ہے:

1. **لوکل CLIs** (اگر انسٹال ہوں)
   - `sherpa-onnx-offline` (کے لیے `SHERPA_ONNX_MODEL_DIR` درکار ہے جس میں encoder/decoder/joiner/tokens شامل ہوں)
   - `whisper-cli` (`whisper-cpp` سے؛ `WHISPER_CPP_MODEL` یا بنڈلڈ tiny ماڈل استعمال کرتا ہے)
   - `whisper` (Python CLI؛ ماڈلز خودکار طور پر ڈاؤن لوڈ کرتا ہے)
2. **Gemini CLI** (`gemini`) بذریعہ `read_many_files`
3. **Provider keys** (OpenAI → Groq → Deepgram → Google)

To disable auto-detection, set `tools.media.audio.enabled: false`.
To customize, set `tools.media.audio.models`.
نوٹ: بائنری کی شناخت macOS/Linux/Windows پر بہترین کوشش کی بنیاد پر ہوتی ہے؛ یقینی بنائیں کہ CLI `PATH` پر موجود ہو (ہم `~` کو توسیع دیتے ہیں)، یا مکمل کمانڈ پاتھ کے ساتھ ایک واضح CLI ماڈل سیٹ کریں۔

## کنفیگ مثالیں

### Provider + CLI فال بیک (OpenAI + Whisper CLI)

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        maxBytes: 20971520,
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          {
            type: "cli",
            command: "whisper",
            args: ["--model", "base", "{{MediaPath}}"],
            timeoutSeconds: 45,
          },
        ],
      },
    },
  },
}
```

### اسکوپ گیٹنگ کے ساتھ صرف Provider

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        scope: {
          default: "allow",
          rules: [{ action: "deny", match: { chatType: "group" } }],
        },
        models: [{ provider: "openai", model: "gpt-4o-mini-transcribe" }],
      },
    },
  },
}
```

### صرف Provider (Deepgram)

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [{ provider: "deepgram", model: "nova-3" }],
      },
    },
  },
}
```

## نوٹس اور حدود

- Provider کی تصدیق معیاری ماڈل تصدیقی ترتیب کی پیروی کرتی ہے (auth پروفائلز، env vars، `models.providers.*.apiKey`)۔
- Deepgram، `provider: "deepgram"` استعمال ہونے پر `DEEPGRAM_API_KEY` اٹھا لیتا ہے۔
- Deepgram سیٹ اپ کی تفصیلات: [Deepgram (audio transcription)](/providers/deepgram)۔
- آڈیو providers، `tools.media.audio` کے ذریعے `baseUrl`، `headers`، اور `providerOptions` کو اووررائیڈ کر سکتے ہیں۔
- Default size cap is 20MB (`tools.media.audio.maxBytes`). Oversize audio is skipped for that model and the next entry is tried.
- Default `maxChars` for audio is **unset** (full transcript). Set `tools.media.audio.maxChars` or per-entry `maxChars` to trim output.
- OpenAI کا خودکار ڈیفالٹ `gpt-4o-mini-transcribe` ہے؛ زیادہ درستگی کے لیے `model: "gpt-4o-transcribe"` سیٹ کریں۔
- متعدد وائس نوٹس پراسیس کرنے کے لیے `tools.media.audio.attachments` استعمال کریں (`mode: "all"` + `maxAttachments`)۔
- ٹرانسکرپٹ ٹیمپلیٹس میں `{{Transcript}}` کے طور پر دستیاب ہے۔
- CLI stdout کی حد (5MB) ہے؛ CLI آؤٹ پٹ مختصر رکھیں۔

## گروپس میں مینشن کی شناخت

جب کسی گروپ چیٹ کے لیے `requireMention: true` سیٹ کیا جاتا ہے تو OpenClaw اب مینشن چیک کرنے سے **پہلے** آڈیو کو ٹرانسکرائب کرتا ہے۔ اس سے وائس نوٹس کو اس وقت بھی پروسیس کیا جا سکتا ہے جب ان میں مینشن شامل ہوں۔

**یہ کیسے کام کرتا ہے:**

1. اگر وائس میسج میں کوئی ٹیکسٹ باڈی نہ ہو اور گروپ میں مینشن درکار ہو تو OpenClaw "preflight" ٹرانسکرپشن انجام دیتا ہے۔
2. ٹرانسکرپٹ میں مینشن پیٹرنز (مثلاً `@BotName`، ایموجی ٹرگرز) کو چیک کیا جاتا ہے۔
3. اگر مینشن مل جائے تو پیغام مکمل ریپلائی پائپ لائن کے ذریعے آگے بڑھتا ہے۔
4. ٹرانسکرپٹ کو مینشن کی شناخت کے لیے استعمال کیا جاتا ہے تاکہ وائس نوٹس مینشن گیٹ سے گزر سکیں۔

**Fallback behavior:**

- اگر پری فلائٹ کے دوران ٹرانسکرپشن ناکام ہو جائے (ٹائم آؤٹ، API ایرر وغیرہ)، تو پیغام کو صرف متن پر مبنی مینشن ڈیٹیکشن کی بنیاد پر پروسیس کیا جاتا ہے۔
- یہ یقینی بناتا ہے کہ مکس پیغامات (متن + آڈیو) کبھی بھی غلطی سے ڈراپ نہ ہوں۔

**Example:** ایک صارف Telegram گروپ میں `requireMention: true` کے ساتھ ایک وائس نوٹ بھیجتا ہے جس میں کہتا ہے "Hey @Claude, what's the weather?"۔ وائس نوٹ کو ٹرانسکرائب کیا جاتا ہے، مینشن کی شناخت ہوتی ہے، اور ایجنٹ جواب دیتا ہے۔

## Gotchas

- Scope rules use first-match wins. `chatType` is normalized to `direct`, `group`, or `room`.
- یقینی بنائیں کہ آپ کا CLI 0 کے ساتھ ایگزٹ کرے اور سادہ متن پرنٹ کرے؛ JSON کو `jq -r .text` کے ذریعے درست کرنا ہوگا۔
- جوابی قطار کو بلاک ہونے سے بچانے کے لیے ٹائم آؤٹس مناسب رکھیں (`timeoutSeconds`، بطورِ طے شدہ 60s)۔
- پری فلائٹ ٹرانسکرپشن مینشن ڈیٹیکشن کے لیے صرف **پہلی** آڈیو اٹیچمنٹ کو پروسیس کرتی ہے۔ اضافی آڈیو کو مین میڈیا انڈراسٹینڈنگ مرحلے کے دوران پروسیس کیا جاتا ہے۔

