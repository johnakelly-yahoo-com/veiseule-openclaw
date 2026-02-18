---
title: "Microsoft Teams"
---

# Microsoft Teams (प्लगइन)

> "जो यहाँ प्रवेश करे, सारी आशा त्याग दे।"

अद्यतन: 2026-01-21

स्थिति: टेक्स्ट + DM अटैचमेंट्स समर्थित हैं; चैनल/ग्रुप में फ़ाइल भेजने के लिए `sharePointSiteId` + Graph अनुमतियाँ आवश्यक हैं (देखें [ग्रुप चैट्स में फ़ाइलें भेजना](#sending-files-in-group-chats))। पोल्स Adaptive Cards के माध्यम से भेजे जाते हैं।

## प्लगइन आवश्यक

Microsoft Teams एक प्लगइन के रूप में उपलब्ध है और कोर इंस्टॉल में शामिल नहीं है।

**Breaking change (2026.1.15):** MS Teams moved out of core. If you use it, you must install the plugin.

स्पष्टीकरण: इससे कोर इंस्टॉल हल्के रहते हैं और MS Teams की निर्भरताएँ स्वतंत्र रूप से अपडेट हो सकती हैं।

CLI के माध्यम से इंस्टॉल करें (npm रजिस्ट्री):

```bash
openclaw plugins install @openclaw/msteams
```

लोकल चेकआउट (जब git रिपॉजिटरी से चला रहे हों):

```bash
openclaw plugins install ./extensions/msteams
```

यदि आप configure/onboarding के दौरान Teams चुनते हैं और git चेकआउट पाया जाता है, तो OpenClaw स्वतः लोकल इंस्टॉल पाथ ऑफ़र करेगा।

विवरण: [Plugins](/tools/plugin)

## त्वरित सेटअप (शुरुआती)

1. Microsoft Teams प्लगइन इंस्टॉल करें।
2. एक **Azure Bot** बनाएँ (App ID + client secret + tenant ID)।
3. उन क्रेडेंशियल्स के साथ OpenClaw को कॉन्फ़िगर करें।
4. `/api/messages` (डिफ़ॉल्ट रूप से पोर्ट 3978) को सार्वजनिक URL या टनल के माध्यम से एक्सपोज़ करें।
5. Teams ऐप पैकेज इंस्टॉल करें और Gateway प्रारंभ करें।

न्यूनतम विन्यास:

```json5
{
  channels: {
    msteams: {
      enabled: true,
      appId: "<APP_ID>",
      appPassword: "<APP_PASSWORD>",
      tenantId: "<TENANT_ID>",
      webhook: { port: 3978, path: "/api/messages" },
    },
  },
}
```

नोट: ग्रुप चैट्स डिफ़ॉल्ट रूप से ब्लॉक हैं (`channels.msteams.groupPolicy: "allowlist"`)। ग्रुप रिप्लाई की अनुमति देने के लिए `channels.msteams.groupAllowFrom` सेट करें (या किसी भी सदस्य को अनुमति देने के लिए `groupPolicy: "open"` का उपयोग करें, mention-gated)।

## लक्ष्य

- Teams DMs, समूह चैट या चैनलों के माध्यम से OpenClaw से बात करना।
- रूटिंग को निर्धारक रखना: उत्तर हमेशा उसी चैनल में वापस जाएँ जहाँ से वे आए।
- सुरक्षित चैनल व्यवहार को डिफ़ॉल्ट रखना (जब तक अन्यथा कॉन्फ़िगर न हो, mentions आवश्यक)।

## Config लिखना

डिफ़ॉल्ट रूप से, Microsoft Teams को `/config set|unset` द्वारा ट्रिगर किए गए config अपडेट लिखने की अनुमति है (इसके लिए `commands.config: true` आवश्यक है)।

अक्षम करने के लिए:

```json5
{
  channels: { msteams: { configWrites: false } },
}
```

## प्रवेश नियंत्रण (DMs + समूह)

**DM प्रवेश**

- डिफ़ॉल्ट: `channels.msteams.dmPolicy = "pairing"`। अज्ञात प्रेषकों को स्वीकृति मिलने तक अनदेखा किया जाता है।
- `channels.msteams.allowFrom` AAD object IDs, UPNs, या display names स्वीकार करता है। जब क्रेडेंशियल्स अनुमति देते हैं, तो विज़ार्ड Microsoft Graph के माध्यम से नामों को IDs में resolve करता है।

**समूह प्रवेश**

- डिफ़ॉल्ट: `channels.msteams.groupPolicy = "allowlist"` (जब तक आप `groupAllowFrom` नहीं जोड़ते, तब तक ब्लॉक रहता है)। जब सेट न हो, तो डिफ़ॉल्ट को ओवरराइड करने के लिए `channels.defaults.groupPolicy` का उपयोग करें।
- `channels.msteams.groupAllowFrom` नियंत्रित करता है कि समूह चैट/चैनलों में कौन से प्रेषक ट्रिगर कर सकते हैं (fallback: `channels.msteams.allowFrom`)।
- किसी भी सदस्य को अनुमति देने के लिए `groupPolicy: "open"` सेट करें (डिफ़ॉल्ट रूप से अभी भी mention‑gated)।
- **कोई चैनल नहीं** अनुमति देने के लिए `channels.msteams.groupPolicy: "disabled"` सेट करें।

उदाहरण:

```json5
{
  channels: {
    msteams: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["user@org.com"],
    },
  },
}
```

**Teams + चैनल allowlist**

- `channels.msteams.teams` के अंतर्गत teams और channels सूचीबद्ध करके समूह/चैनल उत्तरों का दायरा सीमित करें।
- Keys टीम IDs या नाम हो सकते हैं; चैनल keys conversation IDs या नाम हो सकते हैं।
- जब `groupPolicy="allowlist"` और teams allowlist मौजूद हो, तो केवल सूचीबद्ध teams/channels स्वीकार किए जाते हैं (mention‑gated)।
- configure विज़ार्ड `Team/Channel` प्रविष्टियाँ स्वीकार करता है और उन्हें आपके लिए सहेज देता है।
- स्टार्टअप पर, OpenClaw टीम/चैनल और उपयोगकर्ता allowlist नामों को IDs में resolve करता है (जब Graph अनुमतियाँ अनुमति दें) और मैपिंग लॉग करता है; जो resolve न हों, वे टाइप किए गए रूप में ही रखे जाते हैं।

उदाहरण:

```json5
{
  channels: {
    msteams: {
      groupPolicy: "allowlist",
      teams: {
        "My Team": {
          channels: {
            General: { requireMention: true },
          },
        },
      },
    },
  },
}
```

## यह कैसे काम करता है

1. Microsoft Teams प्लगइन इंस्टॉल करें।
2. एक **Azure Bot** बनाएँ (App ID + secret + tenant ID)।
3. एक **Teams ऐप पैकेज** बनाएँ जो बॉट को संदर्भित करे और नीचे दी गई RSC अनुमतियाँ शामिल करे।
4. Teams ऐप को किसी टीम में (या DMs के लिए personal scope में) अपलोड/इंस्टॉल करें।
5. `~/.openclaw/openclaw.json` (या env vars) में `msteams` कॉन्फ़िगर करें और Gateway प्रारंभ करें।
6. Gateway डिफ़ॉल्ट रूप से `/api/messages` पर Bot Framework webhook ट्रैफ़िक सुनता है।

## Azure Bot सेटअप (पूर्वापेक्षाएँ)

OpenClaw को कॉन्फ़िगर करने से पहले, आपको Azure Bot संसाधन बनाना होगा।

### चरण 1: Azure Bot बनाएँ

1. [Create Azure Bot](https://portal.azure.com/#create/Microsoft.AzureBot) पर जाएँ
2. **Basics** टैब भरें:

   | फ़ील्ड             | मान |
   | ------------------ | --- |
   | **बॉट हैंडल**     | आपका बॉट नाम, जैसे `openclaw-msteams` (अद्वितीय होना चाहिए) |
   | **सब्सक्रिप्शन**   | अपनी Azure subscription चुनें |
   | **रिसोर्स ग्रुप** | नया बनाएँ या मौजूदा का उपयोग करें |
   | **प्राइसिंग टियर**   | dev/testing के लिए **Free** |
   | **Type of App**    | **Single Tenant** (अनुशंसित - नीचे नोट देखें) |
   | **Creation type**  | **Create new Microsoft App ID** |

> **अप्रचलन सूचना:** नए मल्टी-टेनेंट बॉट्स का निर्माण 2025-07-31 के बाद अप्रचलित कर दिया गया। नए बॉट्स के लिए **Single Tenant** का उपयोग करें।

3. **Review + create** → **Create** पर क्लिक करें (~1-2 मिनट प्रतीक्षा करें)

### चरण 2: क्रेडेंशियल्स प्राप्त करें

1. अपने Azure Bot संसाधन → **Configuration** पर जाएँ  
2. **Microsoft App ID** कॉपी करें → यही आपका `appId` है  
3. **Manage Password** पर क्लिक करें → App Registration पर जाएँ  
4. **Certificates & secrets** → **New client secret** → **Value** कॉपी करें → यही आपका `appPassword` है  
5. **Overview** → **Directory (tenant) ID** कॉपी करें → यही आपका `tenantId` है  

### चरण 3: Messaging Endpoint कॉन्फ़िगर करें

1. Azure Bot → **Configuration**  
2. **Messaging endpoint** को अपने webhook URL पर सेट करें:  
   - Production: `https://your-domain.com/api/messages`  
   - Local dev: एक टनल का उपयोग करें (नीचे [Local Development](#local-development-tunneling) देखें)  

### चरण 4: Teams चैनल सक्षम करें

1. Azure Bot → **Channels**  
2. **Microsoft Teams** → Configure → Save पर क्लिक करें  
3. सेवा की शर्तें स्वीकार करें  

## Local Development (Tunneling)

Teams `localhost` तक नहीं पहुँच सकता। लोकल डेवलपमेंट के लिए टनल का उपयोग करें:

**विकल्प A: ngrok**

```bash
ngrok http 3978
# Copy the https URL, e.g., https://abc123.ngrok.io
# Set messaging endpoint to: https://abc123.ngrok.io/api/messages
```

**विकल्प B: Tailscale Funnel**

```bash
tailscale funnel 3978
# Use your Tailscale funnel URL as the messaging endpoint
```

## Teams Developer Portal (वैकल्पिक)

मैन्युअली manifest ZIP बनाने के बजाय, आप [Teams Developer Portal](https://dev.teams.microsoft.com/apps) का उपयोग कर सकते हैं:

1. **+ New app** पर क्लिक करें  
2. बुनियादी जानकारी भरें (नाम, विवरण, डेवलपर जानकारी)  
3. **App features** → **Bot** पर जाएँ  
4. **Enter a bot ID manually** चुनें और अपना Azure Bot App ID पेस्ट करें  
5. Scopes चुनें: **Personal**, **Team**, **Group Chat**  
6. **Distribute** → **Download app package**  
7. Teams में: **Apps** → **Manage your apps** → **Upload a custom app** → ZIP चुनें  

यह अक्सर JSON manifests को हाथ से एडिट करने से आसान होता है।