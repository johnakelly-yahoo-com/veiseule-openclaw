---
summary: "Telegram बॉट समर्थन की स्थिति, क्षमताएँ और विन्यास"
read_when:
  - Telegram फीचर्स या वेबहुक्स पर काम करते समय
title: "Telegram"
---

# Telegram (Bot API)

स्थिति: grammY के माध्यम से बॉट DMs + ग्रुप्स के लिए प्रोडक्शन-रेडी। डिफ़ॉल्ट रूप से लॉन्ग-पोलिंग; वेबहुक वैकल्पिक।

## त्वरित सेटअप (शुरुआती)

1. **@BotFather** के साथ एक बॉट बनाएं ([डायरेक्ट लिंक](https://t.me/BotFather))। पुष्टि करें कि हैंडल बिल्कुल `@BotFather` है, फिर टोकन कॉपी करें।
2. टोकन सेट करें:
   - Env: `TELEGRAM_BOT_TOKEN=...`
   - या config: `channels.telegram.botToken: "..."`।
   - यदि दोनों सेट हैं, तो config को प्राथमिकता मिलेगी (env fallback केवल default-account के लिए है)।
3. Gateway प्रारंभ करें।
4. DM एक्सेस डिफ़ॉल्ट रूप से pairing है; पहली बार संपर्क पर pairing कोड को अनुमोदित करें।

न्यूनतम config:

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123:abc",
      dmPolicy: "pairing",
    },
  },
}
```

## यह क्या है

- Gateway के स्वामित्व वाला Telegram Bot API चैनल।
- Deterministic routing: उत्तर वापस Telegram पर जाते हैं; मॉडल कभी चैनल नहीं चुनता।
- DMs एजेंट के मुख्य सत्र को साझा करते हैं; समूह अलग-थलग रहते हैं (`agent:<agentId>:telegram:group:<chatId>`)।

## सेटअप (त्वरित मार्ग)

### 1. बॉट टोकन बनाएँ (BotFather)

1. Telegram खोलें और **@BotFather** से चैट करें ([डायरेक्ट लिंक](https://t.me/BotFather))। पुष्टि करें कि हैंडल बिल्कुल `@BotFather` है।
2. `/newbot` चलाएँ, फिर प्रॉम्प्ट्स का पालन करें (नाम + उपयोगकर्ता नाम जो `bot` पर समाप्त होता हो)।
3. टोकन कॉपी करें और सुरक्षित रखें।

वैकल्पिक BotFather सेटिंग्स:

- `/setjoingroups` — बॉट को समूहों में जोड़ने की अनुमति/अस्वीकृति।
- `/setprivacy` — नियंत्रित करें कि बॉट सभी समूह संदेश देखे या नहीं।

### 2. टोकन कॉन्फ़िगर करें (env या config)

उदाहरण:

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123:abc",
      dmPolicy: "pairing",
      groups: { "*": { requireMention: true } },
    },
  },
}
```

Env विकल्प: `TELEGRAM_BOT_TOKEN=...` (डिफ़ॉल्ट अकाउंट के लिए काम करता है)।
यदि env और config दोनों सेट हैं, तो config को प्राथमिकता मिलेगी।

मल्टी-अकाउंट समर्थन: प्रति-अकाउंट टोकन और वैकल्पिक `name` के साथ `channels.telegram.accounts` का उपयोग करें। शेयर्ड पैटर्न के लिए [`gateway/configuration`](/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) देखें।

3. Gateway प्रारंभ करें। टोकन रेज़ॉल्व होने पर Telegram शुरू होता है (पहले कॉन्फ़िग, फिर env फ़ॉलबैक)।
4. DM एक्सेस डिफ़ॉल्ट रूप से पेयरिंग पर सेट होता है। जब बॉट से पहली बार संपर्क किया जाए, तो कोड को स्वीकृत करें।
5. समूहों के लिए: बॉट जोड़ें, privacy/admin व्यवहार तय करें (नीचे), फिर mention gating + allowlists नियंत्रित करने के लिए `channels.telegram.groups` सेट करें।

## Token + privacy + permissions (Telegram पक्ष)

### टोकन निर्माण (BotFather)

- `/newbot` बॉट बनाता है और टोकन लौटाता है (इसे गोपनीय रखें)।
- यदि टोकन लीक हो जाए, तो @BotFather के माध्यम से इसे revoke/regenerate करें और अपना config अपडेट करें।

### समूह संदेश दृश्यता (Privacy Mode)

Telegram बॉट्स डिफ़ॉल्ट रूप से **Privacy Mode** में होते हैं, जो यह सीमित करता है कि वे कौन से ग्रुप संदेश प्राप्त करते हैं।
यदि आपके बॉट को सभी ग्रुप संदेश देखने हों, तो आपके पास दो विकल्प हैं:

- `/setprivacy` से privacy mode अक्षम करें **या**
- बॉट को समूह **admin** के रूप में जोड़ें (admin बॉट सभी संदेश प्राप्त करते हैं)।

**Note:** privacy mode बदलने पर, Telegram परिवर्तन प्रभावी होने के लिए प्रत्येक समूह से बॉट हटाकर पुनः जोड़ने की आवश्यकता करता है।

### समूह अनुमतियाँ (admin अधिकार)

एडमिन स्टेटस ग्रुप के अंदर (Telegram UI) सेट किया जाता है। एडमिन बॉट्स हमेशा सभी ग्रुप संदेश प्राप्त करते हैं, इसलिए यदि आपको पूर्ण दृश्यता चाहिए तो एडमिन का उपयोग करें।

