---
summary: "แผน: แยก browser act:evaluate ออกจากคิว Playwright โดยใช้ CDP พร้อมกำหนดเส้นตายแบบ end-to-end และการจัดการ ref resolution ที่ปลอดภัยยิ่งขึ้น"
owner: "openclaw"
status: "draft"
last_updated: "2026-02-10"
title: "Browser Evaluate CDP Refactor"
---

# แผนการรีแฟกเตอร์ Browser Evaluate CDP

## บริบท

`act:evaluate` ทำการรัน JavaScript ที่ผู้ใช้ระบุภายในหน้าเว็บ ปัจจุบันทำงานผ่าน Playwright
(`page.evaluate` หรือ `locator.evaluate`) Playwright จะจัดลำดับคำสั่ง CDP ต่อหนึ่งหน้าแบบอนุกรม ดังนั้น evaluate ที่ค้างหรือทำงานนานอาจบล็อกคิวคำสั่งของหน้า และทำให้ทุกการกระทำถัดไปบนแท็บนั้นดูเหมือน "ค้าง"

PR #13498 เพิ่มกลไกความปลอดภัยเชิงปฏิบัติ (bounded evaluate, การส่งต่อ abort และการกู้คืนแบบพยายามเต็มที่) เอกสารนี้อธิบายการรีแฟกเตอร์ขนาดใหญ่ที่ทำให้ `act:evaluate` ถูกแยกออกจาก Playwright โดยเนื้อแท้ เพื่อให้ evaluate ที่ค้างไม่สามารถทำให้การทำงานปกติของ Playwright ติดขัดได้

## เป้าหมาย

- `act:evaluate` ต้องไม่สามารถบล็อกการทำงานของเบราว์เซอร์ถัดไปบนแท็บเดียวกันได้อย่างถาวร
- Timeout ต้องมีแหล่งอ้างอิงเดียวแบบ end-to-end เพื่อให้ผู้เรียกสามารถยึดตามงบเวลาได้
- Abort และ timeout ต้องถูกจัดการในลักษณะเดียวกันทั้งผ่าน HTTP และการเรียกใช้ภายในโปรเซส
- รองรับการระบุองค์ประกอบ (element targeting) สำหรับ evaluate โดยไม่ต้องย้ายทุกอย่างออกจาก Playwright
- คงความเข้ากันได้ย้อนหลังสำหรับผู้เรียกและ payload ที่มีอยู่

## สิ่งที่ไม่อยู่ในขอบเขต

- แทนที่การทำงานของเบราว์เซอร์ทั้งหมด (เช่น click, type, wait ฯลฯ) ด้วยการใช้งานผ่าน CDP
- ลบกลไกความปลอดภัยที่มีอยู่ซึ่งเพิ่มเข้ามาใน PR #13498 (ยังคงเป็น fallback ที่มีประโยชน์)
- แนะนำความสามารถที่ไม่ปลอดภัยใหม่เพิ่มเติม นอกเหนือจากเกต `browser.evaluateEnabled` ที่มีอยู่
- เพิ่มการแยก process (worker process/thread) สำหรับ evaluate หากหลังจากการปรับโครงสร้างครั้งนี้
  เรายังคงพบสถานะค้างที่กู้คืนได้ยาก นั่นอาจเป็นแนวคิดสำหรับดำเนินการต่อในลำดับถัดไป

## สถาปัตยกรรมปัจจุบัน (เหตุใดจึงเกิดอาการค้าง)

ในภาพรวม:

- ผู้เรียกส่ง `act:evaluate` ไปยังบริการควบคุมเบราว์เซอร์
- route handler เรียกใช้ Playwright เพื่อรัน JavaScript
- Playwright จัดลำดับคำสั่งของหน้าแบบอนุกรม ดังนั้น evaluate ที่ไม่สิ้นสุดจะบล็อกคิว
- เมื่อคิวค้าง การดำเนินการ click/type/wait ที่ตามมาบนแท็บอาจดูเหมือนค้างไปด้วย

## สถาปัตยกรรมที่เสนอ

### 1. การส่งต่อ Deadline

แนะนำแนวคิดเรื่องงบเวลา (budget) เดียว และให้ทุกอย่างอ้างอิงจากงบนี้:

- ผู้เรียกกำหนด `timeoutMs` (หรือ deadline ในอนาคต)
- timeout ของคำขอภายนอก, ลอจิกของ route handler และงบเวลาการประมวลผลภายในหน้า
  ใช้งบเดียวกัน โดยเผื่อเวลาเล็กน้อยสำหรับ overhead จากการ serialize
- การยกเลิก (abort) ถูกส่งต่อเป็น `AbortSignal` ในทุกส่วน เพื่อให้การยกเลิกสอดคล้องกัน

แนวทางการพัฒนา:

