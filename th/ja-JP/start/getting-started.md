---
read_when:
  - การตั้งค่าครั้งแรกตั้งแต่ศูนย์
  - ต้องการเส้นทางที่เร็วที่สุดไปยังแชทที่ใช้งานได้
summary: ติดตั้ง OpenClaw และเริ่มแชทแรกของคุณภายในไม่กี่นาที
title: เริ่มต้นใช้งาน
x-i18n:
  generated_at: "2026-02-08T17:15:16Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 27aeeb3d18c495380e94e6b011b0df3def518535c9f1eee504f04871d8a32269
  source_path: start/getting-started.md
  workflow: 15
---

# เริ่มต้นใช้งาน

เป้าหมาย: สร้างแชทที่ใช้งานได้ครั้งแรกจากศูนย์ด้วยการตั้งค่าขั้นต่ำ

<Info>
วิธีแชทที่เร็วที่สุด: เปิด Control UI (ไม่ต้องตั้งค่าช่องทาง) รัน `openclaw dashboard` แล้วแชทผ่านเบราว์เซอร์ หรือ
Gateway โฮสต์
โดยเปิด `http://127.0.0.1:18789/`
เอกสาร: [Dashboard](/web/dashboard) และ [Control UI](/web/control-ui)
<Tooltip headline="Gatewayホスト" tip="OpenClaw Gatewayサービスを実行しているマシン。">ข้อกำหนดเบื้องต้น</Tooltip>Node 22 ขึ้นไป
</Info>

## หากไม่แน่ใจ ให้ตรวจสอบเวอร์ชันของ Node ด้วย `node --version`

- การตั้งค่าอย่างรวดเร็ว (CLI)

<Tip>
        ```bash
        curl -fsSL https://openclaw.ai/install.sh | bash
        ```
      
</Tip>

````
    ```powershell
    iwr -useb https://openclaw.ai/install.ps1 | iex
    ```
````
----

<Steps>
  <Step title="OpenClawをインストール（推奨）">
    <Tabs>
      <Tab title="macOS/Linux"><Note>
วิธีการติดตั้งและข้อกำหนดอื่น ๆ: [ติดตั้ง](/install)
</Note>
</Tab>
      <Tab title="Windows (PowerShell)">  
</Step>
  
</Tab>
    
</Tabs>

    ```
    
        ```bash
        openclaw onboard --install-daemon
        ```
    ```

  
</Step>
  <Step title="オンボーディングウィザードを実行">
    ```bash
    openclaw onboard --install-daemon
    ```

    ```
    ตัวช่วยสร้างจะกำหนดค่าการยืนยันตัวตน การตั้งค่า Gateway และช่องทางเสริม (ถ้ามี)
    ดูรายละเอียดได้ที่[オンボーディングウィザード](/start/wizard)
    ```

  
</Step>
  <Step title="Gatewayを確認">
    หากคุณได้ติดตั้งบริการไว้ บริการควรทำงานอยู่แล้ว:

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
เมื่อ Control UI โหลดสำเร็จ แสดงว่า Gateway พร้อมใช้งานแล้ว
</Check>

## ตัวเลือกเพิ่มเติมและฟีเจอร์เสริม

<AccordionGroup>
  <Accordion title="Gatewayをフォアグラウンドで実行">
    สะดวกสำหรับการทดสอบอย่างรวดเร็วและการแก้ไขปัญหา


    ````
    ```bash
    openclaw gateway --port 18789
    ```
    ````

  
</Accordion>
  <Accordion title="テストメッセージを送信">
    จำเป็นต้องมีช่องทางที่กำหนดค่าไว้แล้ว


    ````
    ```bash
    openclaw message send --target +15555550123 --message "Hello from OpenClaw"
    ```
    ````

  
</Accordion>
</AccordionGroup>

## ข้อมูลเพิ่มเติม

<Columns>
  <Card title="オンボーディングウィザード（詳細）" href="/start/wizard">
    เอกสารอ้างอิง CLI wizard ฉบับเต็มและตัวเลือกขั้นสูง
  
</Card>
  <Card title="macOSアプリのオンボーディング" href="/start/onboarding">
    ขั้นตอนการเปิดใช้งานครั้งแรกของแอป macOS
  
</Card>
</Columns>

## สถานะหลังเสร็จสิ้น

- Gateway ที่กำลังทำงาน
- การยืนยันตัวตนที่กำหนดค่าแล้ว
- การเข้าถึง Control UI หรือมีช่องทางที่เชื่อมต่อแล้ว

## ขั้นตอนถัดไป

- ความปลอดภัยและการอนุมัติ DM: [การจับคู่](/channels/pairing)
- เชื่อมต่อช่องทางเพิ่มเติม: [ช่องทาง](/channels)
- เวิร์กโฟลว์ขั้นสูงและการบิลด์จากซอร์ส: [การตั้งค่า](/start/setup)