## यह कैसे काम करता है (व्यवहार)

- इनबाउंड संदेश reply context और media placeholders के साथ साझा चैनल envelope में normalize किए जाते हैं।
- समूह उत्तरों के लिए डिफ़ॉल्ट रूप से mention आवश्यक है (native @mention या `agents.list[].groupChat.mentionPatterns` / `messages.groupChat.mentionPatterns`)।
- Multi-agent override: `agents.list[].groupChat.mentionPatterns` पर प्रति-एजेंट पैटर्न सेट करें।
- उत्तर हमेशा उसी Telegram चैट पर वापस रूट होते हैं।
- Long-polling grammY runner का उपयोग प्रति-चैट sequencing के साथ करता है; समग्र concurrency `agents.defaults.maxConcurrent` द्वारा सीमित है।
- Telegram Bot API read receipts का समर्थन नहीं करता; कोई `sendReadReceipts` विकल्प नहीं है।

## Draft streaming

OpenClaw `sendMessageDraft` का उपयोग करके Telegram DMs में आंशिक उत्तर स्ट्रीम कर सकता है।

आवश्यकताएँ:

- @BotFather में बॉट के लिए Threaded Mode सक्षम (forum topic mode)।
- केवल private chat threads (Telegram इनबाउंड संदेशों पर `message_thread_id` शामिल करता है)।
- `channels.telegram.streamMode` को `"off"` पर सेट न किया गया हो (डिफ़ॉल्ट: `"partial"`, `"block"` chunked draft updates सक्षम करता है)।

Draft streaming केवल DMs के लिए है; Telegram इसे समूहों या चैनलों में समर्थन नहीं करता।

## Formatting (Telegram HTML)

- आउटबाउंड Telegram टेक्स्ट `parse_mode: "HTML"` (Telegram द्वारा समर्थित टैग उपसमुच्चय) का उपयोग करता है।
- Markdown-जैसा इनपुट **Telegram-safe HTML** (bold/italic/strike/code/links) में रेंडर होता है; block elements को newlines/bullets के साथ टेक्स्ट में flatten किया जाता है।
- मॉडलों से आने वाला raw HTML Telegram parse त्रुटियों से बचने के लिए escaped किया जाता है।
- यदि Telegram HTML payload अस्वीकार करता है, तो OpenClaw वही संदेश plain text के रूप में पुनः प्रयास करता है।

## Commands (native + custom)

OpenClaw स्टार्टअप पर Telegram के बॉट मेनू के साथ नेटिव कमांड (जैसे `/status`, `/reset`, `/model`) रजिस्टर करता है।
आप कॉन्फ़िग के माध्यम से मेनू में कस्टम कमांड जोड़ सकते हैं:

```json5
{
  channels: {
    telegram: {
      customCommands: [
        { command: "backup", description: "Git backup" },
        { command: "generate", description: "Create an image" },
      ],
    },
  },
}
```

## सेटअप समस्या-निवारण (commands)

- लॉग्स में `setMyCommands failed` आमतौर पर यह दर्शाता है कि `api.telegram.org` के लिए outbound HTTPS/DNS अवरुद्ध है।
- यदि `sendMessage` या `sendChatAction` विफलताएँ दिखें, तो IPv6 routing और DNS जाँचें।

अधिक सहायता: [Channel troubleshooting](/channels/troubleshooting).

Notes:

