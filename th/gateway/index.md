---
summary: "คู่มือปฏิบัติงานสำหรับบริการ Gateway วงจรชีวิต และการดำเนินงาน"
read_when:
  - เมื่อรันหรือดีบักกระบวนการGateway
title: "คู่มือปฏิบัติการ Gateway"
---

# คู่มือปฏิบัติการ Gateway

ใช้หน้านี้สำหรับการเริ่มต้นใช้งานวันแรก (day-1) และการดำเนินงานในวันถัดไป (day-2) ของบริการ Gateway

<CardGroup cols={2}>
  <Card title="Deep troubleshooting" icon="siren" href="/gateway/troubleshooting">
    การวินิจฉัยโดยเริ่มจากอาการ พร้อมลำดับคำสั่งที่ชัดเจนและลายเซ็นของล็อก
  
</Card>
  <Card title="Configuration" icon="sliders" href="/gateway/configuration">
    คู่มือการตั้งค่าแบบเน้นงาน + เอกสารอ้างอิงการกำหนดค่าฉบับสมบูรณ์
  
</Card>
</CardGroup>

## การเริ่มต้นใช้งานในเครื่องภายใน 5 นาที

<Steps>
  <Step title="Start the Gateway">

```bash
openclaw gateway --port 18789
# for full debug/trace logs in stdio:
openclaw gateway --port 18789 --verbose
# if the port is busy, terminate listeners then start:
openclaw gateway --force
# dev loop (auto-reload on TS changes):
pnpm gateway:watch
```

  
</Step>

  <Step title="Verify service health">

```bash
Linux: `openclaw-gateway-<profile>.service`
```

สถานะปกติ: `Runtime: running` และ `RPC probe: ok`.

  
</Step>

  <Step title="Validate channel readiness">

```bash
openclaw channels status --probe
```

  
</Step>
</Steps>

<Note>
การรีโหลดคอนฟิกของ Gateway จะเฝ้าดูพาธไฟล์คอนฟิกที่ใช้งานอยู่ (กำหนดจากค่าเริ่มต้นของโปรไฟล์/สถานะ หรือ `OPENCLAW_CONFIG_PATH` หากมีการตั้งค่า)
โหมดเริ่มต้นคือ `gateway.reload.mode="hybrid"`.
</Note>

## โมเดลรันไทม์

- หนึ่งโปรเซสที่ทำงานตลอดเวลา สำหรับการกำหนดเส้นทาง, control plane และการเชื่อมต่อช่องทาง
- พอร์ตแบบมัลติเพล็กซ์เดียวสำหรับ:
  - WebSocket control/RPC
  - OpenResponses (HTTP): [`/v1/responses`](/gateway/openresponses-http-api).
  - UI ควบคุมและ hooks
- โหมด bind เริ่มต้น: `loopback`.
- ต้องมีการยืนยันตัวตนของGatewayโดยค่าเริ่มต้น: ตั้งค่า `gateway.auth.token` (หรือ `OPENCLAW_GATEWAY_TOKEN`) หรือ `gateway.auth.password`.

### ลำดับความสำคัญของพอร์ตและการ bind

| การตั้งค่า    | ลำดับการตัดสินค่า (Resolution order)                                                                         |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| พอร์ต Gateway | ลำดับความสำคัญพอร์ต: `--port` > `OPENCLAW_GATEWAY_PORT` > `gateway.port` > ค่าเริ่มต้น `18789`. |
| โหมดการ bind  | CLI/override → `gateway.bind` → `loopback`                                                                                      |

### โหมด Hot reload

