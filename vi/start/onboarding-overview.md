---
summary: "Tổng quan về các tùy chọn và luồng onboarding của OpenClaw"
read_when:
  - Chọn lộ trình onboarding
  - Thiết lập môi trường mới
title: "Tổng quan Onboarding"
sidebarTitle: "Tổng quan Onboarding"
---

# Tổng quan Onboarding

OpenClaw hỗ trợ nhiều lộ trình onboarding tùy thuộc vào nơi Gateway chạy
và cách bạn muốn cấu hình provider.

## Chọn lộ trình onboarding của bạn

- **CLI wizard** cho macOS, Linux và Windows (thông qua WSL2).
- **Ứng dụng macOS** cho lần chạy đầu tiên có hướng dẫn trên máy Mac Apple silicon hoặc Intel.

## Trình hướng dẫn onboarding CLI

Chạy trình hướng dẫn trong terminal:

```bash
openclaw onboard
```

Sử dụng CLI wizard khi bạn muốn toàn quyền kiểm soát Gateway, workspace,
channels và skills. Tài liệu:

- [Trình hướng dẫn khởi tạo (CLI)](/start/wizard)
- [Lệnh `openclaw onboard`](/cli/onboard)

## Khởi tạo ứng dụng macOS

Sử dụng ứng dụng OpenClaw khi bạn muốn thiết lập được hướng dẫn đầy đủ trên macOS. Tài liệu:

- [Khởi tạo (Ứng dụng macOS)](/start/onboarding)

## Nhà cung cấp tùy chỉnh

Nếu bạn cần một endpoint không có trong danh sách, bao gồm các nhà cung cấp được host
cung cấp API OpenAI hoặc Anthropic tiêu chuẩn, hãy chọn **Custom Provider** trong
trình hướng dẫn CLI. Bạn sẽ được yêu cầu:

- Chọn tương thích OpenAI, tương thích Anthropic hoặc **Unknown** (tự động phát hiện).
- Nhập base URL và API key (nếu nhà cung cấp yêu cầu).
- Cung cấp model ID và bí danh (alias) tùy chọn.
- Chọn một Endpoint ID để nhiều endpoint tùy chỉnh có thể cùng tồn tại.

Để biết các bước chi tiết, hãy làm theo tài liệu khởi tạo CLI ở trên.
