---
summary: "OpenClaw onboarding ရွေးချယ်စရာများနှင့် လုပ်ဆောင်ပုံများ၏ အကျဉ်းချုပ်"
read_when:
  - Onboarding လမ်းကြောင်း ရွေးချယ်ခြင်း
  - ပတ်ဝန်းကျင် အသစ်တစ်ခု တည်ဆောက်ခြင်း
title: "Onboarding အကျဉ်းချုပ်"
sidebarTitle: "Onboarding အကျဉ်းချုပ်"
---

# Onboarding အကျဉ်းချုပ်

OpenClaw သည် Gateway ကို မည်သည့်နေရာတွင် လည်ပတ်စေသည်နှင့် provider များကို မည်သို့ ပြင်ဆင်လိုသည်အပေါ် မူတည်၍ onboarding လမ်းကြောင်း အမျိုးမျိုးကို ပံ့ပိုးပေးပါသည်။

## သင့်အတွက် သင့်လျော်သော onboarding လမ်းကြောင်းကို ရွေးချယ်ပါ

- macOS, Linux နှင့် Windows (WSL2 မှတစ်ဆင့်) အတွက် **CLI wizard**။
- Apple silicon သို့မဟုတ် Intel Macs အတွက် လမ်းညွှန်ပါဝင်သော ပထမဆုံး run အတွက် **macOS app**။

## CLI onboarding wizard

terminal တွင် wizard ကို လည်ပတ်ပါ။

```bash
openclaw onboard
```

Gateway၊ workspace၊ channels နှင့် skills များကို အပြည့်အဝ ထိန်းချုပ်လိုပါက CLI wizard ကို အသုံးပြုပါ။ စာရွက်စာတမ်းများ:

- [Onboarding Wizard (CLI)](/start/wizard)
- [`openclaw onboard` command](/cli/onboard)

## macOS app onboarding

macOS ပေါ်တွင် လမ်းညွှန်အပြည့်အစုံပါသော setup ကို လိုချင်ပါက OpenClaw app ကို အသုံးပြုပါ။ စာရွက်စာတမ်းများ:

- [Onboarding (macOS App)](/start/onboarding)

## Custom Provider

စာရင်းတွင် မပါဝင်သော endpoint တစ်ခုကို လိုအပ်ပါက (standard OpenAI သို့မဟုတ် Anthropic APIs ကို ဖော်ပြပေးသော hosted providers များပါဝင်သည်) CLI wizard တွင် **Custom Provider** ကို ရွေးချယ်ပါ။ သင့်အား အောက်ပါတို့ကို မေးမြန်းပါမည်။

- OpenAI-compatible, Anthropic-compatible သို့မဟုတ် **Unknown** (auto-detect) ကို ရွေးချယ်ပါ။
- base URL နှင့် API key (provider မှ လိုအပ်ပါက) ကို ထည့်သွင်းပါ။
- model ID နှင့် ရွေးချယ်နိုင်သော alias ကို ပေးပါ။
- custom endpoints များစွာကို အတူတကွ အသုံးပြုနိုင်ရန် Endpoint ID ကို ရွေးချယ်ပါ။

အသေးစိတ် အဆင့်များအတွက် အထက်ပါ CLI onboarding စာရွက်စာတမ်းများကို လိုက်နာပါ။
