---
title: "मीडिया समझ"
---

# मीडिया समझ (इनबाउंड) — 2026-01-17

OpenClaw **इनबाउंड मीडिया** (image/audio/video) को reply pipeline चलने से पहले summarize कर सकता है। यह स्वतः पहचानता है कि local tools या provider keys उपलब्ध हैं या नहीं, और इसे अक्षम या अनुकूलित किया जा सकता है। यदि understanding बंद है, तो भी मॉडल्स को मूल फ़ाइलें/URLs सामान्य रूप से मिलते रहते हैं।

## लक्ष्य

- वैकल्पिक: तेज़ रूटिंग + बेहतर कमांड पार्सिंग के लिए इनबाउंड मीडिया को छोटे पाठ में पूर्व‑डाइजेस्ट करना।
- मूल मीडिया की डिलीवरी को मॉडल तक सुरक्षित रखना (हमेशा)।
- **प्रदाता APIs** और **CLI फॉलबैक** का समर्थन।
- क्रमबद्ध फॉलबैक के साथ कई मॉडलों की अनुमति (त्रुटि/आकार/टाइमआउट)।

## उच्च‑स्तरीय व्यवहार

1. इनबाउंड अटैचमेंट्स एकत्र करें (`MediaPaths`, `MediaUrls`, `MediaTypes`)।
2. प्रत्येक सक्षम क्षमता (image/audio/video) के लिए नीति अनुसार अटैचमेंट चुनें (डिफ़ॉल्ट: **पहला**)।
3. पहला पात्र मॉडल एंट्री चुनें (आकार + क्षमता + प्रमाणीकरण)।
4. यदि कोई मॉडल विफल होता है या मीडिया बहुत बड़ा है, तो **अगली एंट्री पर फॉलबैक** करें।
5. सफलता पर:
   - `Body` एक `[Image]`, `[Audio]`, या `[Video]` ब्लॉक बन जाता है।
   - ऑडियो `{{Transcript}}` सेट करता है; कमांड पार्सिंग उपलब्ध होने पर कैप्शन पाठ का उपयोग करती है, अन्यथा ट्रांसक्रिप्ट का।
   - कैप्शन ब्लॉक के भीतर `User text:` के रूप में सुरक्षित रहते हैं।

यदि समझ विफल हो जाती है या अक्षम है, तो **reply flow जारी रहता है**—मूल body + attachments के साथ।

## Config overview

`tools.media` **साझा मॉडलों** के साथ प्रति‑क्षमता overrides का समर्थन करता है:

- `tools.media.models`: साझा मॉडल सूची (gate करने के लिए `capabilities` का उपयोग करें)।
- `tools.media.image` / `tools.media.audio` / `tools.media.video`:
  - defaults (`prompt`, `maxChars`, `maxBytes`, `timeoutSeconds`, `language`)
  - provider overrides (`baseUrl`, `headers`, `providerOptions`)
  - Deepgram ऑडियो विकल्प via `tools.media.audio.providerOptions.deepgram`
  - वैकल्पिक **प्रति‑क्षमता `models` सूची** (shared models से पहले प्राथमिकता)
  - `attachments` नीति (`mode`, `maxAttachments`, `prefer`)
  - `scope` (channel/chatType/session key द्वारा वैकल्पिक gating)
- `tools.media.concurrency`: अधिकतम concurrent capability runs (डिफ़ॉल्ट **2**)।

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

### Model entries

प्रत्येक `models[]` एंट्री **provider** या **CLI** हो सकती है:

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

CLI templates इनका भी उपयोग कर सकते हैं:

- `{{MediaDir}}` (मीडिया फ़ाइल वाली directory)
- `{{OutputDir}}` (इस run के लिए बनाई गई scratch directory)
- `{{OutputBase}}` (scratch फ़ाइल का base path, बिना extension)

## Defaults and limits

अनुशंसित defaults:

- `maxChars`: image/video के लिए **500** (संक्षिप्त, command‑friendly)
- `maxChars`: audio के लिए **unset** (जब तक आप सीमा न सेट करें, पूरा transcript)
- `maxBytes`:
  - image: **10MB**
  - audio: **20MB**
  - video: **50MB**

नियम:

- यदि media `maxBytes` से अधिक है, तो वह model skip हो जाता है और **next model tried**।
- यदि model `maxChars` से अधिक लौटाता है, तो output trim कर दिया जाता है।
- `prompt` डिफ़ॉल्ट रूप से सरल “Describe the {media}.” होता है, साथ में `maxChars` guidance (केवल image/video)।
- यदि `<capability>.enabled: true` लेकिन कोई models configure नहीं हैं, तो OpenClaw **active reply model** को आज़माता है जब उसका provider उस capability का समर्थन करता हो।