| ปิดใช้งานด้วย `gateway.reload.mode="off"`. | พฤติกรรม Keepalive                                         |
| ---------------------------------------------------------- | ---------------------------------------------------------- |
| `off`                                                      | ไม่มีการรีโหลดคอนฟิก                                       |
| `hot`                                                      | ใช้การเปลี่ยนแปลงเฉพาะที่ปลอดภัยสำหรับ hot reload เท่านั้น |
| `restart`                                                  | รีสตาร์ตเมื่อมีการเปลี่ยนแปลงที่ต้องรีโหลดใหม่             |
| `hybrid` (ค่าเริ่มต้น)                  | ใช้ hot reload เมื่อปลอดภัย และรีสตาร์ตเมื่อจำเป็น         |

## ชุดคำสั่งสำหรับผู้ปฏิบัติงาน

```bash
openclaw gateway status
openclaw gateway install
openclaw gateway stop
openclaw gateway restart
openclaw logs --follow
```

## การเข้าถึงระยะไกล

แนะนำ Tailscale/VPN; มิฉะนั้นใช้อุโมงค์SSH:
ทางเลือกสำรอง: SSH tunnel.

```bash
ssh -N -L 18789:127.0.0.1:18789 user@host
```

จากนั้นไคลเอนต์เชื่อมต่อไปยัง `ws://127.0.0.1:18789` ผ่านอุโมงค์

<Warning>
หากตั้งค่าโทเคนไว้ ไคลเอนต์ต้องแนบใน `connect.params.auth.token` แม้ผ่านอุโมงค์
</Warning>

ดู: [Remote Gateway](/gateway/remote), [Authentication](/gateway/authentication), [Tailscale](/gateway/tailscale).

## การกำกับดูแลและวงจรชีวิตของบริการ

ใช้การรันแบบ supervised เพื่อความเสถียรในระดับโปรดักชัน

<Tabs>
  <Tab title="macOS (launchd)">

```bash
openclaw --profile main gateway install
openclaw --profile rescue gateway install
```

ป้ายกำกับ LaunchAgent คือ `ai.openclaw.gateway` (ค่าเริ่มต้น) หรือ `ai.openclaw.<profile>` (โปรไฟล์ที่ตั้งชื่อ). `openclaw doctor` ตรวจสอบและซ่อมแซมความคลาดเคลื่อนของคอนฟิกบริการ

  
</Tab>

  <Tab title="Linux (systemd user)">

```bash
systemctl --user enable --now openclaw-gateway[-<profile>].service
```

เปิดใช้ lingering (จำเป็นเพื่อให้บริการผู้ใช้ยังอยู่หลังออกจากระบบ/ว่าง):

```bash
sudo loginctl enable-linger youruser
```

  
</Tab>

  <Tab title="Linux (system service)">

ใช้ system unit สำหรับโฮสต์ที่มีผู้ใช้หลายคน/ทำงานตลอดเวลา

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now openclaw-gateway[-<profile>].service
```

  
</Tab>
</Tabs>

## หลายGateway (โฮสต์เดียวกัน)

การตั้งค่าส่วนใหญ่ควรเรียกใช้ Gateway เพียง **หนึ่ง** ตัว
ใช้หลายตัวเฉพาะเมื่อจำเป็นต้องแยกการทำงานอย่างเข้มงวดหรือเพื่อความซ้ำซ้อน (เช่น โปรไฟล์กู้คืน)

เช็กลิสต์ต่ออินสแตนซ์:

- `gateway.port` ไม่ซ้ำ
- `OPENCLAW_CONFIG_PATH` ไม่ซ้ำ
- `OPENCLAW_STATE_DIR` ไม่ซ้ำ
- `agents.defaults.workspace` ไม่ซ้ำ

ตัวอย่าง:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json OPENCLAW_STATE_DIR=~/.openclaw-a openclaw gateway --port 19001
OPENCLAW_CONFIG_PATH=~/.openclaw/b.json OPENCLAW_STATE_DIR=~/.openclaw-b openclaw gateway --port 19002
```

See [Multiple gateways](/gateway/multiple-gateways).

### เส้นทางด่วนสำหรับโปรไฟล์ Dev

