---
title: "Mattermost"
---

# Mattermost (plugin)

Trạng thái: được hỗ trợ thông qua plugin (bot token + sự kiện WebSocket). Hỗ trợ kênh, nhóm và tin nhắn trực tiếp (DM).
Mattermost is a self-hostable team messaging platform; see the official site at
[mattermost.com](https://mattermost.com) for product details and downloads.

## Yêu cầu plugin

Mattermost được phân phối dưới dạng plugin và không được gộp sẵn trong bản cài đặt lõi.

Cài đặt qua CLI (npm registry):

```bash
openclaw plugins install @openclaw/mattermost
```

Cài đặt từ mã nguồn cục bộ (khi chạy từ repo git):

```bash
openclaw plugins install ./extensions/mattermost
```

Nếu bạn chọn Mattermost trong quá trình cấu hình/hướng dẫn ban đầu và phát hiện có bản checkout git,
OpenClaw sẽ tự động đề xuất đường dẫn cài đặt cục bộ.

Chi tiết: [Plugins](/tools/plugin)

## Khởi động nhanh

1. Cài đặt plugin Mattermost.
2. Tạo tài khoản bot Mattermost và sao chép **bot token**.
3. Sao chép **base URL** của Mattermost (ví dụ: `https://chat.example.com`).
4. Cấu hình OpenClaw và khởi động gateway.

Cấu hình tối thiểu:

```json5
{
  channels: {
    mattermost: {
      enabled: true,
      botToken: "mm-token",
      baseUrl: "https://chat.example.com",
      dmPolicy: "pairing",
    },
  },
}
```

## Biến môi trường (tài khoản mặc định)

Thiết lập các biến này trên máy chủ gateway nếu bạn предпоч thích dùng biến môi trường:

- `MATTERMOST_BOT_TOKEN=...`
- `MATTERMOST_URL=https://chat.example.com`

Biến môi trường (env vars) chỉ áp dụng cho tài khoản **mặc định** (`default`). Các tài khoản khác phải sử dụng giá trị trong cấu hình.

## Chế độ chat

Mattermost tự động phản hồi tin nhắn trực tiếp (DM). Hành vi trong kênh được kiểm soát bởi `chatmode`:

- `oncall` (mặc định): chỉ phản hồi khi được @mention trong kênh.
- `onmessage`: phản hồi mọi tin nhắn trong kênh.
- `onchar`: phản hồi khi tin nhắn bắt đầu bằng tiền tố kích hoạt.

Ví dụ cấu hình:

```json5
{
  channels: {
    mattermost: {
      chatmode: "onchar",
      oncharPrefixes: [">", "!"],
    },
  },
}
```

Ghi chú:

- `onchar` vẫn phản hồi các @mention rõ ràng.
- `channels.mattermost.requireMention` vẫn được tôn trọng cho cấu hình cũ nhưng `chatmode` được ưu tiên.

## Kiểm soát truy cập (DM)

- Mặc định: `channels.mattermost.dmPolicy = "pairing"` (người gửi chưa biết sẽ nhận mã ghép cặp).
- Phê duyệt qua:
  - `openclaw pairing list mattermost`
  - `openclaw pairing approve mattermost <CODE>`
- DM công khai: `channels.mattermost.dmPolicy="open"` cộng với `channels.mattermost.allowFrom=["*"]`.

## Kênh (nhóm)

- Mặc định: `channels.mattermost.groupPolicy = "allowlist"` (giới hạn theo mention).
- Cho phép người gửi theo danh sách cho phép bằng `channels.mattermost.groupAllowFrom` (ID người dùng hoặc `@username`).
- Kênh mở: `channels.mattermost.groupPolicy="open"` (giới hạn theo mention).

## Đích để gửi ra ngoài

Sử dụng các định dạng đích này với `openclaw message send` hoặc cron/webhooks:

- `channel:<id>` cho một kênh
- `user:<id>` cho một DM
- `@username` cho một DM (được phân giải qua API Mattermost)

ID trần được coi là kênh.

## Đa tài khoản

Mattermost hỗ trợ nhiều tài khoản dưới `channels.mattermost.accounts`:

```json5
{
  channels: {
    mattermost: {
      accounts: {
        default: { name: "Primary", botToken: "mm-token", baseUrl: "https://chat.example.com" },
        alerts: { name: "Alerts", botToken: "mm-token-2", baseUrl: "https://alerts.example.com" },
      },
    },
  },
}
```

## Xử lý sự cố

- Không có phản hồi trong kênh: đảm bảo bot đã ở trong kênh và được mention (oncall), dùng tiền tố kích hoạt (onchar), hoặc đặt `chatmode: "onmessage"`.
- Lỗi xác thực: kiểm tra bot token, base URL và việc tài khoản có được bật hay không.
- Vấn đề đa tài khoản: biến môi trường chỉ áp dụng cho tài khoản `default`.

