---
title: "browser"
---

# `openclaw browser`

OpenClaw ၏ browser ထိန်းချုပ်မှု server ကို စီမံခန့်ခွဲပြီး browser လုပ်ဆောင်ချက်များ (tab များ၊ snapshot များ၊ screenshot များ၊ navigation၊ click၊ စာရိုက်ခြင်း) ကို လုပ်ဆောင်ပါ။

ဆက်စပ်အကြောင်းအရာများ:

- ဘရောက်ဇာ ကိရိယာ + API: [ဘရောက်ဇာ ကိရိယာ](/tools/browser)
- Chrome တိုးချဲ့ပရိုဂရမ် ပြန်လည်ပို့ဆောင်ရေး: [Chrome တိုးချဲ့ပရိုဂရမ်](/tools/chrome-extension)

## အထွေထွေ flags

- `--url <gatewayWsUrl>`: Gateway WebSocket URL (config မှ ပုံမှန်တန်ဖိုးကို အသုံးပြုသည်)။
- `--token <token>`: Gateway token (လိုအပ်ပါက)။
- `--timeout <ms>`: request timeout (ms)။
- `--browser-profile <name>`: browser profile ကို ရွေးချယ်ရန် (config မှ ပုံမှန်တန်ဖိုး)။
- `--json`: စက်ဖြင့်ဖတ်နိုင်သော output (ထောက်ပံ့ထားသောနေရာများတွင်)။

## အမြန် စတင်ခြင်း (ဒေသတွင်း)

```bash
openclaw browser --browser-profile chrome tabs
openclaw browser --browser-profile openclaw start
openclaw browser --browser-profile openclaw open https://example.com
openclaw browser --browser-profile openclaw snapshot
```

## ပရိုဖိုင်များ

Profiles များသည် browser routing configs များကို အမည်ပေးထားခြင်း ဖြစ်သည်။ လက်တွေ့အသုံးချမှုတွင်:

- `openclaw`: OpenClaw မှ စီမံခန့်ခွဲသော Chrome instance သီးသန့်တစ်ခုကို စတင်ခြင်း/ချိတ်ဆက်ခြင်း (သီးခြား user data dir ဖြင့် ခွဲခြားထားသည်)။
- `chrome`: Chrome extension relay မှတဆင့် သင်၏ လက်ရှိ Chrome tab များကို ထိန်းချုပ်ခြင်း။

```bash
openclaw browser profiles
openclaw browser create-profile --name work --color "#FF5A36"
openclaw browser delete-profile --name work
```

Profile တစ်ခုကို သီးသန့်အသုံးပြုရန်—

```bash
openclaw browser --browser-profile work tabs
```

## တැබ်များ

```bash
openclaw browser tabs
openclaw browser open https://docs.openclaw.ai
openclaw browser focus <targetId>
openclaw browser close <targetId>
```

## စနပ်ရှော့ / စကရင်ရှော့ / လုပ်ဆောင်ချက်များ

စနပ်ရှော့:

```bash
openclaw browser snapshot
```

စကရင်ရှော့:

```bash
openclaw browser screenshot
```

Navigate/click/type (ref-based UI automation):

```bash
openclaw browser navigate https://example.com
openclaw browser click <ref>
openclaw browser type <ref> "hello"
```

## Chrome extension relay (toolbar ခလုတ်ဖြင့် attach ပြုလုပ်ခြင်း)

ဤ mode သည် သင်လက်ဖြင့် attach လုပ်ထားသော Chrome tab ရှိပြီးသားကို agent မှ ထိန်းချုပ်နိုင်စေပါသည် (auto-attach မလုပ်ပါ)။

Unpacked extension ကို တည်ငြိမ်သော path တစ်ခုတွင် ထည့်သွင်းပါ—

```bash
openclaw browser extension install
openclaw browser extension path
```

ထို့နောက် Chrome → `chrome://extensions` → “Developer mode” ကို ဖွင့် → “Load unpacked” → ပြထားသော folder ကို ရွေးချယ်ပါ။

လမ်းညွှန်အပြည့်အစုံ: [Chrome extension](/tools/chrome-extension)

## Remote browser control (node host proxy)

Gateway ကို browser နဲ့ မတူတဲ့ စက်ပေါ်မှာ chạy နေပါက Chrome/Brave/Edge/Chromium ရှိတဲ့ စက်ပေါ်မှာ **node host** ကို chạy လုပ်ပါ။ Gateway သည် ထို node သို့ browser actions များကို proxy လုပ်ပေးမည် (သီးခြား browser control server မလိုအပ်ပါ)။

Auto-routing ကို ထိန်းချုပ်ရန် `gateway.nodes.browser.mode` ကို အသုံးပြုပါ၊ node များ အများအပြား ချိတ်ဆက်ထားပါက သီးသန့် node တစ်ခုကို ချိတ်ရန် `gateway.nodes.browser.node` ကို အသုံးပြုပါ။

လုံခြုံရေး + အဝေးမှ setup: [Browser tool](/tools/browser), [Remote access](/gateway/remote), [Tailscale](/gateway/tailscale), [Security](/gateway/security)

