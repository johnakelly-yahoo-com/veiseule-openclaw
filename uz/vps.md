---
summary: "OpenClaw uchun VPS hosting xabi (Oracle/Fly/Hetzner/GCP/exe.dev)"
read_when:
  - Siz Gateway’ni bulutda ishga tushirmoqchisiz
  - Sizga VPS/hosting qo‘llanmalari bo‘yicha tezkor yo‘l xarita kerak
title: "VPS xostingi"
---

# VPS xostingi

Ushbu xab qo‘llab-quvvatlanadigan VPS/hosting qo‘llanmalariga havolalar beradi va bulut
deplomentlari yuqori darajada qanday ishlashini tushuntiradi.

## Provayderni tanlang

- **Railway** (bir bosishda + brauzer orqali sozlash): [Railway](/install/railway)
- **Northflank** (bir bosishda + brauzer orqali sozlash): [Northflank](/install/northflank)
- **Oracle Cloud (Always Free)**: [Oracle](/platforms/oracle) — $0/oy (Always Free, ARM; sig‘im/ro‘yxatdan o‘tish ba’zan muammoli bo‘lishi mumkin)
- **Fly.io**: [Fly.io](/install/fly)
- **Hetzner (Docker)**: [Hetzner](/install/hetzner)
- **GCP (Compute Engine)**: [GCP](/install/gcp)
- **exe.dev** (VM + HTTPS proxy): [exe.dev](/install/exe-dev)
- **AWS (EC2/Lightsail/free tier)**: works well too. Video guide:
  [https://x.com/techfrenAJ/status/2014934471095812547](https://x.com/techfrenAJ/status/2014934471095812547)

## Bulut sozlamalari qanday ishlaydi

- **Gateway VPS’da ishlaydi** va holat + ishchi muhitga egalik qiladi.
- Siz noutbuk/telefoningizdan **Control UI** yoki **Tailscale/SSH** orqali ulanasiz.
- VPS’ni yagona ishonchli manba deb hisoblang va holat + ishchi muhitni **zaxiralang**.
- Secure default: keep the Gateway on loopback and access it via SSH tunnel or Tailscale Serve.
  If you bind to `lan`/`tailnet`, require `gateway.auth.token` or `gateway.auth.password`.

Masofaviy kirish: [Gateway remote](/gateway/remote)  
Platformalar xabi: [Platforms](/platforms)

## VPS bilan birga node’lardan foydalanish

You can keep the Gateway in the cloud and pair **nodes** on your local devices
(Mac/iOS/Android/headless). Nodes provide local screen/camera/canvas and `system.run`
capabilities while the Gateway stays in the cloud.

Hujjatlar: [Nodes](/nodes), [Nodes CLI](/cli/nodes)

