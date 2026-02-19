---
summary: "Gateway xizmati, hayotiy sikli va operatsiyalari bo‘yicha qo‘llanma"
read_when:
  - Running or debugging the gateway process
title: "Gateway qo‘llanmasi"
---

# Gateway service runbook

Last updated: 2025-12-09

<CardGroup cols={2}>
  <Card title="Deep troubleshooting" icon="siren" href="/gateway/troubleshooting">
    Alomatlardan boshlanadigan diagnostika aniq buyruqlar ketma-ketligi va log imzolari bilan.
  
</Card>
  <Card title="Configuration" icon="sliders" href="/gateway/configuration">
    Vazifaga yo‘naltirilgan o‘rnatish qo‘llanmasi + to‘liq konfiguratsiya ma’lumotnomasi.
  
</Card>
</CardGroup>

## 5 daqiqalik lokal ishga tushirish

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
openclaw gateway status
openclaw status
openclaw logs --follow
```

Sog‘lom asosiy holat: `Runtime: running` va `RPC probe: ok`.

  
</Step>

  <Step title="Validate channel readiness">

```bash
openclaw channels status --probe
```

  
</Step>
</Steps>

<Note>
Gateway konfiguratsiyasini qayta yuklash faol konfiguratsiya fayli yo‘lini kuzatadi (profil/state sukut bo‘yicha qiymatlaridan aniqlanadi yoki `OPENCLAW_CONFIG_PATH` o‘rnatilgan bo‘lsa, o‘shandan olinadi).
Sukut bo‘yicha rejim `gateway.reload.mode="hybrid"`.
</Note>

## Runtime modeli

- Marshrutlash, boshqaruv tekisligi va kanal ulanishlari uchun doimiy ishlaydigan bitta jarayon.
- Quyidagilar uchun yagona multiplekslangan port:
  - WebSocket boshqaruvi/RPC
  - HTTP API’lar (OpenAI-mos, Responses, tools invoke)
  - Boshqaruv UI va hook’lar
- Sukut bo‘yicha bind rejimi: `loopback`.
- Sukut bo‘yicha autentifikatsiya talab qilinadi (`gateway.auth.token` / `gateway.auth.password` yoki `OPENCLAW_GATEWAY_TOKEN` / `OPENCLAW_GATEWAY_PASSWORD`).

### Dev profili (`--dev`)

| Sozlama       | Aniqlash tartibi                                              |
| ------------- | ------------------------------------------------------------- |
| Gateway porti | `--port` → `OPENCLAW_GATEWAY_PORT` → `gateway.port` → `18789` |
| Bind rejimi   | CLI/override → `gateway.bind` → `loopback`                    |

### Hot reload rejimlari

| `gateway.reload.mode`                        | Xatti-harakat                                                 |
| -------------------------------------------- | ------------------------------------------------------------- |
| `off`                                        | Konfiguratsiya qayta yuklanmaydi                              |
| `hot`                                        | Faqat hot uchun xavfsiz o‘zgarishlarni qo‘llash               |
| `restart`                                    | Qayta yuklash talab qilinadigan o‘zgarishlarda restart qilish |
| `hybrid` (sukut bo‘yicha) | Xavfsiz bo‘lsa hot qo‘llash, zarur bo‘lsa restart qilish      |

## Operator buyruqlari to‘plami

```bash
openclaw gateway status
openclaw gateway status --deep
openclaw gateway status --json
openclaw gateway install
openclaw gateway restart
openclaw gateway stop
openclaw logs --follow
openclaw doctor
```

## Remote access

Tavsiya etiladi: Tailscale/VPN.
Zaxira usul: SSH tunnel.

```bash
ssh -N -L 18789:127.0.0.1:18789 user@host
```

Service install per profile:

<Warning>
Agar gateway auth sozlangan bo‘lsa, mijozlar hatto SSH tunnel orqali ham auth (`token`/`password`) yuborishi shart.
</Warning>

Qarang: [Remote Gateway](/gateway/remote), [Authentication](/gateway/authentication), [Tailscale](/gateway/tailscale).

## Nazorat va xizmat hayotiy sikli

Ishlab chiqarish muhitiga yaqin ishonchlilik uchun supervised ishga tushirishdan foydalaning.

<Tabs>
  <Tab title="macOS (launchd)">

```bash
openclaw gateway install
openclaw gateway status
openclaw gateway restart
openclaw gateway stop
```

LaunchAgent yorliqlari `ai.openclaw.gateway` (standart) yoki `ai.openclaw.<profile>` (nomlangan profil). `openclaw doctor` xizmat konfiguratsiyasidagi tafovutlarni tekshiradi va tuzatadi.

  
</Tab>

  <Tab title="Linux (systemd user)">

```bash
openclaw gateway install
systemctl --user enable --now openclaw-gateway[-<profile>].service
openclaw gateway status
```

Tizimdan chiqqandan keyin ham saqlanib qolishi uchun lingering’ni yoqing:

```bash
sudo loginctl enable-linger <user>
```

  
</Tab>

  <Tab title="Linux (system service)">

Ko‘p foydalanuvchili/doimiy ishlaydigan hostlar uchun system unit’dan foydalaning.

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now openclaw-gateway[-<profile>].service
```

  
</Tab>
</Tabs>

