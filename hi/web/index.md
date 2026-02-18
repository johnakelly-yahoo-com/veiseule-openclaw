---
title: "वेब"
---

# वेब (Gateway)

Gateway, Gateway WebSocket के समान पोर्ट से एक छोटा **ब्राउज़र कंट्रोल UI** (Vite + Lit) परोसता है:

- डिफ़ॉल्ट: `http://<host>:18789/`
- वैकल्पिक प्रीफ़िक्स: `gateway.controlUi.basePath` सेट करें (उदा. `/openclaw`)

क्षमताएँ [Control UI](/web/control-ui) में उपलब्ध हैं।
This page focuses on bind modes, security, and web-facing surfaces.

## वेबहुक्स

जब `hooks.enabled=true` होता है, तो Gateway उसी HTTP सर्वर पर एक छोटा webhook endpoint भी उपलब्ध कराता है।
See [Gateway configuration](/gateway/configuration) → `hooks` for auth + payloads.

## कॉन्फ़िग (डिफ़ॉल्ट-ऑन)

जब assets मौजूद हों (`dist/control-ui`), तो Control UI **डिफ़ॉल्ट रूप से सक्षम** होता है।
You can control it via config:

```json5
{
  gateway: {
    controlUi: { enabled: true, basePath: "/openclaw" }, // basePath optional
  },
}
```

## Tailscale एक्सेस

### Integrated Serve (अनुशंसित)

Gateway को loopback पर रखें और Tailscale Serve से इसे प्रॉक्सी करें:

```json5
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "serve" },
  },
}
```

फिर Gateway शुरू करें:

```bash
openclaw gateway
```

खोलें:

- `https://<magicdns>/` (या आपका कॉन्फ़िगर किया हुआ `gateway.controlUi.basePath`)

### Tailnet बाइंड + टोकन

```json5
{
  gateway: {
    bind: "tailnet",
    controlUi: { enabled: true },
    auth: { mode: "token", token: "your-token" },
  },
}
```

फिर Gateway शुरू करें (नॉन-loopback बाइंड्स के लिए टोकन आवश्यक):

```bash
openclaw gateway
```

खोलें:

- `http://<tailscale-ip>:18789/` (या आपका कॉन्फ़िगर किया हुआ `gateway.controlUi.basePath`)

### सार्वजनिक इंटरनेट (Funnel)

```json5
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "funnel" },
    auth: { mode: "password" }, // or OPENCLAW_GATEWAY_PASSWORD
  },
}
```

## सुरक्षा नोट्स

- Gateway प्रमाणीकरण डिफ़ॉल्ट रूप से आवश्यक है (टोकन/पासवर्ड या Tailscale पहचान हेडर्स)।
- नॉन-loopback बाइंड्स के लिए अभी भी **आवश्यक** है एक साझा टोकन/पासवर्ड (`gateway.auth` या env)।
- विज़ार्ड डिफ़ॉल्ट रूप से एक Gateway टोकन जनरेट करता है (loopback पर भी)।
- UI `connect.params.auth.token` या `connect.params.auth.password` भेजता है।
- कंट्रोल UI एंटी-क्लिकजैकिंग हेडर्स भेजता है और केवल same-origin ब्राउज़र
  वेब-सॉकेट कनेक्शनों को स्वीकार करता है, जब तक कि `gateway.controlUi.allowedOrigins` सेट न हो।
- Serve के साथ, Tailscale identity headers auth को संतुष्ट कर सकते हैं जब
`gateway.auth.allowTailscale` `true` हो (token/password की आवश्यकता नहीं)। सेट करें
  `gateway.auth.allowTailscale: false` to require explicit credentials. See
  [Tailscale](/gateway/tailscale) and [Security](/gateway/security).
- `gateway.tailscale.mode: "funnel"` के लिए `gateway.auth.mode: "password"` (साझा पासवर्ड) आवश्यक है।

## UI का निर्माण

1. गेटवे `dist/control-ui` से स्थिर फ़ाइलें परोसता है। 2. इन्हें इस तरह बिल्ड करें:

```bash
pnpm ui:build # auto-installs UI deps on first run
```
