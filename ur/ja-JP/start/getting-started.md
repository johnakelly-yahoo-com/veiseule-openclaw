---
read_when:
  - ابتدا سے پہلا سیٹ اپ
  - کیا آپ کام کرنے والی چیٹ تک سب سے مختصر راستہ جاننا چاہتے ہیں
summary: OpenClaw انسٹال کریں اور چند منٹوں میں اپنی پہلی چیٹ چلائیں۔
title: شروعات کریں
x-i18n:
  generated_at: "2026-02-08T17:15:16Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 27aeeb3d18c495380e94e6b011b0df3def518535c9f1eee504f04871d8a32269
  source_path: start/getting-started.md
  workflow: 15
---

# تعارف

مقصد: صفر سے کم سے کم سیٹ اپ کے ساتھ پہلی فعال چیٹ حاصل کرنا۔

<Info>
تیز ترین طریقہ برائے چیٹ: Control UI کھولیں (چینل کی ترتیب درکار نہیں)۔ `openclaw dashboard` چلائیں اور براؤزر میں چیٹ کریں، یا
<Tooltip headline="Gatewayホスト" tip="OpenClaw Gatewayサービスを実行しているマシン。">Gateway ہوسٹ</Tooltip>میں `http://127.0.0.1:18789/` کھولیں۔
دستاویزات: [Dashboard](/web/dashboard) اور [Control UI](/web/control-ui)۔
</Info>

## پیشگی شرائط

- Node 22 یا اس کے بعد کا ورژن

<Tip>
اگر یقین نہ ہو تو `node --version` چلا کر Node کا ورژن چیک کریں۔
</Tip>

## فوری سیٹ اپ (CLI)

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
    دیگر انسٹالیشن طریقے اور تقاضے: [انسٹال کریں](/install)۔
    
</Note>
    ```

  
</Step>
  <Step title="オンボーディングウィザードを実行">
    ```bash
    openclaw onboard --install-daemon
    ```

    ```
    وزرڈ تصدیق، Gateway کی ترتیب، اور اختیاری چینلز کو کنفیگر کرے گا۔
    مزید تفصیل کے لیے [آن بورڈنگ وزرڈ](/start/wizard) دیکھیں۔
    ```

  
</Step>
  <Step title="Gatewayを確認">
    اگر آپ نے سروس انسٹال کی ہے تو یہ پہلے ہی چل رہی ہونی چاہیے:


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
Control UI لوڈ ہو جانے کے بعد، Gateway استعمال کے لیے تیار ہے۔
</Check>

## اختیاری تصدیق اور اضافی خصوصیات

<AccordionGroup>
  <Accordion title="Gatewayをフォアグラウンドで実行">
    فوری ٹیسٹنگ یا ٹربل شوٹنگ کے لیے مفید۔


    ````
    ```bash
    openclaw gateway --port 18789
    ```
    ````

  
</Accordion>
  <Accordion title="テストメッセージを送信">
    ایک کنفیگر شدہ چینل درکار ہے۔


    ````
    ```bash
    openclaw message send --target +15555550123 --message "Hello from OpenClaw"
    ```
    ````

  
</Accordion>
</AccordionGroup>

## مزید جانیں

<Columns>
  <Card title="オンボーディングウィザード（詳細）" href="/start/wizard">
    مکمل CLI وزرڈ ریفرنس اور جدید اختیارات۔
  
</Card>
  <Card title="macOSアプリのオンボーディング" href="/start/onboarding">
    macOS ایپ کا پہلی بار چلانے کا عمل۔
  
</Card>
</Columns>

## مکمل ہونے کے بعد کی حالت

- چلتا ہوا Gateway
- کنفیگر شدہ تصدیق
- Control UI تک رسائی یا منسلک چینل

## اگلے مراحل

- DM کی سیکیورٹی اور منظوری: [پیئرنگ](/channels/pairing)
- مزید چینلز منسلک کریں: [چینلز](/channels)
- ایڈوانسڈ ورک فلو اور سورس سے بلڈ: [سیٹ اپ](/start/setup)
