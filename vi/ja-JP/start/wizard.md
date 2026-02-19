---
read_when:
  - Khi chạy hoặc cấu hình trình hướng dẫn onboarding
  - Khi thiết lập máy mới
sidebarTitle: Wizard (CLI)
summary: "Trình hướng dẫn onboarding CLI: Thiết lập tương tác Gateway, workspace, kênh và Skills"
title: Trình hướng dẫn onboarding (CLI)
x-i18n:
  generated_at: "2026-02-08T17:15:18Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 9a650d46044a930aa4aaec30b35f1273ca3969bf676ab67bf4e1575b5c46db4c
  source_path: start/wizard.md
  workflow: 15
---

# Trình hướng dẫn onboarding (CLI)

Trình hướng dẫn onboarding CLI là lộ trình được khuyến nghị để thiết lập OpenClaw trên macOS, Linux và Windows (thông qua WSL2). Ngoài việc kết nối Gateway cục bộ hoặc từ xa, nó còn cấu hình cài đặt mặc định của workspace, kênh và Skills.

```bash
openclaw onboard
```

<Info>
Cách nhanh nhất để bắt đầu cuộc trò chuyện đầu tiên: Mở Control UI (không cần cấu hình kênh). Chạy `openclaw dashboard` để trò chuyện trên trình duyệt. Tài liệu: [Dashboard](/web/dashboard).
</Info>

## Khởi động nhanh vs Thiết lập chi tiết

Trình hướng dẫn bắt đầu bằng cách chọn **Khởi động nhanh** (cài đặt mặc định) hoặc **Thiết lập chi tiết** (toàn quyền kiểm soát).

<Tabs>
  <Tab title="クイックスタート（デフォルト設定）">
    - Gateway cục bộ trên loopback
    - Workspace hiện có hoặc workspace mặc định
    - Cổng Gateway `18789`
    - Token xác thực Gateway được tạo tự động (cũng được tạo trên loopback)
    - Tailscale public bị tắt
    - DM Telegram và WhatsApp mặc định dùng danh sách cho phép (có thể yêu cầu nhập số điện thoại)
  
</Tab>
  <Tab title="詳細設定（完全な制御）">
    - Hiển thị toàn bộ luồng nhắc cho chế độ, workspace, Gateway, kênh, daemon và Skills
  
</Tab>
</Tabs>

## Chi tiết onboarding CLI

<Columns>
  <Card title="CLIリファレンス" href="/start/wizard-cli-reference">
    Mô tả đầy đủ về luồng cục bộ và từ xa, ma trận xác thực và model, đầu ra cấu hình, RPC của trình hướng dẫn và hoạt động của signal-cli.
  
</Card>
  <Card title="自動化とスクリプト" href="/start/wizard-cli-automation">
    Công thức onboarding không tương tác và ví dụ `agents add` tự động.
  
</Card>
</Columns>

## Các lệnh tiếp theo thường dùng

```bash
openclaw configure
openclaw agents add <name>
```

<Note>
`--json` không có nghĩa là chế độ không tương tác. Trong script, hãy sử dụng `--non-interactive`.
</Note>

<Tip>
Khuyến nghị: Thiết lập Brave Search API key để agent có thể sử dụng `web_search` (`web_fetch` hoạt động mà không cần key). Cách đơn giản nhất: chạy `openclaw configure --section web` để lưu `tools.web.search.apiKey`. Tài liệu: [Webツール](/tools/web).
</Tip>

## Tài liệu liên quan

- Tham khảo lệnh CLI: [`openclaw onboard`](/cli/onboard)
- Onboarding ứng dụng macOS: [Onboarding](/start/onboarding)
- Hướng dẫn khởi động agent lần đầu: [Agent Bootstrap](/start/bootstrapping)
