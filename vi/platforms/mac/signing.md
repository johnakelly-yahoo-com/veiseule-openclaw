---
title: "Ký macOS"
---

# ký mac (bản build debug)

Ứng dụng này thường được build từ [`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh), script hiện nay:

- đặt một bundle identifier debug ổn định: `ai.openclaw.mac.debug`
- ghi Info.plist với bundle id đó (có thể ghi đè qua `BUNDLE_ID=...`)
- calls [`scripts/codesign-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/codesign-mac-app.sh) to sign the main binary and app bundle so macOS treats each rebuild as the same signed bundle and keeps TCC permissions (notifications, accessibility, screen recording, mic, speech). mặc định dùng `CODESIGN_TIMESTAMP=auto`; nó bật trusted timestamps cho chữ ký Developer ID.
- uses `CODESIGN_TIMESTAMP=auto` by default; it enables trusted timestamps for Developer ID signatures. Đặt `CODESIGN_TIMESTAMP=off` để bỏ qua việc đóng dấu thời gian (các bản build debug ngoại tuyến).
- chèn metadata build vào Info.plist: `OpenClawBuildTimestamp` (UTC) và `OpenClawGitCommit` (hash ngắn) để bảng About có thể hiển thị build, git, và kênh debug/release.
- **Đóng gói yêu cầu Node 22+**: script chạy các bản build TS và build Control UI.
- Thêm `export SIGN_IDENTITY="Apple Development: Your Name (TEAMID)"` (hoặc chứng chỉ Developer ID Application của bạn) vào shell rc để luôn ký bằng chứng chỉ của bạn. Ký ad-hoc yêu cầu opt-in rõ ràng qua `ALLOW_ADHOC_SIGNING=1` hoặc `SIGN_IDENTITY="-"` (không khuyến nghị cho việc kiểm thử quyền). Ad-hoc signing requires explicit opt-in via `ALLOW_ADHOC_SIGNING=1` or `SIGN_IDENTITY="-"` (not recommended for permission testing).
- chạy kiểm tra Team ID sau khi ký và sẽ thất bại nếu bất kỳ Mach-O nào bên trong app bundle được ký bởi Team ID khác. Đặt `SKIP_TEAM_ID_CHECK=1` để bỏ qua.

## Cách dùng

```bash
# from repo root
scripts/package-mac-app.sh               # auto-selects identity; errors if none found
SIGN_IDENTITY="Developer ID Application: Your Name" scripts/package-mac-app.sh   # real cert
ALLOW_ADHOC_SIGNING=1 scripts/package-mac-app.sh    # ad-hoc (permissions will not stick)
SIGN_IDENTITY="-" scripts/package-mac-app.sh        # explicit ad-hoc (same caveat)
DISABLE_LIBRARY_VALIDATION=1 scripts/package-mac-app.sh   # dev-only Sparkle Team ID mismatch workaround
```

### Lưu ý về ký ad-hoc

Điều này là cần thiết để tránh crash khi ứng dụng cố tải các framework nhúng (như Sparkle) không dùng chung Team ID. This is necessary to prevent crashes when the app attempts to load embedded frameworks (like Sparkle) that do not share the same Team ID. Ad-hoc signatures also break TCC permission persistence; see [macOS permissions](/platforms/mac/permissions) for recovery steps.

## Metadata build cho About

`package-mac-app.sh` đóng dấu bundle với:

- `OpenClawBuildTimestamp`: ISO8601 UTC tại thời điểm đóng gói
- `OpenClawGitCommit`: hash git ngắn (hoặc `unknown` nếu không có)

Tab About đọc các khóa này để hiển thị phiên bản, ngày build, git commit và liệu đây có phải là bản debug hay không (thông qua `#if DEBUG`). Chạy packager để làm mới các giá trị này sau khi thay đổi mã nguồn.

## Lý do

Quyền TCC được gắn với bundle identifier _và_ chữ ký mã. Các bản debug chưa ký với UUID thay đổi liên tục khiến macOS quên các quyền đã cấp sau mỗi lần build lại. Việc ký các tệp nhị phân (mặc định là ad‑hoc) và giữ cố định bundle id/đường dẫn (`dist/OpenClaw.app`) sẽ сохраня các quyền giữa các lần build, tương tự cách tiếp cận của VibeTunnel.

