---
title: "मीडिया समझ"
---

# मीडिया समझ (इनबाउंड) — 2026-01-17

यह स्वतः पहचानता है कि लोकल tools या provider keys उपलब्ध हैं या नहीं, और इसे अक्षम या अनुकूलित किया जा सकता है। यदि understanding बंद है, तो भी मॉडल्स को मूल फ़ाइलें/URLs सामान्य रूप से मिलते रहते हैं। `prompt` डिफ़ॉल्ट रूप से सरल “Describe the {media}.” होता है, साथ में `maxChars` मार्गदर्शन (केवल image/video)।

## लक्ष्य

- वैकल्पिक: तेज़ रूटिंग + बेहतर कमांड पार्सिंग के लिए इनबाउंड मीडिया को छोटे पाठ में पूर्व‑डाइजेस्ट करना।
- मूल मीडिया की डिलीवरी को मॉडल तक सुरक्षित रखना (हमेशा)।
- **प्रदाता API** और **CLI फॉलबैक** का समर्थन।
- क्रमबद्ध फॉलबैक के साथ कई मॉडलों की अनुमति (त्रुटि/आकार/टाइमआउट)।

## उच्च‑स्तरीय व्यवहार

1. इनबाउंड अटैचमेंट्स एकत्र करें (`MediaPaths`, `MediaUrls`, `MediaTypes`)।
2. प्रत्येक सक्षम क्षमता (इमेज/ऑडियो/वीडियो) के लिए नीति अनुसार अटैचमेंट चुनें (डिफ़ॉल्ट: **पहला**)।
3. पहला पात्र मॉडल एंट्री चुनें (आकार + क्षमता + प्रमाणीकरण)।
4. यदि कोई मॉडल विफल होता है या मीडिया बहुत बड़ा है, तो **अगली एंट्री पर फॉलबैक** करें।
5. सफलता पर:
   - `Body` एक `[Image]`, `[Audio]`, या `[Video]` ब्लॉक बन जाता है।
   - ऑडियो `{{Transcript}}` सेट करता है; कमांड पार्सिंग उपलब्ध होने पर कैप्शन पाठ का उपयोग करती है,
     अन्यथा ट्रांसक्रिप्ट का।
   - कैप्शन ब्लॉक के भीतर `User text:` के रूप में सुरक्षित रहते हैं।

यदि समझ विफल हो जाती है या अक्षम है, तो **उत्तर प्रवाह जारी रहता है**—मूल बॉडी + अटैचमेंट्स के साथ।

## कॉन्फ़िग अवलोकन

`tools.media` **साझा मॉडलों** के साथ प्रति‑क्षमता ओवरराइड का समर्थन करता है:

- `tools.media.models`: साझा मॉडल सूची (गेट करने के लिए `capabilities` का उपयोग करें)।
- `tools.media.image` / `tools.media.audio` / `tools.media.video`:
  - डिफ़ॉल्ट्स (`prompt`, `maxChars`, `maxBytes`, `timeoutSeconds`, `language`)
  - प्रदाता ओवरराइड्स (`baseUrl`, `headers`, `providerOptions`)
  - `tools.media.audio.providerOptions.deepgram` के माध्यम से Deepgram ऑडियो विकल्प
  - वैकल्पिक **प्रति‑क्षमता `models` सूची** (साझा मॉडलों से पहले प्राथमिकता)
  - `attachments` नीति (`mode`, `maxAttachments`, `prefer`)
  - `scope` (चैनल/chatType/सत्र कुंजी द्वारा वैकल्पिक गेटिंग)
- `tools.media.concurrency`: अधिकतम समवर्ती क्षमता रन (डिफ़ॉल्ट **2**)।

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

### मॉडल एंट्रीज़

प्रत्येक `models[]` एंट्री **प्रदाता** या **CLI** हो सकती है:

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

CLI टेम्पलेट्स इनका भी उपयोग कर सकते हैं:

- `{{MediaDir}}` (मीडिया फ़ाइल वाली डायरेक्टरी)
- `{{OutputDir}}` (इस रन के लिए बनाई गई स्क्रैच डायरेक्टरी)
- `{{OutputBase}}` (स्क्रैच फ़ाइल का बेस पथ, बिना एक्सटेंशन)

## डिफ़ॉल्ट्स और सीमाएँ

अनुशंसित डिफ़ॉल्ट्स:

- `maxChars`: इमेज/वीडियो के लिए **500** (संक्षिप्त, कमांड‑अनुकूल)
- `maxChars`: ऑडियो के लिए **अनसेट** (जब तक आप सीमा न सेट करें, पूरा ट्रांसक्रिप्ट)
- `maxBytes`:
  - इमेज: **10MB**
  - ऑडियो: **20MB**
  - वीडियो: **50MB**

नियम:

