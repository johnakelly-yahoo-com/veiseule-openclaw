---
title: "Ansible"
---

# การติดตั้ง Ansible

วิธีที่แนะนำในการปรับใช้ OpenClaw ไปยังเซิร์ฟเวอร์โปรดักชันคือผ่าน **[openclaw-ansible](https://github.com/openclaw/openclaw-ansible)** — ตัวติดตั้งอัตโนมัติที่ออกแบบโดยคำนึงถึงความปลอดภัยเป็นหลัก

## เริ่มต้นอย่างรวดเร็ว

ติดตั้งด้วยคำสั่งเดียว:

```bash
curl -fsSL https://raw.githubusercontent.com/openclaw/openclaw-ansible/main/install.sh | bash
```

> **📦 คู่มือฉบับเต็ม: [github.com/openclaw/openclaw-ansible](https://github.com/openclaw/openclaw-ansible)**
>
> > ที่เก็บ openclaw-ansible เป็นแหล่งข้อมูลหลักสำหรับการปรับใช้ด้วย Ansible หน้านี้เป็นภาพรวมโดยสรุปอย่างรวดเร็ว

## สิ่งที่คุณจะได้รับ

- 🔒 **ความปลอดภัยแบบไฟร์วอลล์เป็นอันดับแรก**: UFW + การแยกด้วย Docker (เข้าถึงได้เฉพาะ SSH + Tailscale)
- 🔐 **Tailscale VPN**: การเข้าถึงระยะไกลอย่างปลอดภัยโดยไม่ต้องเปิดบริการสู่สาธารณะ
- 🐳 **Docker**: คอนเทนเนอร์ sandbox ที่แยกอิสระ ผูกพอร์ตเฉพาะ localhost
- 🛡️ **การป้องกันหลายชั้น**: สถาปัตยกรรมความปลอดภัย 4 ชั้น
- 🚀 **ตั้งค่าด้วยคำสั่งเดียว**: ปรับใช้ครบถ้วนภายในไม่กี่นาที
- 🔧 **การผสานกับ Systemd**: เริ่มอัตโนมัติเมื่อบูตพร้อมการเสริมความปลอดภัย

## ข้อกำหนด

- **OS**: Debian 11+ หรือ Ubuntu 20.04+
- **การเข้าถึง**: สิทธิ์ root หรือ sudo
- **เครือข่าย**: การเชื่อมต่ออินเทอร์เน็ตสำหรับติดตั้งแพ็กเกจ
- **Ansible**: 2.14+ (ติดตั้งอัตโนมัติโดยสคริปต์เริ่มต้นอย่างรวดเร็ว)

## สิ่งที่ถูกติดตั้ง

Ansible playbook จะติดตั้งและกำหนดค่า:

1. **Tailscale** (mesh VPN สำหรับการเข้าถึงระยะไกลอย่างปลอดภัย)
2. **ไฟร์วอลล์ UFW** (เปิดเฉพาะพอร์ต SSH + Tailscale)
3. **Docker CE + Compose V2** (สำหรับ sandbox ของเอเจนต์)
4. **Node.js 22.x + pnpm** (ไลบรารีรันไทม์)
5. **OpenClaw** (รันบนโฮสต์โดยตรง ไม่ได้อยู่ในคอนเทนเนอร์)
6. **บริการ Systemd** (เริ่มอัตโนมัติพร้อมการเสริมความปลอดภัย)

หมายเหตุ: Gateway ทำงาน **โดยตรงบนโฮสต์** (ไม่อยู่ใน Docker) แต่ sandbox ของเอเจนต์ใช้ Docker เพื่อการแยกอิสระ ดูรายละเอียดได้ที่ [Sandboxing](/gateway/sandboxing) See [Sandboxing](/gateway/sandboxing) for details.

## การตั้งค่าหลังการติดตั้ง

หลังจากการติดตั้งเสร็จสิ้น ให้สลับไปยังผู้ใช้ openclaw:

```bash
sudo -i -u openclaw
```

สคริปต์หลังการติดตั้งจะนำทางคุณผ่านขั้นตอนต่อไปนี้:

1. **ตัวช่วยเริ่มต้นใช้งาน**: กำหนดค่าการตั้งค่า OpenClaw
2. **การเข้าสู่ระบบผู้ให้บริการ**: เชื่อมต่อ WhatsApp/Telegram/Discord/Signal
3. **การทดสอบ Gateway**: ตรวจสอบการติดตั้ง
4. **การตั้งค่า Tailscale**: เชื่อมต่อกับ VPN mesh ของคุณ

### คำสั่งด่วน

```bash
# Check service status
sudo systemctl status openclaw

# View live logs
sudo journalctl -u openclaw -f

# Restart gateway
sudo systemctl restart openclaw

# Provider login (run as openclaw user)
sudo -i -u openclaw
openclaw channels login
```

## สถาปัตยกรรมความปลอดภัย

### การป้องกัน 4 ชั้น

1. **ไฟร์วอลล์ (UFW)**: เปิดสาธารณะเฉพาะ SSH (22) + Tailscale (41641/udp)
2. **VPN (Tailscale)**: เข้าถึง Gateway ได้เฉพาะผ่าน VPN mesh
3. **การแยกด้วย Docker**: chain iptables ของ DOCKER-USER ป้องกันการเปิดพอร์ตจากภายนอก
4. **การเสริมความปลอดภัยด้วย Systemd**: NoNewPrivileges, PrivateTmp, ผู้ใช้ที่ไม่มีสิทธิ์พิเศษ

### การตรวจสอบ

ทดสอบพื้นผิวการโจมตีจากภายนอก:

```bash
nmap -p- YOUR_SERVER_IP
```

ควรแสดงว่าเปิด **เฉพาะพอร์ต 22** (SSH) เท่านั้น บริการอื่นทั้งหมด (Gateway, Docker) ถูกล็อกอย่างเข้มงวด All other services (gateway, Docker) are locked down.

### ความพร้อมใช้งานของ Docker

Docker ถูกติดตั้งสำหรับ **sandbox ของเอเจนต์** (การรันเครื่องมือแบบแยกอิสระ) ไม่ได้ใช้สำหรับรัน Gateway เอง Gateway จะผูกกับ localhost เท่านั้นและเข้าถึงผ่าน Tailscale VPN The gateway binds to localhost only and is accessible via Tailscale VPN.

ดูรายละเอียดการกำหนดค่า sandbox ได้ที่ [Multi-Agent Sandbox & Tools](/tools/multi-agent-sandbox-tools)

## การติดตั้งด้วยตนเอง

หากคุณต้องการควบคุมกระบวนการอัตโนมัติด้วยตนเอง:

```bash
# 1. Install prerequisites
sudo apt update && sudo apt install -y ansible git

# 2. Clone repository
git clone https://github.com/openclaw/openclaw-ansible.git
cd openclaw-ansible

# 3. Install Ansible collections
ansible-galaxy collection install -r requirements.yml

# 4. Run playbook
./run-playbook.sh

# Or run directly (then manually execute /tmp/openclaw-setup.sh after)
# ansible-playbook playbook.yml --ask-become-pass
```

## การอัปเดต OpenClaw

ตัวติดตั้ง Ansible จะตั้งค่า OpenClaw สำหรับการอัปเดตด้วยตนเอง ดูขั้นตอนมาตรฐานได้ที่ [Updating](/install/updating) See [Updating](/install/updating) for the standard update flow.

หากต้องการรัน Ansible playbook ซ้ำ (เช่น สำหรับการเปลี่ยนแปลงคอนฟิก):

```bash
cd openclaw-ansible
./run-playbook.sh
```

หมายเหตุ: กระบวนการนี้เป็นแบบ idempotent และปลอดภัยในการรันซ้ำหลายครั้ง

## การแก้ไขปัญหา

### ไฟร์วอลล์บล็อกการเชื่อมต่อของฉัน

หากคุณถูกตัดการเข้าถึง:

- ตรวจสอบให้แน่ใจว่าสามารถเข้าถึงผ่าน Tailscale VPN ได้ก่อน
- การเข้าถึง SSH (พอร์ต 22) ถูกอนุญาตเสมอ
- Gateway ถูกออกแบบให้เข้าถึงได้ **เฉพาะ** ผ่าน Tailscale เท่านั้น

### บริการไม่สามารถเริ่มทำงานได้

```bash
# Check logs
sudo journalctl -u openclaw -n 100

# Verify permissions
sudo ls -la /opt/openclaw

# Test manual start
sudo -i -u openclaw
cd ~/openclaw
pnpm start
```

### ปัญหา Docker sandbox

```bash
# Verify Docker is running
sudo systemctl status docker

# Check sandbox image
sudo docker images | grep openclaw-sandbox

# Build sandbox image if missing
cd /opt/openclaw/openclaw
sudo -u openclaw ./scripts/sandbox-setup.sh
```

### การเข้าสู่ระบบผู้ให้บริการล้มเหลว

ตรวจสอบให้แน่ใจว่าคุณกำลังรันในฐานะผู้ใช้ `openclaw`:

```bash
sudo -i -u openclaw
openclaw channels login
```

## การกำหนดค่าขั้นสูง

สำหรับรายละเอียดสถาปัตยกรรมความปลอดภัยและการแก้ไขปัญหาเชิงลึก:

- [Security Architecture](https://github.com/openclaw/openclaw-ansible/blob/main/docs/security.md)
- [Technical Details](https://github.com/openclaw/openclaw-ansible/blob/main/docs/architecture.md)
- [Troubleshooting Guide](https://github.com/openclaw/openclaw-ansible/blob/main/docs/troubleshooting.md)

## เกี่ยวข้อง

- [openclaw-ansible](https://github.com/openclaw/openclaw-ansible) — คู่มือการปรับใช้ฉบับเต็ม
- [Docker](/install/docker) — การตั้งค่า Gateway แบบคอนเทนเนอร์
- [Sandboxing](/gateway/sandboxing) — การกำหนดค่า sandbox ของเอเจนต์
- [Multi-Agent Sandbox & Tools](/tools/multi-agent-sandbox-tools) — การแยกอิสระต่อเอเจนต์


