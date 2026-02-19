---
summary: "OpenClaw Gateway CLI (`openclaw gateway`) — gatewaylarni ishga tushirish, so‘rov yuborish va topish"
read_when:
  - Gateway’ni CLI orqali ishga tushirganda (dev yoki serverlarda)
  - Gateway autentifikatsiyasi, bind rejimlari va ulanishni nosozliklarini tuzatishda
  - Bonjour orqali gatewaylarni topishda (LAN + tailnet)
title: "gateway"
---

# Gateway CLI

Gateway — bu OpenClaw’ning WebSocket serveri (kanallar, tugunlar, sessiyalar, hook’lar).

Ushbu sahifadagi barcha subbuyruqlar `openclaw gateway …` ostida joylashgan.

Tegishli hujjatlar:

- [/gateway/bonjour](/gateway/bonjour)
- [/gateway/discovery](/gateway/discovery)
- [/gateway/configuration](/gateway/configuration)

## Gateway’ni ishga tushirish

Mahalliy Gateway jarayonini ishga tushirish:

```bash
openclaw gateway
```

Foreground uchun alias:

```bash
openclaw gateway run
```

Eslatmalar:

- By default, the Gateway refuses to start unless `gateway.mode=local` is set in `~/.openclaw/openclaw.json`. Use `--allow-unconfigured` for ad-hoc/dev runs.
- Autentifikatsiyasiz loopback’dan tashqariga bind qilish bloklanadi (xavfsizlik chorasi).
- `SIGUSR1` avtorizatsiya qilinganda jarayon ichida qayta ishga tushirishni ishga tushiradi (`commands.restart` ni yoqing yoki gateway tool/config apply/update’dan foydalaning).
- `SIGINT`/`SIGTERM` handlers stop the gateway process, but they don’t restore any custom terminal state. If you wrap the CLI with a TUI or raw-mode input, restore the terminal before exit.

### Parametrlar

- `--port <port>`: WebSocket porti (standart qiymat config/env’dan olinadi; odatda `18789`).
- `--bind <loopback|lan|tailnet|auto|custom>`: tinglovchi (listener) bind rejimi.
- `--auth <token|password>`: auth rejimini majburan o‘rnatish.
- `--token <token>`: token’ni majburan o‘rnatish (jarayon uchun `OPENCLAW_GATEWAY_TOKEN` ni ham o‘rnatadi).
- `--password <password>`: parolni majburan o‘rnatish (jarayon uchun `OPENCLAW_GATEWAY_PASSWORD` ni ham o‘rnatadi).
- `--tailscale <off|serve|funnel>`: Gateway’ni Tailscale orqali ochish.
- `--tailscale-reset-on-exit`: o‘chirilganda Tailscale serve/funnel konfiguratsiyasini tiklash.
- `--allow-unconfigured`: config’da `gateway.mode=local` bo‘lmasa ham gateway’ni ishga tushirishga ruxsat berish.
- `--dev`: agar mavjud bo‘lmasa, dev config + workspace yaratadi (BOOTSTRAP.md’ni o‘tkazib yuboradi).
- `--reset`: dev config + credential’lar + sessiyalar + workspace’ni tiklaydi (`--dev` talab qilinadi).
- `--force`: ishga tushirishdan oldin tanlangan portdagi mavjud listener’ni majburan to‘xtatadi.
- `--verbose`: batafsil loglar.
- `--claude-cli-logs`: konsolda faqat claude-cli loglarini ko‘rsatadi (va uning stdout/stderr’ini yoqadi).
- `--ws-log <auto|full|compact>`: websocket log uslubi (standart `auto`).
- `--compact`: `--ws-log compact` uchun alias.
- `--raw-stream`: modelning raw stream hodisalarini jsonl ko‘rinishida log qiladi.
- `--raw-stream-path <path>`: raw stream jsonl fayl yo‘li.

## Ishlayotgan Gateway’ga so‘rov yuborish

Barcha so‘rov buyruqlari WebSocket RPC’dan foydalanadi.

Chiqish rejimlari:

- Standart: inson o‘qiy oladigan (TTY’da rangli).
- `--json`: mashina o‘qiy oladigan JSON (stil/spinner’siz).
- `--no-color` (yoki `NO_COLOR=1`): inson uchun ko‘rinishni saqlagan holda ANSI ranglarini o‘chiradi.

Umumiy parametrlar (mavjud joylarda):

- `--url <url>`: Gateway WebSocket URL manzili.
- `--token <token>`: Gateway token’i.
- `--password <password>`: Gateway paroli.
- `--timeout <ms>`: timeout/budjet (buyruqqa qarab farq qiladi).
- `--expect-final`: “final” javobni kutish (agent chaqiruvlari).