- เพิ่ม helper ขนาดเล็ก (เช่น `createBudget({ timeoutMs, signal })`) ที่ส่งคืนค่า:
  - `signal`: AbortSignal ที่เชื่อมโยงแล้ว
  - `deadlineAtMs`: deadline แบบค่าสัมบูรณ์
  - `remainingMs()`: เวลางบประมาณที่เหลือสำหรับการดำเนินการย่อย
- ใช้ helper นี้ใน:
  - `src/browser/client-fetch.ts` (HTTP และการ dispatch ภายใน process)
  - `src/node-host/runner.ts` (เส้นทาง proxy)
  - implementation ของ browser action (Playwright และ CDP)

### 2. แยก Evaluate Engine ออกจากกัน (เส้นทาง CDP)

เพิ่ม implementation ของ evaluate ที่อิง CDP ซึ่งไม่ใช้คิวคำสั่งต่อหน้าของ Playwright ร่วมกัน คุณสมบัติสำคัญคือ transport ของ evaluate เป็นการเชื่อมต่อ WebSocket แยกต่างหาก
และเป็น CDP session แยกที่ผูกกับ target

แนวทางการพัฒนา:

- โมดูลใหม่ เช่น `src/browser/cdp-evaluate.ts` ที่:
  - เชื่อมต่อกับ CDP endpoint ที่กำหนดค่าไว้ (socket ระดับเบราว์เซอร์)
  - ใช้ `Target.attachToTarget({ targetId, flatten: true })` เพื่อรับ `sessionId`
  - เรียกใช้หนึ่งในตัวเลือกต่อไปนี้:
    - `Runtime.evaluate` สำหรับ evaluate ระดับหน้า หรือ
    - `DOM.resolveNode` ร่วมกับ `Runtime.callFunctionOn` สำหรับ evaluate ระดับ element
  - เมื่อเกิด timeout หรือ abort:
    - ส่ง `Runtime.terminateExecution` แบบ best-effort สำหรับ session นั้น
    - ปิด WebSocket และส่งคืนข้อผิดพลาดที่ชัดเจน

หมายเหตุ:

- สิ่งนี้ยังคงรัน JavaScript ภายในหน้าเพจ ดังนั้นการยุติการทำงานอาจมีผลข้างเคียงได้ ข้อดี
  คือจะไม่ทำให้คิวของ Playwright ค้าง และสามารถยกเลิกได้ที่เลเยอร์การสื่อสาร
  โดยการปิดเซสชัน CDP

### 3. Ref Story (การกำหนดเป้าหมาย Element โดยไม่ต้องเขียนใหม่ทั้งหมด)

ส่วนที่ยากคือการกำหนดเป้าหมาย element CDP ต้องการ DOM handle หรือ `backendDOMNodeId` ในขณะที่
ปัจจุบันการทำงานของเบราว์เซอร์ส่วนใหญ่ใช้ Playwright locators ที่อ้างอิงจาก ref ใน snapshot

แนวทางที่แนะนำ: คง ref เดิมไว้ แต่เพิ่ม id ที่ CDP สามารถ resolve ได้เป็นตัวเลือกเสริม

#### 3.1 ขยายข้อมูล Ref ที่จัดเก็บ

ขยาย metadata ของ role ref ที่จัดเก็บไว้ให้สามารถมี CDP id ได้แบบไม่บังคับ:

- ปัจจุบัน: `{ role, name, nth }`
- ข้อเสนอ: `{ role, name, nth, backendDOMNodeId?: number }`

แนวทางนี้ทำให้ action ที่อิง Playwright เดิมทั้งหมดยังคงทำงานได้ และทำให้ CDP evaluate สามารถรับค่า `ref` เดียวกันได้เมื่อมี `backendDOMNodeId`

#### 3.2 เติมค่า backendDOMNodeId ตอนสร้าง Snapshot

เมื่อสร้าง role snapshot:

1. สร้าง role ref map แบบเดิมเหมือนปัจจุบัน (role, name, nth)
2. ดึง AX tree ผ่าน CDP (`Accessibility.getFullAXTree`) และคำนวณ map คู่ขนานของ
   `(role, name, nth) -> backendDOMNodeId` โดยใช้กฎการจัดการข้อมูลซ้ำแบบเดียวกัน
3. รวม id กลับเข้าไปในข้อมูล ref ที่จัดเก็บไว้สำหรับแท็บปัจจุบัน

หากการแมปสำหรับ ref ใดล้มเหลว ให้ปล่อย `backendDOMNodeId` เป็น undefined แนวทางนี้ทำให้ฟีเจอร์เป็นแบบ best-effort
และปลอดภัยต่อการทยอยเปิดใช้งาน

#### 3.3 พฤติกรรม Evaluate เมื่อมี Ref

ใน `act:evaluate`:

