---
read_when:
  - ゼロမှ စတင်하는 ပထမဆုံး စနစ်တပ်ဆင်မှု
  - အလုပ်လုပ်နေသော ချက်ချင်းအသုံးပြုနိုင်သည့် ချတ်သို့ အမြန်ဆုံးရောက်နိုင်သော နည်းလမ်းကို သိလိုပါသလား
summary: OpenClaw ကို ထည့်သွင်းပြီး မိနစ်အနည်းငယ်အတွင်း ပထမဆုံး ချတ်ကို စတင်အသုံးပြုလိုက်ပါ။
title: စတင်ရန်
x-i18n:
  generated_at: "2026-02-08T17:15:16Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 27aeeb3d18c495380e94e6b011b0df3def518535c9f1eee504f04871d8a32269
  source_path: start/getting-started.md
  workflow: 15
---

# စတင်ရန်

ရည်ရွယ်ချက် — အနည်းဆုံး စနစ်တပ်ဆင်မှုဖြင့် သုညမှစ၍ ပထမဆုံး အလုပ်လုပ်နိုင်သော ချတ်ကို ရရှိစေရန်။

<Info>
အမြန်ဆုံး ချတ်အသုံးပြုနည်း — Control UI ကို ဖွင့်ပါ (channel ဆက်တင် မလိုအပ်ပါ)။ `openclaw dashboard` ကို လည်ပတ်ပြီး browser မှ ချတ်လုပ်နိုင်သည် သို့မဟုတ်၊
<Tooltip headline="Gatewayホスト" tip="OpenClaw Gatewayサービスを実行しているマシン。">Gateway ဟို့စ်</Tooltip>တွင် `http://127.0.0.1:18789/` ကို ဖွင့်ပါ။
စာရွက်စာတမ်းများ — [Dashboard](/web/dashboard) နှင့် [Control UI](/web/control-ui)。
</Info>

## လိုအပ်ချက်များ

- Node 22 နှင့်အထက်

<Tip>
မသေချာပါက `node --version` ဖြင့် Node ဗားရှင်းကို စစ်ဆေးပါ။
</Tip>

## အမြန် စနစ်တပ်ဆင်မှု (CLI)

<Steps>
  <Step title="OpenClawをインストール（推奨）">
    <Tabs>
      <Tab title="macOS/Linux">
        ```bash
        curl -fsSL https://openclaw.ai/install.sh | bash
        ```
      
</Tab>
      <Tab title="Windows (PowerShell)">
        ```powershell
        iwr -useb https://openclaw.ai/install.ps1 | iex
        ```
      
</Tab>
    
</Tabs>

    ```
    <Note>
    အခြား စနစ်တပ်ဆင်နည်းများနှင့် လိုအပ်ချက်များ — [インストール](/install)。
    
</Note>
    ```

  
</Step>
  <Step title="オンボーディングウィザードを実行">
    ```bash
    openclaw onboard --install-daemon
    ```

    ```
    ဝစ်ဇာဒ်သည် authentication၊ Gateway ဆက်တင်များနှင့် ရွေးချယ်နိုင်သော channel များကို ပြင်ဆင်သတ်မှတ်ပေးသည်။
    အသေးစိတ်အချက်အလက်များအတွက် [オンボーディングウィザード](/start/wizard) ကို ကြည့်ပါ။
    ```

  
</Step>
  <Step title="Gatewayを確認">
    ဝန်ဆောင်မှုကို ထည့်သွင်းထားပါက၊ ၎င်းသည် ယခုပင် လည်ပတ်နေသင့်သည် —

    ````
    ```bash
    openclaw gateway status
    ```
    ````

  
</Step>
  <Step title="Control UIを開く">
    ```bash
    openclaw dashboard
    ```
  
</Step>
</Steps>

<Check>
Control UI တင်ပြီးသား ဖြစ်ပါက၊ Gateway ကို အသုံးပြုရန် အဆင်သင့် ဖြစ်ပါသည်。
</Check>

## ရွေးချယ်နိုင်သော စစ်ဆေးမှုများနှင့် အပိုလုပ်ဆောင်ချက်များ

<AccordionGroup>
  <Accordion title="Gatewayをフォアグラウンドで実行">
    အမြန် စမ်းသပ်ခြင်းနှင့် ပြဿနာရှာဖွေရေးအတွက် အသုံးဝင်သည်。


    ````
    ```bash
    openclaw gateway --port 18789
    ```
    ````

  
</Accordion>
  <Accordion title="テストメッセージを送信">ချန်နယ်ကို ကြိုတင်ဖွဲ့စည်းထားရန် လိုအပ်သည်။

    ````
    ```bash
    openclaw message send --target +15555550123 --message "Hello from OpenClaw"
    ```
    ````

  
</Accordion>
</AccordionGroup>

## အသေးစိတ် ထပ်မံကြည့်ရှုရန်

<Columns>
  <Card title="オンボーディングウィザード（詳細）" href="/start/wizard">CLI ဝိဇာဒ်၏ ပြည့်စုံသော ရည်ညွှန်းချက်နှင့် အဆင့်မြင့် ရွေးချယ်စရာများ။
</Card>
  <Card title="macOSアプリのオンボーディング" href="/start/onboarding">macOS အက်ပ်ကို ပထမဆုံးအကြိမ် လည်ပတ်သည့် လုပ်ငန်းစဉ်။
</Card>
</Columns>

## ပြီးဆုံးပြီးနောက် အခြေအနေ

- လည်ပတ်နေသော Gateway
- ဖွဲ့စည်းပြီးသား အထောက်အထားအတည်ပြုခြင်း
- Control UI ကို ဝင်ရောက်အသုံးပြုနိုင်ခြင်း သို့မဟုတ် ချိတ်ဆက်ပြီးသား ချန်နယ်များ

## နောက်တစ်ဆင့်များ

- DM လုံခြုံရေးနှင့် အတည်ပြုချက်：[Pairing](/channels/pairing)
- နောက်ထပ် ချန်နယ်များ ချိတ်ဆက်ရန်：[Channels](/channels)
- အဆင့်မြင့် workflow များနှင့် source မှ build လုပ်ခြင်း：[Setup](/start/setup)
