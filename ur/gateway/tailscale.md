---
title: "Tailscale"
---

# Tailscale (Gateway ڈیش بورڈ)

OpenClaw خودکار طور پر Tailscale **Serve** (ٹیل نیٹ) یا **Funnel** (عوامی) کو کے لیے ترتیب دے سکتا ہے
Gateway dashboard and WebSocket port. This keeps the Gateway bound to loopback while
Tailscale provides HTTPS, routing, and (for Serve) identity headers.

## موڈز

- `serve`: صرف ٹیل نیٹ Serve بذریعہ `tailscale serve`۔ گیٹ وے `127.0.0.1` پر رہتا ہے۔
- `funnel`: عوامی HTTPS بذریعہ `tailscale funnel`۔ OpenClaw کے لیے ایک مشترکہ پاس ورڈ درکار ہے۔
- `off`: ڈیفالٹ (کوئی Tailscale خودکاری نہیں)۔

## توثیق

ہینڈشیک کو کنٹرول کرنے کے لیے `gateway.auth.mode` سیٹ کریں:

- `token` (جب `OPENCLAW_GATEWAY_TOKEN` سیٹ ہو تو بطورِ طے شدہ)
- `password` (مشترکہ خفیہ `OPENCLAW_GATEWAY_PASSWORD` یا کنفیگ کے ذریعے)

جب `tailscale.mode = "serve"` اور `gateway.auth.allowTailscale` `true` ہو,
valid Serve proxy requests can authenticate via Tailscale identity headers
(`tailscale-user-login`) without supplying a token/password. OpenClaw verifies
the identity by resolving the `x-forwarded-for` address via the local Tailscale
daemon (`tailscale whois`) and matching it to the header before accepting it.
OpenClaw only treats a request as Serve when it arrives from loopback with
Tailscale’s `x-forwarded-for`, `x-forwarded-proto`, and `x-forwarded-host`
headers.
To require explicit credentials, set `gateway.auth.allowTailscale: false` or
force `gateway.auth.mode: "password"`.

## کنفیگ کی مثالیں

### صرف ٹیل نیٹ (Serve)

```json5
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "serve" },
  },
}
```

کھولیں: `https://<magicdns>/` (یا آپ کے کنفیگر کردہ `gateway.controlUi.basePath`)

### Tailnet-only (Tailnet IP پر bind)

جب آپ چاہتے ہوں کہ Gateway براہِ راست Tailnet IP پر سنے (Serve/Funnel کے بغیر) تو یہ استعمال کریں۔

```json5
{
  gateway: {
    bind: "tailnet",
    auth: { mode: "token", token: "your-token" },
  },
}
```

کسی دوسرے Tailnet ڈیوائس سے کنیکٹ کریں:

- کنٹرول UI: `http://<tailscale-ip>:18789/`
- ویب ساکٹ: `ws://<tailscale-ip>:18789`

نوٹ: loopback (`http://127.0.0.1:18789`) اس موڈ میں **کام نہیں** کرے گا۔

### عوامی انٹرنیٹ (Funnel + مشترکہ پاس ورڈ)

```json5
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "funnel" },
    auth: { mode: "password", password: "replace-me" },
  },
}
```

ڈسک پر پاس ورڈ محفوظ کرنے کے بجائے `OPENCLAW_GATEWAY_PASSWORD` کو ترجیح دیں۔

## CLI کی مثالیں

```bash
openclaw gateway --tailscale serve
openclaw gateway --tailscale funnel --auth password
```

## Notes

- Tailscale Serve/Funnel کے لیے `tailscale` CLI کا انسٹال اور لاگ اِن ہونا ضروری ہے۔
- `tailscale.mode: "funnel"` عوامی ایکسپوژر سے بچنے کے لیے اس وقت تک شروع نہیں ہوتا جب تک auth موڈ `password` نہ ہو۔
- اگر آپ چاہتے ہیں کہ OpenClaw شٹ ڈاؤن پر `tailscale serve`
  یا `tailscale funnel` کنفیگریشن کو واپس لے، تو `gateway.tailscale.resetOnExit` سیٹ کریں۔
- `gateway.bind: "tailnet"` براہِ راست Tailnet bind ہے (نہ HTTPS، نہ Serve/Funnel)۔
- `gateway.bind: "auto"` loopback کو ترجیح دیتا ہے؛ اگر Tailnet-only چاہیے تو `tailnet` استعمال کریں۔
- Serve/Funnel only expose the **Gateway control UI + WS**. Nodes connect over
  the same Gateway WS endpoint, so Serve can work for node access.

## Browser control (ریموٹ Gateway + لوکل براؤزر)

If you run the Gateway on one machine but want to drive a browser on another machine,
run a **node host** on the browser machine and keep both on the same tailnet.
The Gateway will proxy browser actions to the node; no separate control server or Serve URL needed.

براؤزر کنٹرول کے لیے Funnel سے پرہیز کریں؛ نوڈ pairing کو آپریٹر رسائی کے طور پر سمجھیں۔

## Tailscale پیشگی تقاضے + حدود

- Serve کے لیے آپ کے tailnet پر HTTPS فعال ہونا ضروری ہے؛ اگر غائب ہو تو CLI اشارہ دیتا ہے۔
- Serve Tailscale شناختی ہیڈرز داخل کرتا ہے؛ Funnel نہیں کرتا۔
- Funnel کے لیے Tailscale v1.38.3+، MagicDNS، HTTPS فعال ہونا، اور funnel نوڈ ایٹریبیوٹ درکار ہے۔
- Funnel TLS پر صرف `443`, `8443`, اور `10000` پورٹس کی حمایت کرتا ہے۔
- macOS پر Funnel کے لیے اوپن سورس Tailscale ایپ ویریئنٹ درکار ہے۔

## Learn more

- Tailscale Serve جائزہ: [https://tailscale.com/kb/1312/serve](https://tailscale.com/kb/1312/serve)
- `tailscale serve` کمانڈ: [https://tailscale.com/kb/1242/tailscale-serve](https://tailscale.com/kb/1242/tailscale-serve)
- Tailscale Funnel جائزہ: [https://tailscale.com/kb/1223/tailscale-funnel](https://tailscale.com/kb/1223/tailscale-funnel)
- `tailscale funnel` کمانڈ: [https://tailscale.com/kb/1311/tailscale-funnel](https://tailscale.com/kb/1311/tailscale-funnel)


