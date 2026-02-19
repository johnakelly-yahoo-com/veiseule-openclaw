---
summary: "Trạng thái hỗ trợ bot Telegram, khả năng và cấu hình"
read_when:
  - Khi làm việc với các tính năng hoặc webhook của Telegram
title: "Telegram"
---

# Telegram (Bot API)

Status: production-ready for bot DMs + groups via grammY. Long polling là chế độ mặc định; chế độ webhook là tùy chọn.

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Chính sách DM mặc định cho Telegram là ghép nối.
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    Chẩn đoán và quy trình khắc phục sự cố liên kênh.
  
</Card>
  <Card title="Gateway configuration" icon="settings" href="/gateway/configuration">
    Các mẫu cấu hình kênh đầy đủ và ví dụ.
  
</Card>
</CardGroup>

## Thiết lập nhanh

<Steps>
  <Step title="Create the bot token in BotFather">
    Mở Telegram và trò chuyện với **@BotFather** (xác nhận chính xác handle là `@BotFather`).

    ```
    {
      channels: {
        telegram: {
          enabled: true,
          botToken: "123:abc",
          dmPolicy: "pairing",
        },
      },
    }
    ```

  
</Step>

  <Step title="Configure token and DM policy">

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

    ```
    Dự phòng bằng biến môi trường: `TELEGRAM_BOT_TOKEN=...` (chỉ áp dụng cho tài khoản mặc định).
    ```

  
</Step>

  <Step title="Start gateway and approve first DM">

