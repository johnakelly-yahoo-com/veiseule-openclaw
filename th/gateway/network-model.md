---
summary: "Gateway โหนด และโฮสต์แคนวาสเชื่อมต่อกันอย่างไร"
read_when:
  - คุณต้องการภาพรวมแบบกระชับของโมเดลเครือข่ายของGateway
title: "โมเดลเครือข่าย"
---

การทำงานส่วนใหญ่จะไหลผ่านGateway(`openclaw gateway`)ซึ่งเป็นโปรเซสเดียวที่รันต่อเนื่องยาวนานและเป็นเจ้าของการเชื่อมต่อช่องทางและคอนโทรลเพลนWebSocket

## กฎหลัก

- One Gateway per host is recommended. It is the only process allowed to own the WhatsApp Web session. สำหรับบอตกู้ภัยหรือการแยกแบบเข้มงวด ให้รันเกตเวย์หลายตัวด้วยโปรไฟล์และพอร์ตที่แยกจากกัน ดู [Multiple gateways](/gateway/multiple-gateways) ดู [Multiple gateways](/gateway/multiple-gateways)
- ลูปแบ็กก่อน: ค่าเริ่มต้นของ Gateway WS คือ `ws://127.0.0.1:18789` วิซาร์ดจะสร้างโทเคนของเกตเวย์ให้โดยค่าเริ่มต้น แม้สำหรับลูปแบ็ก Loopbackก่อน: ค่าเริ่มต้นของGateway WSคือ`ws://127.0.0.1:18789`วิซาร์ดจะสร้างโทเคนของGatewayเป็นค่าเริ่มต้นแม้สำหรับloopbackสำหรับการเข้าถึงผ่านtailnetให้รัน`openclaw gateway --bind tailnet --token ...`เนื่องจากการbindที่ไม่ใช่loopbackต้องใช้โทเคน วิซาร์ดจะสร้างโทเคนของเกตเวย์ให้โดยค่าเริ่มต้น แม้สำหรับลูปแบ็ก Loopbackก่อน: ค่าเริ่มต้นของGateway WSคือ`ws://127.0.0.1:18789`วิซาร์ดจะสร้างโทเคนของGatewayเป็นค่าเริ่มต้นแม้สำหรับloopbackสำหรับการเข้าถึงผ่านtailnetให้รัน`openclaw gateway --bind tailnet --token ...`เนื่องจากการbindที่ไม่ใช่loopbackต้องใช้โทเคน
- โหนดเชื่อมต่อกับGateway WSผ่านLAN, tailnetหรือSSHตามความจำเป็นบริดจ์TCPแบบเดิมถูกเลิกใช้งานแล้ว บริดจ์ TCP แบบเดิมเลิกใช้งานแล้ว บริดจ์ TCP แบบเดิมเลิกใช้งานแล้ว
- โฮสต์แคนวาสเป็นเซิร์ฟเวอร์ไฟล์HTTPที่`canvasHost.port`(ค่าเริ่มต้น`18793`)ให้บริการ`/__openclaw__/canvas/`สำหรับWebViewของโหนดดู[Gateway configuration](/gateway/configuration)(`canvasHost`) ดู [Gateway configuration](/gateway/configuration) (`canvasHost`)
  - `/__openclaw__/canvas/`
  - `/__openclaw__/a2ui/`
    เมื่อกำหนดค่า `gateway.auth` และ Gateway bind เกินกว่า loopback เส้นทางเหล่านี้จะถูกป้องกันด้วยการยืนยันตัวตนของ Gateway (คำขอจาก loopback จะได้รับการยกเว้น). ดู [Gateway configuration](/gateway/configuration) (`canvasHost`, `gateway`).
- การใช้งานจากระยะไกลมักเป็นอุโมงค์SSHหรือVPNแบบtailnetดู[Remote access](/gateway/remote)และ[Discovery](/gateway/discovery) ดู [Remote access](/gateway/remote) และ [Discovery](/gateway/discovery) ดู [Remote access](/gateway/remote) และ [Discovery](/gateway/discovery)

