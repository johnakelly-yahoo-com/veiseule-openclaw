---
title: "Platformalar"
---

# Platformalar

OpenClaw yadrosi TypeScript’da yozilgan. **Tavsiya etilgan ishga tushirish muhiti — Node**.
Gateway uchun Bun tavsiya etilmaydi (WhatsApp/Telegram xatolari sababli).

macOS (menyu paneli ilovasi) va mobil tugunlar (iOS/Android) uchun yordamchi ilovalar mavjud. Windows va
Linux uchun yordamchi ilovalar rejalashtirilgan, biroq Gateway hozirning o‘zida to‘liq qo‘llab-quvvatlanadi.
Windows uchun mahalliy yordamchi ilovalar ham rejalashtirilgan; Gateway’ni WSL2 orqali ishga tushirish tavsiya etiladi.

## OT-ni tanlang

- macOS: [macOS](/platforms/macos)
- iOS: [iOS](/platforms/ios)
- Android: [Android](/platforms/android)
- Windows: [Windows](/platforms/windows)
- Linux: [Linux](/platforms/linux)

## VPS va xosting

- VPS markazi: [VPS hosting](/vps)
- Fly.io: [Fly.io](/install/fly)
- Hetzner (Docker): [Hetzner](/install/hetzner)
- GCP (Compute Engine): [GCP](/install/gcp)
- exe.dev (VM + HTTPS proxy): [exe.dev](/install/exe-dev)

## Umumiy havolalar

- O‘rnatish qo‘llanmasi: [Getting Started](/start/getting-started)
- Gateway bo‘yicha qo‘llanma: [Gateway](/gateway)
- Gateway sozlamalari: [Configuration](/gateway/configuration)
- Xizmat holati: `openclaw gateway status`

## Gateway xizmatini o‘rnatish (CLI)

Quyidagilardan birini ishlating (barchasi qo‘llab-quvvatlanadi):

- Wizard (tavsiya etiladi): `openclaw onboard --install-daemon`
- To‘g‘ridan-to‘g‘ri: `openclaw gateway install`
- Sozlash jarayoni: `openclaw configure` → **Gateway service** ni tanlang
- Ta’mirlash/migratsiya: `openclaw doctor` (xizmatni o‘rnatish yoki tuzatishni taklif qiladi)

Xizmat manzili OT’ga bog‘liq:

- macOS: LaunchAgent (`bot.molt.gateway` yoki `bot.molt.<profile>`; eski `com.openclaw.*`)
- Linux/WSL2: systemd foydalanuvchi xizmati (`openclaw-gateway[-<profile>].service`)

