---
title: "एजेंट रनटाइम"
---

# एजेंट रनटाइम 🤖

OpenClaw **pi-mono** से व्युत्पन्न एक एकल एंबेडेड एजेंट रनटाइम चलाता है।

## वर्कस्पेस (आवश्यक)

OpenClaw एकल एजेंट वर्कस्पेस डायरेक्टरी (`agents.defaults.workspace`) का उपयोग करता है, जो टूल्स और संदर्भ के लिए एजेंट की **एकमात्र** कार्यशील डायरेक्टरी (`cwd`) होती है।

अनुशंसित: यदि अनुपस्थित हो तो `openclaw setup` का उपयोग करके `~/.openclaw/openclaw.json` बनाएँ और वर्कस्पेस फ़ाइलें प्रारंभ करें।

पूर्ण वर्कस्पेस लेआउट + बैकअप मार्गदर्शिका: [Agent workspace](/concepts/agent-workspace)

यदि `agents.defaults.sandbox` सक्षम है, तो गैर-मुख्य सत्र इसे
`agents.defaults.sandbox.workspaceRoot` के अंतर्गत प्रति-सत्र वर्कस्पेस के साथ ओवरराइड कर सकते हैं (देखें
[Gateway configuration](/gateway/configuration))।

## बूटस्ट्रैप फ़ाइलें (इंजेक्ट की गई)

`agents.defaults.workspace` के भीतर, OpenClaw इन उपयोगकर्ता-संपादन योग्य फ़ाइलों की अपेक्षा करता है:

- `AGENTS.md` — संचालन निर्देश + “मेमोरी”
- `SOUL.md` — व्यक्तित्व, सीमाएँ, स्वर
- `TOOLS.md` — उपयोगकर्ता-रक्षित टूल नोट्स (जैसे `imsg`, `sag`, परंपराएँ)
- `BOOTSTRAP.md` — एक-बार का प्रथम-रन अनुष्ठान (पूर्ण होने के बाद हटाया जाता है)
- `IDENTITY.md` — एजेंट नाम/वाइब/इमोजी
- `USER.md` — उपयोगकर्ता प्रोफ़ाइल + पसंदीदा संबोधन

नए सत्र के पहले टर्न पर, OpenClaw इन फ़ाइलों की सामग्री को सीधे एजेंट संदर्भ में इंजेक्ट करता है।

6. Blank files को छोड़ दिया जाता है। 7. बड़े files को trim और truncate किया जाता है, एक marker के साथ, ताकि prompts lean रहें (पूरा content पढ़ने के लिए file देखें)।

यदि कोई फ़ाइल अनुपस्थित है, तो OpenClaw एकल “missing file” मार्कर पंक्ति इंजेक्ट करता है (और `openclaw setup` एक सुरक्षित डिफ़ॉल्ट टेम्पलेट बनाएगा)।

8. `BOOTSTRAP.md` केवल **brand new workspace** के लिए बनाया जाता है (कोई अन्य bootstrap files मौजूद न हों)। 9. यदि आप ritual पूरा करने के बाद इसे delete कर देते हैं, तो बाद की restarts पर इसे दोबारा नहीं बनाया जाना चाहिए।

बूटस्ट्रैप फ़ाइल निर्माण को पूरी तरह अक्षम करने के लिए (प्री-सीडेड वर्कस्पेस के लिए), सेट करें:

```json5
{ agent: { skipBootstrap: true } }
```

## बिल्ट-इन टूल्स

10. Core tools (read/exec/edit/write और संबंधित system tools) हमेशा उपलब्ध रहते हैं,
    tool policy के अधीन। 11. `apply_patch` optional है और
    `tools.exec.applyPatch` द्वारा gated है। 12. `TOOLS.md` यह नियंत्रित नहीं करता कि कौन से tools मौजूद हैं; यह
    इस बात के लिए guidance है कि _आप_ उन्हें कैसे उपयोग करना चाहते हैं।

## Skills

OpenClaw तीन स्थानों से Skills लोड करता है (नाम टकराव पर वर्कस्पेस जीतता है):

- बंडल्ड (इंस्टॉल के साथ शिप्ड)
- प्रबंधित/लोकल: `~/.openclaw/skills`
- वर्कस्पेस: `<workspace>/skills`

