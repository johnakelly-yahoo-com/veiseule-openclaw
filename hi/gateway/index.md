---
summary: "Gateway सेवा, जीवनचक्र और संचालन के लिए रनबुक"
read_when:
  - Gateway प्रक्रिया चलाते या डीबग करते समय
title: "Gateway रनबुक"
---

# Gateway सेवा रनबुक

अंतिम अपडेट: 2025-12-09

<CardGroup cols={2}>
  <Card title="Deep troubleshooting" icon="siren" href="/gateway/troubleshooting">
    सटीक कमांड क्रम और लॉग सिग्नेचर के साथ लक्षण-आधारित निदान।
  
</Card>
  <Card title="Configuration" icon="sliders" href="/gateway/configuration">
    कार्य-उन्मुख सेटअप गाइड + पूर्ण कॉन्फ़िगरेशन संदर्भ।
  
</Card>
</CardGroup>

## 5-मिनट स्थानीय स्टार्टअप

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

स्वस्थ आधाररेखा: `Runtime: running` और `RPC probe: ok`।

  
</Step>

  <Step title="Validate channel readiness">

```bash
openclaw channels status --probe
```

  
</Step>
</Steps>

<Note>
Gateway कॉन्फ़िग रीलोड सक्रिय कॉन्फ़िग फ़ाइल पथ (प्रोफ़ाइल/स्टेट डिफ़ॉल्ट से निर्धारित, या जब सेट हो तो `OPENCLAW_CONFIG_PATH`) को मॉनिटर करता है।
डिफ़ॉल्ट मोड `gateway.reload.mode="hybrid"` है।
</Note>

## रनटाइम मॉडल

- रूटिंग, कंट्रोल प्लेन और चैनल कनेक्शनों के लिए एक हमेशा-चालू प्रक्रिया।
- इसके लिए एकल मल्टीप्लेक्स्ड पोर्ट:
  - WebSocket कंट्रोल/RPC
  - HTTP APIs (OpenAI-संगत, Responses, tools invoke)
  - कंट्रोल UI और hooks
- डिफ़ॉल्ट बाइंड मोड: `loopback`।
- डिफ़ॉल्ट रूप से Auth आवश्यक है (`gateway.auth.token` / `gateway.auth.password`, या `OPENCLAW_GATEWAY_TOKEN` / `OPENCLAW_GATEWAY_PASSWORD`)।

### Dev प्रोफ़ाइल (`--dev`)

| सेटिंग        | रिज़ॉल्यूशन क्रम                                              |
| ------------- | ------------------------------------------------------------- |
| Gateway पोर्ट | `--port` → `OPENCLAW_GATEWAY_PORT` → `gateway.port` → `18789` |
| बाइंड मोड     | CLI/override → `gateway.bind` → `loopback`                    |

### हॉट रीलोड मोड्स

| `gateway.reload.mode`                  | व्यवहार                                                          |
| -------------------------------------- | ---------------------------------------------------------------- |
| `off`                                  | कोई कॉन्फ़िग रीलोड नहीं                                          |
| `hot`                                  | केवल हॉट-सुरक्षित परिवर्तन लागू करें                             |
| `restart`                              | रीलोड-आवश्यक परिवर्तनों पर पुनः प्रारंभ करें                     |
| `hybrid` (डिफ़ॉल्ट) | सुरक्षित होने पर हॉट-लागू करें, आवश्यक होने पर पुनः प्रारंभ करें |

## ऑपरेटर कमांड सेट

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

## रिमोट एक्सेस

प्राथमिक: Tailscale/VPN.
वैकल्पिक: SSH टनल।

```bash
ssh -N -L 18789:127.0.0.1:18789 user@host
```

प्रोफ़ाइल के अनुसार सेवा इंस्टॉल:

<Warning>
यदि gateway auth कॉन्फ़िगर किया गया है, तो क्लाइंट्स को SSH टनल के माध्यम से भी auth (`token`/`password`) भेजना अनिवार्य है।
</Warning>

उदाहरण:

## सुपरविजन और सेवा जीवनचक्र

प्रोडक्शन-जैसी विश्वसनीयता के लिए supervised runs का उपयोग करें।

<Tabs>
  <Tab title="macOS (launchd)">

```bash
openclaw gateway install
openclaw gateway status
openclaw gateway restart
openclaw gateway stop
```

LaunchAgent labels `ai.openclaw.gateway` (default) या `ai.openclaw.<profile>` (named profile)। `openclaw doctor` सेवा कॉन्फ़िगरेशन ड्रिफ्ट का ऑडिट और मरम्मत करता है।

  
</Tab>

  <Tab title="Linux (systemd user)">

```bash
openclaw gateway install
systemctl --user enable --now openclaw-gateway[-<profile>].service
openclaw gateway status
```

लॉगआउट के बाद भी स्थायित्व के लिए, lingering सक्षम करें:

```bash
sudo loginctl enable-linger <user>
```

  
</Tab>

  <Tab title="Linux (system service)">

