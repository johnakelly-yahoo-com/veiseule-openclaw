---
title: "Telegram"
---

# Telegram (Bot API)

Status: production-ready for bot DMs + groups via grammY. Long-polling by default; webhook optional.

## Quick setup (beginner)

1. Create a bot with **@BotFather** ([direct link](https://t.me/BotFather)). Confirm the handle is exactly `@BotFather`, then copy the token.
2. Thiết lập token:
   - Env: `TELEGRAM_BOT_TOKEN=...`
   - Hoặc config: `channels.telegram.botToken: "..."`.
   - Nếu cả hai cùng được thiết lập, config sẽ được ưu tiên (env chỉ dùng làm fallback cho tài khoản mặc định).
3. Khởi động gateway.
4. Quyền truy cập DM mặc định là ghép cặp; phê duyệt mã ghép cặp ở lần liên hệ đầu tiên.

Cấu hình tối thiểu:

```json5
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

## Nó là gì

- Một kênh Telegram Bot API do Gateway sở hữu.
- Định tuyến xác định: phản hồi luôn quay lại Telegram; mô hình không bao giờ tự chọn kênh.
- DM dùng chung phiên chính của tác tử; nhóm được tách biệt (`agent:<agentId>:telegram:group:<chatId>`).

## Setup (fast path)

### 1. Tạo bot token (BotFather)

1. Open Telegram and chat with **@BotFather** ([direct link](https://t.me/BotFather)). Confirm the handle is exactly `@BotFather`.
2. Chạy `/newbot`, sau đó làm theo hướng dẫn (tên + username kết thúc bằng `bot`).
3. Sao chép token và lưu trữ an toàn.

Cài đặt BotFather tùy chọn:

- `/setjoingroups` — cho phép/không cho phép thêm bot vào nhóm.
- `/setprivacy` — kiểm soát việc bot có thấy tất cả tin nhắn trong nhóm hay không.

### 2. Cấu hình token (env hoặc config)

Ví dụ:

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

Env option: `TELEGRAM_BOT_TOKEN=...` (works for the default account).
Nếu cả biến môi trường và cấu hình đều được thiết lập, cấu hình sẽ được ưu tiên.

Hỗ trợ nhiều tài khoản: dùng `channels.telegram.accounts` với token theo từng tài khoản và `name` tùy chọn. See [`gateway/configuration`](/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) for the shared pattern.

3. Khởi động gateway. Telegram starts when a token is resolved (config first, env fallback).
4. DM access defaults to pairing. Approve the code when the bot is first contacted.
5. Với nhóm: thêm bot, quyết định hành vi privacy/admin (bên dưới), sau đó thiết lập `channels.telegram.groups` để kiểm soát gating theo mention + allowlist.

## Token + quyền riêng tư + quyền hạn (phía Telegram)

### Tạo token (BotFather)

- `/newbot` tạo bot và trả về token (giữ bí mật).
- Nếu token bị lộ, thu hồi/tạo lại qua @BotFather và cập nhật cấu hình.

### Khả năng hiển thị tin nhắn nhóm (Privacy Mode)

Telegram bots default to **Privacy Mode**, which limits which group messages they receive.
Nếu bot của bạn phải thấy _tất cả_ tin nhắn trong nhóm, bạn có hai lựa chọn:

- Tắt privacy mode bằng `/setprivacy` **hoặc**
- Thêm bot làm **admin** của nhóm (bot admin nhận tất cả tin nhắn).

**Lưu ý:** Khi thay đổi privacy mode, Telegram yêu cầu xóa + thêm lại bot
vào từng nhóm để thay đổi có hiệu lực.

### Quyền hạn nhóm (admin)

Trạng thái quản trị được thiết lập bên trong nhóm (UI Telegram). Admin bots always receive all
group messages, so use admin if you need full visibility.

## Cách hoạt động (hành vi)

- Tin nhắn đến được chuẩn hóa vào phong bì kênh dùng chung với ngữ cảnh trả lời và placeholder media.
- Trả lời trong nhóm mặc định yêu cầu mention (mention @ gốc hoặc `agents.list[].groupChat.mentionPatterns` / `messages.groupChat.mentionPatterns`).
- Ghi đè đa tác tử: thiết lập pattern theo từng tác tử trên `agents.list[].groupChat.mentionPatterns`.
- Phản hồi luôn quay lại cùng một chat Telegram.
- Long-polling dùng grammY runner với tuần tự theo từng chat; tổng mức song song bị giới hạn bởi `agents.defaults.maxConcurrent`.
- Telegram Bot API không hỗ trợ read receipts; không có tùy chọn `sendReadReceipts`.

## Draft streaming

OpenClaw có thể stream phản hồi từng phần trong Telegram DM bằng `sendMessageDraft`.

Yêu cầu:

- Bật Threaded Mode cho bot trong @BotFather (forum topic mode).
- Chỉ áp dụng cho thread chat riêng (Telegram bao gồm `message_thread_id` trong tin nhắn đến).
- `channels.telegram.streamMode` không được đặt là `"off"` (mặc định: `"partial"`, `"block"` bật cập nhật draft theo khối).

Draft streaming chỉ áp dụng cho DM; Telegram không hỗ trợ trong nhóm hoặc kênh.

## Định dạng (Telegram HTML)

- Văn bản gửi đi dùng `parse_mode: "HTML"` (tập con thẻ được Telegram hỗ trợ).
- Đầu vào dạng Markdown-ish được render thành **HTML an toàn cho Telegram** (đậm/nghiêng/gạch/xuống dòng/code/liên kết); các phần tử khối được làm phẳng thành văn bản với xuống dòng/gạch đầu dòng.
- HTML thô từ mô hình sẽ được escape để tránh lỗi parse của Telegram.
- Nếu Telegram từ chối payload HTML, OpenClaw sẽ thử gửi lại cùng thông điệp dưới dạng plain text.

## Lệnh (gốc + tùy chỉnh)

OpenClaw đăng ký các lệnh native (như `/status`, `/reset`, `/model`) với menu bot của Telegram khi khởi động.
Bạn có thể thêm các lệnh tùy chỉnh vào menu thông qua cấu hình:

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

## Xử lý sự cố thiết lập (lệnh)

- `setMyCommands failed` trong log thường có nghĩa là HTTPS/DNS outbound bị chặn tới `api.telegram.org`.
- Nếu thấy lỗi `sendMessage` hoặc `sendChatAction`, hãy kiểm tra định tuyến IPv6 và DNS.

Thêm trợ giúp: [Channel troubleshooting](/channels/troubleshooting).

Ghi chú:

- Lệnh tùy chỉnh **chỉ là mục menu**; OpenClaw không triển khai logic cho chúng trừ khi bạn xử lý ở nơi khác.
- Some commands can be handled by plugins/skills without being registered in Telegram’s command menu. These still work when typed (they just won't show up in `/commands` / the menu).
- Tên lệnh được chuẩn hóa (loại bỏ `/` ở đầu, chuyển chữ thường) và phải khớp `a-z`, `0-9`, `_` (1–32 ký tự).
- Custom commands **cannot override native commands**. Conflicts are ignored and logged.
- Nếu `commands.native` bị tắt, chỉ các lệnh tùy chỉnh được đăng ký (hoặc bị xóa nếu không có).

### Device pairing commands (`device-pair` plugin)

If the `device-pair` plugin is installed, it adds a Telegram-first flow for pairing a new phone:

1. `/pair` generates a setup code (sent as a separate message for easy copy/paste).
2. Paste the setup code in the iOS app to connect.
3. `/pair approve` approves the latest pending device request.

More details: [Pairing](/channels/pairing#pair-via-telegram-recommended-for-ios).

## Giới hạn

- Văn bản gửi đi được chia khối theo `channels.telegram.textChunkLimit` (mặc định 4000).
- Chia khối theo dòng trống (tùy chọn): đặt `channels.telegram.chunkMode="newline"` để tách theo dòng trống (ranh giới đoạn) trước khi chia theo độ dài.
- Tải xuống/tải lên media bị giới hạn bởi `channels.telegram.mediaMaxMb` (mặc định 5).
- Các yêu cầu Telegram Bot API hết thời gian sau `channels.telegram.timeoutSeconds` (mặc định 500 qua grammY). Set lower to avoid long hangs.
- Group history context uses `channels.telegram.historyLimit` (or `channels.telegram.accounts.*.historyLimit`), falling back to `messages.groupChat.historyLimit`. Set `0` to disable (default 50).
- `(skills, allowlists, auto-reply, system prompts, disable). Ghi đè theo người dùng:`channels.telegram.dms["&lt;user_id&gt;"].historyLimit\`.

## Chế độ kích hoạt nhóm

Theo mặc định, bot chỉ phản hồi các lần nhắc trong nhóm (`@botname` hoặc các mẫu trong `agents.list[].groupChat.mentionPatterns`). To change this behavior:

### Qua config (khuyến nghị)

```json5
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

**Important:** Setting `channels.telegram.groups` creates an **allowlist** - only listed groups (or `"*"`) will be accepted.
Các chủ đề forum kế thừa cấu hình nhóm cha của chúng (allowFrom, requireMention, skills, prompts) trừ khi bạn thêm ghi đè theo từng chủ đề dưới \`channels.telegram.groups.<groupId>`.topics.&lt;topicId&gt;`.

Cho phép tất cả nhóm và luôn phản hồi:

```json5
{
  channels: {
    telegram: {
      groups: {
        "*": { requireMention: false }, // all groups, always respond
      },
    },
  },
}
```

Giữ chế độ chỉ phản hồi khi mention cho tất cả nhóm (mặc định):

```json5
{
  channels: {
    telegram: {
      groups: {
        "*": { requireMention: true }, // or omit groups entirely
      },
    },
  },
}
```

### Qua lệnh (mức phiên)

Gửi trong nhóm:

- `/activation always` - phản hồi mọi tin nhắn
- `/activation mention` - yêu cầu mention (mặc định)

**Note:** Commands update session state only. Để có hành vi bền vững qua các lần khởi động lại, hãy dùng cấu hình.

### Lấy chat ID của nhóm

Chuyển tiếp bất kỳ tin nhắn nào từ nhóm tới `@userinfobot` hoặc `@getidsbot` trên Telegram để xem chat ID (số âm như `-1001234567890`).

**Mẹo:** Để lấy user ID của bạn, DM bot và bot sẽ trả lời user ID (thông báo ghép cặp), hoặc dùng `/whoami` khi lệnh đã được bật.

**Lưu ý về quyền riêng tư:** `@userinfobot` là bot của bên thứ ba. Nếu muốn, hãy thêm bot vào nhóm, gửi một tin nhắn và dùng `openclaw logs --follow` để đọc `chat.id`, hoặc dùng Bot API `getUpdates`.

## Ghi cấu hình

Mặc định, Telegram được phép ghi các cập nhật cấu hình được kích hoạt bởi sự kiện kênh hoặc `/config set|unset`.

Điều này xảy ra khi:

- Một nhóm được nâng cấp thành supergroup và Telegram phát ra `migrate_to_chat_id` (ID chat thay đổi). OpenClaw có thể tự động migrate `channels.telegram.groups`.
- Bạn chạy `/config set` hoặc `/config unset` trong chat Telegram (yêu cầu `commands.config: true`).

Tắt bằng:

```json5
{
  channels: { telegram: { configWrites: false } },
}
```

## Topics (forum supergroups)

Các chủ đề forum của Telegram bao gồm một `message_thread_id` cho mỗi tin nhắn. OpenClaw:

- Nối `:topic:<threadId>` vào khóa phiên nhóm Telegram để mỗi topic được tách biệt.
- Gửi typing indicator và phản hồi với `message_thread_id` để câu trả lời nằm trong topic.
- Topic chung (thread id `1`) là đặc biệt: khi gửi tin nhắn sẽ bỏ `message_thread_id` (Telegram từ chối), nhưng typing indicator vẫn bao gồm.
- Phơi bày `MessageThreadId` + `IsForum` trong ngữ cảnh template để định tuyến/templating.
- Cấu hình theo chủ đề có sẵn dưới `channels.telegram.groups.<chatId>``.topics.<threadId>`Legacy: `capabilities: ["inlineButtons"]` = `inlineButtons: "all"`.
- Cấu hình topic kế thừa thiết lập nhóm (requireMention, allowlists, skills, prompts, enabled) trừ khi ghi đè theo topic.

4. Các cuộc trò chuyện riêng tư có thể bao gồm `message_thread_id` trong một số trường hợp biên. OpenClaw keeps the DM session key unchanged, but still uses the thread id for replies/draft streaming when it is present.

## Inline Buttons

Telegram hỗ trợ inline keyboard với callback buttons.

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

Cho cấu hình theo từng tài khoản:

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

Phạm vi:

- `off` — tắt inline buttons
- `dm` — chỉ DM (chặn mục tiêu nhóm)
- `group` — chỉ nhóm (chặn mục tiêu DM)
- `all` — DM + nhóm
- `allowlist` — DM + nhóm, nhưng chỉ người gửi được cho phép bởi `allowFrom`/`groupAllowFrom` (cùng quy tắc với lệnh điều khiển)

6. Mặc định: `allowlist`.
   .capabilities\`: Các capability theo từng tài khoản, ghi đè các giá trị mặc định toàn cục cho tài khoản cụ thể đó.

### Gửi nút

Dùng message tool với tham số `buttons`:

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

Khi người dùng bấm nút, dữ liệu callback được gửi lại cho tác tử dưới dạng thông điệp với định dạng:
`callback_data: value`

### Tùy chọn cấu hình

Khả năng Telegram có thể cấu hình ở hai mức (hiển thị dạng object ở trên; mảng chuỗi legacy vẫn được hỗ trợ):

- `channels.telegram.capabilities`: Cấu hình khả năng mặc định toàn cục áp dụng cho tất cả tài khoản Telegram trừ khi bị ghi đè.
- `channels.telegram.accounts.<account>Những nhóm nào được phép** (allowlist nhóm qua `channels.telegram.groups\`):

Use the global setting when all Telegram bots/accounts should behave the same. Use per-account configuration when different bots need different behaviors (for example, one account only handles DMs while another is allowed in groups).

## Kiểm soát truy cập (DM + nhóm)

### Quyền truy cập DM

- Mặc định: `channels.telegram.dmPolicy = "pairing"`. Người gửi chưa biết sẽ nhận mã ghép cặp; tin nhắn bị bỏ qua cho đến khi được duyệt (mã hết hạn sau 1 giờ).
- Phê duyệt qua:
  - `openclaw pairing list telegram`
  - `openclaw pairing approve telegram <CODE>`
- Pairing is the default token exchange used for Telegram DMs. Details: [Pairing](/channels/pairing)
- 14. `channels.telegram.allowFrom` chấp nhận ID người dùng dạng số (khuyến nghị) hoặc mục nhập `@username`. It is **not** the bot username; use the human sender’s ID. 16. Trình hướng dẫn chấp nhận `@username` và sẽ phân giải nó thành ID số khi có thể.

#### Tìm Telegram user ID của bạn

An toàn hơn (không dùng bot bên thứ ba):

1. Khởi động gateway và DM bot của bạn.
2. Chạy `openclaw logs --follow` và tìm `from.id`.

Cách khác (Bot API chính thức):

1. DM bot của bạn.
2. Lấy updates bằng token bot và đọc `message.from.id`:

   ```bash
   curl "https://api.telegram.org/bot<bot_token>/getUpdates"
   ```

Bên thứ ba (ít riêng tư hơn):

- DM `@userinfobot` hoặc `@getidsbot` và dùng user id được trả về.

### Quyền truy cập nhóm

Hai kiểm soát độc lập:

17. **1. Những người gửi nào được phép** (lọc người gửi qua `channels.telegram.groupPolicy`):

- Không có cấu hình `groups` = cho phép tất cả nhóm
- Có cấu hình `groups` = chỉ các nhóm được liệt kê hoặc `"*"` được phép
- Ví dụ: `"groups": { "-1001234567890": {}, "*": {} }` cho phép tất cả nhóm

19. \*\*2. Các kênh bọc lịch sử nên đặt `CommandBody` (hoặc
    `RawBody`) thành văn bản tin nhắn gốc và giữ `Body` là prompt đã được kết hợp.

- `"open"` = tất cả người gửi trong các nhóm được phép có thể nhắn
- `"allowlist"` = chỉ người gửi trong `channels.telegram.groupAllowFrom` có thể nhắn
- `"disabled"` = không chấp nhận tin nhắn nhóm nào
  Mặc định là `groupPolicy: "allowlist"` (bị chặn trừ khi bạn thêm `groupAllowFrom`).

Hầu hết người dùng muốn: `groupPolicy: "allowlist"` + `groupAllowFrom` + các nhóm cụ thể được liệt kê trong `channels.telegram.groups`

Để cho phép **bất kỳ thành viên nhóm** nào nói chuyện trong một nhóm cụ thể (vẫn giữ lệnh điều khiển bị giới hạn cho người gửi được ủy quyền), đặt ghi đè theo nhóm:

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

## Long-polling vs webhook

- Mặc định: long-polling (không cần URL công khai).
- Chế độ webhook: đặt `channels.telegram.webhookUrl` và `channels.telegram.webhookSecret` (tùy chọn `channels.telegram.webhookPath`).
  - Listener cục bộ bind tới `0.0.0.0:8787` và phục vụ `POST /telegram-webhook` theo mặc định.
  - Nếu URL công khai của bạn khác, hãy dùng reverse proxy và trỏ `channels.telegram.webhookUrl` tới endpoint công khai.

## Threading phản hồi

Telegram hỗ trợ threading phản hồi tùy chọn qua thẻ:

- `[[reply_to_current]]` -- trả lời tin nhắn kích hoạt.
- `[[reply_to:<id>]]` -- trả lời một message id cụ thể.

Được kiểm soát bởi `channels.telegram.replyToMode`:

- `first` (mặc định), `all`, `off`.

## Tin nhắn âm thanh (voice vs file)

Telegram distinguishes **voice notes** (round bubble) from **audio files** (metadata card).
OpenClaw defaults to audio files for backward compatibility.

Để buộc gửi voice note trong phản hồi của tác tử, hãy chèn thẻ này ở bất kỳ đâu trong câu trả lời:

- `[[audio_as_voice]]` — gửi âm thanh dưới dạng voice note thay vì file.

The tag is stripped from the delivered text. Other channels ignore this tag.

Với message tool, đặt `asVoice: true` cùng URL `media` âm thanh tương thích voice
(`message` là tùy chọn khi có media):

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/voice.ogg",
  asVoice: true,
}
```

## Video messages (video vs video note)

Telegram distinguishes **video notes** (round bubble) from **video files** (rectangular).
OpenClaw defaults to video files.

For message tool sends, set `asVideoNote: true` with a video `media` URL:

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/video.mp4",
  asVideoNote: true,
}
```

(Note: Video notes do not support captions. If you provide a message text, it will be sent as a separate message.)

## Stickers

OpenClaw hỗ trợ nhận và gửi sticker Telegram với bộ nhớ đệm thông minh.

### Nhận sticker

Khi người dùng gửi sticker, OpenClaw xử lý dựa trên loại sticker:

- **Static stickers (WEBP):** Downloaded and processed through vision. The sticker appears as a `<media:sticker>` placeholder in the message content.
- **Sticker động (TGS):** Bỏ qua (định dạng Lottie không được hỗ trợ xử lý).
- **Sticker video (WEBM):** Bỏ qua (định dạng video không được hỗ trợ xử lý).

Trường ngữ cảnh template khả dụng khi nhận sticker:

- `Sticker` — object với:
  - `emoji` — emoji gắn với sticker
  - `setName` — tên bộ sticker
  - `fileId` — Telegram file ID (gửi lại cùng sticker)
  - `fileUniqueId` — ID ổn định để tra cứu cache
  - `cachedDescription` — mô tả vision đã cache khi có

### Bộ nhớ đệm sticker

Stickers are processed through the AI's vision capabilities to generate descriptions. Since the same stickers are often sent repeatedly, OpenClaw caches these descriptions to avoid redundant API calls.

**Cách hoạt động:**

1. **First encounter:** The sticker image is sent to the AI for vision analysis. 30. AI tạo ra một mô tả (ví dụ: "Một chú mèo hoạt hình đang vẫy tay đầy hào hứng").
2. **Lưu cache:** Mô tả được lưu cùng file ID, emoji và tên bộ sticker.
3. **Subsequent encounters:** When the same sticker is seen again, the cached description is used directly. The image is not sent to the AI.

**Vị trí cache:** `~/.openclaw/telegram/sticker-cache.json`

**Định dạng mục cache:**

```json
{
  "fileId": "CAACAgIAAxkBAAI...",
  "fileUniqueId": "AgADBAADb6cxG2Y",
  "emoji": "👋",
  "setName": "CoolCats",
  "description": "A cartoon cat waving enthusiastically",
  "cachedAt": "2026-01-15T10:30:00.000Z"
}
```

**Lợi ích:**

- Giảm chi phí API bằng cách tránh gọi vision lặp lại cho cùng sticker
- Thời gian phản hồi nhanh hơn cho sticker đã cache (không có độ trễ xử lý vision)
- Cho phép tìm kiếm sticker dựa trên mô tả đã cache

33. Bộ nhớ đệm được tự động tạo khi sticker được nhận. 34. Không cần quản lý bộ nhớ đệm thủ công.

### Gửi sticker

The agent can send and search stickers using the `sticker` and `sticker-search` actions. 36. Các tính năng này bị tắt theo mặc định và phải được bật trong cấu hình:

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

**Gửi một sticker:**

```json5
{
  action: "sticker",
  channel: "telegram",
  to: "123456789",
  fileId: "CAACAgIAAxkBAAI...",
}
```

Tham số:

- `fileId` (required) — the Telegram file ID of the sticker. 38. Lấy thông tin này từ `Sticker.fileId` khi nhận sticker, hoặc từ kết quả `sticker-search`.
- `replyTo` (tùy chọn) — message ID để trả lời.
- `threadId` (tùy chọn) — message thread ID cho forum topic.

**Tìm sticker:**

Tác tử có thể tìm sticker đã cache theo mô tả, emoji hoặc tên bộ:

```json5
{
  action: "sticker-search",
  channel: "telegram",
  query: "cat waving",
  limit: 5,
}
```

Trả về các sticker khớp từ cache:

```json5
{
  ok: true,
  count: 2,
  stickers: [
    {
      fileId: "CAACAgIAAxkBAAI...",
      emoji: "👋",
      description: "A cartoon cat waving enthusiastically",
      setName: "CoolCats",
    },
  ],
}
```

Tìm kiếm dùng fuzzy matching trên văn bản mô tả, ký tự emoji và tên bộ.

**Ví dụ với threading:**

```json5
{
  action: "sticker",
  channel: "telegram",
  to: "-1001234567890",
  fileId: "CAACAgIAAxkBAAI...",
  replyTo: 42,
  threadId: 123,
}
```

## Streaming (drafts)

Telegram can stream **draft bubbles** while the agent is generating a response.
40. OpenClaw sử dụng Bot API `sendMessageDraft` (không phải tin nhắn thật) và sau đó gửi

Yêu cầu (Telegram Bot API 9.3+):

- **Chat riêng với topics được bật** (forum topic mode cho bot).
- Tin nhắn đến phải bao gồm `message_thread_id` (private topic thread).
- Streaming bị bỏ qua cho nhóm/supergroup/kênh.

Cấu hình:

- `channels.telegram.streamMode: "off" | "partial" | "block"` (mặc định: `partial`)
  - `partial`: cập nhật draft bubble với văn bản streaming mới nhất.
  - `block`: cập nhật draft bubble theo các khối lớn hơn (chunked).
  - `off`: tắt draft streaming.
- Tùy chọn (chỉ cho `streamMode: "block"`):
  - `channels.telegram.draftChunk: { minChars?, maxChars?, breakPreference? 42. `channels.telegram.draftChunk: { minChars?, maxChars?, breakPreference?
    - mặc định: `minChars: 200`, `maxChars: 800`, `breakPreference: "paragraph"` (giới hạn tới `channels.telegram.textChunkLimit`).

43. }`Block streaming is off by default and requires`channels.telegram.blockStreaming: true\`
    if you want early Telegram messages instead of draft updates.

Reasoning stream (chỉ Telegram):

- `/reasoning stream` stream reasoning vào draft bubble trong khi tạo phản hồi,
  sau đó gửi câu trả lời cuối cùng không kèm reasoning.
- 45. Block streaming bị tắt theo mặc định và yêu cầu `channels.telegram.blockStreaming: true`
      More context: [Streaming + chunking](/concepts/streaming).

## Chính sách retry

47. Nếu `channels.telegram.streamMode` là `off`, stream suy luận sẽ bị vô hiệu hóa. 48. Thêm ngữ cảnh: [Streaming + chunking](/concepts/streaming). See [Retry policy](/concepts/retry).

## Agent tool (messages + reactions)

- Tool: `telegram` với action `sendMessage` (`to`, `content`, tùy chọn `mediaUrl`, `replyToMessageId`, `messageThreadId`).
- Tool: `telegram` với action `react` (`chatId`, `messageId`, `emoji`).
- Tool: `telegram` với action `deleteMessage` (`chatId`, `messageId`).
- Ngữ nghĩa xóa reaction: xem [/tools/reactions](/tools/reactions).
- Gating tool: `channels.telegram.actions.reactions`, `channels.telegram.actions.sendMessage`, `channels.telegram.actions.deleteMessage` (mặc định: bật), và `channels.telegram.actions.sticker` (mặc định: tắt).

## Thông báo reaction

50. Cấu hình qua `channels.telegram.retry`. When a user adds a reaction, OpenClaw:

1. Nhận update `message_reaction` từ Telegram API
2. Chuyển thành **system event** với định dạng: `"Telegram reaction added: {emoji} by {user} on msg {id}"`
3. Đưa system event vào hàng đợi dùng **cùng khóa phiên** với tin nhắn thường
4. Khi tin nhắn tiếp theo đến trong cuộc hội thoại đó, system event sẽ được xả và thêm vào đầu ngữ cảnh của tác tử

Tác tử thấy reaction như **thông báo hệ thống** trong lịch sử hội thoại, không phải metadata của tin nhắn.

**Cấu hình:**

- `channels.telegram.reactionNotifications`: Kiểm soát reaction nào kích hoạt thông báo
  - `"off"` — bỏ qua mọi reaction
  - `"own"` — thông báo khi người dùng react vào tin nhắn của bot (best-effort; trong bộ nhớ) (mặc định)
  - `"all"` — thông báo cho tất cả reaction

- `channels.telegram.reactionLevel`: Kiểm soát khả năng reaction của tác tử
  - `"off"` — tác tử không thể react tin nhắn
  - `"ack"` — bot gửi reaction xác nhận (👀 khi đang xử lý) (mặc định)
  - `"minimal"` — tác tử có thể react tiết kiệm (hướng dẫn: 1 lần mỗi 5–10 lượt trao đổi)
  - `"extensive"` — tác tử có thể react linh hoạt khi phù hợp

**Forum groups:** Reactions in forum groups include `message_thread_id` and use session keys like `agent:main:telegram:group:{chatId}:topic:{threadId}`. This ensures reactions and messages in the same topic stay together.

**Ví dụ cấu hình:**

```json5
{
  channels: {
    telegram: {
      reactionNotifications: "all", // See all reactions
      reactionLevel: "minimal", // Agent can react sparingly
    },
  },
}
```

**Yêu cầu:**

- Bot Telegram phải yêu cầu rõ `message_reaction` trong `allowed_updates` (được OpenClaw cấu hình tự động)
- Với chế độ webhook, reaction được bao gồm trong webhook `allowed_updates`
- Với chế độ polling, reaction được bao gồm trong `getUpdates` `allowed_updates`

## Mục tiêu gửi (CLI/cron)

- Dùng chat id (`123456789`) hoặc username (`@name`) làm mục tiêu.
- Ví dụ: `openclaw message send --channel telegram --target 123456789 --message "hi"`.

## Troubleshooting

**Bot không phản hồi tin nhắn không mention trong nhóm:**

- Nếu bạn đặt `channels.telegram.groups.*.requireMention=false`, **privacy mode** của Telegram Bot API phải bị tắt.
  - BotFather: `/setprivacy` → **Disable** (sau đó xóa + thêm lại bot vào nhóm)
- `openclaw channels status` hiển thị cảnh báo khi config mong đợi tin nhắn nhóm không mention.
- `openclaw channels status --probe` có thể kiểm tra thêm tư cách thành viên cho các group ID số cụ thể (không audit được rule wildcard `"*"`).
- Thử nhanh: `/activation always` (chỉ phiên; dùng config để lưu bền vững)

**Bot không thấy tin nhắn nhóm nào:**

- Nếu `channels.telegram.groups` được đặt, nhóm phải được liệt kê hoặc dùng `"*"`
- Kiểm tra Privacy Settings trong @BotFather → "Group Privacy" phải **OFF**
- Xác minh bot thực sự là thành viên (không chỉ là admin không có quyền đọc)
- Kiểm tra log gateway: `openclaw logs --follow` (tìm "skipping group message")

**Bot phản hồi khi mention nhưng không phản hồi `/activation always`:**

- Lệnh `/activation` cập nhật trạng thái phiên nhưng không lưu vào config
- Để lưu bền vững, thêm nhóm vào `channels.telegram.groups` với `requireMention: false`

**Các lệnh như `/status` không hoạt động:**

- Đảm bảo Telegram user ID của bạn được ủy quyền (qua ghép cặp hoặc `channels.telegram.allowFrom`)
- Lệnh yêu cầu ủy quyền ngay cả trong nhóm với `groupPolicy: "open"`

**Long-polling bị hủy ngay lập tức trên Node 22+ (thường với proxy/custom fetch):**

- Node 22+ nghiêm ngặt hơn với instance `AbortSignal`; signal lạ có thể hủy `fetch` ngay.
- Nâng cấp lên bản OpenClaw chuẩn hóa abort signals, hoặc chạy gateway trên Node 20 cho đến khi nâng cấp được.

**Bot starts, then silently stops responding (or logs `HttpError: Network request ... failed`):**

- Some hosts resolve `api.telegram.org` to IPv6 first. If your server does not have working IPv6 egress, grammY can get stuck on IPv6-only requests.
- Khắc phục bằng cách bật IPv6 egress **hoặc** ép resolve IPv4 cho `api.telegram.org` (ví dụ: thêm mục `/etc/hosts` dùng bản ghi A IPv4, hoặc ưu tiên IPv4 trong DNS OS), rồi khởi động lại gateway.
- Kiểm tra nhanh: `dig +short api.telegram.org A` và `dig +short api.telegram.org AAAA` để xác nhận DNS trả về gì.

## Tham chiếu cấu hình (Telegram)

Cấu hình đầy đủ: [Configuration](/gateway/configuration)

Tùy chọn provider:

- `channels.telegram.enabled`: bật/tắt khởi động kênh.
- `channels.telegram.botToken`: token của bot (BotFather).
- `channels.telegram.tokenFile`: đọc token từ đường dẫn file.
- `channels.telegram.dmPolicy`: `pairing | allowlist | open | disabled` (mặc định: ghép cặp).
- `channels.telegram.allowFrom`: DM allowlist (ids/usernames). `open` requires `"*"`.
- `channels.telegram.groupPolicy`: `open | allowlist | disabled` (mặc định: allowlist).
- `channels.telegram.groupAllowFrom`: allowlist người gửi trong nhóm (id/username).
- `channels.telegram.groups`: mặc định theo nhóm + allowlist (dùng `"*"` cho mặc định toàn cục).
  - `channels.telegram.groups.<id>.groupPolicy`: ghi đè groupPolicy theo từng nhóm (`open | allowlist | disabled`).
  - `channels.telegram.groups.<id>.requireMention`: thiết lập mặc định yêu cầu mention.
  - `channels.telegram.groups.<id>.skills`: bộ lọc skill (bỏ qua = tất cả skills, rỗng = không skill nào).
  - `channels.telegram.groups.<id>.allowFrom`: ghi đè danh sách người gửi được phép theo từng nhóm.
  - `channels.telegram.groups.<id>.systemPrompt`: system prompt bổ sung cho nhóm.
  - `channels.telegram.groups.<id>.enabled`: vô hiệu hóa nhóm khi đặt `false`.
  - `channels.telegram.groups.<id>.topics.<threadId>.*`: ghi đè theo từng topic (các trường giống như ở nhóm).
  - `channels.telegram.groups.<id>.topics.<threadId>.groupPolicy`: ghi đè groupPolicy theo từng topic (`open | allowlist | disabled`).
  - `channels.telegram.groups.<id>.topics.<threadId>.requireMention`: per-topic mention gating override.
- `channels.telegram.capabilities.inlineButtons`: `off | dm | group | all | allowlist` (mặc định: allowlist).
- `channels.telegram.accounts.<account>.capabilities.inlineButtons`: per-account override.
- `channels.telegram.replyToMode`: `off | first | all` (mặc định: `first`).
- `channels.telegram.textChunkLimit`: kích thước chunk gửi đi (ký tự).
- `channels.telegram.chunkMode`: `length` (mặc định) hoặc `newline` để tách theo dòng trống (ranh giới đoạn) trước khi chia theo độ dài.
- `channels.telegram.linkPreview`: bật/tắt preview liên kết cho tin nhắn gửi đi (mặc định: true).
- `channels.telegram.streamMode`: `off | partial | block` (draft streaming).
- `channels.telegram.mediaMaxMb`: giới hạn media inbound/outbound (MB).
- `channels.telegram.retry`: chính sách retry cho Telegram API outbound (attempts, minDelayMs, maxDelayMs, jitter).
- `channels.telegram.network.autoSelectFamily`: override Node autoSelectFamily (true=enable, false=disable). Defaults to disabled on Node 22 to avoid Happy Eyeballs timeouts.
- `channels.telegram.proxy`: URL proxy cho Bot API (SOCKS/HTTP).
- `channels.telegram.webhookUrl`: bật chế độ webhook (yêu cầu `channels.telegram.webhookSecret`).
- `channels.telegram.webhookSecret`: webhook secret (bắt buộc khi đặt webhookUrl).
- `channels.telegram.webhookPath`: đường dẫn webhook cục bộ (mặc định `/telegram-webhook`).
- `channels.telegram.actions.reactions`: gate reaction của Telegram tool.
- `channels.telegram.actions.sendMessage`: gate gửi tin nhắn của Telegram tool.
- `channels.telegram.actions.deleteMessage`: gate xóa tin nhắn của Telegram tool.
- `channels.telegram.actions.sticker`: gate action sticker Telegram — gửi và tìm (mặc định: false).
- `channels.telegram.reactionNotifications`: `off | own | all` — kiểm soát reaction nào kích hoạt system event (mặc định: `own` khi không đặt).
- `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` — kiểm soát khả năng reaction của tác tử (mặc định: `minimal` khi không đặt).

Tùy chọn toàn cục liên quan:

- `agents.list[].groupChat.mentionPatterns` (pattern gating theo mention).
- `messages.groupChat.mentionPatterns` (fallback toàn cục).
- `commands.native` (defaults to `"auto"` → on for Telegram/Discord, off for Slack), `commands.text`, `commands.useAccessGroups` (command behavior). Override with `channels.telegram.commands.native`.
- `messages.responsePrefix`, `messages.ackReaction`, `messages.ackReactionScope`, `messages.removeAckAfterReply`.