## Bitta hostda bir nechta gateway

Ko‘pchilik sozlamalarda **bitta** Gateway ishga tushirilishi kerak.
Bir nechta gateway’dan faqat qat’iy izolyatsiya/yaxlitlik (masalan, qutqaruv profili) uchun foydalaning.

Checklist per instance:

- Noyob `gateway.port`
- Noyob `OPENCLAW_CONFIG_PATH`
- Noyob `OPENCLAW_STATE_DIR`
- Noyob `agents.defaults.workspace`

Example:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json OPENCLAW_STATE_DIR=~/.openclaw-a openclaw gateway --port 19001
OPENCLAW_CONFIG_PATH=~/.openclaw/b.json OPENCLAW_STATE_DIR=~/.openclaw-b openclaw gateway --port 19002
```

Qarang: [Multiple gateways](/gateway/multiple-gateways).

### Gateway service management (CLI)

```bash
openclaw --dev setup
openclaw --dev gateway --allow-unconfigured
openclaw --dev status
```

Standart sozlamalar izolyatsiyalangan state/config va asosiy gateway porti `19001` ni o‘z ichiga oladi.

## Protocol bo‘yicha qisqa ma’lumot (operator ko‘rinishi)

- `gateway status` probes the Gateway RPC by default using the service’s resolved port/config (override with `--url`).
- `gateway status --deep` adds system-level scans (LaunchDaemons/system units).
- `gateway status --no-probe` skips the RPC probe (useful when networking is down).
- `gateway status --json` is stable for scripts.

Bundled mac app:

1. Darhol qabul qilingan tasdiq (`status:"accepted"`)
2. To stop it cleanly, use `openclaw gateway stop` (or `launchctl bootout gui/$UID/bot.molt.gateway`).

To‘liq protocol hujjatlari: [Gateway Protocol](/gateway/protocol).

## Operational checks

### Liveness

- WS oching va `connect` yuboring.
- Snapshot bilan `hello-ok` javobini kuting.

### Readiness

```bash
openclaw gateway status
openclaw channels status --probe
openclaw health
```

### Uzilishdan tiklanish

Events are not replayed. Ketma-ketlikda uzilishlar bo‘lsa, davom etishdan oldin holatni yangilang (`health`, `system-presence`).

## Keng tarqalgan nosozlik belgilari

| Belgi                                                          | Ehtimoliy muammo                                            |
| -------------------------------------------------------------- | ----------------------------------------------------------- |
| `refusing to bind gateway ... without auth`                    | Loopback bo‘lmagan bind token/passwordsiz                   |
| `another gateway instance is already listening` / `EADDRINUSE` | Port ziddiyati                                              |
| `Gateway start blocked: set gateway.mode=local`                | Config remote rejimga o‘rnatilgan                           |
| ulanish vaqtida `unauthorized`                                 | Client va gateway o‘rtasida autentifikatsiya mos kelmasligi |

To‘liq diagnostika bosqichlari uchun [Gateway Troubleshooting](/gateway/troubleshooting) dan foydalaning.

## Windows (WSL2)

- Gateway protokoli clientlari Gateway mavjud bo‘lmaganda darhol xatolik qaytaradi (to‘g‘ridan-to‘g‘ri kanalga avtomatik o‘tish yo‘q).
- Yaroqsiz/birinchi ulanishga tegishli bo‘lmagan freymlar rad etiladi va yopiladi.
- Socket yopilishidan oldin muloyim yopilish `shutdown` hodisasini chiqaradi.

---

Bog‘liq:

- [Troubleshooting](/gateway/troubleshooting)
- [Background Process](/gateway/background-process)
- [Configuration](/gateway/configuration)
- [Health](/gateway/health)
- [Doctor](/gateway/doctor)
- [Authentication](/gateway/authentication)