```bash
openclaw --dev setup
openclaw --dev gateway --allow-unconfigured
# then target the dev instance:
openclaw --dev status
openclaw --dev health
```

ค่าเริ่มต้นประกอบด้วย state/config ที่แยกกันและพอร์ต gateway พื้นฐาน `19001`

## โปรโตคอล (มุมมองผู้ปฏิบัติงาน)

- เฟรมแรกของไคลเอนต์ต้องเป็น `connect`
- Gateway จะส่งสแนปชอต `hello-ok` กลับมา (`presence`, `health`, `stateVersion`, `uptimeMs`, limits/policy)
- คำขอ: `{type:"req", id, method, params}` → `{type:"res", id, ok, payload|error}`
- เหตุการณ์ทั่วไป: `connect.challenge`, `agent`, `chat`, `presence`, `tick`, `health`, `heartbeat`, `shutdown`.

การรัน Agent มีสองขั้นตอน:

1. การตอบรับทันทีว่าได้รับแล้ว (`status:"accepted"`)
2. การตอบกลับ `agent` เป็นสองช่วง: ช่วงแรก `res` ack `{runId,status:"accepted"}`, จากนั้นผลลัพธ์สุดท้าย `res` `{runId,status:"ok"|"error",summary}` หลังรันเสร็จ; เอาต์พุตแบบสตรีมมาถึงเป็น `event:"agent"`.

เอกสารเต็ม: [Gateway protocol](/gateway/protocol) และ [Bridge protocol (legacy)](/gateway/bridge-protocol).

## การตรวจสอบการปฏิบัติงาน

### Liveness

- เปิด WS และส่ง `connect`
- คาดว่าจะได้รับการตอบกลับ `hello-ok` พร้อมสแนปชอต

### Readiness

```bash
`openclaw gateway health|status` — ขอสุขภาพ/สถานะผ่าน Gateway WS.
```

### การกู้คืนช่องว่าง (Gap recovery)

Events are not replayed. Clients detect seq gaps and should refresh (`health` + `system-presence`) before continuing.

## รูปแบบความล้มเหลวที่พบบ่อย

| ลายเซ็น                                                        | ปัญหาที่เป็นไปได้                                             |
| -------------------------------------------------------------- | ------------------------------------------------------------- |
| `refusing to bind gateway ... without auth`                    | การ bind กับที่อยู่ที่ไม่ใช่ loopback โดยไม่มี token/password |
| `another gateway instance is already listening` / `EADDRINUSE` | พอร์ตขัดแย้งกัน                                               |
| `Gateway start blocked: set gateway.mode=local`                | ตั้งค่า config เป็นโหมด remote                                |
| `unauthorized` ระหว่างการเชื่อมต่อ                             | การยืนยันตัวตนไม่ตรงกันระหว่าง client และ gateway             |

สำหรับขั้นตอนการวินิจฉัยแบบละเอียด โปรดดู [Gateway Troubleshooting](/gateway/troubleshooting).

## การรับประกันด้านความปลอดภัย

- ไคลเอนต์ที่ใช้โปรโตคอล Gateway จะล้มเหลวทันทีเมื่อ Gateway ไม่พร้อมใช้งาน (ไม่มีการ fallback ไปยัง direct-channel โดยอัตโนมัติ)
- เฟรมแรกที่ไม่ใช่การเชื่อมต่อหรือ JSON ที่ผิดรูปแบบจะถูกปฏิเสธและปิดซ็อกเก็ต
- ปิดอย่างนุ่มนวล: ส่งอีเวนต์ `shutdown` ก่อนปิด; ไคลเอนต์ต้องจัดการการปิด+เชื่อมต่อใหม่

---

ที่เกี่ยวข้อง:

- [Troubleshooting](/gateway/troubleshooting)
- [Background Process](/gateway/background-process)
- [Configuration](/gateway/configuration)
- [Health](/gateway/health)
- [Doctor](/gateway/doctor)
- [Authentication](/gateway/authentication)
