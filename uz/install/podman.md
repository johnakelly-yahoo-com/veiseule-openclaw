---
summary: "OpenClaw’ni rootless Podman konteynerida ishga tushirish"
read_when:
  - Docker o‘rniga Podman bilan konteynerlashtirilgan gateway’dan foydalanmoqchisiz
title: "Podman"
---

# Podman

OpenClaw gateway’ni **rootless** Podman konteynerida ishga tushiring. Docker bilan bir xil image’dan foydalanadi (repo’dagi [Dockerfile](https://github.com/openclaw/openclaw/blob/main/Dockerfile) dan build qiling).

## Talablar

- Podman (rootless)
- Bir martalik sozlash uchun Sudo (foydalanuvchi yaratish, image build qilish)

## Tezkor boshlash

**1. Bir martalik sozlash** (repo ildiz papkasidan; foydalanuvchi yaratadi, image build qiladi, ishga tushirish skriptini o‘rnatadi):

```bash
./setup-podman.sh
```

Bu, shuningdek, minimal `~openclaw/.openclaw/openclaw.json` faylini yaratadi (`gateway.mode="local"` ni o‘rnatadi), shunda gateway wizard’ni ishga tushirmasdan start olishi mumkin.

Standart holatda konteyner systemd xizmati sifatida o‘rnatilmaydi, uni qo‘lda ishga tushirasiz (quyida qarang). Ishlab chiqarish uslubidagi sozlama uchun, avtomatik ishga tushirish va qayta ishga tushirish bilan, uni systemd Quadlet foydalanuvchi xizmati sifatida o‘rnating:

```bash
./setup-podman.sh --quadlet
```

(Yoki `OPENCLAW_PODMAN_QUADLET=1` ni o‘rnating; faqat konteyner va ishga tushirish skriptini o‘rnatish uchun `--container` dan foydalaning.)

**2. Gateway’ni ishga tushiring** (qo‘lda, tezkor sinov uchun):

```bash
./scripts/run-openclaw-podman.sh launch
```

\*\*3. **Onboarding ustasi** (masalan, kanallar yoki provayderlarni qo‘shish uchun):

```bash
./scripts/run-openclaw-podman.sh launch setup
```

So‘ng `http://127.0.0.1:18789/` manzilini oching va `~openclaw/.openclaw/.env` ichidagi token’dan (yoki setup chiqargan qiymatdan) foydalaning.

## Systemd (Quadlet, ixtiyoriy)

Agar siz `./setup-podman.sh --quadlet` (yoki `OPENCLAW_PODMAN_QUADLET=1`) ni ishga tushirgan bo‘lsangiz, [Podman Quadlet](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html) birligi o‘rnatiladi va gateway openclaw foydalanuvchisi uchun systemd foydalanuvchi xizmati sifatida ishlaydi. Xizmat setup yakunida yoqiladi va ishga tushiriladi.

- **Ishga tushirish:** `sudo systemctl --machine openclaw@ --user start openclaw.service`
- **To‘xtatish:** `sudo systemctl --machine openclaw@ --user stop openclaw.service`
- **Holati:** `sudo systemctl --machine openclaw@ --user status openclaw.service`
- **Loglar:** `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`

Quadlet fayli `~openclaw/.config/containers/systemd/openclaw.container` manzilida joylashgan. Portlar yoki env’ni o‘zgartirish uchun, o‘sha faylni (yoki u foydalanadigan `.env` ni) tahrir qiling, so‘ng `sudo systemctl --machine openclaw@ --user daemon-reload` buyrug‘ini bajarib, xizmatni qayta ishga tushiring. Tizim yuklanganda, agar openclaw uchun lingering yoqilgan bo‘lsa, xizmat avtomatik ishga tushadi (agar loginctl mavjud bo‘lsa, setup buni bajaradi).

Quadlet’ni dastlab undan foydalanilmagan setup’dan **keyin** qo‘shish uchun quyidagini qayta ishga tushiring: `./setup-podman.sh --quadlet`.

## openclaw foydalanuvchisi (login qilmaydigan)

`setup-podman.sh` maxsus tizim foydalanuvchisi `openclaw` ni yaratadi:

- **Shell:** `nologin` — interaktiv login yo‘q; hujum yuzasini kamaytiradi.

- **Home:** masalan, `/home/openclaw` — `~/.openclaw` (konfiguratsiya, ish maydoni) va `run-openclaw-podman.sh` ishga tushirish skriptini saqlaydi.

- **Rootless Podman:** Foydalanuvchida **subuid** va **subgid** diapazoni bo‘lishi kerak. Ko‘plab distributivlar foydalanuvchi yaratilganda bularni avtomatik tayinlaydi. Agar setup ogohlantirish chiqarsa, `/etc/subuid` va `/etc/subgid` fayllariga quyidagi qatorlarni qo‘shing:

  ```text
  openclaw:100000:65536
  ```

  So‘ng gateway’ni o‘sha foydalanuvchi nomidan ishga tushiring (masalan, cron yoki systemd orqali):

  ```bash
  sudo -u openclaw /home/openclaw/run-openclaw-podman.sh
  sudo -u openclaw /home/openclaw/run-openclaw-podman.sh setup
  ```

- **Konfiguratsiya:** Faqat `openclaw` va root `/home/openclaw/.openclaw` ga kira oladi. Konfiguratsiyani tahrirlash uchun: gateway ishlayotganda Control UI’dan foydalaning yoki `sudo -u openclaw $EDITOR /home/openclaw/.openclaw/openclaw.json` buyrug‘ini bajaring.

## Muhit va konfiguratsiya

- **Token:** `~openclaw/.openclaw/.env` ichida `OPENCLAW_GATEWAY_TOKEN` sifatida saqlanadi. `setup-podman.sh` va `run-openclaw-podman.sh` u mavjud bo‘lmasa uni yaratadi (`openssl`, `python3` yoki `od` dan foydalanadi).
- **Ixtiyoriy:** O‘sha `.env` ichida provayder kalitlarini (masalan, `GROQ_API_KEY`, `OLLAMA_API_KEY`) va boshqa OpenClaw env o‘zgaruvchilarini belgilashingiz mumkin.
- **Host portlari:** Standart bo‘yicha skript `18789` (gateway) va `18790` (bridge) ni xaritalaydi. Ishga tushirishda **host** port xaritalashini `OPENCLAW_PODMAN_GATEWAY_HOST_PORT` va `OPENCLAW_PODMAN_BRIDGE_HOST_PORT` orqali o‘zgartiring.
- **Yo‘llar:** Host konfiguratsiyasi va ish maydoni standart bo‘yicha `~openclaw/.openclaw` va `~openclaw/.openclaw/workspace` ga o‘rnatiladi. Launch skripti tomonidan ishlatiladigan host yo‘llarini `OPENCLAW_CONFIG_DIR` va `OPENCLAW_WORKSPACE_DIR` orqali o‘zgartiring.

## Foydali buyruqlar

- **Loglar:** Quadlet bilan: `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`. Skript bilan: `sudo -u openclaw podman logs -f openclaw`
- **To‘xtatish:** Quadlet bilan: `sudo systemctl --machine openclaw@ --user stop openclaw.service`. Skript bilan: `sudo -u openclaw podman stop openclaw`
- **Qayta ishga tushirish:** Quadlet bilan: `sudo systemctl --machine openclaw@ --user start openclaw.service`. Skript bilan: launch skriptini qayta ishga tushiring yoki `podman start openclaw`
- **Konteynerni o‘chirish:** `sudo -u openclaw podman rm -f openclaw` — hostdagi config va workspace saqlanib qoladi

## Nosozliklarni bartaraf etish

- **Config yoki auth-profiles uchun Permission denied (EACCES):** Konteyner standart bo‘yicha `--userns=keep-id` dan foydalanadi va skriptni ishga tushirayotgan host foydalanuvchisi bilan bir xil uid/gid ostida ishlaydi. Hostdagi `OPENCLAW_CONFIG_DIR` va `OPENCLAW_WORKSPACE_DIR` shu foydalanuvchiga tegishli ekanligiga ishonch hosil qiling.
- **Gateway ishga tushishi bloklandi (yo‘q `gateway.mode=local`):** `~openclaw/.openclaw/openclaw.json` mavjud ekanligini va unda `gateway.mode="local"` o‘rnatilganini tekshiring. Agar mavjud bo‘lmasa, `setup-podman.sh` ushbu faylni yaratadi.
- **Rootless Podman openclaw foydalanuvchisi uchun ishlamayapti:** `/etc/subuid` va `/etc/subgid` fayllarida `openclaw` uchun qator borligini tekshiring (masalan, `openclaw:100000:65536`). Agar mavjud bo‘lmasa, qo‘shing va qayta ishga tushiring.
- **Konteyner nomi allaqachon ishlatilmoqda:** Launch skripti `podman run --replace` dan foydalanadi, shuning uchun qayta ishga tushirganda mavjud konteyner almashtiriladi. Qo‘lda tozalash uchun: `podman rm -f openclaw`.
- **openclaw sifatida ishga tushirilganda skript topilmadi:** `setup-podman.sh` ishga tushirilganiga va `run-openclaw-podman.sh` openclaw foydalanuvchisining home papkasiga (masalan, `/home/openclaw/run-openclaw-podman.sh`) ko‘chirilganiga ishonch hosil qiling.
- **Quadlet xizmati topilmadi yoki ishga tushmadi:** `.container` faylini tahrirlgandan so‘ng `sudo systemctl --machine openclaw@ --user daemon-reload` buyrug‘ini bajaring. Quadlet cgroups v2 ni talab qiladi: `podman info --format '{{.Host.CgroupsVersion}}'` natijasi `2` bo‘lishi kerak.

## Ixtiyoriy: o‘z foydalanuvchingiz sifatida ishga tushirish

Gateway’ni oddiy foydalanuvchingiz sifatida ishga tushirish uchun (alohida openclaw foydalanuvchisiz): image’ni build qiling, `~/.openclaw/.env` ichida `OPENCLAW_GATEWAY_TOKEN` yarating va konteynerni `--userns=keep-id` hamda `~/.openclaw` ga mount bilan ishga tushiring. Launch skripti openclaw foydalanuvchi oqimi uchun mo‘ljallangan; bitta foydalanuvchi uchun sozlamada skriptdagi `podman run` buyrug‘ini qo‘lda ishga tushirib, config va workspace’ni o‘z home papkangizga yo‘naltirishingiz mumkin. Ko‘pchilik foydalanuvchilar uchun tavsiya etiladi: `setup-podman.sh` dan foydalaning va config hamda jarayon izolyatsiya qilinishi uchun openclaw foydalanuvchisi sifatida ishga tushiring.