### Auto-detect media understanding (default)

यदि `tools.media.<capability>.enabled` को `false` पर **set नहीं** किया गया है और आपने models configure नहीं किए हैं, तो OpenClaw इस क्रम में auto‑detect करता है और **पहले working option पर रुक जाता है**:

1. **Local CLIs** (केवल audio; यदि installed हों)
   - `sherpa-onnx-offline` (requires `SHERPA_ONNX_MODEL_DIR` with encoder/decoder/joiner/tokens)
   - `whisper-cli` (`whisper-cpp`; uses `WHISPER_CPP_MODEL` or the bundled tiny model)
   - `whisper` (Python CLI; models automatically download करता है)
2. **Gemini CLI** (`gemini`) using `read_many_files`
3. **Provider keys**
   - Audio: OpenAI → Groq → Deepgram → Google
   - Image: OpenAI → Anthropic → Google → MiniMax
   - Video: Google

Auto-detection disable करने के लिए, set करें:

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

Note: Binary detection macOS/Linux/Windows पर best‑effort है; सुनिश्चित करें कि CLI `PATH` पर है (हम `~` expand करते हैं), या full command path के साथ explicit CLI model set करें।

## Capabilities (optional)

यदि आप `capabilities` set करते हैं, तो entry केवल उन्हीं media types के लिए run होती है। Shared lists के लिए, OpenClaw defaults infer कर सकता है:

- `openai`, `anthropic`, `minimax`: **image**
- `google` (Gemini API): **image + audio + video**
- `groq`: **audio**
- `deepgram`: **audio**

CLI entries के लिए, **`capabilities` स्पष्ट रूप से set करें** ताकि अप्रत्याशित matches से बचा जा सके। यदि आप `capabilities` omit करते हैं, तो entry उस list के लिए eligible होती है जिसमें वह दिखाई देती है।

## Provider support matrix (OpenClaw integrations)

| Capability | Provider integration                             | Notes                                              |
| ---------- | ------------------------------------------------ | -------------------------------------------------- |
| Image      | OpenAI / Anthropic / Google / others via `pi-ai` | Registry में कोई भी image-capable model काम करता है। |
| Audio      | OpenAI, Groq, Deepgram, Google                   | Provider transcription (Whisper/Deepgram/Gemini).  |
| Video      | Google (Gemini API)                              | Provider video understanding.                      |

## Recommended providers

**Image**

- यदि आपका active model images support करता है, तो उसे prefer करें।
- अच्छे defaults: `openai/gpt-5.2`, `anthropic/claude-opus-4-6`, `google/gemini-3-pro-preview`।

**Audio**

- `openai/gpt-4o-mini-transcribe`, `groq/whisper-large-v3-turbo`, या `deepgram/nova-3`।
- CLI fallback: `whisper-cli` (whisper-cpp) या `whisper`।
- Deepgram setup: [Deepgram (audio transcription)](/providers/deepgram).

**Video**

- `google/gemini-3-flash-preview` (तेज़), `google/gemini-3-pro-preview` (अधिक समृद्ध)।
- CLI fallback: `gemini` CLI (video/audio पर `read_file` support करता है)।

## Attachment policy

प्रति‑capability `attachments` नियंत्रित करता है कि कौन‑से attachments process हों:

- `mode`: `first` (डिफ़ॉल्ट) या `all`
- `maxAttachments`: process किए जाने वालों की संख्या limit करें (डिफ़ॉल्ट **1**)
- `prefer`: `first`, `last`, `path`, `url`

जब `mode: "all"`, outputs को `[Image 1/2]`, `[Audio 2/2]`, आदि label किया जाता है।

## Config examples

### 1) Shared models list + overrides

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

### 2) Audio + Video only (image off)

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

### 3) Optional image understanding

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

### 4) Multi‑modal single entry (explicit capabilities)

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

## Status output

जब media understanding चलती है, `/status` में एक संक्षिप्त summary line शामिल होती है:

```
📎 Media: image ok (openai/gpt-5.2) · audio skipped (maxBytes)
```

यह प्रति‑capability outcomes और लागू होने पर चुने गए provider/model को दिखाता है।

## Notes

- Understanding **best‑effort** है। Errors replies को block नहीं करते।
- Understanding disable होने पर भी attachments models तक pass किए जाते हैं।
- जहाँ understanding चले उसे limit करने के लिए `scope` का उपयोग करें (जैसे केवल DMs)।

## Related docs

- [Configuration](/gateway/configuration)
- [छवि और मीडिया समर्थन](/nodes/images)