- หากมี `ref` และมี `backendDOMNodeId` ให้รัน element evaluate ผ่าน CDP
- หากมี `ref` แต่ไม่มี `backendDOMNodeId` ให้ fallback ไปใช้เส้นทางของ Playwright (พร้อม safety net)

ทางเลือกเพิ่มเติม (escape hatch):

- ขยายรูปแบบคำขอให้รับ `backendDOMNodeId` โดยตรงสำหรับผู้ใช้ขั้นสูง (และ
  สำหรับการดีบัก) โดยยังคงให้ `ref` เป็นอินเทอร์เฟซหลัก

### 4. คงเส้นทางการกู้คืนแบบทางเลือกสุดท้ายไว้

แม้จะใช้ CDP evaluate ก็ยังมีวิธีอื่นที่ทำให้แท็บหรือการเชื่อมต่อค้างได้ ให้คง
กลไกการกู้คืนที่มีอยู่ (terminate execution + disconnect Playwright) ไว้เป็นทางเลือกสุดท้าย
สำหรับ:

- ผู้เรียกใช้งานแบบเดิม (legacy callers)
- สภาพแวดล้อมที่ถูกบล็อกการเชื่อมต่อ CDP
- กรณี edge case ที่ไม่คาดคิดของ Playwright

## แผนการดำเนินการ (รอบเดียวจบ)

### สิ่งที่จะส่งมอบ

- เอนจิน evaluate ที่ใช้ CDP และทำงานนอกคิวคำสั่งต่อหน้าเพจของ Playwright
- งบเวลา timeout/abort แบบ end-to-end ชุดเดียวที่ผู้เรียกและตัวจัดการใช้อย่างสอดคล้องกัน
- metadata ของ ref ที่สามารถมี `backendDOMNodeId` สำหรับ element evaluate ได้แบบไม่บังคับ
- `act:evaluate` จะเลือกใช้เอนจิน CDP ก่อนเมื่อเป็นไปได้ และ fallback ไปใช้ Playwright เมื่อไม่สามารถใช้ได้
- การทดสอบที่พิสูจน์ได้ว่า evaluate ที่ค้างจะไม่ทำให้ action ถัดไปค้างตาม
- Logs/metrics ที่ทำให้มองเห็นความล้มเหลวและการ fallback ได้ชัดเจน

### รายการตรวจสอบการติดตั้งใช้งาน

1. เพิ่มตัวช่วย "budget" แบบใช้ร่วมกันเพื่อเชื่อม `timeoutMs` + `AbortSignal` ต้นทาง เข้าสู่:
   - `AbortSignal` เดียว
   - เส้นตายแบบกำหนดเวลาสัมบูรณ์
   - ตัวช่วย `remainingMs()` สำหรับการทำงานลำดับถัดไป
2. อัปเดตเส้นทางการเรียกใช้งานทั้งหมดให้ใช้ตัวช่วยดังกล่าว เพื่อให้ `timeoutMs` มีความหมายสอดคล้องกันทุกที่:
   - `src/browser/client-fetch.ts` (HTTP และการเรียกใช้งานภายในโปรเซส)
   - `src/node-host/runner.ts` (เส้นทาง node proxy)
   - CLI wrappers ที่เรียก `/act` (เพิ่ม `--timeout-ms` ให้กับ `browser evaluate`)
3. ติดตั้งใช้งาน `src/browser/cdp-evaluate.ts`:
   - เชื่อมต่อกับซ็อกเก็ต CDP ระดับเบราว์เซอร์
   - เรียก `Target.attachToTarget` เพื่อรับ `sessionId`
   - เรียก `Runtime.evaluate` สำหรับการ evaluate ระดับเพจ
   - เรียก `DOM.resolveNode` + `Runtime.callFunctionOn` สำหรับการ evaluate ระดับองค์ประกอบ
   - เมื่อ timeout/abort: เรียก `Runtime.terminateExecution` แบบ best-effort จากนั้นปิดซ็อกเก็ต
4. ขยายข้อมูลเมตาของ role ref ที่จัดเก็บไว้ให้สามารถรวม `backendDOMNodeId` แบบไม่บังคับได้:
   - คงพฤติกรรมเดิม `{ role, name, nth }` สำหรับการทำงานของ Playwright
   - เพิ่ม `backendDOMNodeId?: number` สำหรับการระบุเป้าหมายองค์ประกอบผ่าน CDP
5. กำหนดค่า `backendDOMNodeId` ระหว่างการสร้าง snapshot (แบบ best-effort):
   - ดึง AX tree ผ่าน CDP (`Accessibility.getFullAXTree`)
   - คำนวณการแมป `(role, name, nth) -> backendDOMNodeId` และรวมเข้ากับ ref map ที่จัดเก็บไว้
   - หากการแมปกำกวมหรือไม่พบ ให้ปล่อย id เป็น undefined
