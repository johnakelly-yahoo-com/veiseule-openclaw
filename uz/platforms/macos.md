---
summary: "OpenClaw macOS yordamchi ilovasi (menyu paneli + gateway broker)"
read_when:
  - Implementing macOS app features
  - macOS’da gateway hayotiy siklini yoki node bog‘lanishini o‘zgartirish
title: "macOS Ilovasi"
---

# OpenClaw macOS Yordamchisi (menyu paneli + gateway broker)

macOS ilovasi OpenClaw uchun **menyu‑panel yordamchisi** hisoblanadi. U ruxsatlarga egalik qiladi,
Gateway’ni lokal boshqaradi/unga ulanadi (launchd yoki qo‘lda) va macOS imkoniyatlarini agentga node sifatida taqdim etadi.

## U nima qiladi

- Menyu panelida mahalliy bildirishnomalar va holatni ko‘rsatadi.
- TCC so‘rovlariga egalik qiladi (Bildirishnomalar, Accessibility, Screen Recording, Microphone,
  Nutqni aniqlash, Avtomatlashtirish/AppleScript).
- Gateway’ni ishga tushiradi yoki unga ulanadi (lokal yoki masofaviy).
- Faqat macOS’ga xos vositalarni taqdim etadi (Canvas, Camera, Screen Recording, `system.run`).
- Mahalliy node host xizmatini **masofaviy** rejimda (launchd) ishga tushiradi va **lokal** rejimda to‘xtatadi.
- Ixtiyoriy ravishda UI avtomatlashtirish uchun **PeekabooBridge**’ni joylashtiradi.
- So‘rovga binoan global CLI (`openclaw`) ni npm/pnpm orqali o‘rnatadi (Gateway runtime uchun bun tavsiya etilmaydi).

## Lokal va masofaviy rejim

- **Local** (standart): ilova mavjud bo‘lsa, ishlayotgan mahalliy Gateway’ga ulanadi;
  aks holda `openclaw gateway install` orqali launchd xizmatini yoqadi.
- **Remote**: the app connects to a Gateway over SSH/Tailscale and never starts
  a local process.
  Ilova masofaviy Gateway ushbu Mac’ga yetib borishi uchun lokal **node host xizmati**ni ishga tushiradi.
  Ilova Gateway’ni child process sifatida ishga tushirmaydi.

## Launchd boshqaruvi

Ilova har bir foydalanuvchi uchun `bot.molt.gateway` yorlig‘iga ega LaunchAgent’ni boshqaradi
(yoki `bot.molt.<profile>`` ` `--profile`/`OPENCLAW_PROFILE` ishlatilganda; eski `com.openclaw.*` hali ham unload qilinadi).

```bash
launchctl kickstart -k gui/$UID/bot.molt.gateway
launchctl bootout gui/$UID/bot.molt.gateway
```

Nomlangan profil ishga tushirilganda yorliqni `bot.molt.<profile>` bilan almashtiring.Agar LaunchAgent o‘rnatilmagan bo‘lsa, uni ilovadan yoqing yoki
`openclaw gateway install` ni ishga tushiring.

Node imkoniyatlari (mac)

## macOS ilovasi o‘zini node sifatida taqdim etadi.

Umumiy buyruqlar: Canvas: `canvas.present`, `canvas.navigate`, `canvas.eval`, `canvas.snapshot`, `canvas.a2ui.*`

- Camera: `camera.snap`, `camera.clip`
- Camera: `camera.snap`, `camera.clip`
- Screen: `screen.record`
- System: `system.run`, `system.notify`

Node xizmati + ilova IPC:

Headless node host xizmati ishlayotganda (masofaviy rejim), u Gateway WS’iga node sifatida ulanadi.

- `system.run` macOS ilovasida (UI/TCC konteksti) lokal Unix soketi orqali bajariladi; so‘rovlar va chiqish ilova ichida qoladi.
- Diagramma (SCI):

Gateway -> Node Service (WS)
|  IPC (UDS + token + HMAC + TTL)
v
Mac App (UI + TCC + system.run)

```
Exec tasdiqlashlari (system.run)
```

## `system.run` macOS ilovasida **Exec tasdiqlashlari** orqali boshqariladi (Sozlamalar → Exec tasdiqlashlari).

Xavfsizlik + so‘rash + allowlist lokal ravishda Mac’da quyidagi joyda saqlanadi:
~/.openclaw/exec-approvals.json

```
Misol:
```