- यदि मीडिया `maxBytes` से अधिक है, तो वह मॉडल स्किप हो जाता है और **अगला मॉडल आज़माया जाता है**।
- यदि मॉडल `maxChars` से अधिक लौटाता है, तो आउटपुट ट्रिम कर दिया जाता है।
- यदि \`tools.media.<capability>
- यदि `<capability>.enabled: true` लेकिन कोई मॉडल कॉन्फ़िगर नहीं है, तो OpenClaw
  **सक्रिय उत्तर मॉडल** को आज़माता है जब उसका प्रदाता उस क्षमता का समर्थन करता हो।

### मीडिया समझ का ऑटो‑डिटेक्ट (डिफ़ॉल्ट)

.enabled`को`false`पर **सेट नहीं** किया गया है और आपने मॉडल्स कॉन्फ़िगर नहीं किए हैं, तो OpenClaw इस क्रम में auto‑detect करता है और **पहले काम करने वाले विकल्प पर रुक जाता है**:यदि आप`capabilities\` सेट करते हैं, तो एंट्री केवल उन्हीं media types के लिए चलती है।

1. **स्थानीय CLI** (केवल ऑडियो; यदि इंस्टॉल हों)
   - `sherpa-onnx-offline` (आवश्यक: `SHERPA_ONNX_MODEL_DIR` जिसमें एन्कोडर/डिकोडर/जॉइनर/टोकन)
   - `whisper-cli` (`whisper-cpp`; `WHISPER_CPP_MODEL` या बंडल्ड tiny मॉडल का उपयोग)
   - `whisper` (Python CLI; मॉडल स्वचालित रूप से डाउनलोड करता है)
2. **Gemini CLI** (`gemini`) का उपयोग करते हुए `read_many_files`
3. **प्रदाता कुंजियाँ**
   - ऑडियो: OpenAI → Groq → Deepgram → Google
   - इमेज: OpenAI → Anthropic → Google → MiniMax
   - वीडियो: Google

ऑटो‑डिटेक्शन अक्षम करने के लिए, सेट करें:

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

टिप्पणी: बाइनरी डिटेक्शन macOS/Linux/Windows पर best‑effort है; सुनिश्चित करें कि CLI `PATH` पर है (हम `~` का विस्तार करते हैं), या पूर्ण कमांड पथ के साथ एक स्पष्ट CLI मॉडल सेट करें।

## क्षमताएँ (वैकल्पिक)

Shared सूचियों के लिए, OpenClaw डिफ़ॉल्ट्स का अनुमान लगा सकता है: CLI एंट्रीज़ के लिए, **`capabilities` स्पष्ट रूप से सेट करें** ताकि अप्रत्याशित matches से बचा जा सके।

- `openai`, `anthropic`, `minimax`: **इमेज**
- `google` (Gemini API): **इमेज + ऑडियो + वीडियो**
- `groq`: **ऑडियो**
- `deepgram`: **ऑडियो**

यदि आप `capabilities` छोड़ देते हैं, तो एंट्री उस सूची के लिए पात्र होती है जिसमें वह दिखाई देती है।
Understanding **best‑effort** है।

## प्रदाता समर्थन मैट्रिक्स (OpenClaw एकीकरण)

| क्षमता | प्रदाता एकीकरण                                 | टिप्पणियाँ                                                           |
| ------ | ---------------------------------------------- | -------------------------------------------------------------------- |
| इमेज   | OpenAI / Anthropic / Google / अन्य via `pi-ai` | रजिस्ट्री में कोई भी इमेज‑समर्थ मॉडल काम करता है।                    |
| ऑडियो  | OpenAI, Groq, Deepgram, Google                 | प्रदाता ट्रांसक्रिप्शन (Whisper/Deepgram/Gemini)। |
| वीडियो | Google (Gemini API)         | प्रदाता वीडियो समझ।                                                  |

## अनुशंसित प्रदाता

**इमेज**

- यदि आपका सक्रिय मॉडल इमेज का समर्थन करता है, तो उसे प्राथमिकता दें।
- अच्छे डिफ़ॉल्ट्स: `openai/gpt-5.2`, `anthropic/claude-opus-4-6`, `google/gemini-3-pro-preview`।

**ऑडियो**

- `openai/gpt-4o-mini-transcribe`, `groq/whisper-large-v3-turbo`, या `deepgram/nova-3`।
- CLI फॉलबैक: `whisper-cli` (whisper-cpp) या `whisper`।
- Deepgram सेटअप: [Deepgram (audio transcription)](/providers/deepgram)।

**वीडियो**

- `google/gemini-3-flash-preview` (तेज़), `google/gemini-3-pro-preview` (अधिक समृद्ध)।
- CLI फॉलबैक: `gemini` CLI (वीडियो/ऑडियो पर `read_file` का समर्थन करता है)।

## अटैचमेंट नीति

प्रति‑क्षमता `attachments` यह नियंत्रित करता है कि कौन‑से अटैचमेंट प्रोसेस हों:

- `mode`: `first` (डिफ़ॉल्ट) या `all`
- `maxAttachments`: प्रोसेस किए जाने वालों की संख्या सीमित करें (डिफ़ॉल्ट **1**)
- `prefer`: `first`, `last`, `path`, `url`

जब `mode: "all"`, आउटपुट्स को `[Image 1/2]`, `[Audio 2/2]`, आदि लेबल किया जाता है।

## कॉन्फ़िग उदाहरण

### 1. साझा मॉडल सूची + ओवरराइड्स

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

### 2. केवल ऑडियो + वीडियो (इमेज बंद)

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

### 3. वैकल्पिक इमेज समझ

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

### 4. मल्टी‑मोडल एकल एंट्री (स्पष्ट क्षमताएँ)

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

## स्थिति आउटपुट

जब मीडिया समझ चलती है, `/status` में एक संक्षिप्त सार पंक्ति शामिल होती है:

```
📎 Media: image ok (openai/gpt-5.2) · audio skipped (maxBytes)
```

यह प्रति‑क्षमता परिणामों और लागू होने पर चुने गए प्रदाता/मॉडल को दिखाता है।

## टिप्पणियाँ

- Errors replies को ब्लॉक नहीं करते। यदि pairing गायब है, तो पहले node device को approve करें।
- समझ अक्षम होने पर भी अटैचमेंट्स मॉडल तक भेजे जाते हैं।
- जहाँ समझ चले उसे सीमित करने के लिए `scope` का उपयोग करें (जैसे केवल DMs)।

## संबंधित दस्तावेज़

- [Configuration](/gateway/configuration)
- [छवि और मीडिया समर्थन](/nodes/images)