मल्टी-यूज़र/हमेशा-चालू होस्ट्स के लिए system unit का उपयोग करें।

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now openclaw-gateway[-<profile>].service
```

  
</Tab>
</Tabs>

## एक ही होस्ट पर अनेक gateways

अधिकांश सेटअप में **एक** Gateway चलाना चाहिए।
कड़े आइसोलेशन/रिडंडेंसी (उदाहरण के लिए rescue profile) के लिए ही अनेक का उपयोग करें।

प्रति-इंस्टेंस चेकलिस्ट:

- अद्वितीय `gateway.port`
- अद्वितीय `OPENCLAW_CONFIG_PATH`
- अद्वितीय `OPENCLAW_STATE_DIR`
- अद्वितीय `agents.defaults.workspace`

उदाहरण:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json OPENCLAW_STATE_DIR=~/.openclaw-a openclaw gateway --port 19001
OPENCLAW_CONFIG_PATH=~/.openclaw/b.json OPENCLAW_STATE_DIR=~/.openclaw-b openclaw gateway --port 19002
```

देखें: [Multiple gateways](/gateway/multiple-gateways)।

### Gateway सेवा प्रबंधन (CLI)

```bash
openclaw --dev setup
openclaw --dev gateway --allow-unconfigured
openclaw --dev status
```

डिफ़ॉल्ट में पृथक state/config और बेस gateway पोर्ट `19001` शामिल हैं।

## प्रोटोकॉल त्वरित संदर्भ (ऑपरेटर दृश्य)

- `gateway status` डिफ़ॉल्ट रूप से सेवा के रेज़ॉल्व्ड पोर्ट/कॉन्फ़िग का उपयोग करके Gateway RPC को प्रोब करता है (`--url` से ओवरराइड करें)।
- `gateway status --deep` सिस्टम-स्तरीय स्कैन (LaunchDaemons/system units) जोड़ता है।
- `gateway status --no-probe` RPC प्रोब को स्किप करता है (नेटवर्किंग डाउन होने पर उपयोगी)।
- `gateway status --json` स्क्रिप्ट्स के लिए स्थिर है।

बंडल्ड mac ऐप:

1. तुरंत स्वीकृत ack (`status:"accepted"`)
2. इसे साफ़-साफ़ रोकने के लिए `openclaw gateway stop` (या `launchctl bootout gui/$UID/bot.molt.gateway`) का उपयोग करें।

पूर्ण प्रोटोकॉल दस्तावेज़ देखें: [Gateway Protocol](/gateway/protocol)।

## ऑपरेशनल चेक्स

### लाइवनेस

- WS खोलें और `connect` भेजें।
- snapshot के साथ `hello-ok` प्रतिक्रिया की अपेक्षा करें।

### रेडिनेस

```bash
sudo loginctl enable-linger youruser
```

### गैप रिकवरी

Events are not replayed. सीक्वेंस गैप होने पर, आगे बढ़ने से पहले state (`health`, `system-presence`) रिफ्रेश करें।

## सामान्य विफलता संकेत

| संकेत                                                          | संभावित समस्या                           |
| -------------------------------------------------------------- | ---------------------------------------- |
| `refusing to bind gateway ... without auth`                    | token/password के बिना non-loopback bind |
| `another gateway instance is already listening` / `EADDRINUSE` | पोर्ट टकराव                              |
| `Gateway start blocked: set gateway.mode=local`                | रिमोट मोड पर कॉन्फ़िग सेट किया गया       |
| कनेक्ट के दौरान `unauthorized`                                 | क्लाइंट और Gateway के बीच Auth असंगति    |

पूर्ण निदान चरणों के लिए, [Gateway Troubleshooting](/gateway/troubleshooting) का उपयोग करें।

## सुरक्षा गारंटी

- जब Gateway उपलब्ध नहीं होता है, तो Gateway प्रोटोकॉल क्लाइंट तुरंत विफल हो जाते हैं (कोई अप्रत्यक्ष डायरेक्ट-चैनल फॉलबैक नहीं)।
- अमान्य/नॉन-कनेक्ट पहली फ्रेम्स को अस्वीकार कर बंद कर दिया जाता है।
- सॉकेट बंद होने से पहले Graceful shutdown `shutdown` इवेंट जारी करता है।

---

संबंधित:

- डिफ़ॉल्ट रूप से प्रति होस्ट एक Gateway मानें; यदि कई प्रोफ़ाइल चलाते हैं, तो पोर्ट्स/स्टेट को अलग रखें और सही इंस्टेंस को टार्गेट करें।
- सीधे Baileys कनेक्शनों पर कोई फ़ॉलबैक नहीं; यदि Gateway डाउन है, तो sends तेज़ी से विफल होते हैं।
- non-connect प्रथम फ़्रेम या malformed JSON अस्वीकृत किए जाते हैं और सॉकेट बंद कर दिया जाता है।
- ग्रेसफ़ुल शटडाउन: बंद करने से पहले `shutdown` इवेंट उत्सर्जित करें; क्लाइंट्स को close + reconnect संभालना चाहिए।
- [Doctor](/gateway/doctor)
- [Authentication](/gateway/authentication)
