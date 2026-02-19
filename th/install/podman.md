---
summary: "รัน OpenClaw ในคอนเทนเนอร์ Podman แบบ rootless"
read_when:
  - คุณต้องการ gateway แบบคอนเทนเนอร์โดยใช้ Podman แทน Docker
title: "Podman"
---

# Podman

รัน OpenClaw gateway ในคอนเทนเนอร์ Podman แบบ **rootless** ใช้ image เดียวกันกับ Docker (สร้างจาก repo [Dockerfile](https://github.com/openclaw/openclaw/blob/main/Dockerfile))

## ข้อกำหนด

- Podman (rootless)
- ต้องใช้ sudo สำหรับการตั้งค่าครั้งเดียว (สร้างผู้ใช้ สร้าง image)

## เริ่มต้นอย่างรวดเร็ว

**1. การตั้งค่าครั้งเดียว** (จากโฟลเดอร์รากของ repo; สร้างผู้ใช้ สร้าง image และติดตั้งสคริปต์เริ่มต้น):

```bash
./setup-podman.sh
```

ขั้นตอนนี้จะสร้าง `~openclaw/.openclaw/openclaw.json` ขั้นต่ำให้ด้วย (ตั้งค่า `gateway.mode="local"`) เพื่อให้ gateway สามารถเริ่มทำงานได้โดยไม่ต้องรันวิซาร์ด

โดยค่าเริ่มต้น คอนเทนเนอร์จะ **ไม่** ถูกติดตั้งเป็นบริการ systemd คุณต้องเริ่มต้นด้วยตนเอง (ดูด้านล่าง) สำหรับการตั้งค่าแบบ production ที่มีการเริ่มอัตโนมัติและรีสตาร์ตอัตโนมัติ ให้ติดตั้งเป็นบริการผู้ใช้ systemd Quadlet แทน:

```bash
./setup-podman.sh --quadlet
```

(หรือกำหนด `OPENCLAW_PODMAN_QUADLET=1`; ใช้ `--container` เพื่อติดตั้งเฉพาะคอนเทนเนอร์และสคริปต์เปิดใช้งานเท่านั้น)

**2. เริ่มต้น gateway** (แบบแมนนวล สำหรับการทดสอบแบบรวดเร็ว):

```bash
./scripts/run-openclaw-podman.sh launch
```

**3. ตัวช่วยตั้งค่าเริ่มต้น** (เช่น เพื่อเพิ่มช่องทางหรือผู้ให้บริการ):

```bash
./scripts/run-openclaw-podman.sh launch setup
```

จากนั้นเปิด `http://127.0.0.1:18789/` และใช้โทเค็นจาก `~openclaw/.openclaw/.env` (หรือค่าที่แสดงโดยขั้นตอน setup)

## Systemd (Quadlet, ทางเลือก)

หากคุณรัน `./setup-podman.sh --quadlet` (หรือ `OPENCLAW_PODMAN_QUADLET=1`) จะมีการติดตั้งยูนิต [Podman Quadlet](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html) เพื่อให้ gateway ทำงานเป็นบริการ systemd ของผู้ใช้สำหรับผู้ใช้ openclaw บริการจะถูกเปิดใช้งานและเริ่มทำงานเมื่อสิ้นสุดขั้นตอน setup

- **เริ่มต้น:** `sudo systemctl --machine openclaw@ --user start openclaw.service`
- **หยุด:** `sudo systemctl --machine openclaw@ --user stop openclaw.service`
- **สถานะ:** `sudo systemctl --machine openclaw@ --user status openclaw.service`
- **บันทึก (Logs):** `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`

ไฟล์ quadlet อยู่ที่ `~openclaw/.config/containers/systemd/openclaw.container` หากต้องการเปลี่ยนพอร์ตหรือตัวแปร env ให้แก้ไขไฟล์ดังกล่าว (หรือไฟล์ `.env` ที่ถูกอ้างอิง) จากนั้นรัน `sudo systemctl --machine openclaw@ --user daemon-reload` และรีสตาร์ทบริการ เมื่อบูตระบบ บริการจะเริ่มทำงานอัตโนมัติหากเปิดใช้งาน lingering สำหรับ openclaw (ขั้นตอน setup จะดำเนินการนี้เมื่อมี loginctl ให้ใช้งาน)

หากต้องการเพิ่ม quadlet **หลังจาก** การตั้งค่าเริ่มต้นที่ไม่ได้ใช้ ให้รันใหม่: `./setup-podman.sh --quadlet`.

## ผู้ใช้ openclaw (ไม่สามารถล็อกอินได้)

`setup-podman.sh` จะสร้างผู้ใช้ระบบเฉพาะชื่อ `openclaw`:

- **เชลล์:** `nologin` — ไม่สามารถล็อกอินแบบโต้ตอบได้; ลดความเสี่ยงด้านความปลอดภัย

- **โฮมไดเรกทอรี:** เช่น `/home/openclaw` — เก็บ `~/.openclaw` (คอนฟิก, เวิร์กสเปซ) และสคริปต์เปิดใช้งาน `run-openclaw-podman.sh`

- **Rootless Podman:** ผู้ใช้ต้องมีช่วง **subuid** และ **subgid** ดิสทริบิวชันจำนวนมากจะกำหนดค่าเหล่านี้ให้อัตโนมัติเมื่อสร้างผู้ใช้ หาก setup แสดงคำเตือน ให้เพิ่มบรรทัดต่อไปนี้ใน `/etc/subuid` และ `/etc/subgid`:

  ```text
  openclaw:100000:65536
  ```

  จากนั้นเริ่มต้น gateway ในฐานะผู้ใช้นั้น (เช่น จาก cron หรือ systemd):

  ```bash
  sudo -u openclaw /home/openclaw/run-openclaw-podman.sh
  sudo -u openclaw /home/openclaw/run-openclaw-podman.sh setup
  ```

- **คอนฟิก:** เฉพาะ `openclaw` และ root เท่านั้นที่สามารถเข้าถึง `/home/openclaw/.openclaw` ได้ หากต้องการแก้ไขคอนฟิก: ใช้ Control UI เมื่อ gateway ทำงานแล้ว หรือรัน `sudo -u openclaw $EDITOR /home/openclaw/.openclaw/openclaw.json`.

## สภาพแวดล้อมและคอนฟิก

- **โทเค็น:** จัดเก็บใน `~openclaw/.openclaw/.env` เป็น `OPENCLAW_GATEWAY_TOKEN` `setup-podman.sh` และ `run-openclaw-podman.sh` จะสร้างให้หากไม่มี (ใช้ `openssl`, `python3` หรือ `od`)
- **ทางเลือก:** ในไฟล์ `.env` ดังกล่าว คุณสามารถกำหนดคีย์ของผู้ให้บริการ (เช่น `GROQ_API_KEY`, `OLLAMA_API_KEY`) และตัวแปร env อื่น ๆ ของ OpenClaw ได้
- **พอร์ตโฮสต์:** โดยค่าเริ่มต้น สคริปต์จะแมป `18789` (gateway) และ `18790` (bridge) แทนที่การแมปพอร์ตฝั่ง **โฮสต์** ด้วย `OPENCLAW_PODMAN_GATEWAY_HOST_PORT` และ `OPENCLAW_PODMAN_BRIDGE_HOST_PORT` ตอนเปิดใช้งาน
- **พาธ:** คอนฟิกและเวิร์กสเปซบนโฮสต์จะใช้ค่าเริ่มต้นเป็น `~openclaw/.openclaw` และ `~openclaw/.openclaw/workspace` แทนที่พาธบนโฮสต์ที่สคริปต์เปิดใช้งานใช้ด้วย `OPENCLAW_CONFIG_DIR` และ `OPENCLAW_WORKSPACE_DIR`.

## คำสั่งที่มีประโยชน์

- **Logs:** เมื่อใช้ quadlet: `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`. เมื่อใช้สคริปต์: `sudo -u openclaw podman logs -f openclaw`
- **หยุดการทำงาน:** เมื่อใช้ quadlet: `sudo systemctl --machine openclaw@ --user stop openclaw.service`. เมื่อใช้สคริปต์: `sudo -u openclaw podman stop openclaw`
- **เริ่มใหม่อีกครั้ง:** เมื่อใช้ quadlet: `sudo systemctl --machine openclaw@ --user start openclaw.service`. เมื่อใช้สคริปต์: รันสคริปต์เปิดใช้งานอีกครั้ง หรือใช้ `podman start openclaw`
- **ลบคอนเทนเนอร์:** `sudo -u openclaw podman rm -f openclaw` — การตั้งค่าและ workspace บนโฮสต์จะยังคงอยู่

## การแก้ไขปัญหา

- **Permission denied (EACCES) บน config หรือ auth-profiles:** คอนเทนเนอร์จะใช้ค่าเริ่มต้นเป็น `--userns=keep-id` และทำงานด้วย uid/gid เดียวกับผู้ใช้บนโฮสต์ที่รันสคริปต์ ตรวจสอบให้แน่ใจว่า `OPENCLAW_CONFIG_DIR` และ `OPENCLAW_WORKSPACE_DIR` บนโฮสต์เป็นเจ้าของโดยผู้ใช้นั้น
- **Gateway เริ่มต้นไม่สำเร็จ (ขาด `gateway.mode=local`):** ตรวจสอบให้แน่ใจว่าไฟล์ `~openclaw/.openclaw/openclaw.json` มีอยู่และตั้งค่า `gateway.mode="local"` `setup-podman.sh` จะสร้างไฟล์นี้หากไม่มีอยู่
- **Rootless Podman ล้มเหลวสำหรับผู้ใช้ openclaw:** ตรวจสอบว่า `/etc/subuid` และ `/etc/subgid` มีบรรทัดสำหรับ `openclaw` (เช่น `openclaw:100000:65536`) เพิ่มบรรทัดดังกล่าวหากไม่มี แล้วรีสตาร์ท
- **ชื่อคอนเทนเนอร์ถูกใช้งานอยู่:** สคริปต์เปิดใช้งานใช้ `podman run --replace` ดังนั้นคอนเทนเนอร์เดิมจะถูกแทนที่เมื่อคุณเริ่มใหม่อีกครั้ง หากต้องการล้างข้อมูลด้วยตนเอง: `podman rm -f openclaw`.
- **ไม่พบสคริปต์เมื่อรันในฐานะ openclaw:** ตรวจสอบว่าได้รัน `setup-podman.sh` แล้ว เพื่อให้ `run-openclaw-podman.sh` ถูกคัดลอกไปยังโฮมของ openclaw (เช่น `/home/openclaw/run-openclaw-podman.sh`)
- **ไม่พบบริการ Quadlet หรือเริ่มต้นไม่สำเร็จ:** รัน `sudo systemctl --machine openclaw@ --user daemon-reload` หลังจากแก้ไขไฟล์ `.container` Quadlet ต้องใช้ cgroups v2: `podman info --format '{{.Host.CgroupsVersion}}'` ควรแสดงค่า `2`

## ตัวเลือกเพิ่มเติม: รันในฐานะผู้ใช้ของคุณเอง

หากต้องการรัน gateway ด้วยผู้ใช้ปกติของคุณ (โดยไม่ต้องมีผู้ใช้ openclaw เฉพาะ): สร้าง image, สร้างไฟล์ `~/.openclaw/.env` พร้อม `OPENCLAW_GATEWAY_TOKEN` แล้วรันคอนเทนเนอร์ด้วย `--userns=keep-id` และ mount ไปยัง `~/.openclaw` ของคุณ สคริปต์เปิดใช้งานถูกออกแบบมาสำหรับรูปแบบผู้ใช้ openclaw; สำหรับการตั้งค่าแบบผู้ใช้คนเดียว คุณสามารถรันคำสั่ง `podman run` จากสคริปต์ด้วยตนเอง โดยชี้ config และ workspace ไปยังโฮมของคุณ แนะนำสำหรับผู้ใช้ส่วนใหญ่: ใช้ `setup-podman.sh` และรันในฐานะผู้ใช้ openclaw เพื่อแยกการตั้งค่าและโปรเซสออกจากกัน

