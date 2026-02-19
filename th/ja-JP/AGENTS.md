# AGENTS.md - พื้นที่ทำงานแปลเอกสาร ja-JP

## อ่านเมื่อ

- ดูแลรักษา `docs/ja-JP/**`
- อัปเดตกระบวนการแปลภาษาญี่ปุ่น (glossary/TM/prompt)
- จัดการข้อเสนอแนะหรือปัญหาย้อนกลับของการแปลภาษาญี่ปุ่น

## Pipeline (docs-i18n)

- เอกสารต้นฉบับ: `docs/**/*.md`
- เอกสารปลายทาง: `docs/ja-JP/**/*.md`
- Glossary: `docs/.i18n/glossary.ja-JP.json`
- Translation memory: `docs/.i18n/ja-JP.tm.jsonl`
- กฎของ Prompt: `scripts/docs-i18n/prompt.go`

การรันที่ใช้บ่อย:

```bash
# แบบกลุ่ม (โหมด doc; ใช้ parallel ได้)
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode doc -parallel 6 ../../docs/**/*.md

# ไฟล์เดียว
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode doc ../../docs/start/getting-started.md

# แก้ไขเล็กน้อย (โหมด segment; ใช้ TM; ไม่ใช้ parallel)
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode segment ../../docs/start/getting-started.md
```

หมายเหตุ:

- แนะนำให้ใช้โหมด `doc` สำหรับการแปลทั้งหน้า; ใช้โหมด `segment` สำหรับการแก้ไขเล็กน้อย
- หากไฟล์มีขนาดใหญ่มากแล้วเกิด timeout ให้แก้ไขเฉพาะจุดหรือแบ่งหน้าออกก่อน แล้วจึงรันใหม่อีกครั้ง
- หลังการแปล ให้ตรวจสอบแบบสุ่ม: โค้ดแบบ span/block ต้องไม่เปลี่ยนแปลง, ลิงก์/แองเคอร์ต้องไม่เปลี่ยนแปลง, และตัวแปร placeholder ต้องคงไว้