- Custom commands **केवल menu entries** हैं; OpenClaw उन्हें तब तक लागू नहीं करता जब तक आप उन्हें कहीं और हैंडल न करें।
- Some commands can be handled by plugins/skills without being registered in Telegram’s command menu. These still work when typed (they just won't show up in `/commands` / the menu).
- Command नाम normalize किए जाते हैं (leading `/` हटाया जाता है, lowercased) और `a-z`, `0-9`, `_` (1–32 chars) से मेल खाने चाहिए।
- कस्टम कमांड **नेटिव कमांड को ओवरराइड नहीं कर सकते**। कॉन्फ्लिक्ट्स को अनदेखा किया जाता है और लॉग किया जाता है।
- यदि `commands.native` अक्षम है, तो केवल custom commands रजिस्टर किए जाते हैं (या यदि कोई नहीं, तो साफ़ कर दिए जाते हैं)।

### Device pairing commands (`device-pair` plugin)

If the `device-pair` plugin is installed, it adds a Telegram-first flow for pairing a new phone:

1. `/pair` generates a setup code (sent as a separate message for easy copy/paste).
2. Paste the setup code in the iOS app to connect.
3. `/pair approve` approves the latest pending device request.

More details: [Pairing](/channels/pairing#pair-via-telegram-recommended-for-ios).

## Limits

- आउटबाउंड टेक्स्ट `channels.telegram.textChunkLimit` तक chunk किया जाता है (डिफ़ॉल्ट 4000)।
- वैकल्पिक newline chunking: लंबाई chunking से पहले blank lines (paragraph boundaries) पर विभाजित करने के लिए `channels.telegram.chunkMode="newline"` सेट करें।
- Media downloads/uploads `channels.telegram.mediaMaxMb` द्वारा सीमित हैं (डिफ़ॉल्ट 5)।
- Telegram Bot API रिक्वेस्ट्स `channels.telegram.timeoutSeconds` (grammY के माध्यम से डिफ़ॉल्ट 500) के बाद टाइम आउट हो जाती हैं। लंबे हैंग से बचने के लिए कम मान सेट करें।
- ग्रुप हिस्ट्री कॉन्टेक्स्ट `channels.telegram.historyLimit` (या `channels.telegram.accounts.*.historyLimit`) का उपयोग करता है, और `messages.groupChat.historyLimit` पर फ़ॉलबैक करता है। अक्षम करने के लिए `0` सेट करें (डिफ़ॉल्ट 50)।
- DM हिस्ट्री को `channels.telegram.dmHistoryLimit` (यूज़र टर्न्स) के साथ सीमित किया जा सकता है। प्रति-यूज़र ओवरराइड्स: `channels.telegram.dms["<user_id>"].historyLimit`।

## Group activation modes

डिफ़ॉल्ट रूप से, बॉट ग्रुप्स में केवल मेंशन पर ही प्रतिक्रिया देता है (`@botname` या `agents.list[].groupChat.mentionPatterns` में पैटर्न)। इस व्यवहार को बदलने के लिए:

### Config के माध्यम से (अनुशंसित)

```json5
{
  channels: {
    telegram: {
      groups: {
        "-1001234567890": { requireMention: false }, // always respond in this group
      },
    },
  },
}
```

**महत्वपूर्ण:** `channels.telegram.groups` सेट करने से एक **allowlist** बनती है — केवल सूचीबद्ध ग्रुप्स (या `"*"`) ही स्वीकार किए जाएंगे।
फ़ोरम टॉपिक्स अपने पैरेंट ग्रुप कॉन्फ़िग (allowFrom, requireMention, skills, prompts) को इनहेरिट करते हैं, जब तक कि आप `channels.telegram.groups.<groupId>` के तहत प्रति-टॉपिक ओवरराइड्स न जोड़ें।1. .topics.<topicId>\`.

सभी समूहों को always-respond के साथ अनुमति देने के लिए:

```json5
{
  channels: {
    telegram: {
      groups: {
        "*": { requireMention: false }, // all groups, always respond
      },
    },
  },
}
```

सभी समूहों के लिए mention-only बनाए रखने के लिए (डिफ़ॉल्ट व्यवहार):

```json5
{
  channels: {
    telegram: {
      groups: {
        "*": { requireMention: true }, // or omit groups entirely
      },
    },
  },
}
```

### Command के माध्यम से (session-level)

समूह में भेजें:

- `/activation always` - सभी संदेशों पर प्रतिक्रिया दें
- `/activation mention` - mentions आवश्यक (डिफ़ॉल्ट)

**नोट:** कमांड्स केवल सत्र की स्थिति को अपडेट करते हैं। रीस्टार्ट के बाद भी स्थायी व्यवहार के लिए, config का उपयोग करें।

### Group chat ID प्राप्त करना

किसी भी समूह संदेश को `@userinfobot` या `@getidsbot` पर forward करें ताकि chat ID (जैसे `-1001234567890` जैसी नकारात्मक संख्या) दिखाई दे।

**Tip:** अपने user ID के लिए, बॉट को DM करें और यह आपके user ID (pairing संदेश) के साथ उत्तर देगा, या commands सक्षम होने पर `/whoami` का उपयोग करें।

**गोपनीयता नोट:** `@userinfobot` एक थर्ड-पार्टी बॉट है। यदि आप चाहें, तो बॉट को समूह में जोड़ें, एक संदेश भेजें, और `chat.id` पढ़ने के लिए `openclaw logs --follow` का उपयोग करें, या Bot API `getUpdates` का उपयोग करें।

## Config writes

डिफ़ॉल्ट रूप से, Telegram को चैनल इवेंट्स या `/config set|unset` द्वारा ट्रिगर किए गए config अपडेट लिखने की अनुमति है।

यह तब होता है जब:

- किसी समूह को सुपरग्रुप में अपग्रेड किया जाता है और Telegram `migrate_to_chat_id` जारी करता है (चैट ID बदल जाती है)। OpenClaw `channels.telegram.groups` को स्वचालित रूप से माइग्रेट कर सकता है।
- आप Telegram चैट में `/config set` या `/config unset` चलाते हैं (इसके लिए `commands.config: true` आवश्यक है)।

अक्षम करने के लिए:

```json5
{
  channels: { telegram: { configWrites: false } },
}
```

## Topics (forum supergroups)

Telegram फ़ोरम टॉपिक्स में प्रत्येक संदेश के लिए एक `message_thread_id` शामिल होता है। OpenClaw:

- प्रत्येक टॉपिक को अलग रखने के लिए Telegram group session key में `:topic:<threadId>` जोड़ता है।
- typing indicators भेजता है और `message_thread_id` के साथ उत्तर देता है ताकि प्रतिक्रियाएँ उसी टॉपिक में रहें।
- General topic (thread id `1`) विशेष है: संदेश भेजते समय `message_thread_id` छोड़ा जाता है (Telegram इसे अस्वीकार करता है), लेकिन typing indicators में यह शामिल रहता है।
- routing/templating के लिए template context में `MessageThreadId` + `IsForum` उजागर करता है।
- टॉपिक-विशिष्ट कॉन्फ़िगरेशन `channels.telegram.groups.<chatId>11. .topics.<threadId>` (skills, allowlists, auto-reply, system prompts, disable).
- टॉपिक configs समूह सेटिंग्स (requireMention, allowlists, skills, prompts, enabled) को विरासत में लेते हैं जब तक प्रति-टॉपिक override न किया जाए।

Private chats can include `message_thread_id` in some edge cases. OpenClaw DM सत्र कुंजी को अपरिवर्तित रखता है, लेकिन जब यह मौजूद हो तो उत्तरों/ड्राफ्ट स्ट्रीमिंग के लिए थ्रेड ID का उपयोग करता है।

## Inline Buttons

Telegram callback buttons के साथ inline keyboards का समर्थन करता है।

```json5
{
  channels: {
    telegram: {
      capabilities: {
        inlineButtons: "allowlist",
      },
    },
  },
}
```

प्रति-खाता configuration के लिए:

```json5
{
  channels: {
    telegram: {
      accounts: {
        main: {
          capabilities: {
            inlineButtons: "allowlist",
          },
        },
      },
    },
  },
}
```

Scopes:

- `off` — inline buttons अक्षम
- `dm` — केवल DMs (group targets अवरुद्ध)
- `group` — केवल समूह (DM targets अवरुद्ध)
- `all` — DMs + समूह
- `allowlist` — DMs + समूह, लेकिन केवल `allowFrom`/`groupAllowFrom` द्वारा अनुमत प्रेषक (control commands जैसे ही नियम)

डिफ़ॉल्ट: `allowlist`.
लीगेसी: `capabilities: ["inlineButtons"]` = `inlineButtons: "all"`.

### Buttons भेजना

message tool का उपयोग `buttons` पैरामीटर के साथ करें:

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  message: "Choose an option:",
  buttons: [
    [
      { text: "Yes", callback_data: "yes" },
      { text: "No", callback_data: "no" },
    ],
    [{ text: "Cancel", callback_data: "cancel" }],
  ],
}
```

जब कोई उपयोगकर्ता बटन क्लिक करता है, तो callback data एजेंट को इस फ़ॉर्मैट में संदेश के रूप में वापस भेजी जाती है:
`callback_data: value`

### Configuration विकल्प

Telegram क्षमताएँ दो स्तरों पर कॉन्फ़िगर की जा सकती हैं (ऊपर object form दिखाया गया है; legacy string arrays भी समर्थित हैं):

- `channels.telegram.capabilities`: सभी Telegram खातों पर लागू global default capability config, जब तक override न किया जाए।
- `channels.telegram.accounts.<account>.capabilities`: प्रति-अकाउंट क्षमताएँ जो उस विशिष्ट अकाउंट के लिए ग्लोबल डिफ़ॉल्ट्स को ओवरराइड करती हैं।

जब सभी Telegram बॉट्स/अकाउंट्स का व्यवहार समान होना चाहिए, तब ग्लोबल सेटिंग का उपयोग करें। जब अलग-अलग बॉट्स को अलग व्यवहार चाहिए (उदाहरण के लिए, एक अकाउंट केवल DMs संभालता है जबकि दूसरे को समूहों में अनुमति है), तब प्रति-अकाउंट कॉन्फ़िगरेशन का उपयोग करें।

## Access control (DMs + groups)

### DM access

- डिफ़ॉल्ट: `channels.telegram.dmPolicy = "pairing"`। अज्ञात प्रेषकों को एक पेयरिंग कोड मिलता है; स्वीकृति तक संदेश अनदेखे रहते हैं (कोड 1 घंटे बाद समाप्त हो जाते हैं)।
- अनुमोदन करें:
  - `openclaw pairing list telegram`
  - `openclaw pairing approve telegram <CODE>`
- पेयरिंग Telegram DMs के लिए डिफ़ॉल्ट टोकन एक्सचेंज है। विवरण: [Pairing](/channels/pairing)
- `channels.telegram.allowFrom` संख्यात्मक यूज़र IDs (अनुशंसित) या `@username` एंट्रीज़ स्वीकार करता है। यह बॉट का यूज़रनेम नहीं है; मानव प्रेषक की ID का उपयोग करें। विज़ार्ड `@username` स्वीकार करता है और जब संभव हो तो उसे संख्यात्मक ID में रिज़ॉल्व करता है।

#### अपना Telegram user ID ढूँढना

अधिक सुरक्षित (कोई third-party बॉट नहीं):

1. Gateway प्रारंभ करें और अपने बॉट को DM करें।
2. `openclaw logs --follow` चलाएँ और `from.id` देखें।

वैकल्पिक (official Bot API):

1. अपने बॉट को DM करें।
2. अपने बॉट टोकन के साथ updates प्राप्त करें और `message.from.id` पढ़ें:

   ```bash
   curl "https://api.telegram.org/bot<bot_token>/getUpdates"
   ```

Third-party (कम निजी):

- `@userinfobot` या `@getidsbot` को DM करें और लौटाया गया user id उपयोग करें।

### Group access

दो स्वतंत्र नियंत्रण:

\*\*1. **कौन से समूह अनुमत हैं** (`channels.telegram.groups` के माध्यम से समूह allowlist):

- कोई `groups` config नहीं = सभी समूह अनुमत
- `groups` config के साथ = केवल सूचीबद्ध समूह या `"*"` अनुमत
- उदाहरण: `"groups": { "-1001234567890": {}, "*": {} }` सभी समूहों की अनुमति देता है

\*\*2. **कौन से प्रेषक अनुमत हैं** (`channels.telegram.groupPolicy` के माध्यम से प्रेषक फ़िल्टरिंग):

- `"open"` = अनुमत समूहों के सभी प्रेषक संदेश भेज सकते हैं
- `"allowlist"` = केवल `channels.telegram.groupAllowFrom` में प्रेषक संदेश भेज सकते हैं
- `"disabled"` = कोई भी समूह संदेश स्वीकार नहीं
  डिफ़ॉल्ट `groupPolicy: "allowlist"` है (जब तक आप `groupAllowFrom` न जोड़ें, तब तक अवरुद्ध)।

अधिकांश उपयोगकर्ता चाहते हैं: `groupPolicy: "allowlist"` + `groupAllowFrom` + `channels.telegram.groups` में सूचीबद्ध विशिष्ट समूह

किसी विशिष्ट समूह में **किसी भी सदस्य** को बात करने की अनुमति देने के लिए (जबकि control commands केवल authorized senders तक सीमित रहें), प्रति-समूह override सेट करें:

```json5
{
  channels: {
    telegram: {
      groups: {
        "-1001234567890": {
          groupPolicy: "open",
          requireMention: false,
        },
      },
    },
  },
}
```

## Long-polling बनाम webhook

- डिफ़ॉल्ट: long-polling (कोई सार्वजनिक URL आवश्यक नहीं)।
- Webhook मोड: `channels.telegram.webhookUrl` और `channels.telegram.webhookSecret` सेट करें (वैकल्पिक `channels.telegram.webhookPath`)।
  - local listener `0.0.0.0:8787` पर bind करता है और डिफ़ॉल्ट रूप से `POST /telegram-webhook` सर्व करता है।
  - यदि आपका public URL अलग है, तो reverse proxy का उपयोग करें और `channels.telegram.webhookUrl` को public endpoint पर point करें।

## Reply threading

Telegram टैग्स के माध्यम से वैकल्पिक threaded replies का समर्थन करता है:

- `[[reply_to_current]]` -- triggering संदेश को reply।
- `[[reply_to:<id>]]` -- किसी विशिष्ट message id को reply।

`channels.telegram.replyToMode` द्वारा नियंत्रित:

- `first` (डिफ़ॉल्ट), `all`, `off`।

## Audio messages (voice बनाम file)

Telegram **वॉइस नोट्स** (गोल बबल) और **ऑडियो फ़ाइल्स** (मेटाडेटा कार्ड) में अंतर करता है।
OpenClaw बैकवर्ड कम्पैटिबिलिटी के लिए डिफ़ॉल्ट रूप से ऑडियो फ़ाइल्स का उपयोग करता है।

एजेंट उत्तरों में voice note bubble को मजबूर करने के लिए, उत्तर में कहीं भी यह टैग शामिल करें:

- `[[audio_as_voice]]` — audio को file के बजाय voice note के रूप में भेजें।

डिलीवर किए गए टेक्स्ट से टैग हटा दिया जाता है। अन्य चैनल इस टैग को अनदेखा करते हैं।

message tool भेजने के लिए, voice-compatible audio `media` URL के साथ `asVoice: true` सेट करें
(जब media मौजूद हो, तो `message` वैकल्पिक है):

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/voice.ogg",
  asVoice: true,
}
```

## Video messages (video vs video note)

Telegram distinguishes **video notes** (round bubble) from **video files** (rectangular).
OpenClaw defaults to video files.

For message tool sends, set `asVideoNote: true` with a video `media` URL:

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/video.mp4",
  asVideoNote: true,
}
```

(Note: Video notes do not support captions. If you provide a message text, it will be sent as a separate message.)

## Stickers

OpenClaw बुद्धिमान caching के साथ Telegram stickers प्राप्त करने और भेजने का समर्थन करता है।

### Stickers प्राप्त करना

जब कोई उपयोगकर्ता sticker भेजता है, तो OpenClaw sticker प्रकार के आधार पर इसे संभालता है:

- **स्टैटिक स्टिकर्स (WEBP):** डाउनलोड किए जाते हैं और विज़न के माध्यम से प्रोसेस किए जाते हैं। स्टिकर संदेश सामग्री में `<media:sticker>` प्लेसहोल्डर के रूप में दिखाई देता है।
- **Animated stickers (TGS):** छोड़े जाते हैं (Lottie फ़ॉर्मैट प्रोसेसिंग के लिए समर्थित नहीं)।
- **Video stickers (WEBM):** छोड़े जाते हैं (video फ़ॉर्मैट प्रोसेसिंग के लिए समर्थित नहीं)।

Stickers प्राप्त करते समय उपलब्ध template context field:

- `Sticker` — object जिसमें:
  - `emoji` — sticker से जुड़ा emoji
  - `setName` — sticker set का नाम
  - `fileId` — Telegram file ID (वही sticker वापस भेजने के लिए)
  - `fileUniqueId` — cache lookup के लिए stable ID
  - `cachedDescription` — उपलब्ध होने पर cached vision description

### Sticker cache

स्टिकर्स को विवरण उत्पन्न करने के लिए AI की विज़न क्षमताओं के माध्यम से प्रोसेस किया जाता है। चूँकि वही स्टिकर्स अक्सर बार-बार भेजे जाते हैं, OpenClaw अनावश्यक API कॉल्स से बचने के लिए इन विवरणों को कैश करता है।

**यह कैसे काम करता है:**

1. **पहली बार सामना:** स्टिकर इमेज को विज़न विश्लेषण के लिए AI को भेजा जाता है। AI एक विवरण उत्पन्न करता है (उदाहरण के लिए, "एक कार्टून बिल्ली उत्साहपूर्वक हाथ हिलाती हुई").
2. **Cache storage:** विवरण sticker के file ID, emoji और set name के साथ सहेजा जाता है।
3. **बाद के अवसर:** जब वही स्टिकर फिर से देखा जाता है, तो कैश किया हुआ विवरण सीधे उपयोग किया जाता है। इमेज को AI को नहीं भेजा जाता।

**Cache location:** `~/.openclaw/telegram/sticker-cache.json`

**Cache entry format:**

```json
{
  "fileId": "CAACAgIAAxkBAAI...",
  "fileUniqueId": "AgADBAADb6cxG2Y",
  "emoji": "👋",
  "setName": "CoolCats",
  "description": "A cartoon cat waving enthusiastically",
  "cachedAt": "2026-01-15T10:30:00.000Z"
}
```

**लाभ:**

- समान sticker के लिए दोहराए गए vision calls से बचकर API लागत कम होती है
- cached stickers के लिए तेज़ प्रतिक्रिया समय (कोई vision प्रोसेसिंग देरी नहीं)
- cached विवरणों के आधार पर sticker search कार्यक्षमता सक्षम होती है

स्टिकर्स प्राप्त होने पर कैश स्वचालित रूप से भर जाता है। मैनुअल कैश प्रबंधन की आवश्यकता नहीं होती।

### Stickers भेजना

एजेंट `sticker` और `sticker-search` एक्शन्स का उपयोग करके स्टिकर्स भेज और खोज सकता है। ये डिफ़ॉल्ट रूप से अक्षम होते हैं और config में सक्षम किए जाने चाहिए:

```json5
{
  channels: {
    telegram: {
      actions: {
        sticker: true,
      },
    },
  },
}
```

**Sticker भेजें:**

```json5
{
  action: "sticker",
  channel: "telegram",
  to: "123456789",
  fileId: "CAACAgIAAxkBAAI...",
}
```

Parameters:

- `fileId` (आवश्यक) — स्टिकर का Telegram फ़ाइल ID। इसे स्टिकर प्राप्त करते समय `Sticker.fileId` से, या `sticker-search` परिणाम से प्राप्त करें।
- `replyTo` (optional) — reply करने के लिए message ID।
- `threadId` (optional) — forum topics के लिए message thread ID।

**Stickers खोजें:**

एजेंट विवरण, emoji या set name के आधार पर cached stickers खोज सकता है:

```json5
{
  action: "sticker-search",
  channel: "telegram",
  query: "cat waving",
  limit: 5,
}
```

Cache से मिलते-जुलते stickers लौटाता है:

```json5
{
  ok: true,
  count: 2,
  stickers: [
    {
      fileId: "CAACAgIAAxkBAAI...",
      emoji: "👋",
      description: "A cartoon cat waving enthusiastically",
      setName: "CoolCats",
    },
  ],
}
```

खोज विवरण टेक्स्ट, emoji characters और set names में fuzzy matching का उपयोग करती है।

**Threading के साथ उदाहरण:**

```json5
{
  action: "sticker",
  channel: "telegram",
  to: "-1001234567890",
  fileId: "CAACAgIAAxkBAAI...",
  replyTo: 42,
  threadId: 123,
}
```

## Streaming (drafts)

Telegram एजेंट के उत्तर जनरेट करते समय **ड्राफ्ट बबल्स** स्ट्रीम कर सकता है।
OpenClaw Bot API `sendMessageDraft` (वास्तविक संदेश नहीं) का उपयोग करता है और फिर अंतिम उत्तर को सामान्य संदेश के रूप में भेजता है।

आवश्यकताएँ (Telegram Bot API 9.3+):

- **topics सक्षम private chats** (बॉट के लिए forum topic mode)।
- इनकमिंग संदेशों में `message_thread_id` (private topic thread) शामिल होना चाहिए।
- समूह/supergroups/channels के लिए streaming अनदेखा किया जाता है।

Config:

- `channels.telegram.streamMode: "off" | "partial" | "block"` (डिफ़ॉल्ट: `partial`)
  - `partial`: नवीनतम streaming टेक्स्ट के साथ draft bubble अपडेट करें।
  - `block`: draft bubble को बड़े blocks (chunked) में अपडेट करें।
  - `off`: draft streaming अक्षम करें।
- वैकल्पिक (केवल `streamMode: "block"` के लिए):
  - `channels.telegram.draftChunk: { minChars?, maxChars?, breakPreference?  }`
    - defaults: `minChars: 200`, `maxChars: 800`, `breakPreference: "paragraph"` (`channels.telegram.textChunkLimit` तक clamp किया गया)।

Note: draft streaming is separate from **block streaming** (channel messages).
Block streaming is off by default and requires `channels.telegram.blockStreaming: true`
if you want early Telegram messages instead of draft updates.

Reasoning stream (केवल Telegram):

- `/reasoning stream` उत्तर जनरेट होते समय reasoning को draft bubble में स्ट्रीम करता है,
  फिर बिना reasoning के अंतिम उत्तर भेजता है।
- If `channels.telegram.streamMode` is `off`, reasoning stream is disabled.
  More context: [Streaming + chunking](/concepts/streaming).

## Retry policy

Outbound Telegram API calls retry on transient network/429 errors with exponential backoff and jitter. Configure via `channels.telegram.retry`. See [Retry policy](/concepts/retry).

## Agent tool (messages + reactions)

- Tool: `telegram` with `sendMessage` action (`to`, `content`, optional `mediaUrl`, `replyToMessageId`, `messageThreadId`)।
- Tool: `telegram` with `react` action (`chatId`, `messageId`, `emoji`)।
- Tool: `telegram` with `deleteMessage` action (`chatId`, `messageId`)।
- Reaction removal semantics: [/tools/reactions](/tools/reactions) देखें।
- Tool gating: `channels.telegram.actions.reactions`, `channels.telegram.actions.sendMessage`, `channels.telegram.actions.deleteMessage` (डिफ़ॉल्ट: सक्षम), और `channels.telegram.actions.sticker` (डिफ़ॉल्ट: अक्षम)।

## Reaction notifications

**How reactions work:**
Telegram reactions arrive as **separate `message_reaction` events**, not as properties in message payloads. When a user adds a reaction, OpenClaw:

1. Telegram API से `message_reaction` update प्राप्त करता है
2. इसे इस फ़ॉर्मैट के साथ **system event** में बदलता है: `"Telegram reaction added: {emoji} by {user} on msg {id}"`
3. system event को नियमित संदेशों के समान **session key** का उपयोग करके enqueue करता है
4. जब उसी बातचीत में अगला संदेश आता है, तो system events drain होकर एजेंट के context में prepend हो जाते हैं

एजेंट reactions को बातचीत इतिहास में **system notifications** के रूप में देखता है, न कि message metadata के रूप में।

**Configuration:**

- `channels.telegram.reactionNotifications`: नियंत्रित करता है कि कौन-सी reactions notifications ट्रिगर करें
  - `"off"` — सभी reactions अनदेखी करें
  - `"own"` — जब उपयोगकर्ता बॉट संदेशों पर react करें तो notify करें (best-effort; in-memory) (डिफ़ॉल्ट)
  - `"all"` — सभी reactions के लिए notify करें

- `channels.telegram.reactionLevel`: एजेंट की reaction क्षमता नियंत्रित करता है
  - `"off"` — एजेंट messages पर react नहीं कर सकता
  - `"ack"` — बॉट acknowledgment reactions भेजता है (प्रोसेसिंग के दौरान 👀) (डिफ़ॉल्ट)
  - `"minimal"` — एजेंट सीमित रूप से react कर सकता है (मार्गदर्शन: हर 5–10 exchanges में 1)
  - `"extensive"` — उपयुक्त होने पर एजेंट उदारता से react कर सकता है

**Forum groups:** Reactions in forum groups include `message_thread_id` and use session keys like `agent:main:telegram:group:{chatId}:topic:{threadId}`. This ensures reactions and messages in the same topic stay together.

**Example config:**

```json5
{
  channels: {
    telegram: {
      reactionNotifications: "all", // See all reactions
      reactionLevel: "minimal", // Agent can react sparingly
    },
  },
}
```

**Requirements:**

- Telegram bots को `allowed_updates` में स्पष्ट रूप से `message_reaction` अनुरोध करना होता है (OpenClaw द्वारा स्वचालित रूप से कॉन्फ़िगर)
- Webhook मोड के लिए, reactions webhook `allowed_updates` में शामिल होती हैं
- Polling मोड के लिए, reactions `getUpdates` `allowed_updates` में शामिल होती हैं

## Delivery targets (CLI/cron)

- लक्ष्य के रूप में chat id (`123456789`) या username (`@name`) का उपयोग करें।
- उदाहरण: `openclaw message send --channel telegram --target 123456789 --message "hi"`।

## Troubleshooting

**समूह में non-mention संदेशों पर बॉट प्रतिक्रिया नहीं देता:**

- यदि आपने `channels.telegram.groups.*.requireMention=false` सेट किया है, तो Telegram Bot API **privacy mode** अक्षम होना चाहिए।
  - BotFather: `/setprivacy` → **Disable** (फिर समूह से बॉट हटाकर पुनः जोड़ें)
- `openclaw channels status` चेतावनी दिखाता है जब config unmentioned group messages की अपेक्षा करता है।
- `openclaw channels status --probe` स्पष्ट numeric group IDs के लिए सदस्यता की अतिरिक्त जाँच कर सकता है (यह wildcard `"*"` नियमों का audit नहीं कर सकता)।
- Quick test: `/activation always` (केवल session; स्थायित्व के लिए config का उपयोग करें)

**बॉट समूह संदेश बिल्कुल नहीं देख पा रहा:**

- यदि `channels.telegram.groups` सेट है, तो समूह सूचीबद्ध होना चाहिए या `"*"` का उपयोग करना चाहिए
- @BotFather में Privacy Settings जाँचें → "Group Privacy" **OFF** होना चाहिए
- सत्यापित करें कि बॉट वास्तव में सदस्य है (सिर्फ admin नहीं, बिना read access)
- Gateway logs जाँचें: `openclaw logs --follow` ("skipping group message" देखें)

**बॉट mentions पर प्रतिक्रिया देता है लेकिन `/activation always` पर नहीं:**

- `/activation` command session state अपडेट करता है लेकिन config में स्थायी नहीं करता
- स्थायी व्यवहार के लिए, समूह को `channels.telegram.groups` में `requireMention: false` के साथ जोड़ें

**`/status` जैसे commands काम नहीं करते:**

- सुनिश्चित करें कि आपका Telegram user ID अधिकृत है (pairing या `channels.telegram.allowFrom` के माध्यम से)
- commands को authorization की आवश्यकता होती है, भले ही समूह में `groupPolicy: "open"` हो

**Node 22+ पर long-polling तुरंत abort हो जाता है (अक्सर proxies/custom fetch के साथ):**

- Node 22+ `AbortSignal` instances के बारे में अधिक सख्त है; foreign signals तुरंत `fetch` calls abort कर सकते हैं।
- abort signals को normalize करने वाले OpenClaw build में अपग्रेड करें, या अपग्रेड तक Gateway को Node 20 पर चलाएँ।

**Bot starts, then silently stops responding (or logs `HttpError: Network request ... failed`):**

- Some hosts resolve `api.telegram.org` to IPv6 first. If your server does not have working IPv6 egress, grammY can get stuck on IPv6-only requests.
- समाधान: IPv6 egress सक्षम करें **या** `api.telegram.org` के लिए IPv4 resolution को मजबूर करें (उदाहरण: IPv4 A record का उपयोग करके `/etc/hosts` entry जोड़ें, या अपने OS DNS stack में IPv4 को प्राथमिकता दें), फिर Gateway पुनः प्रारंभ करें।
- Quick check: DNS क्या लौटाता है यह पुष्टि करने के लिए `dig +short api.telegram.org A` और `dig +short api.telegram.org AAAA`।

## Configuration reference (Telegram)

पूर्ण configuration: [Configuration](/gateway/configuration)

Provider विकल्प:

- `channels.telegram.enabled`: चैनल स्टार्टअप सक्षम/अक्षम करें।
- `channels.telegram.botToken`: बॉट टोकन (BotFather)।
- `channels.telegram.tokenFile`: फ़ाइल पथ से टोकन पढ़ें।
- `channels.telegram.dmPolicy`: `pairing | allowlist | open | disabled` (डिफ़ॉल्ट: pairing)।
- `channels.telegram.allowFrom`: DM allowlist (ids/usernames). `open` requires `"*"`.
- `channels.telegram.groupPolicy`: `open | allowlist | disabled` (डिफ़ॉल्ट: allowlist)।
- `channels.telegram.groupAllowFrom`: group sender allowlist (ids/usernames)।
- `channels.telegram.groups`: प्रति-समूह defaults + allowlist (global defaults के लिए `"*"` का उपयोग करें)।
  - `channels.telegram.groups.<id>.groupPolicy`: groupPolicy (`open | allowlist | disabled`) के लिए प्रति-समूह ओवरराइड।
  - `channels.telegram.groups.<id>.requireMention`: मेंशन गेटिंग का डिफ़ॉल्ट।
  - `channels.telegram.groups.<id>.skills`: स्किल फ़िल्टर (omit = सभी स्किल्स, empty = कोई नहीं)।
  - `channels.telegram.groups.<id>.allowFrom`: प्रति-समूह प्रेषक allowlist ओवरराइड।
  - `channels.telegram.groups.<id>.systemPrompt`: समूह के लिए अतिरिक्त सिस्टम प्रॉम्प्ट।
  - `channels.telegram.groups.<id>.enabled`: `false` होने पर समूह को अक्षम करें।
  - `channels.telegram.groups.<id>.topics.<threadId>.*`: प्रति-टॉपिक ओवरराइड (समूह जैसे ही फ़ील्ड्स)।
  - `channels.telegram.groups.<id>.topics.<threadId>.groupPolicy`: groupPolicy (`open | allowlist | disabled`) के लिए प्रति-टॉपिक ओवरराइड।
  - `channels.telegram.groups.<id>.topics.<threadId>.requireMention`: per-topic mention gating override.
- `channels.telegram.capabilities.inlineButtons`: `off | dm | group | all | allowlist` (डिफ़ॉल्ट: allowlist)।
- `channels.telegram.accounts.<account>.capabilities.inlineButtons`: per-account override.
- `channels.telegram.replyToMode`: `off | first | all` (डिफ़ॉल्ट: `first`)।
- `channels.telegram.textChunkLimit`: outbound chunk size (chars)।
- `channels.telegram.chunkMode`: `length` (डिफ़ॉल्ट) या लंबाई chunking से पहले blank lines (paragraph boundaries) पर विभाजित करने के लिए `newline`।
- `channels.telegram.linkPreview`: outbound संदेशों के लिए link previews toggle करें (डिफ़ॉल्ट: true)।
- `channels.telegram.streamMode`: `off | partial | block` (draft streaming)।
- `channels.telegram.mediaMaxMb`: inbound/outbound media cap (MB)।
- `channels.telegram.retry`: outbound Telegram API calls के लिए retry policy (attempts, minDelayMs, maxDelayMs, jitter)।
- `channels.telegram.network.autoSelectFamily`: override Node autoSelectFamily (true=enable, false=disable). Defaults to disabled on Node 22 to avoid Happy Eyeballs timeouts.
- `channels.telegram.proxy`: Bot API calls के लिए proxy URL (SOCKS/HTTP)।
- `channels.telegram.webhookUrl`: webhook मोड सक्षम करें (इसके लिए `channels.telegram.webhookSecret` आवश्यक)।
- `channels.telegram.webhookSecret`: webhook secret (जब webhookUrl सेट हो तो आवश्यक)।
- `channels.telegram.webhookPath`: local webhook path (डिफ़ॉल्ट `/telegram-webhook`)।
- `channels.telegram.actions.reactions`: Telegram tool reactions gate करें।
- `channels.telegram.actions.sendMessage`: Telegram tool message sends gate करें।
- `channels.telegram.actions.deleteMessage`: Telegram tool message deletes gate करें।
- `channels.telegram.actions.sticker`: Telegram sticker actions — send और search gate करें (डिफ़ॉल्ट: false)।
- `channels.telegram.reactionNotifications`: `off | own | all` — नियंत्रित करें कि कौन-सी reactions system events ट्रिगर करें (डिफ़ॉल्ट: सेट न होने पर `own`)।
- `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` — एजेंट की reaction क्षमता नियंत्रित करें (डिफ़ॉल्ट: सेट न होने पर `minimal`)।

Related global विकल्प:

- `agents.list[].groupChat.mentionPatterns` (mention gating patterns)।
- `messages.groupChat.mentionPatterns` (global fallback)।
- `commands.native` (defaults to `"auto"` → on for Telegram/Discord, off for Slack), `commands.text`, `commands.useAccessGroups` (command behavior). Override with `channels.telegram.commands.native`.
- `messages.responsePrefix`, `messages.ackReaction`, `messages.ackReactionScope`, `messages.removeAckAfterReply`।
