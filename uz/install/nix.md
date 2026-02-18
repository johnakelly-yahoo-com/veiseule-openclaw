---
summary: "45. OpenClaw’ni Nix yordamida deklarativ tarzda o‘rnating"
read_when:
  - 46. Siz qayta ishlab chiqariladigan, orqaga qaytariladigan o‘rnatishlarni xohlaysiz
  - 47. Siz allaqachon Nix/NixOS/Home Manager’dan foydalanmoqdasiz
  - 48. Siz hammasi pinlangan va deklarativ boshqarilishini xohlaysiz
title: "49. Nix"
---

# 50. Nix o‘rnatilishi

OpenClaw’ni Nix bilan ishga tushirishning tavsiya etilgan usuli — **[nix-openclaw](https://github.com/openclaw/nix-openclaw)** orqali, ya’ni barcha kerakli narsalarni o‘z ichiga olgan Home Manager moduli.

## Tezkor boshlash

Buni AI agentingizga (Claude, Cursor va boshqalar) joylashtiring:

```text
I want to set up nix-openclaw on my Mac.
Repository: github:openclaw/nix-openclaw

What I need you to do:
1. Check if Determinate Nix is installed (if not, install it)
2. Create a local flake at ~/code/openclaw-local using templates/agent-first/flake.nix
3. Help me create a Telegram bot (@BotFather) and get my chat ID (@userinfobot)
4. Set up secrets (bot token, Anthropic key) - plain files at ~/.secrets/ is fine
5. Fill in the template placeholders and run home-manager switch
6. Verify: launchd running, bot responds to messages

Reference the nix-openclaw README for module options.
```

> **📦 Full guide: [github.com/openclaw/nix-openclaw](https://github.com/openclaw/nix-openclaw)**
>
> nix-openclaw repozitoriyasi Nix o‘rnatish uchun yagona ishonchli manba hisoblanadi. Ushbu sahifa esa shunchaki qisqacha ko‘rinishdir.

## Nimalarga ega bo‘lasiz

- Gateway + macOS ilovasi + vositalar (whisper, spotify, cameras) — barchasi qat’iy versiyalarga biriktirilgan
- Qayta yuklashlardan keyin ham ishlashda davom etadigan Launchd xizmati
- Deklarativ konfiguratsiyaga ega plagin tizimi
- Bir zumda ortga qaytarish: `home-manager switch --rollback`

---

## Nix Mode Runtime Behavior

When `OPENCLAW_NIX_MODE=1` is set (automatic with nix-openclaw):

OpenClaw supports a **Nix mode** that makes configuration deterministic and disables auto-install flows.
Enable it by exporting:

```bash
OPENCLAW_NIX_MODE=1
```

On macOS, the GUI app does not automatically inherit shell env vars. You can
also enable Nix mode via defaults:

```bash
defaults write bot.molt.mac openclaw.nixMode -bool true
```

### Config + state paths

OpenClaw JSON5 konfiguratsiyasini `OPENCLAW_CONFIG_PATH` dan o‘qiydi va o‘zgaruvchan ma’lumotlarni `OPENCLAW_STATE_DIR` da saqlaydi.
Zarur bo‘lganda, ichki yo‘l aniqlashlari uchun ishlatiladigan asosiy uy katalogini boshqarish uchun `OPENCLAW_HOME` ni ham o‘rnatishingiz mumkin.

- `OPENCLAW_HOME` (standart ustuvorlik: `HOME` / `USERPROFILE` / `os.homedir()`)
- `OPENCLAW_STATE_DIR` (default: `~/.openclaw`)
- `OPENCLAW_CONFIG_PATH` (default: `$OPENCLAW_STATE_DIR/openclaw.json`)

When running under Nix, set these explicitly to Nix-managed locations so runtime state and config
stay out of the immutable store.

### Runtime behavior in Nix mode

- Auto-install and self-mutation flows are disabled
- Missing dependencies surface Nix-specific remediation messages
- UI surfaces a read-only Nix mode banner when present

## Packaging note (macOS)

The macOS packaging flow expects a stable Info.plist template at:

```
apps/macos/Sources/OpenClaw/Resources/Info.plist
```

[`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh) copies this template into the app bundle and patches dynamic fields
(bundle ID, version/build, Git SHA, Sparkle keys). This keeps the plist deterministic for SwiftPM
packaging and Nix builds (which do not rely on a full Xcode toolchain).

## Related

- [nix-openclaw](https://github.com/openclaw/nix-openclaw) — full setup guide
- [Wizard](/start/wizard) — non-Nix CLI setup
- [Docker](/install/docker) — containerized setup
