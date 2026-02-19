---
read_when:
  - الإعداد الأولي من الصفر
  - هل تريد معرفة أسرع طريق إلى دردشة تعمل؟
summary: قم بتثبيت OpenClaw وشغّل أول دردشة خلال دقائق.
title: البدء
x-i18n:
  generated_at: "2026-02-08T17:15:16Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 27aeeb3d18c495380e94e6b011b0df3def518535c9f1eee504f04871d8a32269
  source_path: start/getting-started.md
  workflow: 15
---

# البدء

الهدف: تحقيق أول دردشة تعمل بإعداد بسيط من الصفر.

<Info>
أسرع طريقة للدردشة: افتح Control UI (لا يلزم إعداد قناة). شغّل `openclaw dashboard` للدردشة عبر المتصفح، أو<Tooltip headline="Gatewayホスト" tip="OpenClaw Gatewayサービスを実行しているマシン。">مضيف Gateway</Tooltip>افتح `http://127.0.0.1:18789/`.
الوثائق: [Dashboard](/web/dashboard) و[Control UI](/web/control-ui).
</Info>

## المتطلبات الأساسية

- Node 22 أو أحدث

<Tip>
إذا لم تكن متأكدًا، تحقق من إصدار Node باستخدام `node --version`.
</Tip>

## الإعداد السريع (CLI)

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
    طرق ومتطلبات تثبيت أخرى: [التثبيت](/install).
    
</Note>
    ```

  
</Step>
  <Step title="オンボーディングウィザードを実行">
    ```bash
    openclaw onboard --install-daemon
    ```

    ```
    سيقوم المعالج بتكوين المصادقة وإعدادات Gateway والقنوات الاختيارية.
    لمزيد من التفاصيل، راجع [معالج الإعداد](/start/wizard).
    ```

  
</Step>
  <Step title="Gatewayを確認">
    إذا قمت بتثبيت الخدمة، فيفترض أنها قيد التشغيل بالفعل:

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
إذا تم تحميل Control UI، فإن Gateway جاهز للاستخدام.
</Check>

## التحقق الاختياري والميزات الإضافية

<AccordionGroup>
  <Accordion title="Gatewayをフォアグラウンドで実行">
    مفيد للاختبارات السريعة واستكشاف الأخطاء وإصلاحها.

    ````
    ```bash
    openclaw gateway --port 18789
    ```
    ````

  
</Accordion>
  <Accordion title="テストメッセージを送信">
    يتطلب قنوات مُكوَّنة مسبقًا.

    ````
    ```bash
    openclaw message send --target +15555550123 --message "Hello from OpenClaw"
    ```
    ````

  
</Accordion>
</AccordionGroup>

## مزيد من المعلومات

<Columns>
  <Card title="オンボーディングウィザード（詳細）" href="/start/wizard">
    مرجع كامل لمعالج CLI وخيارات متقدمة.
  
</Card>
  <Card title="macOSアプリのオンボーディング" href="/start/onboarding">
    سير عمل التشغيل الأول لتطبيق macOS.
  
</Card>
</Columns>

## الحالة بعد الإكمال

- Gateway قيد التشغيل
- مصادقة مُكوَّنة
- الوصول إلى Control UI أو قنوات متصلة

## الخطوات التالية

- أمان DM والموافقة: [الاقتران](/channels/pairing)
- توصيل المزيد من القنوات: [القنوات](/channels)
- سير عمل متقدم والبناء من المصدر: [الإعداد](/start/setup)