6. อัปเดตการกำหนดเส้นทาง `act:evaluate`:
   - หากไม่มี `ref`: ใช้ CDP evaluate เสมอ
   - หาก `ref` แปลงเป็น `backendDOMNodeId` ได้: ใช้ CDP element evaluate
   - กรณีอื่น: ย้อนกลับไปใช้ Playwright evaluate (ยังคงมีการจำกัดเวลาและยกเลิกได้)
7. คงเส้นทางกู้คืน "last resort" ที่มีอยู่เป็นทางเลือกสำรอง ไม่ใช่เส้นทางเริ่มต้น
8. เพิ่มการทดสอบ:
   - evaluate ที่ค้างอยู่ต้อง timeout ภายในงบเวลาที่กำหนด และการคลิก/พิมพ์ครั้งถัดไปต้องสำเร็จ
   - การ abort ต้องยกเลิก evaluate (client ตัดการเชื่อมต่อหรือ timeout) และปลดบล็อกการทำงานถัดไป
   - ความล้มเหลวของการแมปต้องย้อนกลับไปใช้ Playwright ได้อย่างถูกต้อง
9. เพิ่มการสังเกตการณ์:
   - ระยะเวลา evaluate และตัวนับ timeout
   - การใช้งาน terminateExecution
   - อัตราการ fallback (CDP -> Playwright) และเหตุผล

### เกณฑ์การยอมรับ

- `act:evaluate` ที่ถูกทำให้ค้างโดยตั้งใจต้องส่งผลลัพธ์กลับภายในงบเวลาของผู้เรียก และต้องไม่ทำให้
  แท็บค้างจนกระทบการทำงานถัดไป
- `timeoutMs` ทำงานสอดคล้องกันใน CLI, เครื่องมือเอเจนต์, node proxy และการเรียกใช้งานภายในโปรเซส
- หาก `ref` สามารถแมปเป็น `backendDOMNodeId` ได้ การ evaluate ระดับองค์ประกอบต้องใช้ CDP; มิฉะนั้น
  เส้นทาง fallback ยังคงถูกจำกัดเวลาและกู้คืนได้

## แผนการทดสอบ

- การทดสอบหน่วย (Unit tests):
  - ตรรกะการจับคู่ `(role, name, nth)` ระหว่าง role refs และโหนดใน AX tree
  - พฤติกรรมของตัวช่วยจัดการงบประมาณ (headroom, การคำนวณเวลาที่เหลือ)
- การทดสอบแบบบูรณาการ (Integration tests):
  - การหมดเวลา CDP evaluate จะต้องส่งคืนภายในงบประมาณที่กำหนดและไม่บล็อกการดำเนินการถัดไป
  - การยกเลิก (Abort) ต้องยกเลิก evaluate และพยายามเรียกใช้การยุติการทำงาน (termination) อย่างเต็มที่
- การทดสอบสัญญา (Contract tests):
  - ตรวจสอบให้แน่ใจว่า `BrowserActRequest` และ `BrowserActResponse` ยังคงเข้ากันได้

## ความเสี่ยงและแนวทางลดผลกระทบ

- การแมปปิงไม่สมบูรณ์แบบ:
  - แนวทางลดผลกระทบ: แมปแบบ best-effort, สำรองไปใช้ Playwright evaluate และเพิ่มเครื่องมือดีบัก
- `Runtime.terminateExecution` มีผลข้างเคียง:
  - แนวทางลดผลกระทบ: ใช้เฉพาะเมื่อเกิด timeout/abort เท่านั้น และบันทึกพฤติกรรมนี้ไว้ในข้อความข้อผิดพลาด
- ภาระงานเพิ่มเติม:
  - แนวทางลดผลกระทบ: ดึง AX tree เฉพาะเมื่อมีการร้องขอ snapshot, แคชต่อเป้าหมาย และคงอายุ CDP session ให้สั้น
    CDP session ให้มีอายุสั้น
- ข้อจำกัดของ Extension relay:
  - แนวทางลดผลกระทบ: ใช้ browser level attach APIs เมื่อไม่สามารถใช้ per page sockets ได้ และ
    คงเส้นทาง Playwright ปัจจุบันไว้เป็นทางเลือกสำรอง

## คำถามที่ยังเปิดอยู่

- ควรให้ engine ใหม่สามารถตั้งค่าเป็น `playwright`, `cdp` หรือ `auto` ได้หรือไม่?
- เราควรเปิดเผยรูปแบบ "nodeRef" ใหม่สำหรับผู้ใช้ขั้นสูง หรือคงไว้เฉพาะ `ref` เท่านั้น?
- frame snapshots และ selector scoped snapshots ควรมีส่วนร่วมในการแมป AX อย่างไร?
