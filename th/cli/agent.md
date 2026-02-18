---
title: "เอเจนต์"
---

# `openclaw agent`

รันเทิร์นของเอเจนต์ผ่าน Gateway (ใช้ `--local` สำหรับแบบฝัง)
รันหนึ่งรอบของเอเจนต์ผ่านGateway（เกตเวย์）(ใช้ `--local` สำหรับแบบฝังตัว)
ใช้ `--agent <id>` เพื่อกำหนดเป้าหมายไปยังเอเจนต์ที่ตั้งค่าไว้โดยตรง

เกี่ยวข้อง:

- เครื่องมือส่งเอเจนต์: [ส่งเอเจนต์](/tools/agent-send)

## ตัวอย่าง

```bash
openclaw agent --to +15555550123 --message "status update" --deliver
openclaw agent --agent ops --message "Summarize logs"
openclaw agent --session-id 1234 --message "Summarize inbox" --thinking medium
openclaw agent --agent ops --message "Generate report" --deliver --reply-channel slack --reply-to "#reports"
```


