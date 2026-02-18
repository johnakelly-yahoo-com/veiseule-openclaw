---
title: "GCP"
---

# GCP Compute Engine’da OpenClaw (Docker, Production VPS qo‘llanma)

## Maqsad

Docker yordamida GCP Compute Engine VM’da doimiy OpenClaw Gateway’ni ishga tushirish — barqaror holat, image ichiga joylangan binar fayllar va xavfsiz qayta ishga tushirish xatti-harakati bilan.

Agar siz “OpenClaw 24/7 ~$5-12/oy” variantini istasangiz, bu Google Cloud’da ishonchli sozlama hisoblanadi.  
Narx VM turi va hududga qarab farq qiladi; yuklamangizga mos eng kichik VM’ni tanlang va agar OOM yuz bersa, resursni oshiring.

## Nima qilamiz (oddiy tilda)?

- GCP loyihasini yaratamiz va billing’ni yoqamiz
- Compute Engine VM yaratamiz
- Docker o‘rnatamiz (izolyatsiyalangan ilova muhiti)
- OpenClaw Gateway’ni Docker’da ishga tushiramiz
- Host’da `~/.openclaw` + `~/.openclaw/workspace` papkalarini saqlaymiz (restart/rebuild’dan keyin ham saqlanadi)
- SSH tunnel orqali noutbukingizdan Control UI’ga kiramiz

Gateway’ga quyidagicha kirish mumkin:

- Noutbukdan SSH port forwarding orqali
- Agar firewall va tokenlarni o‘zingiz boshqarsangiz, portni to‘g‘ridan-to‘g‘ri ochish orqali

Ushbu qo‘llanma GCP Compute Engine’da Debian’dan foydalanadi.  
Ubuntu ham ishlaydi; paketlarni mos ravishda moslashtiring.  
Umumiy Docker jarayoni uchun [Docker](/install/docker) sahifasiga qarang.

---

## Tez yo‘l (tajribali operatorlar uchun)

1. GCP loyihasini yarating + Compute Engine API’ni yoqing
2. Compute Engine VM yarating (e2-small, Debian 12, 20GB)
3. VM’ga SSH orqali kiring
4. Docker o‘rnating
5. OpenClaw repository’sini klon qiling
6. Doimiy host papkalarini yarating
7. `.env` va `docker-compose.yml` ni sozlang
8. Kerakli binar fayllarni image ichiga joylab, build qiling va ishga tushiring

---

## Sizga kerak bo‘ladi

- GCP akkaunti (e2-micro uchun free tier mavjud)
- gcloud CLI o‘rnatilgan (yoki Cloud Console’dan foydalaning)
- Noutbukingizdan SSH kirish
- SSH + copy/paste bilan ishlash bo‘yicha asosiy tushuncha
- ~20–30 daqiqa
- Docker va Docker Compose
- Model autentifikatsiya ma’lumotlari
- Ixtiyoriy provayder ma’lumotlari
  - WhatsApp QR
  - Telegram bot token
  - Gmail OAuth

---

## 1) gcloud CLI o‘rnatish (yoki Console’dan foydalanish)

**Variant A: gcloud CLI** (avtomatlashtirish uchun tavsiya etiladi)

