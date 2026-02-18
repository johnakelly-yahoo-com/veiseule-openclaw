---
summary: "Cài đặt OpenClaw theo cách khai báo với Nix"
read_when:
  - Bạn muốn cài đặt có thể tái tạo và quay lui
  - Bạn đã sử dụng Nix/NixOS/Home Manager
  - Bạn muốn mọi thứ được ghim phiên bản và quản lý theo cách khai báo
title: "Nix"
---

# Cài đặt Nix

Cách được khuyến nghị để chạy OpenClaw với Nix là thông qua **[nix-openclaw](https://github.com/openclaw/nix-openclaw)** — một module Home Manager đầy đủ pin kèm sẵn.

## Khởi động nhanh

Dán đoạn này cho tác tử AI của bạn (Claude, Cursor, v.v.):

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

> **📦 Hướng dẫn đầy đủ: [github.com/openclaw/nix-openclaw](https://github.com/openclaw/nix-openclaw)**
>
> Repo nix-openclaw là nguồn chân lý cho cài đặt Nix. This page is just a quick overview.

## Những gì bạn nhận được

- Gateway + ứng dụng macOS + công cụ (whisper, spotify, cameras) — tất cả đều được ghim phiên bản
- Dịch vụ Launchd tồn tại qua các lần khởi động lại
- Hệ thống plugin với cấu hình khai báo
- Quay lui tức thì: `home-manager switch --rollback`

---

## Hành vi runtime ở chế độ Nix

Khi `OPENCLAW_NIX_MODE=1` được thiết lập (tự động với nix-openclaw):

OpenClaw hỗ trợ **chế độ Nix** giúp cấu hình có tính xác định và vô hiệu hóa các quy trình tự động cài đặt.
Enable it by exporting:

```bash
OPENCLAW_NIX_MODE=1
```

Trên macOS, ứng dụng GUI không tự động kế thừa các biến môi trường của shell. Bạn có thể
also enable Nix mode via defaults:

```bash
defaults write bot.molt.mac openclaw.nixMode -bool true
```

### Đường dẫn cấu hình + trạng thái

OpenClaw đọc cấu hình JSON5 từ `OPENCLAW_CONFIG_PATH` và lưu dữ liệu có thể thay đổi trong `OPENCLAW_STATE_DIR`.
When needed, you can also set `OPENCLAW_HOME` to control the base home directory used for internal path resolution.

- `OPENCLAW_HOME` (thứ tự ưu tiên mặc định: `HOME` / `USERPROFILE` / `os.homedir()`)
- `OPENCLAW_STATE_DIR` (mặc định: `~/.openclaw`)
- `OPENCLAW_CONFIG_PATH` (mặc định: `$OPENCLAW_STATE_DIR/openclaw.json`)

Khi chạy dưới Nix, hãy thiết lập rõ ràng các giá trị này tới các vị trí do Nix quản lý để trạng thái runtime và cấu hình
không nằm trong kho bất biến.

### Hành vi runtime trong chế độ Nix

- Các luồng tự cài đặt và tự thay đổi bị vô hiệu hóa
- Phụ thuộc bị thiếu sẽ hiển thị thông báo khắc phục dành riêng cho Nix
- UI hiển thị banner chế độ Nix chỉ đọc khi có

## Ghi chú đóng gói (macOS)

Quy trình đóng gói macOS yêu cầu một template Info.plist ổn định tại:

```
apps/macos/Sources/OpenClaw/Resources/Info.plist
```

[`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh) copies this template into the app bundle and patches dynamic fields
(bundle ID, version/build, Git SHA, Sparkle keys). Điều này giữ cho plist có tính xác định cho việc đóng gói SwiftPM
và các bản build Nix (không phụ thuộc vào đầy đủ toolchain Xcode).

## Liên quan

- [nix-openclaw](https://github.com/openclaw/nix-openclaw) — hướng dẫn thiết lập đầy đủ
- [Wizard](/start/wizard) — thiết lập CLI không dùng Nix
- [Docker](/install/docker) — thiết lập dạng container
