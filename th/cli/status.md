---
title: "สถานะ"
---

# `openclaw status`

การวินิจฉัยสำหรับช่องทาง+เซสชัน

```bash
openclaw status
openclaw status --all
openclaw status --deep
openclaw status --usage
```

หมายเหตุ:

- `--deep` รันโพรบแบบสด (WhatsApp Web + Telegram + Discord + Google Chat + Slack + Signal)
- เอาต์พุตรวมสโตร์เซสชันต่อเอเจนต์เมื่อมีการกำหนดค่าเอเจนต์หลายตัว
- ภาพรวมรวมสถานะการติดตั้ง/รันไทม์ของGateway+บริการโฮสต์โหนดเมื่อมีให้ใช้งาน
- ภาพรวมรวมช่องทางอัปเดต+git SHA (สำหรับการเช็กซอร์ส)
- ข้อมูลอัปเดตจะแสดงในภาพรวม; หากมีอัปเดตให้ใช้งาน สถานะจะแสดงคำแนะนำให้รัน `openclaw update` (ดู [Updating](/install/updating))