```bash
openclaw gateway
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

    ```
    Mã ghép nối hết hạn sau 1 giờ.
    ```

  
</Step>

  <Step title="Add the bot to a group">
    Thêm bot vào nhóm của bạn, sau đó đặt `channels.telegram.groups` và `groupPolicy` phù hợp với mô hình truy cập của bạn.
  
</Step>
</Steps>

<Note>
Thứ tự phân giải token phụ thuộc theo từng tài khoản. Trên thực tế, giá trị trong config sẽ được ưu tiên hơn biến môi trường dự phòng, và `TELEGRAM_BOT_TOKEN` chỉ áp dụng cho tài khoản mặc định.
</Note>

## Cài đặt phía Telegram

<AccordionGroup>
  <Accordion title="Privacy mode and group visibility">
    Bot Telegram mặc định bật **Privacy Mode**, điều này giới hạn các tin nhắn nhóm mà bot có thể nhận được.

    ```
    Nếu bot cần xem tất cả tin nhắn trong nhóm, hãy:
    
    - tắt privacy mode bằng `/setprivacy`, hoặc
    - cấp quyền admin cho bot trong nhóm.
    
    Khi thay đổi privacy mode, hãy xóa + thêm lại bot trong từng nhóm để Telegram áp dụng thay đổi.
    ```

  
</Accordion>

  <Accordion title="Group permissions">
    Trạng thái admin được quản lý trong cài đặt nhóm Telegram.

    ```
    Admin bot nhận được tất cả tin nhắn trong nhóm, điều này hữu ích cho các hành vi nhóm luôn hoạt động.
    ```

  
</Accordion>

  <Accordion title="Helpful BotFather toggles">

    ```
    - `/setjoingroups` để cho phép/từ chối việc thêm vào nhóm
    - `/setprivacy` để cấu hình chế độ hiển thị trong nhóm
    ```

  
</Accordion>
</AccordionGroup>

## Kiểm soát truy cập và kích hoạt

<Tabs>
  <Tab title="DM policy">
    `channels.telegram.dmPolicy` kiểm soát quyền truy cập tin nhắn trực tiếp:

    ```
    - `pairing` (mặc định)
    - `allowlist`
    - `open` (yêu cầu `allowFrom` bao gồm `"*"`)
    - `disabled`
    
    `channels.telegram.allowFrom` chấp nhận ID người dùng Telegram dạng số. Tiền tố `telegram:` / `tg:` được chấp nhận và sẽ được chuẩn hóa.
    Trình hướng dẫn khởi tạo (onboarding wizard) chấp nhận đầu vào `@username` và sẽ phân giải thành ID dạng số.
    Nếu bạn đã nâng cấp và cấu hình của bạn chứa các mục allowlist dạng `@username`, hãy chạy `openclaw doctor --fix` để phân giải chúng (best-effort; yêu cầu Telegram bot token).
    
    ### Tìm Telegram user ID của bạn
    
    Cách an toàn hơn (không dùng bot bên thứ ba):
    
    1. Gửi DM cho bot của bạn.
    2. Chạy `openclaw logs --follow`.
    3. Đọc giá trị `from.id`.
    
    Phương pháp chính thức qua Bot API:
    ```

```bash
curl "https://api.telegram.org/bot<bot_token>/getUpdates"
```

    ```
    Phương pháp bên thứ ba (kém riêng tư hơn): `@userinfobot` hoặc `@getidsbot`.
    ```

  
</Tab>

  <Tab title="Group policy and allowlists">
    Có hai cơ chế kiểm soát độc lập:

    ```
    1. **Những nhóm nào được phép** (`channels.telegram.groups`)
       - không có cấu hình `groups`: tất cả nhóm đều được phép
       - có cấu hình `groups`: hoạt động như allowlist (ID cụ thể hoặc `"*"`)
    
    2. **Những người gửi nào được phép trong nhóm** (`channels.telegram.groupPolicy`)
       - `open`
       - `allowlist` (mặc định)
       - `disabled`
    
    `groupAllowFrom` được dùng để lọc người gửi trong nhóm. Nếu không được thiết lập, Telegram sẽ fallback về `allowFrom`.
    Các mục trong `groupAllowFrom` phải là Telegram user ID dạng số.
    
    Ví dụ: cho phép mọi thành viên trong một nhóm cụ thể:
    ```

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

  <Tab title="Mention behavior">
    Phản hồi trong nhóm mặc định yêu cầu phải mention.

    ```
    Mention có thể đến từ:
    
    - mention gốc `@botusername`, hoặc
    - các mẫu mention trong:
      - `agents.list[].groupChat.mentionPatterns`
      - `messages.groupChat.mentionPatterns`
    
    Các lệnh bật/tắt ở cấp phiên:
    
    - `/activation always`
    - `/activation mention`
    
    Các lệnh này chỉ cập nhật trạng thái phiên. Sử dụng cấu hình để lưu vĩnh viễn.
    
    Ví dụ cấu hình lưu vĩnh viễn:
    ```

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

    ```
    Lấy group chat ID:
    
    - chuyển tiếp một tin nhắn trong nhóm đến `@userinfobot` / `@getidsbot`
    - hoặc đọc `chat.id` từ `openclaw logs --follow`
    - hoặc kiểm tra Bot API `getUpdates`
    ```

  
</Tab>
</Tabs>

## Hành vi runtime

- Telegram được quản lý bởi tiến trình gateway.
- Định tuyến là xác định: tin nhắn vào từ Telegram sẽ phản hồi lại Telegram (model không chọn kênh).
- Tin nhắn đầu vào được chuẩn hóa vào channel envelope dùng chung với metadata phản hồi và placeholder media.
- Phiên nhóm được tách biệt theo group ID. Forum topics được nối thêm `:topic:<threadId>` để giữ các chủ đề tách biệt.
- Tin nhắn DM có thể chứa `message_thread_id`; OpenClaw định tuyến chúng bằng session key nhận biết theo thread và giữ nguyên thread ID khi phản hồi.
- Long polling sử dụng grammY runner với cơ chế tuần tự theo từng chat/từng thread. Mức độ concurrency tổng thể của runner sink sử dụng `agents.defaults.maxConcurrent`.
- Telegram Bot API không hỗ trợ read-receipt (`sendReadReceipts` không áp dụng).

## Tham chiếu tính năng

<AccordionGroup>
  <Accordion title="Live stream preview (message edits)">
    OpenClaw có thể stream phản hồi từng phần bằng cách gửi một tin nhắn Telegram tạm thời và chỉnh sửa nó khi văn bản được tạo ra.

    ```
    Yêu cầu:
    
    - `channels.telegram.streamMode` không phải `"off"` (mặc định: `"partial"`)
    
    Các chế độ:
    
    - `off`: không có xem trước trực tiếp
    - `partial`: cập nhật xem trước thường xuyên từ văn bản từng phần
    - `block`: cập nhật xem trước theo từng khối dựa trên `channels.telegram.draftChunk`
    
    `draftChunk` mặc định cho `streamMode: "block"`:
    
    - `minChars: 200`
    - `maxChars: 800`
    - `breakPreference: "paragraph"`
    
    `maxChars` bị giới hạn bởi `channels.telegram.textChunkLimit`.
    
    Hoạt động trong chat trực tiếp và nhóm/topic.
    
    Đối với phản hồi chỉ có văn bản, OpenClaw giữ nguyên cùng một tin nhắn xem trước và thực hiện chỉnh sửa cuối cùng ngay tại chỗ (không gửi tin nhắn thứ hai).
    
    Đối với phản hồi phức tạp (ví dụ payload media), OpenClaw sẽ fallback sang cách gửi cuối cùng thông thường rồi dọn dẹp tin nhắn xem trước.
    
    `streamMode` tách biệt với block streaming. Khi block streaming được bật rõ ràng cho Telegram, OpenClaw sẽ bỏ qua preview stream để tránh stream kép.
    
    Telegram-only reasoning stream:
    
    - `/reasoning stream` gửi reasoning vào phần xem trước trực tiếp trong khi đang tạo
    - câu trả lời cuối cùng được gửi mà không kèm văn bản reasoning
    ```

  
</Accordion>

  <Accordion title="Formatting and HTML fallback">
    Văn bản gửi đi sử dụng Telegram `parse_mode: "HTML"`.

    ```
    - Văn bản kiểu Markdown được render sang HTML an toàn cho Telegram.
    - HTML thô từ model được escape để giảm lỗi parse của Telegram.
    - Nếu Telegram từ chối HTML đã parse, OpenClaw sẽ thử lại dưới dạng văn bản thuần.
    
    Link preview được bật mặc định và có thể tắt bằng `channels.telegram.linkPreview: false`.
    ```

  
</Accordion>

  <Accordion title="Native commands and custom commands">
    Đăng ký menu lệnh Telegram được xử lý khi khởi động bằng `setMyCommands`.

    ```
    Mặc định lệnh native:
    
    - `commands.native: "auto"` bật lệnh native cho Telegram
    
    Thêm các mục lệnh tùy chỉnh vào menu:
    ```

```json5
{
  channels: {
    telegram: {
      customCommands: [
        { command: "backup", description: "Git backup" },
        { command: "generate", description: "Create an image" },
      ],
    },
  },
}
```

    ```
    {
      channels: {
        telegram: {
          groups: {
            "-1001234567890": { requireMention: false }, // always respond in this group
          },
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="Inline buttons">
    Cấu hình phạm vi inline keyboard:

```json5
{
  channels: {
    telegram: {
      capabilities: {
        inlineButtons: "allowlist",
      },
    },
  },
}
```

    ```
    Ghi đè theo từng tài khoản:
    ```

```json5
{
  channels: {
    telegram: {
      accounts: {
        main: {
          capabilities: {
            inlineButtons: "allowlist",
          },
        },
      },
    },
  },
}
```

    ```
    Phạm vi:
    
    - `off`
    - `dm`
    - `group`
    - `all`
    - `allowlist` (mặc định)
    
    Legacy `capabilities: ["inlineButtons"]` được ánh xạ thành `inlineButtons: "all"`.
    
    Ví dụ message action:
    ```

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  message: "Choose an option:",
  buttons: [
    [
      { text: "Yes", callback_data: "yes" },
      { text: "No", callback_data: "no" },
    ],
    [{ text: "Cancel", callback_data: "cancel" }],
  ],
}
```

    ```
    Callback click được truyền đến agent dưới dạng văn bản:
    `callback_data: <value>`
    ```

  
</Accordion>

  <Accordion title="Telegram message actions for agents and automation">
    Các tool action của Telegram bao gồm:

    ```
    - `sendMessage` (`to`, `content`, tùy chọn `mediaUrl`, `replyToMessageId`, `messageThreadId`)
    - `react` (`chatId`, `messageId`, `emoji`)
    - `deleteMessage` (`chatId`, `messageId`)
    - `editMessage` (`chatId`, `messageId`, `content`)
    
    Channel message action cung cấp các alias thân thiện (`send`, `react`, `delete`, `edit`, `sticker`, `sticker-search`).
    
    Cơ chế gating:
    
    - `channels.telegram.actions.sendMessage`
    - `channels.telegram.actions.editMessage`
    - `channels.telegram.actions.deleteMessage`
    - `channels.telegram.actions.reactions`
    - `channels.telegram.actions.sticker` (mặc định: disabled)
    
    Ngữ nghĩa gỡ reaction: [/tools/reactions](/tools/reactions)
    ```

  
</Accordion>

  <Accordion title="Reply threading tags">
    Telegram hỗ trợ thẻ reply threading tường minh trong nội dung được tạo:

    ```
    - `[[reply_to_current]]` trả lời tin nhắn kích hoạt
    - `[[reply_to:<id>]]` trả lời một Telegram message ID cụ thể
    
    `channels.telegram.replyToMode` kiểm soát cách xử lý:
    
    - `off` (mặc định)
    - `first`
    - `all`
    
    Lưu ý: `off` vô hiệu hóa reply threading ngầm định. Các thẻ tường minh `[[reply_to_*]]` vẫn được tôn trọng.
    ```

  
</Accordion>

  <Accordion title="Forum topics and thread behavior">
    Forum supergroups:

    ```
    - khóa phiên theo chủ đề được thêm `:topic:<threadId>`
    - phản hồi và trạng thái đang nhập sẽ nhắm tới luồng chủ đề
    - đường dẫn cấu hình chủ đề:
      `channels.telegram.groups.<chatId>.topics.<threadId>`
    
    Trường hợp đặc biệt với chủ đề chung (`threadId=1`):
    
    - khi gửi tin nhắn sẽ bỏ qua `message_thread_id` (Telegram từ chối `sendMessage(...thread_id=1)`)
    - hành động đang nhập vẫn bao gồm `message_thread_id`
    
    Kế thừa chủ đề: các mục chủ đề sẽ kế thừa cài đặt của nhóm trừ khi bị ghi đè (`requireMention`, `allowFrom`, `skills`, `systemPrompt`, `enabled`, `groupPolicy`).
    
    Ngữ cảnh template bao gồm:
    
    - `MessageThreadId`
    - `IsForum`
    
    Hành vi luồng DM:
    
    - các cuộc trò chuyện riêng tư có `message_thread_id` vẫn giữ định tuyến DM nhưng sử dụng khóa phiên và mục tiêu phản hồi theo luồng.
    ```

  
</Accordion>

  <Accordion title="Audio, video, and stickers">### Audio messages

    ```
    Telegram phân biệt giữa voice note và tệp âm thanh.
    
    - mặc định: hành vi như tệp âm thanh
    - thêm thẻ `[[audio_as_voice]]` trong phản hồi của agent để buộc gửi dưới dạng voice note
    
    Ví dụ hành động tin nhắn:
    ```

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/voice.ogg",
  asVoice: true,
}
```

    ```
    ### Video messages
    
    Telegram phân biệt giữa tệp video và video note.
    
    Ví dụ hành động tin nhắn:
    ```

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/video.mp4",
  asVideoNote: true,
}
```

    ```
    Video note không hỗ trợ chú thích; nội dung văn bản sẽ được gửi riêng.
    
    ### Stickers
    
    Xử lý sticker đầu vào:
    
    - WEBP tĩnh: được tải xuống và xử lý (placeholder `<media:sticker>`)
    - TGS động: bỏ qua
    - WEBM video: bỏ qua
    
    Các trường ngữ cảnh sticker:
    
    - `Sticker.emoji`
    - `Sticker.setName`
    - `Sticker.fileId`
    - `Sticker.fileUniqueId`
    - `Sticker.cachedDescription`
    
    Tệp cache sticker:
    
    - `~/.openclaw/telegram/sticker-cache.json`
    
    Sticker sẽ được mô tả một lần (khi có thể) và lưu cache để giảm các lần gọi vision lặp lại.
    
    Bật các hành động sticker:
    ```

```json5
{
  channels: {
    telegram: {
      actions: {
        sticker: true,
      },
    },
  },
}
```

    ```
    Gửi hành động sticker:
    ```

```json5
{
  action: "sticker",
  channel: "telegram",
  to: "123456789",
  fileId: "CAACAgIAAxkBAAI...",
}
```

    ```
    Tìm kiếm sticker đã cache:
    ```

```json5
{
  action: "sticker-search",
  channel: "telegram",
  query: "cat waving",
  limit: 5,
}
```

  
</Accordion>

  <Accordion title="Reaction notifications">Telegram reactions đến dưới dạng cập nhật `message_reaction` (tách biệt với payload tin nhắn).

    ```
    Khi được bật, OpenClaw sẽ đưa vào hàng đợi các sự kiện hệ thống như:
    
    - `Telegram reaction added: 👍 by Alice (@alice) on msg 42`
    
    Cấu hình:
    
    - `channels.telegram.reactionNotifications`: `off | own | all` (mặc định: `own`)
    - `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` (mặc định: `minimal`)
    
    Lưu ý:
    
    - `own` nghĩa là chỉ phản ứng của người dùng đối với các tin nhắn do bot gửi (best-effort thông qua cache tin nhắn đã gửi).
    - Telegram không cung cấp thread ID trong cập nhật reaction.
      - nhóm không phải forum sẽ định tuyến đến phiên chat nhóm
      - nhóm forum sẽ định tuyến đến phiên chủ đề chung của nhóm (`:topic:1`), không phải đúng chủ đề gốc
    
    `allowed_updates` cho polling/webhook tự động bao gồm `message_reaction`.
    ```

  
</Accordion>

  <Accordion title="Ack reactions">`ackReaction` gửi một emoji xác nhận trong khi OpenClaw đang xử lý tin nhắn đầu vào.

    ```
    Thứ tự phân giải:
    
    - `channels.telegram.accounts.<accountId>.ackReaction`
    - `channels.telegram.ackReaction`
    - `messages.ackReaction`
    - emoji nhận diện của agent làm phương án dự phòng (`agents.list[].identity.emoji`, nếu không có thì "👀")
    
    Lưu ý:
    
    - Telegram yêu cầu emoji unicode (ví dụ "👀").
    - Sử dụng `""` để vô hiệu hóa reaction cho một kênh hoặc tài khoản.
    ```

  
</Accordion>

  <Accordion title="Config writes from Telegram events and commands">Ghi cấu hình kênh được bật theo mặc định (`configWrites !== false`).

    ```
    Các thao tác ghi được kích hoạt bởi Telegram bao gồm:
    
    - sự kiện di chuyển nhóm (`migrate_to_chat_id`) để cập nhật `channels.telegram.groups`
    - `/config set` và `/config unset` (yêu cầu bật lệnh)
    ```

```json5
Vô hiệu hóa:
```

  
</Accordion>

  <Accordion title="Long polling vs webhook">{
  channels: {
    telegram: {
      configWrites: false,
    },
  },
}

    ```
    Mặc định: long polling.
    ```

  
</Accordion>

  <Accordion title="Limits, retry, and CLI targets">
    Chế độ webhook:

- đặt `channels.telegram.webhookUrl`
- đặt `channels.telegram.webhookSecret` (bắt buộc khi đã đặt webhook URL)
- tùy chọn `channels.telegram.webhookPath` (mặc định `/telegram-webhook`)
- tùy chọn `channels.telegram.webhookHost` (mặc định `127.0.0.1`)

Trình lắng nghe cục bộ mặc định cho chế độ webhook bind tới `127.0.0.1:8787`.

Nếu endpoint công khai của bạn khác, hãy đặt reverse proxy phía trước và trỏ `webhookUrl` tới URL công khai.
Đặt `webhookHost` (ví dụ `0.0.0.0`) khi bạn chủ ý cần nhận truy cập từ bên ngoài.
    - `channels.telegram.textChunkLimit` mặc định là 4000.
    - `channels.telegram.chunkMode="newline"` ưu tiên ranh giới đoạn văn (dòng trống) trước khi tách theo độ dài.
    - `channels.telegram.mediaMaxMb` (mặc định 5) giới hạn kích thước tải xuống/xử lý media Telegram đầu vào.
    - `channels.telegram.timeoutSeconds` ghi đè thời gian chờ của Telegram API client (nếu không đặt, mặc định của grammY sẽ được áp dụng).
    - lịch sử ngữ cảnh nhóm sử dụng `channels.telegram.historyLimit` hoặc `messages.groupChat.historyLimit` (mặc định 50); `0` để tắt.<user_id>- điều khiển lịch sử DM:
      - `channels.telegram.dmHistoryLimit`
      - `channels.telegram.dms["

    ```
    "].historyLimit`
        - số lần retry Telegram API đầu ra có thể cấu hình qua `channels.telegram.retry`.
    ```

```bash
Mục tiêu gửi CLI có thể là chat ID dạng số hoặc username:
```

  
</Accordion>
</AccordionGroup>

## Troubleshooting

<AccordionGroup>
  <Accordion title="Bot does not respond to non mention group messages">

    ```
    
</Accordion>
    ```

  
</Accordion>

  <Accordion title="Bot not seeing group messages at all">

    ```
    - Nếu `requireMention=false`, chế độ privacy của Telegram phải cho phép hiển thị đầy đủ.
      - BotFather: `/setprivacy` -> Disable
      - sau đó xóa + thêm lại bot vào nhóm
    - `openclaw channels status` sẽ cảnh báo khi cấu hình mong đợi tin nhắn nhóm không cần mention.
    - `openclaw channels status --probe` có thể kiểm tra các group ID dạng số cụ thể; ký tự đại diện `"*"` không thể được kiểm tra membership.
    - kiểm tra nhanh phiên: `/activation always`.
    ```

  
</Accordion>

  <Accordion title="Commands work partially or not at all">

    ```
    - khi `channels.telegram.groups` tồn tại, nhóm phải được liệt kê (hoặc bao gồm `"*"`)
    - xác minh bot là thành viên của nhóm
    - xem log: `openclaw logs --follow` để biết lý do bị bỏ qua
    ```

  
</Accordion>

  <Accordion title="Polling or network instability">

    ```
    - ủy quyền danh tính người gửi của bạn (ghép cặp và/hoặc `allowFrom` dạng số)
    - ủy quyền lệnh vẫn được áp dụng ngay cả khi chính sách nhóm là `open`
    - `setMyCommands failed` thường cho thấy sự cố DNS/HTTPS khi truy cập `api.telegram.org`
    ```

```bash
- Node 22+ + fetch/proxy tùy chỉnh có thể gây hủy ngay lập tức nếu kiểu AbortSignal không khớp.
- Một số host phân giải `api.telegram.org` sang IPv6 trước; IPv6 egress bị lỗi có thể gây lỗi Telegram API không ổn định.
- Xác thực kết quả DNS:
```

  
</Accordion>
</AccordionGroup>

Telegram hỗ trợ threading phản hồi tùy chọn qua thẻ:

## 
</Accordion>

Được kiểm soát bởi `channels.telegram.replyToMode`:

- `first` (mặc định), `all`, `off`.

- `channels.telegram.botToken`: bot token (BotFather).

- `channels.telegram.tokenFile`: đọc token từ đường dẫn file.

- `channels.telegram.dmPolicy`: `pairing | allowlist | open | disabled` (mặc định: ghép cặp).

- Tham chiếu cấu hình Telegram `open` requires `"*"`. `channels.telegram.allowFrom`: danh sách cho phép DM (Telegram user ID dạng số).

- `channels.telegram.groupPolicy`: `open | allowlist | disabled` (mặc định: allowlist).

- `openclaw doctor --fix` có thể chuyển các mục `@username` cũ sang ID. `channels.telegram.groupAllowFrom`: danh sách cho phép người gửi trong nhóm (Telegram user ID dạng số).

- `channels.telegram.groups`: mặc định theo nhóm + allowlist (dùng `"*"` cho mặc định toàn cục).
  - `channels.telegram.groups.<id>.groupPolicy`: per-group override for groupPolicy (`open | allowlist | disabled`).
  - `channels.telegram.groups.<id>.requireMention`: mention gating default.
  - `channels.telegram.groups.<id>.skills`: bộ lọc skill (bỏ qua = tất cả skills, rỗng = không skill nào).
  - `channels.telegram.groups.<id>.allowFrom`: per-group sender allowlist override.
  - `channels.telegram.groups.<id>.systemPrompt`: extra system prompt for the group.
  - `channels.telegram.groups.<id>.enabled`: disable the group when `false`.
  - `channels.telegram.groups.<id>`.topics.<threadId>`.*`: per-topic overrides (same fields as group).
  - `channels.telegram.groups.<id>.topics.<threadId>.groupPolicy`: per-topic override for groupPolicy (`open | allowlist | disabled`).
  - `channels.telegram.groups.<id>.topics.<threadId>.requireMention`: per-topic mention gating override.

- `channels.telegram.capabilities.inlineButtons`: `off | dm | group | all | allowlist` (mặc định: allowlist).

- `channels.telegram.accounts.<account>.capabilities.inlineButtons`: per-account override.

- `openclaw doctor --fix` có thể chuyển các mục `@username` cũ sang ID.

- `channels.telegram.textChunkLimit`: kích thước chunk gửi đi (ký tự).

- `channels.telegram.chunkMode`: `length` (mặc định) hoặc `newline` để tách theo dòng trống (ranh giới đoạn) trước khi chia theo độ dài.

- `channels.telegram.linkPreview`: bật/tắt preview liên kết cho tin nhắn gửi đi (mặc định: true).

- `channels.telegram.replyToMode`: `off | first | all` (mặc định: `off`).

- `channels.telegram.mediaMaxMb`: giới hạn media inbound/outbound (MB).

- `channels.telegram.retry`: chính sách retry cho Telegram API outbound (attempts, minDelayMs, maxDelayMs, jitter).

- `channels.telegram.network.autoSelectFamily`: override Node autoSelectFamily (true=enable, false=disable). Defaults to disabled on Node 22 to avoid Happy Eyeballs timeouts.

- `channels.telegram.proxy`: URL proxy cho Bot API (SOCKS/HTTP).

- `channels.telegram.webhookUrl`: bật chế độ webhook (yêu cầu `channels.telegram.webhookSecret`).

- `channels.telegram.webhookSecret`: webhook secret (bắt buộc khi đặt webhookUrl).

- `channels.telegram.webhookPath`: đường dẫn webhook cục bộ (mặc định `/telegram-webhook`).

- `channels.telegram.streamMode`: `off | partial | block` (xem trước live stream).

- `channels.telegram.actions.reactions`: gate reaction của Telegram tool.

- `channels.telegram.actions.sendMessage`: gate gửi tin nhắn của Telegram tool.

- `channels.telegram.actions.deleteMessage`: gate xóa tin nhắn của Telegram tool.

- `channels.telegram.actions.sticker`: gate action sticker Telegram — gửi và tìm (mặc định: false).

- `channels.telegram.reactionNotifications`: `off | own | all` — kiểm soát reaction nào kích hoạt system event (mặc định: `own` khi không đặt).

- `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` — kiểm soát khả năng reaction của tác tử (mặc định: `minimal` khi không đặt).

- [Tham chiếu cấu hình - Telegram](/gateway/configuration-reference#telegram)

Các trường tín hiệu quan trọng dành riêng cho Telegram:

- khởi động/xác thực: `enabled`, `botToken`, `tokenFile`, `accounts.*`
- kiểm soát truy cập: `dmPolicy`, `allowFrom`, `groupPolicy`, `groupAllowFrom`, `groups`, `groups.*.topics.*`
- lệnh/menu: `commands.native`, `customCommands`
- luồng hội thoại/trả lời: `replyToMode`
- streaming: `streamMode` (xem trước), `draftChunk`, `blockStreaming`
- định dạng/gửi tin: `textChunkLimit`, `chunkMode`, `linkPreview`, `responsePrefix`
- media/mạng: `mediaMaxMb`, `timeoutSeconds`, `retry`, `network.autoSelectFamily`, `proxy`
- webhook: `webhookUrl`, `webhookSecret`, `webhookPath`, `webhookHost`
- hành động/khả năng: `capabilities.inlineButtons`, `actions.sendMessage|editMessage|deleteMessage|reactions|sticker`
- phản ứng: `reactionNotifications`, `reactionLevel`
- ghi/cập nhật lịch sử: `configWrites`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`

## Liên quan

- `[[audio_as_voice]]` — gửi âm thanh dưới dạng voice note thay vì file.
- [Định tuyến kênh](/channels/channel-routing)
- [Khắc phục sự cố](/channels/troubleshooting)