Skills को कॉन्फ़िग/एन्व द्वारा गेट किया जा सकता है (देखें `skills` [Gateway configuration](/gateway/configuration) में)।

## pi-mono एकीकरण

OpenClaw pi-mono कोडबेस (मॉडल/टूल्स) के हिस्सों का पुन: उपयोग करता है, लेकिन **सत्र प्रबंधन, डिस्कवरी, और टूल वायरिंग OpenClaw के स्वामित्व में हैं**।

- कोई pi-coding एजेंट रनटाइम नहीं।
- `~/.pi/agent` या `<workspace>/.pi` सेटिंग्स पर विचार नहीं किया जाता।

## सत्र

सत्र ट्रांसक्रिप्ट JSONL के रूप में यहाँ संग्रहीत होते हैं:

- `~/.openclaw/agents/<agentId>/sessions/<SessionId>.jsonl`

13. Session ID स्थिर होता है और OpenClaw द्वारा चुना जाता है।
14. Legacy Pi/Tau session folders **नहीं** पढ़े जाते हैं।

## स्ट्रीमिंग के दौरान स्टीयरिंग

15. जब queue mode `steer` होता है, inbound messages को current run में inject किया जाता है।
16. Queue को **हर tool call के बाद** check किया जाता है; यदि कोई queued message मौजूद है,
    current assistant message से remaining tool calls को skip कर दिया जाता है (error tool
    results के साथ "Skipped due to queued user message."), फिर queued user
    message को अगले assistant response से पहले inject किया जाता है।

17. जब queue mode `followup` या `collect` होता है, inbound messages को तब तक रोका जाता है जब तक
    current turn समाप्त न हो जाए, फिर queued payloads के साथ एक नया agent turn शुरू होता है। 18. Mode + debounce/cap behavior के लिए देखें
    [Queue](/concepts/queue)।

19. Block streaming पूर्ण हुए assistant blocks को जैसे ही वे खत्म होते हैं भेज देता है; यह
    **default रूप से off** होता है (`agents.defaults.blockStreamingDefault: "off"`)।
20. Boundary को `agents.defaults.blockStreamingBreak` के माध्यम से tune करें (`text_end` बनाम `message_end`; default `text_end`)।
21. Soft block chunking को `agents.defaults.blockStreamingChunk` से नियंत्रित करें (default
    800–1200 chars; paragraph breaks को प्राथमिकता, फिर newlines; sentences अंत में)।
22. Streamed chunks को `agents.defaults.blockStreamingCoalesce` के साथ coalesce करें ताकि
    single-line spam कम हो (send से पहले idle-based merging)। 23. Non-Telegram channels में block replies सक्षम करने के लिए
    explicit `*.blockStreaming: true` की आवश्यकता होती है।
23. Verbose tool summaries tool start पर emit होते हैं (कोई debounce नहीं); Control UI
    tool output को agent events के माध्यम से stream करता है जब उपलब्ध हो।
24. अधिक विवरण: [Streaming + chunking](/concepts/streaming)।

## मॉडल रेफ़्स

कॉन्फ़िग में मॉडल रेफ़्स (उदाहरण के लिए `agents.defaults.model` और `agents.defaults.models`) को **पहले** `/` पर विभाजित करके पार्स किया जाता है।

- मॉडल कॉन्फ़िगर करते समय `provider/model` का उपयोग करें।
- यदि मॉडल ID में स्वयं `/` (OpenRouter-शैली) शामिल है, तो प्रदाता प्रीफ़िक्स शामिल करें (उदाहरण: `openrouter/moonshotai/kimi-k2`)।
- यदि आप प्रदाता छोड़ देते हैं, तो OpenClaw इनपुट को किसी उपनाम या **डिफ़ॉल्ट प्रदाता** के लिए मॉडल मानता है (यह केवल तब काम करता है जब मॉडल ID में कोई `/` न हो)।

## विन्यास (न्यूनतम)

कम-से-कम, सेट करें:

- `agents.defaults.workspace`
- `channels.whatsapp.allowFrom` (दृढ़ता से अनुशंसित)

---

_आगे: [Group Chats](/channels/group-messages)_ 🦞

