---
read_when:
  - शुरुआत से प्रारंभिक सेटअप
  - कार्यरत चैट तक पहुँचने का सबसे तेज़ तरीका जानना चाहते हैं
summary: OpenClaw इंस्टॉल करें और कुछ ही मिनटों में अपनी पहली चैट चलाएँ।
title: शुरुआत करें
x-i18n:
  generated_at: "2026-02-08T17:15:16Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 27aeeb3d18c495380e94e6b011b0df3def518535c9f1eee504f04871d8a32269
  source_path: start/getting-started.md
  workflow: 15
---

# परिचय

लक्ष्य: शून्य से न्यूनतम सेटअप के साथ पहली कार्यशील चैट शुरू करना।

<Info>
सबसे तेज़ चैट तरीका: Control UI खोलें (चैनल कॉन्फ़िगरेशन की आवश्यकता नहीं है)। `openclaw dashboard` चलाकर ब्राउज़र में चैट करें, या
<Tooltip headline="Gatewayホスト" tip="OpenClaw Gatewayサービスを実行しているマシン。">Gateway होस्ट</Tooltip>
में `http://127.0.0.1:18789/` खोलें।
डॉक्यूमेंटेशन: [Dashboard](/web/dashboard) और [Control UI](/web/control-ui)।
</Info>

## पूर्वापेक्षाएँ

- Node 22 या बाद का संस्करण

<Tip>
यदि सुनिश्चित न हों, तो `node --version` से अपना Node संस्करण जाँचें।
</Tip>

## त्वरित सेटअप (CLI)

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
    अन्य इंस्टॉलेशन तरीकों और आवश्यकताओं के लिए: [इंस्टॉलेशन](/install)।
    
</Note>
    ```

  
</Step>
  <Step title="オンボーディングウィザードを実行">
    ```bash
    openclaw onboard --install-daemon
    ```

    ```
    विज़ार्ड प्रमाणीकरण, Gateway कॉन्फ़िगरेशन और वैकल्पिक चैनल सेट करता है।
    अधिक जानकारी के लिए [ऑनबोर्डिंग विज़ार्ड](/start/wizard) देखें।
    ```

  
</Step>
  <Step title="Gatewayを確認">
    यदि आपने सेवा इंस्टॉल की है, तो यह पहले से चल रही होनी चाहिए:


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
यदि Control UI लोड हो जाता है, तो Gateway उपयोग के लिए तैयार है।
</Check>

## वैकल्पिक जाँच और अतिरिक्त सुविधाएँ

<AccordionGroup>
  <Accordion title="Gatewayをフォアグラウンドで実行">
    त्वरित परीक्षण या समस्या निवारण के लिए उपयोगी।


    ````
    ```bash
    openclaw gateway --port 18789
    ```
    ````

  
</Accordion>
  <Accordion title="テストメッセージを送信">
    कॉन्फ़िगर किया हुआ चैनल आवश्यक है।


    ````
    ```bash
    openclaw message send --target +15555550123 --message "Hello from OpenClaw"
    ```
    ````

  
</Accordion>
</AccordionGroup>

## और जानें

<Columns>
  <Card title="オンボーディングウィザード（詳細）" href="/start/wizard">
    पूर्ण CLI विज़ार्ड संदर्भ और उन्नत विकल्प।
  
</Card>
  <Card title="macOSアプリのオンボーディング" href="/start/onboarding">
    macOS ऐप के पहले लॉन्च की प्रक्रिया।
  
</Card>
</Columns>

## पूर्ण होने के बाद की स्थिति

- चल रहा Gateway
- कॉन्फ़िगर किया हुआ प्रमाणीकरण
- Control UI एक्सेस या कनेक्टेड चैनल

## अगले चरण

- DM की सुरक्षा और अनुमोदन: [पेयरिंग](/channels/pairing)
- अधिक चैनल कनेक्ट करें: [चैनल](/channels)
- उन्नत वर्कफ़्लो और स्रोत से बिल्ड: [सेटअप](/start/setup)