Quyidan o‘rnating: [https://cloud.google.com/sdk/docs/install](https://cloud.google.com/sdk/docs/install)

Initsializatsiya va autentifikatsiya:

```bash
gcloud init
gcloud auth login
```

**Variant B: Cloud Console**

Barcha bosqichlarni web interfeys orqali bajarish mumkin:  
[https://console.cloud.google.com](https://console.cloud.google.com)

---

## 2) GCP loyihasini yaratish

**CLI:**

```bash
gcloud projects create my-openclaw-project --name="OpenClaw Gateway"
gcloud config set project my-openclaw-project
```

Billing’ni yoqing:  
[https://console.cloud.google.com/billing](https://console.cloud.google.com/billing) (Compute Engine uchun majburiy).

Compute Engine API’ni yoqing:

```bash
gcloud services enable compute.googleapis.com
```

**Console:**

1. IAM & Admin > Create Project’ga o‘ting
2. Nom bering va yarating
3. Loyiha uchun billing’ni yoqing
4. APIs & Services > Enable APIs > "Compute Engine API" ni qidirib, Enable bosing

---

## 3) VM yaratish

**Mashina turlari:**

| Type     | Specs                    | Cost               | Notes                     |
| -------- | ------------------------ | ------------------ | ------------------------- |
| e2-small | 2 vCPU, 2GB RAM          | ~$12/oy            | Tavsiya etiladi           |
| e2-micro | 2 vCPU (shared), 1GB RAM | Free tier mavjud   | Yuklama ostida OOM bo‘lishi mumkin |

**CLI:**

```bash
gcloud compute instances create openclaw-gateway \
  --zone=us-central1-a \
  --machine-type=e2-small \
  --boot-disk-size=20GB \
  --image-family=debian-12 \
  --image-project=debian-cloud
```

**Console:**

1. Compute Engine > VM instances > Create instance
2. Name: `openclaw-gateway`
3. Region: `us-central1`, Zone: `us-central1-a`
4. Machine type: `e2-small`
5. Boot disk: Debian 12, 20GB
6. Create

---

## 4) VM’ga SSH orqali kirish

**CLI:**

```bash
gcloud compute ssh openclaw-gateway --zone=us-central1-a
```

**Console:**

Compute Engine panelida VM yonidagi "SSH" tugmasini bosing.

Eslatma: VM yaratilgandan keyin SSH kalitlari 1–2 daqiqa ichida tarqaladi. Agar ulanish rad etilsa, biroz kutib qayta urinib ko‘ring.

---

## 5) Docker o‘rnatish (VM ichida)

```bash
sudo apt-get update
sudo apt-get install -y git curl ca-certificates
curl -fsSL https://get.docker.com | sudo sh
sudo usermod -aG docker $USER
```

Guruh o‘zgarishi kuchga kirishi uchun chiqib, qayta kiring:

```bash
exit
```

Keyin yana SSH orqali kiring:

```bash
gcloud compute ssh openclaw-gateway --zone=us-central1-a
```

Tekshiring:

```bash
docker --version
docker compose version
```

---

## 6) OpenClaw repository’sini klon qilish

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
```

Ushbu qo‘llanma binar fayllar doimiy saqlanishini kafolatlash uchun custom image build qilishni nazarda tutadi.

---

## 7) Doimiy host papkalarini yaratish

Docker konteynerlari ephemeral (vaqtinchalik).  
Barcha uzoq muddatli holat host’da saqlanishi kerak.

```bash
mkdir -p ~/.openclaw
mkdir -p ~/.openclaw/workspace
```

---

## 8) Muhit o‘zgaruvchilarini sozlash

Repository root’da `.env` fayl yarating.

```bash
OPENCLAW_IMAGE=openclaw:latest
OPENCLAW_GATEWAY_TOKEN=change-me-now
OPENCLAW_GATEWAY_BIND=lan
OPENCLAW_GATEWAY_PORT=18789

OPENCLAW_CONFIG_DIR=/home/$USER/.openclaw
OPENCLAW_WORKSPACE_DIR=/home/$USER/.openclaw/workspace

GOG_KEYRING_PASSWORD=change-me-now
XDG_CONFIG_HOME=/home/node/.openclaw
```

Kuchli secret yarating:

```bash
openssl rand -hex 32
```

**Bu faylni commit qilmang.**

---

## 9) Docker Compose konfiguratsiyasi

`docker-compose.yml` ni yarating yoki yangilang.

```yaml
services:
  openclaw-gateway:
    image: ${OPENCLAW_IMAGE}
    build: .
    restart: unless-stopped
    env_file:
      - .env
    environment:
      - HOME=/home/node
      - NODE_ENV=production
      - TERM=xterm-256color
      - OPENCLAW_GATEWAY_BIND=${OPENCLAW_GATEWAY_BIND}
      - OPENCLAW_GATEWAY_PORT=${OPENCLAW_GATEWAY_PORT}
      - OPENCLAW_GATEWAY_TOKEN=${OPENCLAW_GATEWAY_TOKEN}
      - GOG_KEYRING_PASSWORD=${GOG_KEYRING_PASSWORD}
      - XDG_CONFIG_HOME=${XDG_CONFIG_HOME}
      - PATH=/home/linuxbrew/.linuxbrew/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    volumes:
      - ${OPENCLAW_CONFIG_DIR}:/home/node/.openclaw
      - ${OPENCLAW_WORKSPACE_DIR}:/home/node/.openclaw/workspace
    ports:
      # Tavsiya etiladi: Gateway’ni VM ichida faqat loopback’da qoldiring; SSH tunnel orqali kiring.
      # Agar ommaviy ochmoqchi bo‘lsangiz, `127.0.0.1:` prefiksini olib tashlang va firewall’ni mos ravishda sozlang.
      - "127.0.0.1:${OPENCLAW_GATEWAY_PORT}:18789"
    command:
      [
        "node",
        "dist/index.js",
        "gateway",
        "--bind",
        "${OPENCLAW_GATEWAY_BIND}",
        "--port",
        "${OPENCLAW_GATEWAY_PORT}",
      ]
```

---

## 10) Kerakli binar fayllarni image ichiga joylash (muhim)

Ishlayotgan konteyner ichida binar o‘rnatish — xato yondashuv.  
Runtime’da o‘rnatilgan har qanday narsa restart’dan keyin yo‘qoladi.

Skill’lar talab qiladigan barcha tashqi binar fayllar image build vaqtida o‘rnatilishi kerak.

Quyidagi misollar faqat uchta keng tarqalgan binarni ko‘rsatadi:

- `gog` — Gmail uchun
- `goplaces` — Google Places uchun
- `wacli` — WhatsApp uchun

Bular faqat misol, to‘liq ro‘yxat emas.  
Xuddi shu usul bilan istalgancha binar qo‘shishingiz mumkin.

Agar keyinroq qo‘shimcha binarga bog‘liq yangi skill qo‘shsangiz, quyidagilarni bajarishingiz kerak:

1. Dockerfile’ni yangilang  
2. Image’ni qayta build qiling  
3. Konteynerlarni qayta ishga tushiring  

**Namuna Dockerfile**

```dockerfile
FROM node:22-bookworm

RUN apt-get update && apt-get install -y socat && rm -rf /var/lib/apt/lists/*

# Example binary 1: Gmail CLI
RUN curl -L https://github.com/steipete/gog/releases/latest/download/gog_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/gog

# Example binary 2: Google Places CLI
RUN curl -L https://github.com/steipete/goplaces/releases/latest/download/goplaces_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/goplaces

# Example binary 3: WhatsApp CLI
RUN curl -L https://github.com/steipete/wacli/releases/latest/download/wacli_Linux_x86_64.tar.gz \
  | tar -xz -C /usr/local/bin && chmod +x /usr/local/bin/wacli

# Add more binaries below using the same pattern

WORKDIR /app
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml .npmrc ./
COPY ui/package.json ./ui/package.json
COPY scripts ./scripts

RUN corepack enable
RUN pnpm install --frozen-lockfile

COPY . .
RUN pnpm build
RUN pnpm ui:install
RUN pnpm ui:build

ENV NODE_ENV=production

CMD ["node","dist/index.js"]
```

---

## 11) Build qilish va ishga tushirish

```bash
docker compose build
docker compose up -d openclaw-gateway
```

Binarlarni tekshirish:

```bash
docker compose exec openclaw-gateway which gog
docker compose exec openclaw-gateway which goplaces
docker compose exec openclaw-gateway which wacli
```

Kutilgan natija:

```
/usr/local/bin/gog
/usr/local/bin/goplaces
/usr/local/bin/wacli
```

---

## 12) Gateway’ni tekshirish

```bash
docker compose logs -f openclaw-gateway
```

Muvaffaqiyatli holat:

```
[gateway] listening on ws://0.0.0.0:18789
```

---

## 13) Noutbukdan kirish

Gateway portini forward qilish uchun SSH tunnel yarating:

```bash
gcloud compute ssh openclaw-gateway --zone=us-central1-a -- -L 18789:127.0.0.1:18789
```

Brauzeringizda oching:

`http://127.0.0.1:18789/`

Gateway token’ingizni kiriting.

---

## Nima qayerda saqlanadi (haqiqiy manba)

OpenClaw Docker’da ishlaydi, lekin Docker haqiqiy manba emas.  
Barcha uzoq muddatli holat restart, rebuild va reboot’dan omon qolishi kerak.

| Component           | Location                          | Persistence mechanism | Notes                              |
| ------------------- | --------------------------------- | --------------------- | ---------------------------------- |
| Gateway config      | `/home/node/.openclaw/`           | Host volume mount     | `openclaw.json`, tokenlar          |
| Model auth profiles | `/home/node/.openclaw/`           | Host volume mount     | OAuth tokenlar, API kalitlar       |
| Skill configs       | `/home/node/.openclaw/skills/`    | Host volume mount     | Skill darajasidagi holat           |
| Agent workspace     | `/home/node/.openclaw/workspace/` | Host volume mount     | Kod va agent artefaktlari          |
| WhatsApp session    | `/home/node/.openclaw/`           | Host volume mount     | QR login saqlanadi                 |
| Gmail keyring       | `/home/node/.openclaw/`           | Host volume + password| `GOG_KEYRING_PASSWORD` talab etiladi |
| External binaries   | `/usr/local/bin/`                 | Docker image          | Build vaqtida image ichiga joylanadi |
| Node runtime        | Container filesystem              | Docker image          | Har build’da qayta yaratiladi      |
| OS packages         | Container filesystem              | Docker image          | Runtime’da o‘rnatmang              |
| Docker container    | Ephemeral                         | Restart qilinadi      | O‘chirish xavfsiz                  |

---

## Yangilash

VM’da OpenClaw’ni yangilash:

```bash
cd ~/openclaw
git pull
docker compose build
docker compose up -d
```

---

## Muammolarni hal qilish

**SSH connection refused**

VM yaratilgandan keyin SSH kalitlari 1–2 daqiqa ichida tarqaladi. Kuting va qayta urinib ko‘ring.

**OS Login muammolari**

OS Login profilingizni tekshiring:

```bash
gcloud compute os-login describe-profile
```

Hisobingizda kerakli IAM ruxsatlari (Compute OS Login yoki Compute OS Admin Login) mavjudligiga ishonch hosil qiling.

**Xotira yetishmasligi (OOM)**

Agar e2-micro ishlatayotgan bo‘lsangiz va OOM yuz bersa, e2-small yoki e2-medium’ga o‘ting:

```bash
# Avval VM’ni to‘xtating
gcloud compute instances stop openclaw-gateway --zone=us-central1-a

# Mashina turini o‘zgartiring
gcloud compute instances set-machine-type openclaw-gateway \
  --zone=us-central1-a \
  --machine-type=e2-small

# VM’ni ishga tushiring
gcloud compute instances start openclaw-gateway --zone=us-central1-a
```

---

## Service account’lar (xavfsizlik bo‘yicha eng yaxshi amaliyot)

Shaxsiy foydalanish uchun default user account yetarli.

Avtomatlashtirish yoki CI/CD pipeline’lar uchun minimal ruxsatlarga ega alohida service account yarating:

1. Service account yarating:

   ```bash
   gcloud iam service-accounts create openclaw-deploy \
     --display-name="OpenClaw Deployment"
   ```

2. Compute Instance Admin rolini (yoki yanada tor custom rolni) bering:

   ```bash
   gcloud projects add-iam-policy-binding my-openclaw-project \
     --member="serviceAccount:openclaw-deploy@my-openclaw-project.iam.gserviceaccount.com" \
     --role="roles/compute.instanceAdmin.v1"
   ```

Avtomatlashtirish uchun Owner rolidan foydalanmang. Eng kam ruxsat tamoyiliga amal qiling.

IAM rollari haqida batafsil:  
[https://cloud.google.com/iam/docs/understanding-roles](https://cloud.google.com/iam/docs/understanding-roles)

---

## Keyingi qadamlar

- Xabar almashish kanallarini sozlang: [Channels](/channels)
- Lokal qurilmalarni node sifatida ulang: [Nodes](/nodes)
- Gateway’ni sozlang: [Gateway configuration](/gateway/configuration)
