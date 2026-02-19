---
summary: "รองรับ iMessage แบบเดิมผ่าน imsg (JSON-RPC ผ่าน stdio) การตั้งค่าใหม่ควรใช้ BlueBubbles การตั้งค่าใหม่ควรใช้ BlueBubbles การตั้งค่าใหม่ควรใช้ BlueBubbles"
read_when:
  - การตั้งค่าการรองรับ iMessage
  - การดีบักการส่ง/รับ iMessage
title: "iMessage"
---

# iMessage (legacy: imsg)

<Warning>
**แนะนำ:** ใช้ [BlueBubbles](/channels/bluebubbles) สำหรับการตั้งค่า iMessage ใหม่

ช่องทาง `imsg` เป็นการผสานรวม CLI ภายนอกแบบเดิม และอาจถูกนำออกในรีลีสอนาคต 
</Warning>

สถานะ: การผสานรวม CLI ภายนอกแบบเดิม สถานะ: การผสานรวม CLI ภายนอกแบบเดิม Gateway สร้าง `imsg rpc` (JSON-RPC ผ่าน stdio) Gateway จะเรียกใช้ `imsg rpc` และสื่อสารผ่าน JSON-RPC บน stdio (ไม่มี daemon/พอร์ตแยกต่างหาก)

<CardGroup cols={3}>
  <Card title="BlueBubbles (recommended)" icon="message-circle" href="/channels/bluebubbles">
    เส้นทาง iMessage ที่แนะนำสำหรับการตั้งค่าใหม่
  
</Card>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    ข้อความ DM ของ iMessage จะใช้โหมดการจับคู่ (pairing mode) เป็นค่าเริ่มต้น
  
</Card>
  <Card title="Configuration reference" icon="settings" href="/gateway/configuration-reference#imessage">
    เอกสารอ้างอิงฟิลด์ iMessage ฉบับสมบูรณ์
  
</Card>
</CardGroup>

## Quick setup (beginner)

<Tabs>
  <Tab title="Local Mac (fast path)">
    <Steps>
      <Step title="Install and verify imsg">

```bash
`brew install steipete/tap/imsg`
```

        
</Step>
      
        <Step title="กำหนดค่า OpenClaw">

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "/usr/local/bin/imsg",
      dbPath: "/Users/<you>/Library/Messages/chat.db",
    },
  },
}
```

        
</Step>
      
        <Step title="เริ่มต้น gateway">

```bash
openclaw gateway
```

        
</Step>
      
        <Step title="อนุมัติการจับคู่ DM ครั้งแรก (ค่าเริ่มต้น dmPolicy)">

```bash
`openclaw pairing approve imessage <CODE>`
```

        ```
            คำขอจับคู่จะหมดอายุภายใน 1 ชั่วโมง
          
</Step>
        
