---
title: "40. macOS Dev Setup"
---

# 41. macOS Developer Setup

42. Ushbu qo‘llanma OpenClaw macOS ilovasini manba koddan yig‘ish va ishga tushirish uchun zarur qadamlarni qamrab oladi.

## 43. Oldindan talablar

44. Ilovani yig‘ishdan oldin quyidagilar o‘rnatilganligiga ishonch hosil qiling:

1. 45. **Xcode 26.2+**: Swift’da ishlab chiqish uchun zarur.
2. 46. **Node.js 22+ & pnpm**: Gateway, CLI va paketlash skriptlari uchun zarur.

## 47) 1) 48) Bog‘liqliklarni o‘rnatish

49. Loyiha bo‘yicha umumiy bog‘liqliklarni o‘rnating:

```bash
50. pnpm install
```

## 2. Ilovani yig‘ish va paketlash

macOS ilovasini yig‘ish va uni `dist/OpenClaw.app` ichiga paketlash uchun quyidagini ishga tushiring:

```bash
./scripts/package-mac-app.sh
```

Agar sizda Apple Developer ID sertifikati bo‘lmasa, skript avtomatik ravishda **ad-hoc signing** (`-`) dan foydalanadi.

Dev ishga tushirish rejimlari, imzolash flaglari va Team ID bilan bog‘liq muammolarni hal qilish uchun macOS app README ga qarang:
[https://github.com/openclaw/openclaw/blob/main/apps/macos/README.md](https://github.com/openclaw/openclaw/blob/main/apps/macos/README.md)

> **Eslatma**: Ad-hoc imzolangan ilovalar xavfsizlik ogohlantirishlarini chiqarishi mumkin. Agar ilova darhol "Abort trap 6" xatosi bilan yopilsa, [Troubleshooting](#troubleshooting) bo‘limiga qarang.

## 3. CLI ni o‘rnatish

macOS ilovasi fon vazifalarini boshqarish uchun global `openclaw` CLI o‘rnatilgan bo‘lishini talab qiladi.

**Uni o‘rnatish uchun (tavsiya etiladi):**

1. Open the OpenClaw app.
2. **General** sozlamalar yorlig‘iga o‘ting.
3. Click **"Install CLI"**.

Alternatively, install it manually:

```bash
npm install -g openclaw@<version>
```

## Troubleshooting

### Build Fails: Toolchain or SDK Mismatch

The macOS app build expects the latest macOS SDK and Swift 6.2 toolchain.

**System dependencies (required):**

- **Latest macOS version available in Software Update** (required by Xcode 26.2 SDKs)
- **Xcode 26.2** (Swift 6.2 toolchain)

**Checks:**

```bash
xcodebuild -version
xcrun swift --version
```

If versions don’t match, update macOS/Xcode and re-run the build.

### App Crashes on Permission Grant

If the app crashes when you try to allow **Speech Recognition** or **Microphone** access, it may be due to a corrupted TCC cache or signature mismatch.

**Fix:**

1. Reset the TCC permissions:

   ```bash
   tccutil reset All bot.molt.mac.debug
   ```

2. If that fails, change the `BUNDLE_ID` temporarily in [`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh) to force a "clean slate" from macOS.

### Gateway "Starting..." indefinitely

If the gateway status stays on "Starting...", check if a zombie process is holding the port:

```bash
openclaw gateway status
openclaw gateway stop

# If you’re not using a LaunchAgent (dev mode / manual runs), find the listener:
lsof -nP -iTCP:18789 -sTCP:LISTEN
```

If a manual run is holding the port, stop that process (Ctrl+C). As a last resort, kill the PID you found above.
