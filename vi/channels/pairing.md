---
title: "Ghép cặp"
---

# Ghép cặp

“Pairing” là bước **chủ sở hữu phê duyệt** rõ ràng của OpenClaw.
It is used in two places:

1. **Ghép cặp DM** (ai được phép nói chuyện với bot)
2. **Ghép cặp node** (những thiết bị/node nào được phép tham gia mạng gateway)

Ngữ cảnh bảo mật: [Security](/gateway/security)

## 1. Ghép cặp DM (truy cập chat đến)

Khi một kênh được cấu hình với chính sách DM `pairing`, người gửi chưa xác định sẽ nhận một mã ngắn và tin nhắn của họ **không được xử lý** cho đến khi bạn phê duyệt.

Các chính sách DM mặc định được ghi trong: [Security](/gateway/security)

Mã ghép cặp:

- 8 ký tự, chữ hoa, không có ký tự dễ gây nhầm lẫn (`0O1I`).
- **Hết hạn sau 1 giờ**. Bot chỉ gửi thông báo ghép nối khi một yêu cầu mới được tạo (khoảng một lần mỗi giờ cho mỗi người gửi).
- Các yêu cầu ghép cặp DM đang chờ được giới hạn **3 yêu cầu cho mỗi kênh** theo mặc định; các yêu cầu bổ sung sẽ bị bỏ qua cho đến khi một yêu cầu hết hạn hoặc được phê duyệt.

### Phê duyệt một người gửi

```bash
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

Các kênh được hỗ trợ: `telegram`, `whatsapp`, `signal`, `imessage`, `discord`, `slack`.

### Trạng thái được lưu ở đâu

Lưu dưới `~/.openclaw/credentials/`:

- Yêu cầu đang chờ: `<channel>-pairing.json`
- Kho danh sách cho phép đã phê duyệt: `<channel>-allowFrom.json`

Hãy coi những mục này là nhạy cảm (chúng kiểm soát quyền truy cập vào trợ lý của bạn).

## 2. Ghép cặp thiết bị node (iOS/Android/macOS/node headless)

Các node kết nối đến Gateway dưới dạng **thiết bị** với `role: node`. Gateway
creates a device pairing request that must be approved.

### Ghép nối qua Telegram (khuyến nghị cho iOS)

Nếu bạn sử dụng plugin `device-pair`, bạn có thể thực hiện ghép nối thiết bị lần đầu hoàn toàn từ Telegram:

1. Trong Telegram, nhắn cho bot của bạn: `/pair`
2. Bot sẽ trả lời với hai tin nhắn: một tin hướng dẫn và một tin riêng chứa **mã thiết lập** (dễ sao chép/dán trong Telegram).
3. Trên điện thoại của bạn, mở ứng dụng OpenClaw iOS → Settings → Gateway.
4. Dán mã thiết lập và kết nối.
5. Quay lại Telegram: `/pair approve`

The setup code is a base64-encoded JSON payload that contains:

- `url`: the Gateway WebSocket URL (`ws://...` or `wss://...`)
- `token`: a short-lived pairing token

Treat the setup code like a password while it is valid.

### Phê duyệt một thiết bị node

```bash
openclaw devices list
openclaw devices approve <requestId>
openclaw devices reject <requestId>
```

### Lưu trữ trạng thái ghép cặp node

Lưu dưới `~/.openclaw/devices/`:

- `pending.json` (tồn tại ngắn; các yêu cầu đang chờ sẽ hết hạn)
- `paired.json` (thiết bị đã ghép cặp + token)

### Ghi chú

- The legacy `node.pair.*` API (CLI: `openclaw nodes pending/approve`) is a
  separate gateway-owned pairing store. WS nodes still require device pairing.

## Tài liệu liên quan

- Mô hình bảo mật + prompt injection: [Security](/gateway/security)
- Cập nhật an toàn (chạy doctor): [Updating](/install/updating)
- Cấu hình kênh:
  - Telegram: [Telegram](/channels/telegram)
  - WhatsApp: [WhatsApp](/channels/whatsapp)
  - Signal: [Signal](/channels/signal)
  - BlueBubbles (iMessage): [BlueBubbles](/channels/bluebubbles)
  - iMessage (legacy): [iMessage](/channels/imessage)
  - Discord: [Discord](/channels/discord)
  - Slack: [Slack](/channels/slack)