{
"version": 1,
"defaults": {
"security": "deny",
"ask": "on-miss"
},
"agents": {
"main": {
"security": "allowlist",
"ask": "on-miss",
"allowlist": [{ "pattern": "/opt/homebrew/bin/rg" }]
}
}
}

```json
Eslatmalar:
```

`allowlist` yozuvlari yechilgan binary yo‘llari uchun glob andozalaridir.

- `allowlist` yozuvlari aniqlangan binary yo‘llari uchun glob andozalaridir.
- So‘rov oynasida “Always Allow” tanlansa, o‘sha buyruq allowlist’ga qo‘shiladi.
- `system.run` muhit o‘zgaruvchilari ustidan yozishlar filtrlab olinadi (`PATH`, `DYLD_*`, `LD_*`, `NODE_OPTIONS`, `PYTHON*`, `PERL*`, `RUBYOPT` olib tashlanadi) va so‘ng ilovaning muhiti bilan birlashtiriladi.

## Deep links

The app registers the `openclaw://` URL scheme for local actions.

### `openclaw://agent`

Triggers a Gateway `agent` request.

```bash
open 'openclaw://agent?message=Hello%20from%20deep%20link'
```

Query parameters:

- `message` (required)
- `sessionKey` (optional)
- `thinking` (optional)
- `deliver` / `to` / `channel` (optional)
- `timeoutSeconds` (optional)
- `key` (optional unattended mode key)

Safety:

- Without `key`, the app prompts for confirmation.
- `key` bo‘lmasa, ilova tasdiqlash prompti uchun qisqa xabar cheklovini qo‘llaydi va `deliver` / `to` / `channel` ni e’tiborsiz qoldiradi.
- With a valid `key`, the run is unattended (intended for personal automations).

## Onboarding flow (typical)

1. Install and launch **OpenClaw.app**.
2. Complete the permissions checklist (TCC prompts).
3. Ensure **Local** mode is active and the Gateway is running.
4. Install the CLI if you want terminal access.

## Build & dev workflow (native)

- `cd apps/macos && swift build`
- `swift run OpenClaw` (or Xcode)
- Package app: `scripts/package-mac-app.sh`

## Debug gateway connectivity (macOS CLI)

Use the debug CLI to exercise the same Gateway WebSocket handshake and discovery
logic that the macOS app uses, without launching the app.

```bash
cd apps/macos
swift run openclaw-mac connect --json
swift run openclaw-mac discover --timeout 3000 --json
```

Connect options:

- `--url <ws://host:port>`: override config
- `--mode <local|remote>`: resolve from config (default: config or local)
- `--probe`: force a fresh health probe
- `--timeout <ms>`: request timeout (default: `15000`)
- `--json`: structured output for diffing

Discovery options:

- `--include-local`: include gateways that would be filtered as “local”
- `--timeout <ms>`: overall discovery window (default: `2000`)
- `--json`: structured output for diffing

Tip: compare against `openclaw gateway discover --json` to see whether the
macOS app’s discovery pipeline (NWBrowser + tailnet DNS‑SD fallback) differs from
the Node CLI’s `dns-sd` based discovery.

## Remote connection plumbing (SSH tunnels)

When the macOS app runs in **Remote** mode, it opens an SSH tunnel so local UI
components can talk to a remote Gateway as if it were on localhost.

### Control tunnel (Gateway WebSocket port)

- **Purpose:** health checks, status, Web Chat, config, and other control-plane calls.
- **Local port:** the Gateway port (default `18789`), always stable.
- **Remote port:** the same Gateway port on the remote host.
- **Behavior:** no random local port; the app reuses an existing healthy tunnel
  or restarts it if needed.
- **SSH shape:** `ssh -N -L <local>:127.0.0.1:<remote>` with BatchMode +
  ExitOnForwardFailure + keepalive options.
- **IP reporting:** the SSH tunnel uses loopback, so the gateway will see the node
  IP as `127.0.0.1`. Use **Direct (ws/wss)** transport if you want the real client
  IP to appear (see [macOS remote access](/platforms/mac/remote)).

1. O‘rnatish bosqichlari uchun [macOS remote access](/platforms/mac/remote) ga qarang. 2. Protokol tafsilotlari uchun [Gateway protocol](/gateway/protocol) ga qarang.

## 3. Tegishli hujjatlar

- [Gateway runbook](/gateway)
- [Gateway (macOS)](/platforms/mac/bundled-gateway)
- [macOS permissions](/platforms/mac/permissions)
- [Canvas](/platforms/mac/canvas)