</Steps>
        ```

  
</Tab>

  <Tab title="Remote Mac over SSH">หากต้องการ iMessage บน Mac เครื่องอื่น ให้ตั้งค่า `channels.imessage.cliPath` ไปยัง wrapper ที่รัน `imsg` บนโฮสต์ macOS ระยะไกลผ่าน SSH OpenClaw ต้องการเพียง stdio เท่านั้น OpenClaw ต้องการเพียง stdio

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

    ```
    การกำหนดค่าที่แนะนำเมื่อเปิดใช้งานไฟล์แนบ:
    ```

```json5
{
  channels: {
    imessage: {
      cliPath: "~/imsg-ssh", // SSH wrapper to remote Mac
      remoteHost: "user@gateway-host", // for SCP file transfer
      includeAttachments: true,
    },
  },
}
```

    ```
    หากไม่ได้ตั้งค่า `remoteHost` OpenClaw จะพยายามตรวจจับอัตโนมัติโดยการพาร์สคำสั่ง SSH ในสคริปต์ wrapper ของคุณ แนะนำให้กำหนดค่าแบบชัดเจนเพื่อความเสถียร แนะนำให้กำหนดค่าชัดเจนเพื่อความเชื่อถือได้
    ```

  
</Tab>
</Tabs>

## ข้อกำหนดและสิทธิ์ (macOS)

- macOS ที่ลงชื่อเข้าใช้ Messages แล้ว
- **Full Disk Access**: อนุญาตให้เข้าถึงสำหรับโปรเซสที่รัน OpenClaw (และ wrapper shell/SSH ใดๆ ที่เรียก `imsg`) จำเป็นสำหรับการอ่านฐานข้อมูล Messages (`chat.db`) สิ่งนี้จำเป็นสำหรับการอ่านฐานข้อมูล Messages (`chat.db`)
- สิทธิ์ Automation เมื่อส่งข้อความ

<Tip>
สิทธิ์จะถูกกำหนดแยกตาม process context หาก gateway ทำงานแบบไม่มีหน้าจอ (LaunchAgent/SSH) ให้รันคำสั่งแบบโต้ตอบหนึ่งครั้งใน context เดียวกันนั้นเพื่อเรียกให้แสดงหน้าต่างขอสิทธิ์:

```bash
imsg chats --limit 1
# or
imsg send <handle> "test"
```

</Tip>

## การควบคุมการเข้าถึงและการกำหนดเส้นทาง

<Tabs>
  <Tab title="DM policy">
    `channels.imessage.dmPolicy` ใช้ควบคุมข้อความโดยตรง (DM):

    ```
    `channels.imessage.groupPolicy`: `open | allowlist | disabled` (ค่าเริ่มต้น: allowlist)
    ```

  
</Tab>

  <Tab title="Group policy + mentions">`channels.imessage.groupAllowFrom`: allowlist ผู้ส่งในกลุ่ม

    ```
    {
      channels: {
        imessage: {
          enabled: true,
          accounts: {
            bot: {
              name: "Bot",
              enabled: true,
              cliPath: "/path/to/imsg-bot",
              dbPath: "/Users/<bot-macos-user>/Library/Messages/chat.db",
            },
          },
        },
      },
    }
    ```

  
</Tab>

  <Tab title="Sessions and deterministic replies">
    - DM ใช้การกำหนดเส้นทางแบบ direct; กลุ่มใช้การกำหนดเส้นทางแบบกลุ่ม
    DMs ใช้เซสชันหลักของเอเจนต์ร่วมกัน; กลุ่มจะแยกออก (`agent:<agentId>:imessage:group:<chat_id>`)
    - เซสชันกลุ่มจะแยกอิสระจากกัน (`agent:<agentId>Groups:<chat_id>`)
    การกำหนดเส้นทางแบบกำหนดแน่นอน: การตอบกลับจะกลับไปที่ iMessage เสมอ

    ```
    หากเธรดที่มีผู้เข้าร่วมหลายคนเข้ามาพร้อมกับ `is_group=false` คุณยังสามารถแยกได้โดย `chat_id` ด้วยการใช้ `channels.imessage.groups` (ดู “Group-ish threads” ด้านล่าง)
    ```

  
</Tab>
</Tabs>

## รูปแบบการปรับใช้ (Deployment patterns)

<AccordionGroup>
  <Accordion title="Dedicated bot macOS user (separate iMessage identity)">หากต้องการให้บอตส่งจาก **ตัวตน iMessage แยกต่างหาก** (และทำให้ Messages ส่วนตัวของคุณสะอาด) ให้ใช้ Apple ID เฉพาะ + ผู้ใช้ macOS เฉพาะ

    ```
    เปิด Messages ในผู้ใช้ macOS นั้นและลงชื่อเข้าใช้ iMessage ด้วย Apple ID ของบอต
    ```

  
</Accordion>

  <Accordion title="Remote Mac over Tailscale (example)">
    โครงสร้างทั่วไป:

    ```
    หาก Gateway รันบนโฮสต์/VM Linux แต่ iMessage ต้องรันบน Mac, Tailscale เป็นสะพานที่ง่ายที่สุด: Gateway ติดต่อ Mac ผ่าน tailnet รัน `imsg` ผ่าน SSH และ SCP ไฟล์แนบกลับมา
    ```

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "~/.openclaw/scripts/imsg-ssh",
      remoteHost: "bot@mac-mini.tailnet-1234.ts.net",
      includeAttachments: true,
      dbPath: "/Users/bot/Library/Messages/chat.db",
    },
  },
}
```

```bash
#!/usr/bin/env bash
exec ssh -T bot@mac-mini.tailnet-1234.ts.net imsg "$@"
```

    ```
    ใช้คีย์ SSH เพื่อให้ `ssh bot@mac-mini.tailnet-1234.ts.net` ทำงานได้โดยไม่มีพรอมป์
    ```

  
