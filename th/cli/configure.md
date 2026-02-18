---
title: "configure"
---

# `openclaw configure`

พรอมต์แบบโต้ตอบสำหรับตั้งค่าข้อมูลรับรอง อุปกรณ์ และค่าเริ่มต้นของเอเจนต์

หมายเหตุ: ส่วน **Model** ตอนนี้มีการเลือกได้หลายรายการสำหรับ allowlist ของ
`agents.defaults.models` (สิ่งที่แสดงใน `/model` และตัวเลือกโมเดล)

เคล็ดลับ: เรียกใช้ `openclaw config` โดยไม่ระบุคำสั่งย่อยจะเปิดวิซาร์ดเดียวกัน ใช้
`openclaw config get|set|unset` สำหรับการแก้ไขแบบไม่โต้ตอบ ใช้
`openclaw config get|set|unset` สำหรับการแก้ไขแบบไม่โต้ตอบ

เกี่ยวข้อง:

- เอกสารอ้างอิงการกำหนดค่าGateway: [Configuration](/gateway/configuration)
- กำหนดค่า CLI: [การตั้งค่า](/cli/config)

หมายเหตุ:

- การเลือกตำแหน่งที่Gatewayทำงานจะอัปเดต `gateway.mode` เสมอ คุณสามารถเลือก "Continue" โดยไม่ต้องตั้งค่าส่วนอื่นได้ หากต้องการเพียงเท่านี้ คุณสามารถเลือก "Continue" โดยไม่ต้องเลือกส่วนอื่น หากนั่นคือทั้งหมดที่คุณต้องการ
- บริการที่ยึดตามช่องทาง (Slack/Discord/Matrix/Microsoft Teams) จะถาม allowlist ของช่องทาง/ห้องระหว่างการตั้งค่า คุณสามารถป้อนชื่อหรือ ID ได้ วิซาร์ดจะทำการแปลงชื่อเป็น ID เมื่อเป็นไปได้ คุณสามารถป้อนชื่อหรือ ID ได้; วิซาร์ดจะแปลงชื่อเป็น ID เมื่อเป็นไปได้

## ตัวอย่าง

```bash
openclaw configure
openclaw configure --section models --section channels
```
