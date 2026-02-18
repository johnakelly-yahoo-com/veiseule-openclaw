---
title: "Brauzer muammolarini bartaraf etish"
---

# Brauzer muammolarini bartaraf etish (Linux)

## Muammo: "Failed to start Chrome CDP on port 18800"

7. OpenClaw brauzer boshqaruv serveri Chrome/Brave/Edge/Chromium ni ishga tushira olmaydi va quyidagi xatoni beradi:

```
{"error":"Error: Failed to start Chrome CDP on port 18800 for profile \"openclaw\"."}
```

### Asosiy sabab

Ubuntu’da (va ko‘plab Linux distributsiyalarida) Chromium’ning standart o‘rnatilishi **snap paket** hisoblanadi. Snap’ning AppArmor cheklovlari OpenClaw’ning brauzer jarayonini ishga tushirish va kuzatish usuliga xalaqit beradi.

8. `apt install chromium` buyrug‘i snap’ga yo‘naltiruvchi stub paketni o‘rnatadi:

```
Eslatma, 'chromium' o‘rniga 'chromium-browser' tanlash
chromium-browser allaqachon eng yangi versiya (2:1snap1-0ubuntu2).
```

Bu haqiqiy brauzer EMAS — bu shunchaki o‘ram (wrapper).

### Yechim 1: Google Chrome’ni o‘rnatish (Tavsiya etiladi)

Snap tomonidan sandbox qilinmagan rasmiy Google Chrome `.deb` paketini o‘rnating:

```bash
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo dpkg -i google-chrome-stable_current_amd64.deb
sudo apt --fix-broken install -y  # agar bog‘liqlik xatolari bo‘lsa
```

So‘ng OpenClaw konfiguratsiyasini yangilang (`~/.openclaw/openclaw.json`):

```json
{
  "browser": {
    "enabled": true,
    "executablePath": "/usr/bin/google-chrome-stable",
    "headless": true,
    "noSandbox": true
  }
}
```

### Yechim 2: Snap Chromium’ni Attach-Only rejimida ishlatish

Agar snap Chromium’dan foydalanishingiz shart bo‘lsa, OpenClaw’ni qo‘lda ishga tushirilgan brauzerga ulanadigan qilib sozlang:

1. Konfiguratsiyani yangilang:

```json
{
  "browser": {
    "enabled": true,
    "attachOnly": true,
    "headless": true,
    "noSandbox": true
  }
}
```

2. Chromium'ni qo‘lda ishga tushiring:

```bash
chromium-browser --headless --no-sandbox --disable-gpu \
  --remote-debugging-port=18800 \
  --user-data-dir=$HOME/.openclaw/browser/openclaw/user-data \
  about:blank &
```

3. Ixtiyoriy ravishda Chrome'ni avtomatik ishga tushirish uchun systemd foydalanuvchi xizmatini yarating:

```ini
# ~/.config/systemd/user/openclaw-browser.service
[Unit]
Description=OpenClaw Browser (Chrome CDP)
After=network.target

[Service]
ExecStart=/snap/bin/chromium --headless --no-sandbox --disable-gpu --remote-debugging-port=18800 --user-data-dir=%h/.openclaw/browser/openclaw/user-data about:blank
Restart=on-failure
RestartSec=5

[Install]
WantedBy=default.target
```

Quyidagicha yoqing: `systemctl --user enable --now openclaw-browser.service`

### Brauzer ishlashini tekshirish

9. Holatni tekshirish:

```bash
curl -s http://127.0.0.1:18791/ | jq '{running, pid, chosenBrowser}'
```

Ko‘rishni sinab ko‘ring:

```bash
curl -s -X POST http://127.0.0.1:18791/start
curl -s http://127.0.0.1:18791/tabs
```

### Konfiguratsiya ma’lumotnomasi

| Parametr                   | Tavsif                                                                                                                  | Standart qiymat                                                                        |
| ------------------------ | ---------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------ |
| `browser.enabled`        | Brauzer boshqaruvini yoqish                                                                                                       | `true`                                                                         |
| `browser.executablePath` | 10. Chromium-ga asoslangan brauzer binar fayliga yo‘l (Chrome/Brave/Edge/Chromium) | auto-detected (prefers default browser when Chromium-based) |
| `browser.headless`       | Run without GUI                                                                                                              | `false`                                                                        |
| `browser.noSandbox`      | Add `--no-sandbox` flag (needed for some Linux setups)                                                    | `false`                                                                        |
| `browser.attachOnly`     | Don't launch browser, only attach to existing                                                                                | 11. `false`                                             |
| `browser.cdpPort`        | Chrome DevTools Protocol port                                                                                                | `18800`                                                                        |

### Problem: "Chrome extension relay is running, but no tab is connected"

You’re using the `chrome` profile (extension relay). It expects the OpenClaw
browser extension to be attached to a live tab.

Fix options:

1. **Use the managed browser:** `openclaw browser start --browser-profile openclaw`
   (or set `browser.defaultProfile: "openclaw"`).
2. **Use the extension relay:** install the extension, open a tab, and click the
   OpenClaw extension icon to attach it.

Notes:

- The `chrome` profile uses your **system default Chromium browser** when possible.
- Local `openclaw` profiles auto-assign `cdpPort`/`cdpUrl`; only set those for remote CDP.