</Accordion>

  <Accordion title="Multi-account pattern">ช่องทาง iMessage ที่ทำงานบน `imsg` บน macOS

    ```
    แต่ละบัญชีสามารถกำหนดค่าแทนที่ฟิลด์ เช่น `cliPath`, `dbPath`, `allowFrom`, `groupPolicy`, `mediaMaxMb` และการตั้งค่าประวัติได้
    ```

  
</Accordion>
</AccordionGroup>

## สื่อ (Media), การแบ่งชิ้น (chunking) และปลายทางการจัดส่ง

<AccordionGroup>
  <Accordion title="Attachments and media">การอัปโหลดสื่อถูกจำกัดโดย `channels.imessage.mediaMaxMb` (ค่าเริ่มต้น 16)
</Accordion>

  <Accordion title="Outbound chunking">`channels.imessage.chunkMode`: `length` (ค่าเริ่มต้น) หรือ `newline` เพื่อแบ่งตามบรรทัดว่าง (ขอบเขตย่อหน้า) ก่อนการแบ่งตามความยาว
</Accordion>

  <Accordion title="Addressing formats">
    ปลายทางแบบระบุชัดเจนที่แนะนำ:

    ```
    - `chat_id:123` (แนะนำสำหรับการกำหนดเส้นทางที่เสถียร)
    - `chat_guid:...`
    - `chat_identifier:...`
    
    รองรับการระบุเป้าหมายแบบ handle ด้วย:
    
    - `imessage:+1555...`
    - `sms:+1555...`
    - `user@example.com`
    ```

```bash
imsg chats --limit 20
```

  
</Accordion>
</AccordionGroup>

## Config writes

โดยค่าเริ่มต้น iMessage ได้รับอนุญาตให้เขียนการอัปเดตคอนฟิกที่ถูกกระตุ้นโดย `/config set|unset` (ต้องใช้ `commands.config: true`)

ปิดใช้งานด้วย:

```json5
{
  channels: { imessage: { configWrites: false } },
}
```

## การแก้ไขปัญหา

<AccordionGroup>
  <Accordion title="imsg not found or RPC unsupported">
    ตรวจสอบ binary และการรองรับ RPC:

```bash
imsg rpc --help
openclaw channels status --probe
```

    ```
    หาก probe รายงานว่า RPC ไม่รองรับ ให้อัปเดต `imsg`.
    ```

  
</Accordion>

  <Accordion title="DMs are ignored">อนุมัติผ่าน:

    ```
    `channels.imessage.dmPolicy`: `pairing | allowlist | open | disabled` (ค่าเริ่มต้น: pairing)
    ```

  
</Accordion>

  <Accordion title="Group messages are ignored">หมายเหตุ:

    ```
    `channels.imessage.groupPolicy = open | allowlist | disabled`
    ```

  
</Accordion>

  <Accordion title="Remote attachments fail">เช็กลิสต์:

    ```
    `channels.imessage.remoteHost`: โฮสต์ SSH สำหรับการถ่ายโอนไฟล์แนบผ่าน SCP เมื่อ `cliPath` ชี้ไปยัง Mac ระยะไกล (เช่น `user@gateway-host`) ตรวจจับอัตโนมัติจาก wrapper SSH หากไม่ตั้งค่า Auto-detected from SSH wrapper if not set.
    ```

  
</Accordion>

  <Accordion title="macOS permission prompts were missed">
เรียกใช้อีกครั้งในเทอร์มินัล GUI แบบโต้ตอบภายใต้ผู้ใช้/เซสชันเดียวกัน และอนุมัติพรอมป์ต่าง ๆ:

```bash
imsg chats --limit 1
imsg send <handle> "test"
```

    ```
    Full Disk Access สำหรับ OpenClaw + `imsg` (การเข้าถึงฐานข้อมูล Messages)
    ```

  
</Accordion>
</AccordionGroup>

## ตัวชี้อ้างอิงการกำหนดค่า

- Configuration reference (iMessage)
- คอนฟิกเต็มรูปแบบ: [Configuration](/gateway/configuration)
- [Pairing](/channels/pairing)
- [BlueBubbles](/channels/bluebubbles)