Eslatma: `--url` o‘rnatilganda, CLI konfiguratsiya yoki muhit credentiallariga qaytmaydi.
`--token` yoki `--password` ni aniq ko‘rsating. Aniq ko‘rsatilgan hisob ma’lumotlarining yo‘qligi xato hisoblanadi.

### `gateway health`

```bash
openclaw gateway health --url ws://127.0.0.1:18789
```

### `gateway status`

`gateway status` Gateway servisining holatini (launchd/systemd/schtasks) va ixtiyoriy RPC probe’ni ko‘rsatadi.

```bash
openclaw gateway status
openclaw gateway status --json
```

Parametrlar:

- `--url <url>`: probe URL’ini majburan o‘rnatish.
- `--token <token>`: probe uchun token auth.
- `--password <password>`: probe uchun password auth.
- `--timeout <ms>`: probe timeout’i (standart `10000`).
- `--no-probe`: RPC probe’ni o‘tkazib yuborish (faqat servis ko‘rinishi).
- `--deep`: tizim darajasidagi servislarni ham tekshiradi.

### `gateway probe`

`gateway probe` — “hammasini debug qilish” buyrug‘i. U har doim tekshiradi:

- sozlangan masofaviy gateway (agar o‘rnatilgan bo‘lsa), va
- localhost (loopback) **hatto remote sozlangan bo‘lsa ham**.

Agar bir nechta gateway mavjud bo‘lsa, ularning barchasini chiqaradi. Izolyatsiyalangan profillar/portlardan foydalanganda (masalan, qutqaruv bot), bir nechta gateway qo‘llab-quvvatlanadi, ammo ko‘pchilik o‘rnatmalarda hali ham bitta gateway ishlaydi.

```bash
openclaw gateway probe
openclaw gateway probe --json
```

#### SSH orqali remote (Mac ilovasi bilan mos)

macOS ilovasidagi “Remote over SSH” rejimi lokal port-forward’dan foydalanadi, shunda remote gateway (faqat loopback’ga bind qilingan bo‘lishi mumkin) `ws://127.0.0.1:<port>` orqali mavjud bo‘ladi.

CLI ekvivalenti:

```bash
openclaw gateway probe --ssh user@gateway-host
```

Parametrlar:

- `--ssh <target>`: `user@host` yoki `user@host:port` (port standart `22`).
- `--ssh-identity <path>`: identity fayli.
- `--ssh-auto`: aniqlangan birinchi gateway host’ni SSH target sifatida tanlaydi (LAN/WAB only).

Config (ixtiyoriy, standart sifatida ishlatiladi):

- `gateway.remote.sshTarget`
- `gateway.remote.sshIdentity`

### `gateway call <method>`

Past darajadagi RPC yordamchi vosita.

```bash
openclaw gateway call status
openclaw gateway call logs.tail --params '{"sinceMs": 60000}'
```

## Gateway servisini boshqarish

```bash
openclaw gateway install
openclaw gateway start
openclaw gateway stop
openclaw gateway restart
openclaw gateway uninstall
```

Eslatmalar:

- `gateway install` quyidagilarni qo‘llab-quvvatlaydi: `--port`, `--runtime`, `--token`, `--force`, `--json`.
- Lifecycle buyruqlari skriptlash uchun `--json` ni qabul qiladi.

## Gateway’larni topish (Bonjour)

`gateway discover` Gateway beacon’larini (`_openclaw-gw._tcp`) qidiradi.

- Ko‘p manzilli DNS-SD: `local.`
- Unicast DNS-SD (Wide-Area Bonjour): domen tanlang (masalan: `openclaw.internal.`) va split DNS + DNS server sozlang; qarang [/gateway/bonjour](/gateway/bonjour)

Faqat Bonjour discovery yoqilgan gateway’lar (standart holatda yoqilgan) beacon’ni e’lon qiladi.

Wide-Area discovery yozuvlari (TXT) quyidagilarni o‘z ichiga oladi:

- `role` (gateway roli haqida ishora)
- `transport` (transport turi, masalan `gateway`)
- `gatewayPort` (WebSocket porti, odatda `18789`)
- `sshPort` (SSH porti; ko‘rsatilmagan bo‘lsa standart `22`)
- `tailnetDns` (mavjud bo‘lsa, MagicDNS hostname)
- `gatewayTls` / `gatewayTlsSha256` (TLS yoqilgan + sertifikat fingerprint’i)
- `cliPath` (ixtiyoriy, remote o‘rnatmalar uchun ishora)

### `gateway discover`

```bash
openclaw gateway discover
```

Parametrlar:

- `--timeout <ms>`: har bir buyruq uchun timeout (browse/resolve); standart `2000`.
- `--json`: mashina o‘qiy oladigan chiqish (stil/spinner’ni ham o‘chiradi).

Misollar:

```bash
openclaw gateway discover --timeout 4000
openclaw gateway discover --json | jq '.beacons[].wsUrl'
```
