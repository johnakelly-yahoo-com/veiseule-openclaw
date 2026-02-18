---
summary: "Windows (WSL2) qo‘llab-quvvatlovi + hamroh ilova holati"
read_when:
  - OpenClaw’ni Windows’da o‘rnatayotganda
  - Windows hamroh ilovasi holatini qidirayotganda
title: "Windows (WSL2)"
---

# Windows (WSL2)

Windows’da OpenClaw’dan foydalanish **WSL2 orqali** tavsiya etiladi (Ubuntu tavsiya qilinadi).  
CLI + Gateway Linux ichida ishlaydi, bu esa ish muhitini bir xil saqlaydi va
asboblar bilan moslikni ancha yaxshilaydi (Node/Bun/pnpm, Linux binarlari, skills). Native
Windows biroz murakkabroq bo‘lishi mumkin. WSL2 sizga to‘liq Linux tajribasini beradi — o‘rnatish uchun
bitta buyruq yetarli: `wsl --install`.

Native Windows hamroh ilovalari rejalashtirilgan.

## O‘rnatish (WSL2)

- [Boshlash](/start/getting-started) (WSL ichida foydalaning)
- [O‘rnatish va yangilanishlar](/install/updating)
- Rasmiy WSL2 qo‘llanmasi (Microsoft): [https://learn.microsoft.com/windows/wsl/install](https://learn.microsoft.com/windows/wsl/install)

## Gateway

- [Gateway ishchi yo‘riqnomasi](/gateway)
- [Konfiguratsiya](/gateway/configuration)

## Gateway servisni o‘rnatish (CLI)

WSL2 ichida:

```
openclaw onboard --install-daemon
```

Yoki:

```
openclaw gateway install
```

Yoki:

```
openclaw configure
```

So‘ralganda **Gateway service** ni tanlang.

Tuzatish/migratsiya:

```
openclaw doctor
```

## Kengaytirilgan: WSL servislarini LAN orqali ochish (portproxy)

WSL o‘zining virtual tarmog‘iga ega. Agar boshqa qurilma **WSL ichida**
ishlayotgan servisga (SSH, lokal TTS server yoki Gateway) ulanishi kerak bo‘lsa,
Windows portini joriy WSL IP manziliga yo‘naltirishingiz kerak. WSL IP manzili qayta ishga tushirilgandan
so‘ng o‘zgaradi, shuning uchun yo‘naltirish qoidasini yangilab turish talab qilinishi mumkin.

Misol (PowerShell’ni **Administrator sifatida** ishga tushiring):

```powershell
$Distro = "Ubuntu-24.04"
$ListenPort = 2222
$TargetPort = 22

$WslIp = (wsl -d $Distro -- hostname -I).Trim().Split(" ")[0]
if (-not $WslIp) { throw "WSL IP not found." }

netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=$ListenPort `
  connectaddress=$WslIp connectport=$TargetPort
```

Portni Windows Firewall orqali ruxsat bering (bir martalik):

```powershell
New-NetFirewallRule -DisplayName "WSL SSH $ListenPort" -Direction Inbound `
  -Protocol TCP -LocalPort $ListenPort -Action Allow
```

WSL qayta ishga tushirilgandan so‘ng portproxy’ni yangilang:

```powershell
netsh interface portproxy delete v4tov4 listenport=$ListenPort listenaddress=0.0.0.0 | Out-Null
netsh interface portproxy add v4tov4 listenport=$ListenPort listenaddress=0.0.0.0 `
  connectaddress=$WslIp connectport=$TargetPort | Out-Null
```

Eslatmalar:

- Boshqa qurilmadan SSH ulanish **Windows host IP manziliga** yo‘naltiriladi (masalan: `ssh user@windows-host -p 2222`).
- Masofaviy tugunlar **kirish mumkin bo‘lgan** Gateway URL’ga yo‘naltirilishi kerak (`127.0.0.1` emas); tasdiqlash uchun
  `openclaw status --all` dan foydalaning.
- LAN orqali kirish uchun `listenaddress=0.0.0.0` dan foydalaning; `127.0.0.1` faqat lokal kirishni ta’minlaydi.
- Agar buni avtomatik qilishni istasangiz, login vaqtida yangilash bosqichini
  ishga tushiradigan Scheduled Task ro‘yxatdan o‘tkazing.

## WSL2 o‘rnatish bo‘yicha bosqichma-bosqich qo‘llanma

### 1) WSL2 + Ubuntu o‘rnatish

PowerShell’ni (Admin) oching:

```powershell
wsl --install
# Yoki distributivni aniq tanlang:
wsl --list --online
wsl --install -d Ubuntu-24.04
```

Agar Windows so‘rasa, kompyuterni qayta ishga tushiring.

### 2) systemd ni yoqish (gateway o‘rnatish uchun talab qilinadi)

WSL terminalingizda:

```bash
sudo tee /etc/wsl.conf >/dev/null <<'EOF'
[boot]
systemd=true
EOF
```

So‘ng PowerShell’dan:

```powershell
wsl --shutdown
```

Ubuntu’ni qayta oching, keyin tekshiring:

```bash
systemctl --user status
```

### 3) OpenClaw o‘rnatish (WSL ichida)

WSL ichida Linux uchun Boshlash qo‘llanmasiga amal qiling:

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build # birinchi ishga tushirishda UI bog‘liqliklarini avtomatik o‘rnatadi
pnpm build
openclaw onboard
```

To‘liq qo‘llanma: [Boshlash](/start/getting-started)

## Windows hamroh ilovasi

Hozircha Windows uchun hamroh ilova mavjud emas. Agar buni amalga oshirishga
hissa qo‘shmoqchi bo‘lsangiz, hissalar mamnuniyat bilan qabul qilinadi.
