---
title: "Telegram"
---

# Telegram (Bot API)

Status: production-ready cho bot DM + nhóm qua grammY. Long polling là chế độ mặc định; webhook là tùy chọn.

<CardGroup cols={3}>
  <Card title="Ghép cặp" icon="link" href="/channels/pairing">
    Chính sách DM mặc định cho Telegram là ghép cặp.
  </Card>
  <Card title="Khắc phục sự cố kênh" icon="wrench" href="/channels/troubleshooting">
    Chẩn đoán đa kênh và playbook sửa lỗi.
  </Card>
  <Card title="Cấu hình Gateway" icon="settings" href="/gateway/configuration">
    Mẫu cấu hình kênh đầy đủ và ví dụ.
  </Card>
</CardGroup>

## Quick setup

<Steps>
  <Step title="Tạo bot token trong BotFather">
    Mở Telegram và chat với **@BotFather** (xác nhận handle chính xác là `@BotFather`).

    Chạy `/newbot`, làm theo hướng dẫn và lưu lại token.

  </Step>

  <Step title="Cấu hình token và chính sách DM">

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123:abc",
      dmPolicy: "pairing",
      groups: { "*": { requireMention: true } },
    },
  },
}
```

    Env fallback: `TELEGRAM_BOT_TOKEN=...` (chỉ áp dụng cho tài khoản mặc định).

  </Step>

  <Step title="Khởi động gateway và phê duyệt DM đầu tiên">

```bash
openclaw gateway
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

    Mã ghép cặp hết hạn sau 1 giờ.

  </Step>

  <Step title="Thêm bot vào nhóm">
    Thêm bot vào nhóm của bạn, sau đó thiết lập `channels.telegram.groups` và `groupPolicy` phù hợp với mô hình truy cập của bạn.
  </Step>
</Steps>

<Note>
Thứ tự phân giải token có phân biệt theo tài khoản. Trong thực tế, giá trị trong config được ưu tiên hơn env fallback, và `TELEGRAM_BOT_TOKEN` chỉ áp dụng cho tài khoản mặc định.
</Note>

## Telegram side settings

<AccordionGroup>
  <Accordion title="Privacy mode và khả năng hiển thị trong nhóm">
    Telegram bots mặc định bật **Privacy Mode**, giới hạn các tin nhắn nhóm mà bot nhận được.

    Nếu bot cần thấy tất cả tin nhắn trong nhóm, bạn có thể:

    - tắt privacy mode qua `/setprivacy`, hoặc
    - đặt bot làm admin của nhóm.

    Khi thay đổi privacy mode, hãy xóa + thêm lại bot trong từng nhóm để Telegram áp dụng thay đổi.

  </Accordion>

  <Accordion title="Quyền hạn nhóm">
    Trạng thái admin được thiết lập trong cài đặt nhóm Telegram.

    Bot là admin sẽ nhận tất cả tin nhắn nhóm, hữu ích cho hành vi luôn bật trong nhóm.

  </Accordion>

  <Accordion title="Các tùy chọn hữu ích trong BotFather">

    - `/setjoingroups` để cho phép/từ chối thêm vào nhóm
    - `/setprivacy` để điều chỉnh hành vi hiển thị trong nhóm

  </Accordion>
</AccordionGroup>

## Access control và kích hoạt

<Tabs>
  <Tab title="Chính sách DM">
    `channels.telegram.dmPolicy` kiểm soát quyền truy cập tin nhắn trực tiếp:

    - `pairing` (mặc định)
    - `allowlist`
    - `open` (yêu cầu `allowFrom` bao gồm `"*"`)
    - `disabled`

    `channels.telegram.allowFrom` chấp nhận Telegram user ID dạng số. Tiền tố `telegram:` / `tg:` được chấp nhận và chuẩn hóa.
    Trình hướng dẫn onboarding chấp nhận nhập `@username` và sẽ phân giải sang ID số.
    Nếu bạn đã nâng cấp và config chứa các mục allowlist dạng `@username`, hãy chạy `openclaw doctor --fix` để phân giải chúng (best-effort; yêu cầu bot token Telegram).

    ### Tìm Telegram user ID của bạn

    An toàn hơn (không dùng bot bên thứ ba):

    1. DM bot của bạn.
    2. Chạy `openclaw logs --follow`.
    3. Đọc `from.id`.

    Cách chính thức qua Bot API:

```bash
curl "https://api.telegram.org/bot<bot_token>/getUpdates"
```

    Cách bên thứ ba (ít riêng tư hơn): `@userinfobot` hoặc `@getidsbot`.

  </Tab>

  <Tab title="Chính sách nhóm và allowlist">
    Có hai kiểm soát độc lập:

    1. **Nhóm nào được phép** (`channels.telegram.groups`)
       - không có cấu hình `groups`: cho phép tất cả nhóm
       - có cấu hình `groups`: hoạt động như allowlist (ID cụ thể hoặc `"*"`)

    2. **Người gửi nào được phép trong nhóm** (`channels.telegram.groupPolicy`)
       - `open`
       - `allowlist` (mặc định)
       - `disabled`

    `groupAllowFrom` dùng để lọc người gửi trong nhóm. Nếu không đặt, Telegram sẽ fallback về `allowFrom`.
    Các mục trong `groupAllowFrom` phải là Telegram user ID dạng số.

    Ví dụ: cho phép mọi thành viên trong một nhóm cụ thể:

```json5
{
  channels: {
    telegram: {
      groups: {
        "-1001234567890": {
          groupPolicy: "open",
          requireMention: false,
        },
      },
    },
  },
}
```

  </Tab>

  <Tab title="Hành vi mention">
    Phản hồi trong nhóm mặc định yêu cầu mention.

    Mention có thể đến từ:

    - mention gốc `@botusername`, hoặc
    - các pattern trong:
      - `agents.list[].groupChat.mentionPatterns`
      - `messages.groupChat.mentionPatterns`

    Lệnh bật/tắt ở mức phiên:

    - `/activation always`
    - `/activation mention`

    Các lệnh này chỉ cập nhật trạng thái phiên. Dùng config để lưu bền vững.

    Ví dụ config bền vững:

```json5
{
  channels: {
    telegram: {
      groups: {
        "*": { requireMention: false },
      },
    },
  },
}
```

    Lấy group chat ID:

    - chuyển tiếp tin nhắn nhóm tới `@userinfobot` / `@getidsbot`
    - hoặc đọc `chat.id` từ `openclaw logs --follow`
    - hoặc kiểm tra Bot API `getUpdates`

  </Tab>
</Tabs>

## Runtime behavior

- Telegram được sở hữu bởi tiến trình gateway.
- Định tuyến xác định: tin nhắn vào từ Telegram sẽ phản hồi lại Telegram (model không tự chọn kênh).
- Tin nhắn đến được chuẩn hóa vào phong bì kênh dùng chung với metadata trả lời và placeholder media.
- Phiên nhóm được tách biệt theo group ID. Forum topics nối thêm `:topic:<threadId>` để giữ tách biệt.
- Tin nhắn DM có thể mang `message_thread_id`; OpenClaw định tuyến bằng session key theo thread và giữ thread ID khi phản hồi.
- Long polling dùng grammY runner với tuần tự theo từng chat/từng thread. Tổng mức song song dùng `agents.defaults.maxConcurrent`.
- Telegram Bot API không hỗ trợ read-receipt (`sendReadReceipts` không áp dụng).

## Related

- [Pairing](/channels/pairing)
- [Channel routing](/channels/channel-routing)
- [Troubleshooting](/channels/troubleshooting)