---
title: "Plugin Zalo Personal"
---

# Zalo Cá nhân (plugin)

Hỗ trợ Zalo Personal cho OpenClaw thông qua một plugin, sử dụng `zca-cli` để tự động hóa một tài khoản người dùng Zalo thông thường.

> **Cảnh báo:** Tự động hóa không chính thức có thể dẫn đến việc tài khoản bị đình chỉ/cấm. Hãy tự chịu rủi ro khi sử dụng.

## Đặt tên

Channel id là `zalouser` để làm rõ rằng tính năng này tự động hóa một **tài khoản người dùng Zalo cá nhân** (không chính thức). Chúng tôi giữ `zalo` để dành cho khả năng tích hợp API Zalo chính thức trong tương lai.

## Nơi chạy

Plugin này chạy **bên trong tiến trình Gateway**.

Nếu bạn dùng Gateway từ xa, hãy cài đặt/cấu hình nó trên **máy đang chạy Gateway**, sau đó khởi động lại Gateway.

## Cài đặt

### Tùy chọn A: cài từ npm

```bash
openclaw plugins install @openclaw/zalouser
```

Sau đó khởi động lại Gateway.

### Tùy chọn B: cài từ thư mục cục bộ (dev)

```bash
openclaw plugins install ./extensions/zalouser
cd ./extensions/zalouser && pnpm install
```

Sau đó khởi động lại Gateway.

## Điều kiện tiên quyết: zca-cli

Máy Gateway phải có `zca` trên `PATH`:

```bash
zca --version
```

## Cấu hình

Cấu hình kênh nằm dưới `channels.zalouser` (không phải `plugins.entries.*`):

```json5
{
  channels: {
    zalouser: {
      enabled: true,
      dmPolicy: "pairing",
    },
  },
}
```

## CLI

```bash
openclaw channels login --channel zalouser
openclaw channels logout --channel zalouser
openclaw channels status --probe
openclaw message send --channel zalouser --target <threadId> --message "Hello from OpenClaw"
openclaw directory peers list --channel zalouser --query "name"
```

## Công cụ của tác tử

Tên công cụ: `zalouser`

Hành động: `send`, `image`, `link`, `friends`, `groups`, `me`, `status`

