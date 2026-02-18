---
title: "Ansible"
---

# Ansible orqali o‘rnatish

OpenClaw’ni production serverlarga joylashtirishning tavsiya etilgan usuli — **[openclaw-ansible](https://github.com/openclaw/openclaw-ansible)** orqali, ya’ni xavfsizlikni birinchi o‘ringa qo‘ygan arxitekturaga ega avtomatlashtirilgan o‘rnatuvchi yordamida.

## Tez boshlash

Bitta buyruq bilan o‘rnatish:

```bash
curl -fsSL https://raw.githubusercontent.com/openclaw/openclaw-ansible/main/install.sh | bash
```

> **📦 To‘liq qo‘llanma: [github.com/openclaw/openclaw-ansible](https://github.com/openclaw/openclaw-ansible)**
>
> openclaw-ansible repozitoriyasi Ansible orqali joylashtirish uchun asosiy manba hisoblanadi. Ushbu sahifa esa qisqacha sharhdir.

## Siz nimalarga ega bo‘lasiz

- 🔒 **Firewall-first xavfsizlik**: UFW + Docker izolyatsiyasi (faqat SSH + Tailscale ochiq)
- 🔐 **Tailscale VPN**: Xizmatlarni ommaga ochmasdan xavfsiz masofaviy kirish
- 🐳 **Docker**: Izolyatsiyalangan sandbox konteynerlar, faqat localhost bilan bog‘lanish
- 🛡️ **Ko‘p qatlamli himoya**: 4 qatlamli xavfsizlik arxitekturasi
- 🚀 **Bitta buyruq bilan sozlash**: Bir necha daqiqada to‘liq joylashtirish
- 🔧 **Systemd integratsiyasi**: Yuklanishda avtomatik ishga tushish va qo‘shimcha himoya

## Talablar

- **OS**: Debian 11+ yoki Ubuntu 20.04+
- **Kirish huquqi**: Root yoki sudo imtiyozlari
- **Tarmoq**: Paketlarni o‘rnatish uchun internet aloqasi
- **Ansible**: 2.14+ (tez boshlash skripti orqali avtomatik o‘rnatiladi)

## Nimalar o‘rnatiladi

Ansible playbook quyidagilarni o‘rnatadi va sozlaydi:

1. **Tailscale** (xavfsiz masofaviy kirish uchun mesh VPN)
2. **UFW firewall** (faqat SSH + Tailscale portlari ochiq)
3. **Docker CE + Compose V2** (agent sandboxlari uchun)
4. **Node.js 22.x + pnpm** (runtime bog‘liqliklar)
5. **OpenClaw** (host’da ishlaydi, konteyner ichida emas)
6. **Systemd xizmati** (xavfsizlik bilan mustahkamlangan avtomatik ishga tushish)

Eslatma: Gateway **to‘g‘ridan-to‘g‘ri host’da ishlaydi** (Docker ichida emas), ammo agent sandboxlari izolyatsiya uchun Docker’dan foydalanadi. Batafsil ma’lumot uchun [Sandboxing](/gateway/sandboxing) sahifasiga qarang.

## O‘rnatishdan keyingi sozlash

O‘rnatish tugagach, openclaw foydalanuvchisiga o‘ting:

```bash
sudo -i -u openclaw
```

Post-install skript sizni quyidagilar orqali yo‘naltiradi:

1. **Onboarding ustasi**: OpenClaw sozlamalarini moslash
2. **Provider login**: WhatsApp/Telegram/Discord/Signal ulash
3. **Gateway sinovi**: O‘rnatishni tekshirish
4. **Tailscale sozlash**: VPN mesh tarmog‘iga ulanish

### Tezkor buyruqlar

```bash
# Xizmat holatini tekshirish
sudo systemctl status openclaw

# Jonli loglarni ko‘rish
sudo journalctl -u openclaw -f

# Gateway’ni qayta ishga tushirish
sudo systemctl restart openclaw

# Provider login (openclaw foydalanuvchisi sifatida bajariladi)
sudo -i -u openclaw
openclaw channels login
```

## Xavfsizlik arxitekturasi

### 4 qatlamli himoya

1. **Firewall (UFW)**: Ommaga faqat SSH (22) + Tailscale (41641/udp) ochiq
2. **VPN (Tailscale)**: Gateway faqat VPN mesh orqali kirish mumkin
3. **Docker izolyatsiyasi**: DOCKER-USER iptables zanjiri tashqi portlarni bloklaydi
4. **Systemd himoyasi**: NoNewPrivileges, PrivateTmp, imtiyozsiz foydalanuvchi

### Tekshirish

Tashqi hujum yuzasini sinab ko‘ring:

```bash
nmap -p- YOUR_SERVER_IP
```

Natijada **faqat 22-port** (SSH) ochiq bo‘lishi kerak. Boshqa barcha xizmatlar (gateway, Docker) yopiq bo‘ladi.

### Docker mavjudligi

Docker **agent sandboxlari** (izolyatsiyalangan vosita bajarilishi) uchun o‘rnatiladi, gateway’ning o‘zi uchun emas. Gateway faqat localhost’ga ulanadi va Tailscale VPN orqali kirish mumkin.

Sandbox sozlamalari haqida batafsil: [Multi-Agent Sandbox & Tools](/tools/multi-agent-sandbox-tools).

## Qo‘lda o‘rnatish

Agar avtomatlashtirish o‘rniga qo‘lda boshqarishni xohlasangiz:

```bash
# 1. Kerakli paketlarni o‘rnatish
sudo apt update && sudo apt install -y ansible git

# 2. Repozitoriyani klonlash
git clone https://github.com/openclaw/openclaw-ansible.git
cd openclaw-ansible

# 3. Ansible kolleksiyalarini o‘rnatish
ansible-galaxy collection install -r requirements.yml

# 4. Playbook’ni ishga tushirish
./run-playbook.sh

# Yoki to‘g‘ridan-to‘g‘ri ishga tushirish (so‘ng /tmp/openclaw-setup.sh ni qo‘lda bajarish kerak)
# ansible-playbook playbook.yml --ask-become-pass
```

## OpenClaw’ni yangilash

Ansible o‘rnatuvchisi OpenClaw’ni qo‘lda yangilash uchun sozlaydi. Standart yangilash jarayoni uchun [Updating](/install/updating) sahifasiga qarang.

Ansible playbook’ni qayta ishga tushirish (masalan, konfiguratsiya o‘zgarishlari uchun):

```bash
cd openclaw-ansible
./run-playbook.sh
```

Eslatma: Bu idempotent bo‘lib, bir necha marta ishga tushirish xavfsiz.

## Muammolarni bartaraf etish

### Firewall ulanishni blokladi

Agar tizimga kira olmay qolsangiz:

- Avvalo Tailscale VPN orqali kira olishingizni tekshiring
- SSH (22-port) doimo ochiq
- Gateway ataylab **faqat** Tailscale orqali ochiq

### Xizmat ishga tushmayapti

```bash
# Loglarni tekshirish
sudo journalctl -u openclaw -n 100

# Ruxsatlarni tekshirish
sudo ls -la /opt/openclaw

# Qo‘lda ishga tushirib ko‘rish
sudo -i -u openclaw
cd ~/openclaw
pnpm start
```

### Docker sandbox muammolari

```bash
# Docker ishlayotganini tekshirish
sudo systemctl status docker

# Sandbox image’ni tekshirish
sudo docker images | grep openclaw-sandbox

# Agar image mavjud bo‘lmasa, yaratish
cd /opt/openclaw/openclaw
sudo -u openclaw ./scripts/sandbox-setup.sh
```

### Provider login ishlamayapti

`openclaw` foydalanuvchisi sifatida ishlayotganingizga ishonch hosil qiling:

```bash
sudo -i -u openclaw
openclaw channels login
```

## Kengaytirilgan sozlamalar

Xavfsizlik arxitekturasi va muammolarni bartaraf etish bo‘yicha batafsil ma’lumot:

- [Xavfsizlik arxitekturasi](https://github.com/openclaw/openclaw-ansible/blob/main/docs/security.md)
- [Texnik tafsilotlar](https://github.com/openclaw/openclaw-ansible/blob/main/docs/architecture.md)
- [Muammolarni bartaraf etish qo‘llanmasi](https://github.com/openclaw/openclaw-ansible/blob/main/docs/troubleshooting.md)

## Bog‘liq sahifalar

- [openclaw-ansible](https://github.com/openclaw/openclaw-ansible) — to‘liq joylashtirish qo‘llanmasi
- [Docker](/install/docker) — konteyner asosidagi gateway sozlamasi
- [Sandboxing](/gateway/sandboxing) — agent sandbox sozlamalari
- [Multi-Agent Sandbox & Tools](/tools/multi-agent-sandbox-tools) — har bir agent uchun izolyatsiya

