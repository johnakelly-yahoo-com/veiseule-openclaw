---
title: "Tài liệu tham chiếu cấu hình"
description: "Tài liệu tham chiếu đầy đủ theo từng trường cho ~/.openclaw/openclaw.json"
---

# Tài liệu tham chiếu cấu hình

Mọi trường có sẵn trong `~/.openclaw/openclaw.json`. Để xem tổng quan theo tác vụ, xem [Configuration](/gateway/configuration).

Định dạng cấu hình là **JSON5** (cho phép comment và dấu phẩy cuối). Tất cả các trường đều không bắt buộc — OpenClaw sử dụng giá trị mặc định an toàn khi bị bỏ qua.

---

## Kênh

Mỗi kênh sẽ tự động khởi động khi phần cấu hình của nó tồn tại (trừ khi `enabled: false`).

### Truy cập DM và nhóm

Tất cả các kênh đều hỗ trợ chính sách DM và chính sách nhóm:

| Chính sách DM                           | Hành vi                                                                                                 |
| --------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| `pairing` (mặc định) | Người gửi chưa xác định sẽ nhận mã ghép nối một lần; chủ sở hữu phải phê duyệt                          |
| `allowlist`                             | Chỉ cho phép người gửi trong `allowFrom` (hoặc trong danh sách cho phép đã ghép nối) |
| `open`                                  | Cho phép tất cả DM đến (yêu cầu `allowFrom: ["*"]`)                                  |
| `disabled`                              | Bỏ qua tất cả DM đến                                                                                    |

| Chính sách nhóm                           | Hành vi                                                                       |
| ----------------------------------------- | ----------------------------------------------------------------------------- |
| `allowlist` (mặc định) | Chỉ các nhóm khớp với allowlist đã cấu hình                                   |
| `open`                                    | Bỏ qua allowlist nhóm (vẫn áp dụng kiểm soát theo mention) |
| `disabled`                                | Chặn tất cả tin nhắn nhóm/phòng                                               |

<Note>
`channels.defaults.groupPolicy` đặt giá trị mặc định khi `groupPolicy` của provider chưa được thiết lập.
Mã ghép nối hết hạn sau 1 giờ. Các yêu cầu ghép nối DM đang chờ được giới hạn ở **3 mỗi kênh**.
Slack/Discord có cơ chế dự phòng đặc biệt: nếu toàn bộ phần cấu hình provider bị thiếu, chính sách nhóm khi chạy có thể được đặt thành `open` (kèm cảnh báo khi khởi động).
</Note>

### WhatsApp

WhatsApp chạy thông qua kênh web của gateway (Baileys Web). Nó tự động khởi động khi tồn tại một phiên đã liên kết.

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing", // pairing | allowlist | open | disabled
      allowFrom: ["+15555550123", "+447700900123"],
      textChunkLimit: 4000,
      chunkMode: "length", // length | newline
      mediaMaxMb: 50,
      sendReadReceipts: true, // dấu tick xanh (false trong chế độ tự trò chuyện)
      groups: {
        "*": { requireMention: true },
      },
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
    },
  },
  web: {
    enabled: true,
    heartbeatSeconds: 60,
    reconnect: {
      initialMs: 2000,
      maxMs: 120000,
      factor: 1.4,
      jitter: 0.2,
      maxAttempts: 0,
    },
  },
}
```

<Accordion title="Multi-account WhatsApp">

```json5
{
  channels: {
    whatsapp: {
      accounts: {
        default: {},
        personal: {},
        biz: {
          // authDir: "~/.openclaw/credentials/whatsapp/biz",
        },
      },
    },
  },
}
```

- Các lệnh gửi đi mặc định sử dụng tài khoản `default` nếu có; nếu không thì dùng id tài khoản được cấu hình đầu tiên (đã sắp xếp).
- Thư mục xác thực Baileys một tài khoản cũ được `openclaw doctor` di chuyển sang `whatsapp/default`.
- Ghi đè theo từng tài khoản: `channels.whatsapp.accounts.<id>``.sendReadReceipts`, `channels.whatsapp.accounts.<id>``.dmPolicy`, `channels.whatsapp.accounts.<id>``.allowFrom`.

</Accordion>

### Telegram

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "your-bot-token",
      dmPolicy: "pairing",
      allowFrom: ["tg:123456789"],
      groups: {
        "*": { requireMention: true },
        "-1001234567890": {
          allowFrom: ["@admin"],
          systemPrompt: "Giữ câu trả lời ngắn gọn.",
          topics: {
            "99": {
              requireMention: false,
              skills: ["search"],
              systemPrompt: "Giữ đúng chủ đề.",
            },
          },
        },
      },
      customCommands: [
        { command: "backup", description: "Sao lưu Git" },
        { command: "generate", description: "Tạo hình ảnh" },
      ],
      historyLimit: 50,
      replyToMode: "first", // off | first | all
      linkPreview: true,
      streamMode: "partial", // off | partial | block
      draftChunk: {
        minChars: 200,
        maxChars: 800,
        breakPreference: "paragraph", // paragraph | newline | sentence
      },
      actions: { reactions: true, sendMessage: true },
      reactionNotifications: "own", // off | own | all
      mediaMaxMb: 5,
      retry: {
        attempts: 3,
        minDelayMs: 400,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
      network: { autoSelectFamily: false },
      proxy: "socks5://localhost:9050",
      webhookUrl: "https://example.com/telegram-webhook",
      webhookSecret: "secret",
      webhookPath: "/telegram-webhook",
    },
  },
}
```

- Bot token: `channels.telegram.botToken` hoặc `channels.telegram.tokenFile`, với `TELEGRAM_BOT_TOKEN` làm phương án dự phòng cho tài khoản mặc định.
- `configWrites: false` chặn các thay đổi cấu hình được khởi tạo từ Telegram (di chuyển ID supergroup, `/config set|unset`).
- Xem trước khi stream trên Telegram sử dụng `sendMessage` + `editMessageText` (hoạt động trong chat riêng và nhóm).
- Chính sách thử lại: xem [Retry policy](/concepts/retry).

### Discord

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "your-bot-token",
      mediaMaxMb: 8,
      allowBots: false,
      actions: {
        reactions: true,
        stickers: true,
        polls: true,
        permissions: true,
        messages: true,
        threads: true,
        pins: true,
        search: true,
        memberInfo: true,
        roleInfo: true,
        roles: false,
        channelInfo: true,
        voiceStatus: true,
        events: true,
        moderation: false,
      },
      replyToMode: "off", // off | first | all
      dmPolicy: "pairing",
      allowFrom: ["1234567890", "steipete"],
      dm: { enabled: true, groupEnabled: false, groupChannels: ["openclaw-dm"] },
      guilds: {
        "123456789012345678": {
          slug: "friends-of-openclaw",
          requireMention: false,
          reactionNotifications: "own",
          users: ["987654321098765432"],
          channels: {
            general: { allow: true },
            help: {
              allow: true,
              requireMention: true,
              users: ["987654321098765432"],
              skills: ["docs"],
              systemPrompt: "Chỉ trả lời ngắn gọn.",
            },
          },
        },
      },
      historyLimit: 20,
      textChunkLimit: 2000,
      chunkMode: "length", // length | newline
      maxLinesPerMessage: 17,
      ui: {
        components: {
          accentColor: "#5865F2",
        },
      },
      retry: {
        attempts: 3,
        minDelayMs: 500,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
    },
  },
}
```

- Token: `channels.discord.token`, với `DISCORD_BOT_TOKEN` làm phương án dự phòng cho tài khoản mặc định.
- Sử dụng `user:<id>` (DM) hoặc `channel:<id>` (kênh guild) cho đích gửi; ID số thuần sẽ bị từ chối.
- Slug của guild ở dạng chữ thường và thay khoảng trắng bằng `-`; khóa kênh dùng tên đã slug (không có `#`). Ưu tiên sử dụng ID guild.
- Tin nhắn do bot tạo sẽ bị bỏ qua theo mặc định. `allowBots: true` sẽ bật chúng (tin nhắn của chính bot vẫn bị lọc).
- `maxLinesPerMessage` (mặc định 17) sẽ tách các tin nhắn quá dài theo số dòng ngay cả khi chưa vượt quá 2000 ký tự.
- `channels.discord.ui.components.accentColor` đặt màu nhấn cho các container Discord components v2.

**Chế độ thông báo reaction:** `off` (không), `own` (tin nhắn của bot, mặc định), `all` (tất cả tin nhắn), `allowlist` (từ `guilds.<id>``.users` trên tất cả tin nhắn).

### Google Chat

```json5
{
  channels: {
    googlechat: {
      enabled: true,
      serviceAccountFile: "/path/to/service-account.json",
      audienceType: "app-url", // app-url | project-number
      audience: "https://gateway.example.com/googlechat",
      webhookPath: "/googlechat",
      botUser: "users/1234567890",
      dm: {
        enabled: true,
        policy: "pairing",
        allowFrom: ["users/1234567890"],
      },
      groupPolicy: "allowlist",
      groups: {
        "spaces/AAAA": { allow: true, requireMention: true },
      },
      actions: { reactions: true },
      typingIndicator: "message",
      mediaMaxMb: 20,
    },
  },
}
```

- JSON tài khoản dịch vụ: nội tuyến (`serviceAccount`) hoặc dựa trên tệp (`serviceAccountFile`).
- Biến môi trường dự phòng: `GOOGLE_CHAT_SERVICE_ACCOUNT` hoặc `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE`.
- Sử dụng `spaces/<spaceId>` hoặc `users/<userId|email>` làm mục tiêu gửi.

### Slack

```json5
{
  channels: {
    slack: {
      enabled: true,
      botToken: "xoxb-...",
      appToken: "xapp-...",
      dmPolicy: "pairing",
      allowFrom: ["U123", "U456", "*"],
      dm: { enabled: true, groupEnabled: false, groupChannels: ["G123"] },
      channels: {
        C123: { allow: true, requireMention: true, allowBots: false },
        "#general": {
          allow: true,
          requireMention: true,
          allowBots: false,
          users: ["U123"],
          skills: ["docs"],
          systemPrompt: "Short answers only.",
        },
      },
      historyLimit: 50,
      allowBots: false,
      reactionNotifications: "own",
      reactionAllowlist: ["U123"],
      replyToMode: "off", // off | first | all
      thread: {
        historyScope: "thread", // thread | channel
        inheritParent: false,
      },
      actions: {
        reactions: true,
        messages: true,
        pins: true,
        memberInfo: true,
        emojiList: true,
      },
      slashCommand: {
        enabled: true,
        name: "openclaw",
        sessionPrefix: "slack:slash",
        ephemeral: true,
      },
      textChunkLimit: 4000,
      chunkMode: "length",
      mediaMaxMb: 20,
    },
  },
}
```

- **Chế độ socket** yêu cầu cả `botToken` và `appToken` (`SLACK_BOT_TOKEN` + `SLACK_APP_TOKEN` cho biến môi trường tài khoản mặc định).
- **Chế độ HTTP** yêu cầu `botToken` cùng với `signingSecret` (ở cấp root hoặc theo từng tài khoản).
- `configWrites: false` chặn các thao tác ghi cấu hình được khởi tạo từ Slack.
- Sử dụng `user:<id>` (DM) hoặc `channel:<id>` làm mục tiêu gửi.

**Chế độ thông báo reaction:** `off`, `own` (mặc định), `all`, `allowlist` (từ `reactionAllowlist`).

**Cô lập phiên theo thread:** `thread.historyScope` áp dụng cho từng thread (mặc định) hoặc chia sẻ trên toàn kênh. `thread.inheritParent` sao chép bản ghi hội thoại của kênh cha sang các thread mới.

| Nhóm hành động | Mặc định | Ghi chú                          |
| -------------- | -------- | -------------------------------- |
| reactions      | bật      | Thêm reaction + liệt kê reaction |
| messages       | bật      | Đọc/gửi/chỉnh sửa/xóa            |
| pins           | bật      | Ghim/bỏ ghim/liệt kê             |
| memberInfo     | bật      | Thông tin thành viên             |
| emojiList      | bật      | Danh sách emoji tùy chỉnh        |

### Mattermost

Mattermost được cung cấp dưới dạng plugin: `openclaw plugins install @openclaw/mattermost`.

```json5
{
  channels: {
    mattermost: {
      enabled: true,
      botToken: "mm-token",
      baseUrl: "https://chat.example.com",
      dmPolicy: "pairing",
      chatmode: "oncall", // oncall | onmessage | onchar
      oncharPrefixes: [">", "!"],
      textChunkLimit: 4000,
      chunkMode: "length",
    },
  },
}
```

Chế độ chat: `oncall` (phản hồi khi được @-mention, mặc định), `onmessage` (mọi tin nhắn), `onchar` (tin nhắn bắt đầu bằng tiền tố kích hoạt).

### Signal

```json5
{
  channels: {
    signal: {
      reactionNotifications: "own", // off | own | all | allowlist
      reactionAllowlist: ["+15551234567", "uuid:123e4567-e89b-12d3-a456-426614174000"],
      historyLimit: 50,
    },
  },
}
```

**Chế độ thông báo reaction:** `off`, `own` (mặc định), `all`, `allowlist` (từ `reactionAllowlist`).

### iMessage

OpenClaw khởi chạy `imsg rpc` (JSON-RPC qua stdio). Không yêu cầu daemon hoặc cổng.

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "imsg",
      dbPath: "~/Library/Messages/chat.db",
      remoteHost: "user@gateway-host",
      dmPolicy: "pairing",
      allowFrom: ["+15555550123", "user@example.com", "chat_id:123"],
      historyLimit: 50,
      includeAttachments: false,
      mediaMaxMb: 16,
      service: "auto",
      region: "US",
    },
  },
}
```

- Yêu cầu cấp quyền Full Disk Access cho cơ sở dữ liệu Messages.
- Ưu tiên sử dụng mục tiêu `chat_id:<id>`. Sử dụng `imsg chats --limit 20` để liệt kê các cuộc trò chuyện.
- `cliPath` có thể trỏ tới một SSH wrapper; thiết lập `remoteHost` để tải tệp đính kèm qua SCP.

<Accordion title="iMessage SSH wrapper example">

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

</Accordion>

### Đa tài khoản (tất cả các kênh)

Chạy nhiều tài khoản cho mỗi kênh (mỗi tài khoản có `accountId` riêng):

```json5
{
  channels: {
    telegram: {
      accounts: {
        default: {
          name: "Primary bot",
          botToken: "123456:ABC...",
        },
        alerts: {
          name: "Alerts bot",
          botToken: "987654:XYZ...",
        },
      },
    },
  },
}
```

- `default` được sử dụng khi `accountId` bị bỏ qua (CLI + routing).
- Token môi trường (env) chỉ áp dụng cho tài khoản **default**.
- Cấu hình kênh cơ bản áp dụng cho tất cả tài khoản trừ khi được ghi đè riêng cho từng tài khoản.
- Sử dụng `bindings[].match.accountId` để định tuyến mỗi tài khoản tới một agent khác nhau.

### Kiểm soát đề cập trong chat nhóm

Tin nhắn nhóm mặc định là **yêu cầu đề cập** (đề cập trong metadata hoặc theo mẫu regex). Áp dụng cho các nhóm WhatsApp, Telegram, Discord, Google Chat và iMessage.

**Các loại đề cập:**

- **Đề cập trong metadata**: @-mention gốc của nền tảng. Bị bỏ qua trong chế độ tự chat của WhatsApp.
- **Mẫu văn bản**: Các mẫu regex trong `agents.list[].groupChat.mentionPatterns`. Luôn được kiểm tra.
- Kiểm soát đề cập chỉ được thực thi khi có thể phát hiện (đề cập gốc hoặc có ít nhất một mẫu).

```json5
{
  messages: {
    groupChat: { historyLimit: 50 },
  },
  agents: {
    list: [{ id: "main", groupChat: { mentionPatterns: ["@openclaw", "openclaw"] } }],
  },
}
```

`messages.groupChat.historyLimit` thiết lập giá trị mặc định toàn cục. Các kênh có thể ghi đè bằng `channels.<channel>``.historyLimit` (hoặc theo từng tài khoản). Đặt `0` để vô hiệu hóa.

#### Giới hạn lịch sử DM

```json5
{
  channels: {
    telegram: {
      dmHistoryLimit: 30,
      dms: {
        "123456789": { historyLimit: 50 },
      },
    },
  },
}
```

Thứ tự áp dụng: ghi đè theo từng DM → mặc định của nhà cung cấp → không giới hạn (giữ lại toàn bộ).

Hỗ trợ: `telegram`, `whatsapp`, `discord`, `slack`, `signal`, `imessage`, `msteams`.

#### Chế độ tự chat

Thêm số của bạn vào `allowFrom` để bật chế độ tự chat (bỏ qua @-mention gốc, chỉ phản hồi theo mẫu văn bản):

```json5
{
  channels: {
    whatsapp: {
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } },
    },
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: { mentionPatterns: ["reisponde", "@openclaw"] },
      },
    ],
  },
}
```

### Lệnh (xử lý lệnh trong chat)

```json5
{
  commands: {
    native: "auto", // register native commands when supported
    text: true, // parse /commands in chat messages
    bash: false, // allow ! (alias: /bash)
    bashForegroundMs: 2000,
    config: false, // allow /config
    debug: false, // allow /debug
    restart: false, // allow /restart + gateway restart tool
    allowFrom: {
      "*": ["user1"],
      discord: ["user:123"],
    },
    useAccessGroups: true,
  },
}
```

<Accordion title="Command details">

- Lệnh văn bản phải là các tin nhắn **độc lập** bắt đầu bằng `/`.
- `native: "auto"` bật lệnh gốc cho Discord/Telegram, giữ Slack ở trạng thái tắt.
- Ghi đè theo từng kênh: `channels.discord.commands.native` (bool hoặc `"auto"`). `false` sẽ xóa các lệnh đã được đăng ký trước đó.
- `channels.telegram.customCommands` thêm các mục menu bổ sung cho bot Telegram.
- `bash: true` bật `! <cmd>` cho shell của máy chủ. Yêu cầu `tools.elevated.enabled` và người gửi nằm trong `tools.elevated.allowFrom.<channel>`.
- `config: true` bật `/config` (đọc/ghi `openclaw.json`).
- `channels.<provider>`.configWrites\` kiểm soát quyền thay đổi cấu hình theo từng kênh (mặc định: true).
- `allowFrom` được cấu hình riêng cho từng provider. Khi được đặt, đây là nguồn phân quyền **duy nhất** (danh sách cho phép của kênh/ghép nối và `useAccessGroups` sẽ bị bỏ qua).
- `useAccessGroups: false` cho phép các lệnh bỏ qua chính sách access-group khi `allowFrom` chưa được đặt.

</Accordion>

---

## Giá trị mặc định của agent

### `agents.defaults.workspace`

Mặc định: `~/.openclaw/workspace`.

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
}
```

### `agents.defaults.repoRoot`

Thư mục gốc repository tùy chọn được hiển thị trong dòng Runtime của system prompt. Nếu không được đặt, OpenClaw sẽ tự động phát hiện bằng cách duyệt ngược lên từ workspace.

```json5
{
  agents: { defaults: { repoRoot: "~/Projects/openclaw" } },
}
```

### `agents.defaults.skipBootstrap`

Vô hiệu hóa việc tự động tạo các tệp bootstrap của workspace (`AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, `BOOTSTRAP.md`).

```json5
{
  agents: { defaults: { skipBootstrap: true } },
}
```

### `agents.defaults.bootstrapMaxChars`

Số ký tự tối đa cho mỗi tệp bootstrap của workspace trước khi bị cắt bớt. Mặc định: `20000`.

```json5
{
  agents: { defaults: { bootstrapMaxChars: 20000 } },
}
```

### `agents.defaults.bootstrapTotalMaxChars`

Tổng số ký tự tối đa được chèn từ tất cả các tệp bootstrap của workspace. Mặc định: `24000`.

```json5
{
  agents: { defaults: { bootstrapTotalMaxChars: 24000 } },
}
```

### `agents.defaults.userTimezone`

Múi giờ dùng cho ngữ cảnh system prompt (không phải dấu thời gian của tin nhắn). Nếu không có, sẽ dùng múi giờ của máy chủ.

```json5
{
  agents: { defaults: { userTimezone: "America/Chicago" } },
}
```

### `agents.defaults.timeFormat`

Định dạng thời gian trong system prompt. Mặc định: `auto` (theo tùy chọn của hệ điều hành).

```json5
{
  agents: { defaults: { timeFormat: "auto" } }, // auto | 12 | 24
}
```

### `agents.defaults.model`

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": { alias: "opus" },
        "minimax/MiniMax-M2.1": { alias: "minimax" },
      },
      model: {
        primary: "anthropic/claude-opus-4-6",
        fallbacks: ["minimax/MiniMax-M2.1"],
      },
      imageModel: {
        primary: "openrouter/qwen/qwen-2.5-vl-72b-instruct:free",
        fallbacks: ["openrouter/google/gemini-2.0-flash-vision:free"],
      },
      thinkingDefault: "low",
      verboseDefault: "off",
      elevatedDefault: "on",
      timeoutSeconds: 600,
      mediaMaxMb: 5,
      contextTokens: 200000,
      maxConcurrent: 3,
    },
  },
}
```

- `model.primary`: định dạng `provider/model` (ví dụ: `anthropic/claude-opus-4-6`). Nếu bạn bỏ qua provider, OpenClaw sẽ mặc định là `anthropic` (không còn được khuyến nghị).
- `models`: danh mục model và allowlist được cấu hình cho `/model`. Mỗi mục có thể bao gồm `alias` (bí danh) và `params` (theo từng provider: `temperature`, `maxTokens`).
- `imageModel`: chỉ được sử dụng nếu model chính không hỗ trợ đầu vào hình ảnh.
- `maxConcurrent`: số lượt chạy agent song song tối đa trên các session (mỗi session vẫn được xử lý tuần tự). Mặc định: 1.

**Bí danh rút gọn tích hợp sẵn** (chỉ áp dụng khi model nằm trong `agents.defaults.models`):

| Bí danh        | Model                           |
| -------------- | ------------------------------- |
| `opus`         | `anthropic/claude-opus-4-6`     |
| `sonnet`       | `anthropic/claude-sonnet-4-5`   |
| `gpt`          | `openai/gpt-5.2`                |
| `gpt-mini`     | `openai/gpt-5-mini`             |
| `gemini`       | `google/gemini-3-pro-preview`   |
| `gemini-flash` | `google/gemini-3-flash-preview` |

Các bí danh bạn cấu hình luôn được ưu tiên hơn mặc định.

Các model Z.AI GLM-4.x tự động bật chế độ thinking trừ khi bạn đặt `--thinking off` hoặc tự định nghĩa `agents.defaults.models["zai/<model>"].params.thinking`.

### `agents.defaults.cliBackends`

Các CLI backend tùy chọn cho các lượt chạy dự phòng chỉ văn bản (không gọi tool). Hữu ích làm phương án dự phòng khi các API provider gặp sự cố.

```json5
{
  agents: {
    defaults: {
      cliBackends: {
        "claude-cli": {
          command: "/opt/homebrew/bin/claude",
        },
        "my-cli": {
          command: "my-cli",
          args: ["--json"],
          output: "json",
          modelArg: "--model",
          sessionArg: "--session",
          sessionMode: "existing",
          systemPromptArg: "--system",
          systemPromptWhen: "first",
          imageArg: "--image",
          imageMode: "repeat",
        },
      },
    },
  },
}
```

- CLI backend ưu tiên văn bản; các tool luôn bị vô hiệu hóa.
- Session được hỗ trợ khi `sessionArg` được thiết lập.
- Hỗ trợ truyền hình ảnh khi `imageArg` chấp nhận đường dẫn tệp.

### `agents.defaults.heartbeat`

Chạy heartbeat định kỳ.

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m", // 0m để tắt
        model: "openai/gpt-5.2-mini",
        includeReasoning: false,
        session: "main",
        to: "+15555550123",
        target: "last", // last | whatsapp | telegram | discord | ... | none
        prompt: "Đọc HEARTBEAT.md nếu tồn tại...",
        ackMaxChars: 300,
      },
    },
  },
}
```

- `every`: chuỗi thời lượng (ms/s/m/h). Mặc định: `30m`.
- Theo từng agent: đặt `agents.list[].heartbeat`. Khi bất kỳ agent nào định nghĩa `heartbeat`, **chỉ những agent đó** sẽ chạy heartbeat.
- Heartbeat chạy toàn bộ lượt của agent — khoảng thời gian ngắn hơn sẽ tiêu tốn nhiều token hơn.

### `agents.defaults.compaction`

```json5
{
  agents: {
    defaults: {
      compaction: {
        mode: "safeguard", // default | safeguard
        reserveTokensFloor: 24000,
        memoryFlush: {
          enabled: true,
          softThresholdTokens: 6000,
          systemPrompt: "Session nearing compaction. Store durable memories now.",
          prompt: "Write any lasting notes to memory/YYYY-MM-DD.md; reply with NO_REPLY if nothing to store.",
        },
      },
    },
  },
}
```

- `mode`: `default` hoặc `safeguard` (tóm tắt theo từng phần cho lịch sử dài). Xem [Compaction](/concepts/compaction).
- `memoryFlush`: lượt chạy agent im lặng trước khi tự động compaction để lưu trữ các ký ức lâu dài. Bị bỏ qua khi workspace ở chế độ chỉ đọc.

### `agents.defaults.contextPruning`

Loại bỏ **kết quả tool cũ** khỏi ngữ cảnh trong bộ nhớ trước khi gửi đến LLM. **Không** chỉnh sửa lịch sử phiên được lưu trên đĩa.

```json5
{
  agents: {
    defaults: {
      contextPruning: {
        mode: "cache-ttl", // off | cache-ttl
        ttl: "1h", // duration (ms/s/m/h), default unit: minutes
        keepLastAssistants: 3,
        softTrimRatio: 0.3,
        hardClearRatio: 0.5,
        minPrunableToolChars: 50000,
        softTrim: { maxChars: 4000, headChars: 1500, tailChars: 1500 },
        hardClear: { enabled: true, placeholder: "[Old tool result content cleared]" },
        tools: { deny: ["browser", "canvas"] },
      },
    },
  },
}
```

<Accordion title="cache-ttl mode behavior">

- `mode: "cache-ttl"` bật các lần chạy cắt tỉa.
- `ttl` kiểm soát khoảng thời gian tối thiểu trước khi có thể chạy cắt tỉa lại (sau lần cache chạm cuối cùng).
- Quá trình cắt tỉa sẽ soft-trim các kết quả tool quá lớn trước, sau đó hard-clear các kết quả tool cũ hơn nếu cần.

**Soft-trim** giữ lại phần đầu + phần cuối và chèn `...` vào giữa.

**Hard-clear** thay thế toàn bộ kết quả tool bằng placeholder.

Lưu ý:

- Các khối hình ảnh không bao giờ bị trim/clear.
- Các tỷ lệ được tính theo số ký tự (xấp xỉ), không phải số token chính xác.
- Nếu có ít hơn `keepLastAssistants` thông điệp assistant, quá trình cắt tỉa sẽ bị bỏ qua.

</Accordion>

Xem [Session Pruning](/concepts/session-pruning) để biết chi tiết hành vi.

### Chặn streaming

```json5
{
  agents: {
    defaults: {
      blockStreamingDefault: "off", // on | off
      blockStreamingBreak: "text_end", // text_end | message_end
      blockStreamingChunk: { minChars: 800, maxChars: 1200 },
      blockStreamingCoalesce: { idleMs: 1000 },
      humanDelay: { mode: "natural" }, // off | natural | custom (use minMs/maxMs)
    },
  },
}
```

- Các kênh không phải Telegram yêu cầu cấu hình `*.blockStreaming: true` rõ ràng để bật phản hồi theo khối.
- Ghi đè theo kênh: `channels.<channel>`.`blockStreamingCoalesce` (và các biến thể theo từng tài khoản). Signal/Slack/Discord/Google Chat mặc định `minChars: 1500`.
- `humanDelay`: khoảng dừng ngẫu nhiên giữa các phản hồi theo khối. `natural` = 800–2500ms. Ghi đè theo agent: `agents.list[].humanDelay`.

Xem [Streaming](/concepts/streaming) để biết chi tiết về hành vi + cách chia khối.

### Chỉ báo đang nhập

```json5
{
  agents: {
    defaults: {
      typingMode: "instant", // never | instant | thinking | message
      typingIntervalSeconds: 6,
    },
  },
}
```

- Mặc định: `instant` cho chat trực tiếp/được nhắc đến, `message` cho nhóm chat không được nhắc đến.
- Ghi đè theo phiên: `session.typingMode`, `session.typingIntervalSeconds`.

Xem [Typing Indicators](/concepts/typing-indicators).

### `agents.defaults.sandbox`

**Docker sandboxing** tùy chọn cho agent nhúng. Xem [Sandboxing](/gateway/sandboxing) để biết hướng dẫn đầy đủ.

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main", // off | non-main | all
        scope: "agent", // session | agent | shared
        workspaceAccess: "none", // none | ro | rw
        workspaceRoot: "~/.openclaw/sandboxes",
        docker: {
          image: "openclaw-sandbox:bookworm-slim",
          containerPrefix: "openclaw-sbx-",
          workdir: "/workspace",
          readOnlyRoot: true,
          tmpfs: ["/tmp", "/var/tmp", "/run"],
          network: "none",
          user: "1000:1000",
          capDrop: ["ALL"],
          env: { LANG: "C.UTF-8" },
          setupCommand: "apt-get update && apt-get install -y git curl jq",
          pidsLimit: 256,
          memory: "1g",
          memorySwap: "2g",
          cpus: 1,
          ulimits: {
            nofile: { soft: 1024, hard: 2048 },
            nproc: 256,
          },
          seccompProfile: "/path/to/seccomp.json",
          apparmorProfile: "openclaw-sandbox",
          dns: ["1.1.1.1", "8.8.8.8"],
          extraHosts: ["internal.service:10.0.0.5"],
          binds: ["/home/user/source:/source:rw"],
        },
        browser: {
          enabled: false,
          image: "openclaw-sandbox-browser:bookworm-slim",
          cdpPort: 9222,
          vncPort: 5900,
          noVncPort: 6080,
          headless: false,
          enableNoVnc: true,
          allowHostControl: false,
          autoStart: true,
          autoStartTimeoutMs: 12000,
        },
        prune: {
          idleHours: 24,
          maxAgeDays: 7,
        },
      },
    },
  },
  tools: {
    sandbox: {
      tools: {
        allow: [
          "exec",
          "process",
          "read",
          "write",
          "edit",
          "apply_patch",
          "sessions_list",
          "sessions_history",
          "sessions_send",
          "sessions_spawn",
          "session_status",
        ],
        deny: ["browser", "canvas", "nodes", "cron", "discord", "gateway"],
      },
    },
  },
}
```

<Accordion title="Sandbox details">

**Quyền truy cập workspace:**

- `none`: workspace sandbox theo từng scope dưới `~/.openclaw/sandboxes`
- `ro`: workspace sandbox tại `/workspace`, workspace của agent được mount chỉ đọc tại `/agent`
- `rw`: workspace của agent được mount đọc/ghi tại `/workspace`

**Phạm vi:**

- `session`: container + workspace cho mỗi phiên
- `agent`: một container + workspace cho mỗi agent (mặc định)
- `shared`: container và workspace dùng chung (không cô lập giữa các phiên)

**`setupCommand`** chạy một lần sau khi tạo container (qua `sh -lc`). Yêu cầu network egress, root có thể ghi, người dùng root.

**Container mặc định `network: "none"`** — đặt thành `"bridge"` nếu agent cần truy cập outbound.

**Inbound attachments** được đưa vào `media/inbound/*` trong workspace đang hoạt động.

**`docker.binds`** mount thêm các thư mục host; các bind toàn cục và theo từng agent sẽ được hợp nhất.

**Trình duyệt sandbox** (`sandbox.browser.enabled`): Chromium + CDP trong container. URL noVNC được chèn vào system prompt. Không yêu cầu `browser.enabled` trong cấu hình chính.

- `allowHostControl: false` (mặc định) chặn các phiên sandbox nhắm tới trình duyệt trên host.
- `sandbox.browser.binds` mount thêm các thư mục host chỉ vào container trình duyệt sandbox. Khi được thiết lập (bao gồm cả `[]`), nó sẽ thay thế `docker.binds` cho container trình duyệt.

</Accordion>

Build image:

```bash
scripts/sandbox-setup.sh           # image sandbox chính
scripts/sandbox-browser-setup.sh   # image trình duyệt tùy chọn
```

### `agents.list` (ghi đè theo từng agent)

```json5
{
  agents: {
    list: [
      {
        id: "main",
        default: true,
        name: "Main Agent",
        workspace: "~/.openclaw/workspace",
        agentDir: "~/.openclaw/agents/main/agent",
        model: "anthropic/claude-opus-4-6", // hoặc { primary, fallbacks }
        identity: {
          name: "Samantha",
          theme: "helpful sloth",
          emoji: "🦥",
          avatar: "avatars/samantha.png",
        },
        groupChat: { mentionPatterns: ["@openclaw"] },
        sandbox: { mode: "off" },
        subagents: { allowAgents: ["*"] },
        tools: {
          profile: "coding",
          allow: ["browser"],
          deny: ["canvas"],
          elevated: { enabled: true },
        },
      },
    ],
  },
}
```

- `id`: id agent ổn định (bắt buộc).
- `default`: khi có nhiều mục được đặt, mục đầu tiên sẽ được chọn (ghi log cảnh báo). Nếu không mục nào được đặt, phần tử đầu tiên trong danh sách sẽ là mặc định.
- `model`: dạng chuỗi chỉ ghi đè `primary`; dạng object `{ primary, fallbacks }` ghi đè cả hai (`[]` sẽ vô hiệu hóa fallbacks toàn cục).
- `identity.avatar`: đường dẫn tương đối với workspace, URL `http(s)`, hoặc URI `data:`.
- `identity` suy ra giá trị mặc định: `ackReaction` từ `emoji`, `mentionPatterns` từ `name`/`emoji`.
- `subagents.allowAgents`: danh sách cho phép id agent cho `sessions_spawn` (`["*"]` = bất kỳ; mặc định: chỉ cùng agent).

---

## Định tuyến multi-agent

Chạy nhiều agent cách ly trong một Gateway. Xem [Multi-Agent](/concepts/multi-agent).

```json5
{
  agents: {
    list: [
      { id: "home", default: true, workspace: "~/.openclaw/workspace-home" },
      { id: "work", workspace: "~/.openclaw/workspace-work" },
    ],
  },
  bindings: [
    { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
    { agentId: "work", match: { channel: "whatsapp", accountId: "biz" } },
  ],
}
```

### Các trường khớp Binding

- `match.channel` (bắt buộc)
- `match.accountId` (tùy chọn; `*` = bất kỳ tài khoản nào; bỏ qua = tài khoản mặc định)
- `match.peer` (tùy chọn; `{ kind: direct|group|channel, id }`)
- `match.guildId` / `match.teamId` (tùy chọn; theo từng kênh)

**Thứ tự khớp xác định (deterministic):**

1. `match.peer`
2. `match.guildId`
3. `match.teamId`
4. `match.accountId` (chính xác, không phải peer/guild/team)
5. `match.accountId: "*"` (toàn kênh)
6. Agent mặc định

Trong mỗi cấp, mục `bindings` khớp đầu tiên sẽ được áp dụng.

### Hồ sơ truy cập theo từng agent

<Accordion title="Full access (no sandbox)">

```json5
{
  agents: {
    list: [
      {
        id: "personal",
        workspace: "~/.openclaw/workspace-personal",
        sandbox: { mode: "off" },
      },
    ],
  },
}
```

</Accordion>

<Accordion title="Read-only tools + workspace">

```json5
{
  agents: {
    list: [
      {
        id: "family",
        workspace: "~/.openclaw/workspace-family",
        sandbox: { mode: "all", scope: "agent", workspaceAccess: "ro" },
        tools: {
          allow: [
            "read",
            "sessions_list",
            "sessions_history",
            "sessions_send",
            "sessions_spawn",
            "session_status",
          ],
          deny: ["write", "edit", "apply_patch", "exec", "process", "browser"],
        },
      },
    ],
  },
}
```

</Accordion>

<Accordion title="No filesystem access (messaging only)">

```json5
{
  agents: {
    list: [
      {
        id: "public",
        workspace: "~/.openclaw/workspace-public",
        sandbox: { mode: "all", scope: "agent", workspaceAccess: "none" },
        tools: {
          allow: [
            "sessions_list",
            "sessions_history",
            "sessions_send",
            "sessions_spawn",
            "session_status",
            "whatsapp",
            "telegram",
            "slack",
            "discord",
            "gateway",
          ],
          deny: [
            "read",
            "write",
            "edit",
            "apply_patch",
            "exec",
            "process",
            "browser",
            "canvas",
            "nodes",
            "cron",
            "gateway",
            "image",
          ],
        },
      },
    ],
  },
}
```

</Accordion>

Xem [Multi-Agent Sandbox & Tools](/tools/multi-agent-sandbox-tools) để biết chi tiết về thứ tự ưu tiên.

---

## Phiên

```json5
{
  session: {
    scope: "per-sender",
    dmScope: "main", // main | per-peer | per-channel-peer | per-account-channel-peer
    identityLinks: {
      alice: ["telegram:123456789", "discord:987654321012345678"],
    },
    reset: {
      mode: "daily", // daily | idle
      atHour: 4,
      idleMinutes: 60,
    },
    resetByType: {
      thread: { mode: "daily", atHour: 4 },
      direct: { mode: "idle", idleMinutes: 240 },
      group: { mode: "idle", idleMinutes: 120 },
    },
    resetTriggers: ["/new", "/reset"],
    store: "~/.openclaw/agents/{agentId}/sessions/sessions.json",
    maintenance: {
      mode: "warn", // warn | enforce
      pruneAfter: "30d",
      maxEntries: 500,
      rotateBytes: "10mb",
    },
    mainKey: "main", // legacy (runtime luôn sử dụng "main")
    agentToAgent: { maxPingPongTurns: 5 },
    sendPolicy: {
      rules: [{ action: "deny", match: { channel: "discord", chatType: "group" } }],
      default: "allow",
    },
  },
}
```

<Accordion title="Session field details">

- **`dmScope`**: cách nhóm các tin nhắn DM.
  - `main`: tất cả DM dùng chung phiên chính.
  - `per-peer`: tách riêng theo id người gửi trên các kênh.
  - `per-channel-peer`: tách riêng theo từng kênh + người gửi (khuyến nghị cho inbox nhiều người dùng).
  - `per-account-channel-peer`: tách riêng theo từng tài khoản + kênh + người gửi (khuyến nghị cho nhiều tài khoản).
- **`identityLinks`**: ánh xạ id chuẩn với các peer có tiền tố nhà cung cấp để chia sẻ phiên giữa các kênh.
- **`reset`**: chính sách đặt lại chính. `daily` đặt lại tại `atHour` theo giờ địa phương; `idle` đặt lại sau `idleMinutes`. Khi cả hai cùng được cấu hình, điều kiện hết hạn trước sẽ được áp dụng.
- **`resetByType`**: ghi đè theo từng loại (`direct`, `group`, `thread`). Chấp nhận `dm` cũ như bí danh của `direct`.
- **`mainKey`**: trường cũ. Runtime hiện luôn sử dụng `"main"` cho nhóm chat trực tiếp chính.
- **`sendPolicy`**: khớp theo `channel`, `chatType` (`direct|group|channel`, với bí danh cũ `dm`), `keyPrefix`, hoặc `rawKeyPrefix`. Luật deny khớp đầu tiên sẽ được áp dụng.
- **`maintenance`**: `warn` cảnh báo phiên đang hoạt động khi bị loại bỏ; `enforce` áp dụng việc dọn dẹp và xoay vòng.

</Accordion>

---

## Tin nhắn

```json5
{
  messages: {
    responsePrefix: "🦞", // hoặc "auto"
    ackReaction: "👀",
    ackReactionScope: "group-mentions", // group-mentions | group-all | direct | all
    removeAckAfterReply: false,
    queue: {
      mode: "collect", // steer | followup | collect | steer-backlog | steer+backlog | queue | interrupt
      debounceMs: 1000,
      cap: 20,
      drop: "summarize", // old | new | summarize
      byChannel: {
        whatsapp: "collect",
        telegram: "collect",
      },
    },
    inbound: {
      debounceMs: 2000, // 0 để tắt
      byChannel: {
        whatsapp: 5000,
        slack: 1500,
      },
    },
  },
}
```

### Tiền tố phản hồi

Ghi đè theo từng kênh/tài khoản: `channels.<channel>.responsePrefix`, `channels.<channel>.accounts.<id>.responsePrefix`.

Thứ tự phân giải (cụ thể nhất được ưu tiên): tài khoản → kênh → toàn cục. `""` sẽ vô hiệu hóa và dừng kế thừa. `"auto"` sẽ suy ra `[{identity.name}]`.

**Biến mẫu:**

| Biến              | Mô tả                    | Ví dụ                                   |
| ----------------- | ------------------------ | --------------------------------------- |
| `{model}`         | Tên model ngắn           | `claude-opus-4-6`                       |
| `{modelFull}`     | Định danh model đầy đủ   | `anthropic/claude-opus-4-6`             |
| `{provider}`      | Tên nhà cung cấp         | `anthropic`                             |
| `{thinkingLevel}` | Mức độ suy nghĩ hiện tại | `high`, `low`, `off`                    |
| `{identity.name}` | Tên định danh của agent  | (giống như `"auto"`) |

Biến không phân biệt chữ hoa chữ thường. `{think}` là bí danh của `{thinkingLevel}`.

### Phản ứng xác nhận (ack)

- Mặc định là `identity.emoji` của agent đang hoạt động, nếu không thì là `"👀"`. Đặt `""` để tắt.
- Ghi đè theo từng kênh: `channels.<channel>`.ackReaction`, `channels.<channel>`.accounts.<id>`.ackReaction\`.
- Thứ tự phân giải: account → channel → `messages.ackReaction` → fallback theo identity.
- Phạm vi: `group-mentions` (mặc định), `group-all`, `direct`, `all`.
- `removeAckAfterReply`: xóa ack sau khi trả lời (chỉ áp dụng cho Slack/Discord/Telegram/Google Chat).

### Gộp tin nhắn đầu vào (debounce)

Gộp các tin nhắn chỉ có văn bản gửi nhanh liên tiếp từ cùng một người gửi thành một lượt xử lý của agent. Media/tệp đính kèm được xử lý ngay lập tức. Các lệnh điều khiển bỏ qua cơ chế debounce.

### TTS (chuyển văn bản thành giọng nói)

```json5
{
  messages: {
    tts: {
      auto: "always", // off | always | inbound | tagged
      mode: "final", // final | all
      provider: "elevenlabs",
      summaryModel: "openai/gpt-4.1-mini",
      modelOverrides: { enabled: true },
      maxTextLength: 4000,
      timeoutMs: 30000,
      prefsPath: "~/.openclaw/settings/tts.json",
      elevenlabs: {
        apiKey: "elevenlabs_api_key",
        baseUrl: "https://api.elevenlabs.io",
        voiceId: "voice_id",
        modelId: "eleven_multilingual_v2",
        seed: 42,
        applyTextNormalization: "auto",
        languageCode: "en",
        voiceSettings: {
          stability: 0.5,
          similarityBoost: 0.75,
          style: 0.0,
          useSpeakerBoost: true,
          speed: 1.0,
        },
      },
      openai: {
        apiKey: "openai_api_key",
        model: "gpt-4o-mini-tts",
        voice: "alloy",
      },
    },
  },
}
```

- `auto` điều khiển chế độ tự động TTS. `/tts off|always|inbound|tagged` ghi đè theo từng phiên.
- `summaryModel` ghi đè `agents.defaults.model.primary` cho việc tự động tóm tắt.
- API key sẽ sử dụng giá trị dự phòng từ `ELEVENLABS_API_KEY`/`XI_API_KEY` và `OPENAI_API_KEY`.

---

## Trò chuyện

Mặc định cho chế độ Talk (macOS/iOS/Android).

```json5
{
  talk: {
    voiceId: "elevenlabs_voice_id",
    voiceAliases: {
      Clawd: "EXAVITQu4vr4xnSDxMaL",
      Roger: "CwhRBWXzGAHq8TQ4Fs17",
    },
    modelId: "eleven_v3",
    outputFormat: "mp3_44100_128",
    apiKey: "elevenlabs_api_key",
    interruptOnSpeech: true,
  },
}
```

- Voice ID sẽ mặc định sử dụng `ELEVENLABS_VOICE_ID` hoặc `SAG_VOICE_ID`.
- `apiKey` sẽ mặc định sử dụng `ELEVENLABS_API_KEY`.
- `voiceAliases` cho phép các chỉ thị Talk sử dụng tên thân thiện.

---

## Công cụ

### Hồ sơ công cụ

`tools.profile` thiết lập danh sách cho phép cơ bản trước `tools.allow`/`tools.deny`:

| Hồ sơ       | Bao gồm                                                                                   |
| ----------- | ----------------------------------------------------------------------------------------- |
| `minimal`   | Chỉ `session_status`                                                                      |
| `coding`    | `group:fs`, `group:runtime`, `group:sessions`, `group:memory`, `image`                    |
| `messaging` | `group:messaging`, `sessions_list`, `sessions_history`, `sessions_send`, `session_status` |
| `full`      | Không giới hạn (giống như không thiết lập)                             |

### Nhóm công cụ

| Nhóm               | Công cụ                                                                                  |
| ------------------ | ---------------------------------------------------------------------------------------- |
| `group:runtime`    | `exec`, `process` (`bash` được chấp nhận như bí danh của `exec`)      |
| `group:fs`         | `read`, `write`, `edit`, `apply_patch`                                                   |
| `group:sessions`   | `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status` |
| `group:memory`     | `memory_search`, `memory_get`                                                            |
| `group:web`        | `web_search`, `web_fetch`                                                                |
| `group:ui`         | `browser`, `canvas`                                                                      |
| `group:automation` | `cron`, `gateway`                                                                        |
| `group:messaging`  | `message`                                                                                |
| `group:nodes`      | `nodes`                                                                                  |
| `group:openclaw`   | Tất cả công cụ tích hợp sẵn (không bao gồm plugin của provider)       |

### `tools.allow` / `tools.deny`

Chính sách cho phép/từ chối công cụ toàn cục (deny được ưu tiên). Không phân biệt hoa thường, hỗ trợ ký tự đại diện `*`. Được áp dụng ngay cả khi Docker sandbox bị tắt.

```json5
{
  tools: { deny: ["browser", "canvas"] },
}
```

### `tools.byProvider`

Giới hạn thêm công cụ cho các provider hoặc model cụ thể. Thứ tự: profile cơ sở → profile provider → allow/deny.

```json5
{
  tools: {
    profile: "coding",
    byProvider: {
      "google-antigravity": { profile: "minimal" },
      "openai/gpt-5.2": { allow: ["group:fs", "sessions_list"] },
    },
  },
}
```

### `tools.elevated`

Kiểm soát quyền thực thi (host) nâng cao:

```json5
{
  tools: {
    elevated: {
      enabled: true,
      allowFrom: {
        whatsapp: ["+15555550123"],
        discord: ["steipete", "1234567890123"],
      },
    },
  },
}
```

- Ghi đè theo từng agent (`agents.list[].tools.elevated`) chỉ có thể hạn chế thêm.
- `/elevated on|off|ask|full` lưu trạng thái theo từng phiên; chỉ thị nội tuyến áp dụng cho một tin nhắn duy nhất.
- `exec` nâng cao chạy trên host, bỏ qua cơ chế sandbox.

### `tools.exec`

```json5
{
  tools: {
    exec: {
      backgroundMs: 10000,
      timeoutSec: 1800,
      cleanupMs: 1800000,
      notifyOnExit: true,
      notifyOnExitEmptySuccess: false,
      applyPatch: {
        enabled: false,
        allowModels: ["gpt-5.2"],
      },
    },
  },
}
```

### `tools.web`

```json5
{
  tools: {
    web: {
      search: {
        enabled: true,
        apiKey: "brave_api_key", // hoặc biến môi trường BRAVE_API_KEY
        maxResults: 5,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15,
      },
      fetch: {
        enabled: true,
        maxChars: 50000,
        maxCharsCap: 50000,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15,
        userAgent: "custom-ua",
      },
    },
  },
}
```

### `tools.media`

Cấu hình xử lý media đầu vào (hình ảnh/âm thanh/video):

```json5
{
  tools: {
    media: {
      concurrency: 2,
      audio: {
        enabled: true,
        maxBytes: 20971520,
        scope: {
          default: "deny",
          rules: [{ action: "allow", match: { chatType: "direct" } }],
        },
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          { type: "cli", command: "whisper", args: ["--model", "base", "{{MediaPath}}"] },
        ],
      },
      video: {
        enabled: true,
        maxBytes: 52428800,
        models: [{ provider: "google", model: "gemini-3-flash-preview" }],
      },
    },
  },
}
```

<Accordion title="Media model entry fields">

**Mục provider** (`type: "provider"` hoặc bỏ qua):

- `provider`: id provider API (`openai`, `anthropic`, `google`/`gemini`, `groq`, v.v.)
- `model`: ghi đè id model
- `profile` / `preferredProfile`: chọn profile xác thực

**Mục CLI** (`type: "cli"`):

- `command`: tệp thực thi cần chạy
- `args`: tham số dạng mẫu (hỗ trợ `{{MediaPath}}`, `{{Prompt}}`, `{{MaxChars}}`, v.v.)

**Trường chung:**

- `capabilities`: danh sách tùy chọn (`image`, `audio`, `video`). Mặc định: `openai`/`anthropic`/`minimax` → image, `google` → image+audio+video, `groq` → audio.
- `prompt`, `maxChars`, `maxBytes`, `timeoutSeconds`, `language`: ghi đè theo từng mục.
- Nếu lỗi sẽ chuyển sang mục tiếp theo.

Xác thực provider tuân theo thứ tự chuẩn: profile xác thực → biến môi trường → `models.providers.*.apiKey`.

</Accordion>

### `tools.agentToAgent`

```json5
{
  tools: {
    agentToAgent: {
      enabled: false,
      allow: ["home", "work"],
    },
  },
}
```

### `tools.subagents`

```json5
{
  agents: {
    defaults: {
      subagents: {
        model: "minimax/MiniMax-M2.1",
        maxConcurrent: 1,
        archiveAfterMinutes: 60,
      },
    },
  },
}
```

- `model`: model mặc định cho các sub-agent được khởi tạo. Nếu bỏ qua, các sub-agent sẽ kế thừa model của agent gọi.
- Chính sách công cụ cho từng sub-agent: `tools.subagents.tools.allow` / `tools.subagents.tools.deny`.

---

## Nhà cung cấp tùy chỉnh và base URL

OpenClaw sử dụng danh mục model của pi-coding-agent. Thêm nhà cung cấp tùy chỉnh qua `models.providers` trong config hoặc `~/.openclaw/agents/<agentId>/agent/models.json`.

```json5
{
  models: {
    mode: "merge", // merge (mặc định) | replace
    providers: {
      "custom-proxy": {
        baseUrl: "http://localhost:4000/v1",
        apiKey: "LITELLM_KEY",
        api: "openai-completions", // openai-completions | openai-responses | anthropic-messages | google-generative-ai
        models: [
          {
            id: "llama-3.1-8b",
            name: "Llama 3.1 8B",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 128000,
            maxTokens: 32000,
          },
        ],
      },
    },
  },
}
```

- Sử dụng `authHeader: true` + `headers` cho nhu cầu xác thực tùy chỉnh.
- Ghi đè thư mục gốc config agent bằng `OPENCLAW_AGENT_DIR` (hoặc `PI_CODING_AGENT_DIR`).

### Ví dụ về nhà cung cấp

<Accordion title="Cerebras (GLM 4.6 / 4.7)">

```json5
{
  env: { CEREBRAS_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: {
        primary: "cerebras/zai-glm-4.7",
        fallbacks: ["cerebras/zai-glm-4.6"],
      },
      models: {
        "cerebras/zai-glm-4.7": { alias: "GLM 4.7 (Cerebras)" },
        "cerebras/zai-glm-4.4": { alias: "GLM 4.6 (Cerebras)" }
      },
    },
  },
  models: {
    mode: "merge",
    providers: {
      cerebras: {
        baseUrl: "https://api.cerebras.ai/v1",
        apiKey: "${CEREBRAS_API_KEY}",
        api: "openai-completions",
        models: [
          { id: "zai-glm-4.7", name: "GLM 4.7 (Cerebras)" },
          { id: "zai-glm-4.6", name: "GLM 4.6 (Cerebras)" },
        ],
      },
    },
  },
}
```

Sử dụng `cerebras/zai-glm-4.7` cho Cerebras; `zai/glm-4.7` cho Z.AI trực tiếp.

</Accordion>

<Accordion title="OpenCode Zen">

```json5
{
  agents: {
    defaults: {
      model: { primary: "opencode/claude-opus-4-6" },
      models: { "opencode/claude-opus-4-6": { alias: "Opus" } },
    },
  },
}
```

Thiết lập `OPENCODE_API_KEY` (hoặc `OPENCODE_ZEN_API_KEY`). Lối tắt: `openclaw onboard --auth-choice opencode-zen`.

</Accordion>

<Accordion title="Z.AI (GLM-4.7)">

```json5
{
  agents: {
    defaults: {
      model: { primary: "zai/glm-4.7" },
      models: { "zai/glm-4.7": {} },
    },
  },
}
```

Thiết lập `ZAI_API_KEY`. `z.ai/*` và `z-ai/*` là các alias được chấp nhận. Lối tắt: `openclaw onboard --auth-choice zai-api-key`.

- Endpoint chung: `https://api.z.ai/api/paas/v4`
- Endpoint cho coding (mặc định): `https://api.z.ai/api/coding/paas/v4`
- Với endpoint chung, định nghĩa một nhà cung cấp tùy chỉnh với base URL được ghi đè.

</Accordion>

<Accordion title="Moonshot AI (Kimi)">

```json5
{
  env: { MOONSHOT_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "moonshot/kimi-k2.5" },
      models: { "moonshot/kimi-k2.5": { alias: "Kimi K2.5" } },
    },
  },
  models: {
    mode: "merge",
    providers: {
      moonshot: {
        baseUrl: "https://api.moonshot.ai/v1",
        apiKey: "${MOONSHOT_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "kimi-k2.5",
            name: "Kimi K2.5",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

Với endpoint Trung Quốc: `baseUrl: "https://api.moonshot.cn/v1"` hoặc `openclaw onboard --auth-choice moonshot-api-key-cn`.

</Accordion>

<Accordion title="Kimi Coding">

```json5
{
  env: { KIMI_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "kimi-coding/k2p5" },
      models: { "kimi-coding/k2p5": { alias: "Kimi K2.5" } },
    },
  },
}
```

Tương thích với Anthropic, nhà cung cấp tích hợp sẵn. Lối tắt: `openclaw onboard --auth-choice kimi-code-api-key`.

</Accordion>

<Accordion title="Synthetic (Anthropic-compatible)">

```json5
{
  env: { SYNTHETIC_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "synthetic/hf:MiniMaxAI/MiniMax-M2.1" },
      models: { "synthetic/hf:MiniMaxAI/MiniMax-M2.1": { alias: "MiniMax M2.1" } },
    },
  },
  models: {
    mode: "merge",
    providers: {
      synthetic: {
        baseUrl: "https://api.synthetic.new/anthropic",
        apiKey: "${SYNTHETIC_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "hf:MiniMaxAI/MiniMax-M2.1",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 192000,
            maxTokens: 65536,
          },
        ],
      },
    },
  },
}
```

Base URL không nên bao gồm `/v1` (client Anthropic sẽ tự thêm). Lối tắt: `openclaw onboard --auth-choice synthetic-api-key`.

</Accordion>

<Accordion title="MiniMax M2.1 (direct)">

```json5
{
  agents: {
    defaults: {
      model: { primary: "minimax/MiniMax-M2.1" },
      models: {
        "minimax/MiniMax-M2.1": { alias: "Minimax" },
      },
    },
  },
  models: {
    mode: "merge",
    providers: {
      minimax: {
        baseUrl: "https://api.minimax.io/anthropic",
        apiKey: "${MINIMAX_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "MiniMax-M2.1",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            cost: { input: 15, output: 60, cacheRead: 2, cacheWrite: 10 },
            contextWindow: 200000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

Thiết lập `MINIMAX_API_KEY`. Lối tắt: `openclaw onboard --auth-choice minimax-api`.

</Accordion>

<Accordion title="Local models (LM Studio)">

Xem [Local Models](/gateway/local-models). Tóm lại: chạy MiniMax M2.1 qua LM Studio Responses API trên phần cứng mạnh; giữ các model hosted ở chế độ merge để làm phương án dự phòng.

</Accordion>

---

## Skills

```json5
{
  skills: {
    allowBundled: ["gemini", "peekaboo"],
    load: {
      extraDirs: ["~/Projects/agent-scripts/skills"],
    },
    install: {
      preferBrew: true,
      nodeManager: "npm", // npm | pnpm | yarn
    },
    entries: {
      "nano-banana-pro": {
        apiKey: "GEMINI_KEY_HERE",
        env: { GEMINI_API_KEY: "GEMINI_KEY_HERE" },
      },
      peekaboo: { enabled: true },
      sag: { enabled: false },
    },
  },
}
```

- `allowBundled`: danh sách cho phép tùy chọn chỉ áp dụng cho các skill bundled (không ảnh hưởng đến skill managed/workspace).
- `entries.<skillKey>``.enabled: false` sẽ vô hiệu hóa một skill ngay cả khi nó đã được bundled/cài đặt.
- `entries.<skillKey> `.apiKey\`: tiện ích cho các skills khai báo một biến môi trường chính.

---

## Plugins

```json5
{
  plugins: {
    enabled: true,
    allow: ["voice-call"],
    deny: [],
    load: {
      paths: ["~/Projects/oss/voice-call-extension"],
    },
    entries: {
      "voice-call": {
        enabled: true,
        config: { provider: "twilio" },
      },
    },
  },
}
```

- Được tải từ `~/.openclaw/extensions`, `<workspace>/.openclaw/extensions`, và `plugins.load.paths`.
- **Thay đổi cấu hình yêu cầu khởi động lại gateway.**
- `allow`: danh sách cho phép tùy chọn (chỉ các plugin được liệt kê mới được tải). `deny` được ưu tiên.

Xem [Plugins](/tools/plugin).

---

## Browser

```json5
{
  browser: {
    enabled: true,
    evaluateEnabled: true,
    defaultProfile: "chrome",
    profiles: {
      openclaw: { cdpPort: 18800, color: "#FF4500" },
      work: { cdpPort: 18801, color: "#0066CC" },
      remote: { cdpUrl: "http://10.0.0.42:9222", color: "#00AA00" },
    },
    color: "#FF4500",
    // headless: false,
    // noSandbox: false,
    // executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser",
    // attachOnly: false,
  },
}
```

- `evaluateEnabled: false` vô hiệu hóa `act:evaluate` và `wait --fn`.
- Các profile remote chỉ hỗ trợ attach (không hỗ trợ start/stop/reset).
- Thứ tự tự động phát hiện: trình duyệt mặc định nếu dựa trên Chromium → Chrome → Brave → Edge → Chromium → Chrome Canary.
- Dịch vụ điều khiển: chỉ loopback (cổng được suy ra từ `gateway.port`, mặc định `18791`).

---

## UI

```json5
{
  ui: {
    seamColor: "#FF4500",
    assistant: {
      name: "OpenClaw",
      avatar: "CB", // emoji, short text, image URL, or data URI
    },
  },
}
```

- `seamColor`: màu nhấn cho giao diện ứng dụng native (màu bong bóng Talk Mode, v.v.).
- `assistant`: ghi đè danh tính Control UI. Mặc định sử dụng danh tính agent đang hoạt động.

---

## Gateway

```json5
{
  gateway: {
    mode: "local", // local | remote
    port: 18789,
    bind: "loopback",
    auth: {
      mode: "token", // token | password | trusted-proxy
      token: "your-token",
      // password: "your-password", // or OPENCLAW_GATEWAY_PASSWORD
      // trustedProxy: { userHeader: "x-forwarded-user" }, // for mode=trusted-proxy; see /gateway/trusted-proxy-auth
      allowTailscale: true,
      rateLimit: {
        maxAttempts: 10,
        windowMs: 60000,
        lockoutMs: 300000,
        exemptLoopback: true,
      },
    },
    tailscale: {
      mode: "off", // off | serve | funnel
      resetOnExit: false,
    },
    controlUi: {
      enabled: true,
      basePath: "/openclaw",
      // root: "dist/control-ui",
      // allowInsecureAuth: false,
      // dangerouslyDisableDeviceAuth: false,
    },
    remote: {
      url: "ws://gateway.tailnet:18789",
      transport: "ssh", // ssh | direct
      token: "your-token",
      // password: "your-password",
    },
    trustedProxies: ["10.0.0.1"],
    tools: {
      // Additional /tools/invoke HTTP denies
      deny: ["browser"],
      // Remove tools from the default HTTP deny list
      allow: ["gateway"],
    },
  },
}
```

<Accordion title="Gateway field details">

- `mode`: `local` (chạy gateway) hoặc `remote` (kết nối tới gateway từ xa). Gateway sẽ từ chối khởi động trừ khi ở chế độ `local`.
- `port`: một cổng multiplex duy nhất cho WS + HTTP. Thứ tự ưu tiên: `--port` > `OPENCLAW_GATEWAY_PORT` > `gateway.port` > `18789`.
- `bind`: `auto`, `loopback` (mặc định), `lan` (`0.0.0.0`), `tailnet` (chỉ IP Tailscale), hoặc `custom`.
- **Auth**: bắt buộc theo mặc định. Bind không phải loopback yêu cầu token/password dùng chung. Trình hướng dẫn khởi tạo (onboarding wizard) mặc định tạo một token.
- `auth.mode: "trusted-proxy"`: ủy quyền xác thực cho reverse proxy có nhận diện danh tính và tin cậy các header danh tính từ `gateway.trustedProxies` (xem [Trusted Proxy Auth](/gateway/trusted-proxy-auth)).
- `auth.allowTailscale`: khi `true`, các header danh tính từ Tailscale Serve sẽ thỏa điều kiện xác thực (được xác minh qua `tailscale whois`). Mặc định là `true` khi `tailscale.mode = "serve"`.
- `auth.rateLimit`: giới hạn tùy chọn cho các lần xác thực thất bại. Áp dụng theo từng IP client và từng phạm vi auth (shared-secret và device-token được theo dõi độc lập). Các lần thử bị chặn sẽ trả về `429` + `Retry-After`.
  - `auth.rateLimit.exemptLoopback` mặc định là `true`; đặt thành `false` khi bạn cố ý muốn giới hạn cả lưu lượng localhost (cho môi trường kiểm thử hoặc triển khai proxy nghiêm ngặt).
- `tailscale.mode`: `serve` (chỉ tailnet, bind loopback) hoặc `funnel` (công khai, yêu cầu auth).
- `remote.transport`: `ssh` (mặc định) hoặc `direct` (ws/wss). Với `direct`, `remote.url` phải là `ws://` hoặc `wss://`.
- `gateway.remote.token` chỉ dùng cho các lệnh CLI từ xa; không kích hoạt xác thực gateway cục bộ.
- `trustedProxies`: các IP reverse proxy thực hiện terminate TLS. Chỉ liệt kê các proxy bạn kiểm soát.
- `gateway.tools.deny`: tên các công cụ bổ sung bị chặn cho HTTP `POST /tools/invoke` (mở rộng danh sách chặn mặc định).
- `gateway.tools.allow`: xóa tên công cụ khỏi danh sách chặn HTTP mặc định.

</Accordion>

### Các endpoint tương thích OpenAI

- Chat Completions: bị vô hiệu hóa theo mặc định. Bật bằng `gateway.http.endpoints.chatCompletions.enabled: true`.
- Responses API: `gateway.http.endpoints.responses.enabled`.
- Tăng cường bảo mật URL đầu vào cho Responses:
  - `gateway.http.endpoints.responses.maxUrlParts`
  - `gateway.http.endpoints.responses.files.urlAllowlist`
  - `gateway.http.endpoints.responses.images.urlAllowlist`

### Cô lập nhiều instance

Chạy nhiều gateway trên một host với các cổng và thư mục trạng thái riêng biệt:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json \
OPENCLAW_STATE_DIR=~/.openclaw-a \
openclaw gateway --port 19001
```

Cờ tiện lợi: `--dev` (sử dụng `~/.openclaw-dev` + cổng `19001`), `--profile <name>` (sử dụng `~/.openclaw-<name>`).

Xem [Multiple Gateways](/gateway/multiple-gateways).

---

## Hooks

```json5
{
  hooks: {
    enabled: true,
    token: "shared-secret",
    path: "/hooks",
    maxBodyBytes: 262144,
    defaultSessionKey: "hook:ingress",
    allowRequestSessionKey: false,
    allowedSessionKeyPrefixes: ["hook:"],
    allowedAgentIds: ["hooks", "main"],
    presets: ["gmail"],
    transformsDir: "~/.openclaw/hooks/transforms",
    mappings: [
      {
        match: { path: "gmail" },
        action: "agent",
        agentId: "hooks",
        wakeMode: "now",
        name: "Gmail",
        sessionKey: "hook:gmail:{{messages[0].id}}",
        messageTemplate: "From: {{messages[0].from}}\nSubject: {{messages[0].subject}}\n{{messages[0].snippet}}",
        deliver: true,
        channel: "last",
        model: "openai/gpt-5.2-mini",
      },
    ],
  },
}
```

Xác thực: `Authorization: Bearer <token>` hoặc `x-openclaw-token: <token>`.

**Endpoints:**

- `POST /hooks/wake` → `{ text, mode?: "now"|"next-heartbeat" }`
- `POST /hooks/agent` → `{ message, name?, agentId?, sessionKey?, wakeMode?, deliver?, channel?, to?, model?, thinking?, timeoutSeconds? }`
  - `sessionKey` từ payload của request chỉ được chấp nhận khi `hooks.allowRequestSessionKey=true` (mặc định: `false`).
- `POST /hooks/<name>` → được phân giải thông qua `hooks.mappings`

<Accordion title="Mapping details">

- `match.path` khớp với sub-path sau `/hooks` (ví dụ: `/hooks/gmail` → `gmail`).
- `match.source` khớp với một trường trong payload cho các đường dẫn chung.
- Các template như `{{messages[0].subject}}` đọc dữ liệu từ payload.
- `transform` có thể trỏ tới một module JS/TS trả về một hook action.
  - `transform.module` phải là đường dẫn tương đối và nằm trong `hooks.transformsDir` (đường dẫn tuyệt đối và traversal sẽ bị từ chối).
- `agentId` định tuyến đến một agent cụ thể; ID không xác định sẽ quay về mặc định.
- `allowedAgentIds`: giới hạn định tuyến tường minh (`*` hoặc bỏ trống = cho phép tất cả, `[]` = từ chối tất cả).
- `defaultSessionKey`: khóa session cố định tùy chọn cho các lần chạy hook agent không có `sessionKey` tường minh.
- `allowRequestSessionKey`: cho phép caller của `/hooks/agent` thiết lập `sessionKey` (mặc định: `false`).
- `allowedSessionKeyPrefixes`: danh sách tiền tố được phép tùy chọn cho các giá trị `sessionKey` tường minh (request + mapping), ví dụ `['hook:']`.
- `deliver: true` gửi phản hồi cuối cùng tới một channel; `channel` mặc định là `last`.
- `model` ghi đè LLM cho lần chạy hook này (phải được cho phép nếu có thiết lập model catalog).

</Accordion>

### Tích hợp Gmail

```json5
{
  hooks: {
    gmail: {
      account: "openclaw@gmail.com",
      topic: "projects/<project-id>/topics/gog-gmail-watch",
      subscription: "gog-gmail-watch-push",
      pushToken: "shared-push-token",
      hookUrl: "http://127.0.0.1:18789/hooks/gmail",
      includeBody: true,
      maxBytes: 20000,
      renewEveryMinutes: 720,
      serve: { bind: "127.0.0.1", port: 8788, path: "/" },
      tailscale: { mode: "funnel", path: "/gmail-pubsub" },
      model: "openrouter/meta-llama/llama-3.3-70b-instruct:free",
      thinking: "off",
    },
  },
}
```

- Gateway tự động khởi động `gog gmail watch serve` khi boot nếu đã được cấu hình. Đặt `OPENCLAW_SKIP_GMAIL_WATCHER=1` để vô hiệu hóa.
- Không chạy một tiến trình `gog gmail watch serve` riêng biệt song song với Gateway.

---

## Máy chủ Canvas

```json5
{
  canvasHost: {
    root: "~/.openclaw/workspace/canvas",
    liveReload: true,
    // enabled: false, // or OPENCLAW_SKIP_CANVAS_HOST=1
  },
}
```

- Phục vụ HTML/CSS/JS có thể chỉnh sửa bởi agent và A2UI qua HTTP dưới cổng Gateway:
  - `http://<gateway-host>:<gateway.port>/__openclaw__/canvas/`
  - `http://<gateway-host>:<gateway.port>/__openclaw__/a2ui/`
- Chỉ cục bộ: giữ `gateway.bind: "loopback"` (mặc định).
- Với bind không phải loopback: các route canvas yêu cầu xác thực Gateway (token/password/trusted-proxy), tương tự các bề mặt HTTP khác của Gateway.
- Node WebViews thường không gửi header xác thực; sau khi một node được ghép nối và kết nối, Gateway cho phép cơ chế fallback IP riêng tư để node có thể tải canvas/A2UI mà không làm lộ secret trong URL.
- Chèn client live-reload vào HTML được phục vụ.
- Tự động tạo `index.html` mẫu khi thư mục trống.
- Cũng phục vụ A2UI tại `/__openclaw__/a2ui/`.
- Các thay đổi yêu cầu khởi động lại gateway.
- Tắt live reload cho các thư mục lớn hoặc khi gặp lỗi `EMFILE`.

---

## Khám phá

### mDNS (Bonjour)

```json5
{
  discovery: {
    mdns: {
      mode: "minimal", // minimal | full | off
    },
  },
}
```

- `minimal` (mặc định): bỏ qua `cliPath` + `sshPort` khỏi bản ghi TXT.
- `full`: bao gồm `cliPath` + `sshPort`.
- Hostname mặc định là `openclaw`. Ghi đè bằng `OPENCLAW_MDNS_HOSTNAME`.

### Wide-area (DNS-SD)

```json5
{
  discovery: {
    wideArea: { enabled: true },
  },
}
```

Ghi một zone DNS-SD unicast tại `~/.openclaw/dns/`. Để khám phá xuyên mạng, kết hợp với một máy chủ DNS (khuyến nghị CoreDNS) + Tailscale split DNS.

Thiết lập: `openclaw dns setup --apply`.

---

## Môi trường

### `env` (biến môi trường nội tuyến)

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: {
      GROQ_API_KEY: "gsk-...",
    },
    shellEnv: {
      enabled: true,
      timeoutMs: 15000,
    },
  },
}
```

- Biến môi trường nội tuyến chỉ được áp dụng nếu môi trường tiến trình chưa có key đó.
- Tệp `.env`: `.env` trong CWD + `~/.openclaw/.env` (không cái nào ghi đè các biến đã tồn tại).
- `shellEnv`: nhập các key còn thiếu được mong đợi từ profile shell đăng nhập của bạn.
- Xem [Environment](/help/environment) để biết đầy đủ thứ tự ưu tiên.

### Thay thế biến môi trường

Tham chiếu biến môi trường trong bất kỳ chuỗi cấu hình nào bằng `${VAR_NAME}`:

```json5
{
  gateway: {
    auth: { token: "${OPENCLAW_GATEWAY_TOKEN}" },
  },
}
```

- Chỉ khớp các tên viết hoa: `[A-Z_][A-Z0-9_]*`.
- Biến thiếu/trống sẽ gây lỗi khi tải cấu hình.
- Escape bằng `$${VAR}` để hiển thị literal `${VAR}`.
- Hoạt động với `$include`.

---

## Lưu trữ xác thực

```json5
{
  auth: {
    profiles: {
      "anthropic:me@example.com": { provider: "anthropic", mode: "oauth", email: "me@example.com" },
      "anthropic:work": { provider: "anthropic", mode: "api_key" },
    },
    order: {
      anthropic: ["anthropic:me@example.com", "anthropic:work"],
    },
  },
}
```

- Hồ sơ xác thực theo từng agent được lưu tại `<agentDir>/auth-profiles.json`.
- OAuth cũ được nhập từ `~/.openclaw/credentials/oauth.json`.
- Xem [OAuth](/concepts/oauth).

---

## Ghi log

```json5
{
  logging: {
    level: "info",
    file: "/tmp/openclaw/openclaw.log",
    consoleLevel: "info",
    consoleStyle: "pretty", // pretty | compact | json
    redactSensitive: "tools", // off | tools
    redactPatterns: ["\\bTOKEN\\b\\s*[=:]\\s*([\"']?)([^\\s\"']+)\\1"],
  },
}
```

- Tệp log mặc định: `/tmp/openclaw/openclaw-YYYY-MM-DD.log`.
- Đặt `logging.file` để có đường dẫn cố định.
- `consoleLevel` tăng lên `debug` khi dùng `--verbose`.

---

## Trình hướng dẫn

Metadata được ghi bởi các trình hướng dẫn CLI (`onboard`, `configure`, `doctor`):

```json5
{
  wizard: {
    lastRunAt: "2026-01-01T00:00:00.000Z",
    lastRunVersion: "2026.1.4",
    lastRunCommit: "abc1234",
    lastRunCommand: "configure",
    lastRunMode: "local",
  },
}
```

---

## Định danh

```json5
{
  agents: {
    list: [
      {
        id: "main",
        identity: {
          name: "Samantha",
          theme: "helpful sloth",
          emoji: "🦥",
          avatar: "avatars/samantha.png",
        },
      },
    ],
  },
}
```

Được ghi bởi trợ lý onboarding trên macOS. Suy ra các giá trị mặc định:

- `messages.ackReaction` từ `identity.emoji` (mặc định là 👀 nếu không có)
- `mentionPatterns` từ `identity.name`/`identity.emoji`
- `avatar` chấp nhận: đường dẫn tương đối với workspace, URL `http(s)`, hoặc URI `data:`

---

## Bridge (cũ, đã bị loại bỏ)

Các bản dựng hiện tại không còn bao gồm TCP bridge. Các node kết nối qua Gateway WebSocket. Các khóa `bridge.*` không còn là một phần của schema cấu hình (xác thực sẽ thất bại cho đến khi được xóa; `openclaw doctor --fix` có thể loại bỏ các khóa không xác định).

<Accordion title="Legacy bridge config (historical reference)">

```json
{
  "bridge": {
    "enabled": true,
    "port": 18790,
    "bind": "tailnet",
    "tls": {
      "enabled": true,
      "autoGenerate": true
    }
  }
}
```

</Accordion>

---

## Cron

```json5
{
  cron: {
    enabled: true,
    maxConcurrentRuns: 2,
    sessionRetention: "24h", // duration string or false
  },
}
```

- `sessionRetention`: thời gian giữ các phiên cron đã hoàn thành trước khi dọn dẹp. Mặc định: `24h`.

Xem [Cron Jobs](/automation/cron-jobs).

---

## Biến mẫu model media

Các placeholder trong template được mở rộng trong `tools.media.*.models[].args`:

| Biến               | Mô tả                                                                                                     |
| ------------------ | --------------------------------------------------------------------------------------------------------- |
| `{{Body}}`         | Toàn bộ nội dung message đầu vào                                                                          |
| `{{RawBody}}`      | Nội dung thô (không có wrapper lịch sử/người gửi)                                      |
| `{{BodyStripped}}` | Nội dung đã loại bỏ các đề cập nhóm                                                                       |
| `{{From}}`         | Định danh người gửi                                                                                       |
| `{{To}}`           | Định danh đích                                                                                            |
| `{{MessageSid}}`   | ID tin nhắn kênh                                                                                          |
| `{{SessionId}}`    | UUID phiên hiện tại                                                                                       |
| `{{IsNewSession}}` | `"true"` khi phiên mới được tạo                                                                           |
| `{{MediaUrl}}`     | Pseudo-URL của media đầu vào                                                                              |
| `{{MediaPath}}`    | Đường dẫn media cục bộ                                                                                    |
| `{{MediaType}}`    | Loại media (image/audio/document/…)                                                    |
| `{{Transcript}}`   | Bản ghi âm thanh                                                                                          |
| `{{Prompt}}`       | Prompt media đã được xử lý cho các mục CLI                                                                |
| `{{MaxChars}}`     | Số ký tự đầu ra tối đa đã được xử lý cho các mục CLI                                                      |
| `{{ChatType}}`     | `"direct"` hoặc `"group"`                                                                                 |
| `{{GroupSubject}}` | Chủ đề nhóm (mức tốt nhất có thể)                                                      |
| `{{GroupMembers}}` | Xem trước thành viên nhóm (mức tốt nhất có thể)                                        |
| `{{SenderName}}`   | Tên hiển thị của người gửi (mức tốt nhất có thể)                                       |
| `{{SenderE164}}`   | Số điện thoại người gửi (mức tốt nhất có thể)                                          |
| `{{Provider}}`     | Gợi ý nhà cung cấp (whatsapp, telegram, discord, v.v.) |

---

## Cấu hình bao gồm (`$include`)

Chia cấu hình thành nhiều tệp:

```json5
// ~/.openclaw/openclaw.json
{
  gateway: { port: 18789 },
  agents: { $include: "./agents.json5" },
  broadcast: {
    $include: ["./clients/mueller.json5", "./clients/schmidt.json5"],
  },
}
```

**Hành vi hợp nhất (merge):**

- Một tệp đơn: thay thế toàn bộ đối tượng chứa nó.
- Mảng nhiều tệp: được hợp nhất sâu theo thứ tự (tệp sau ghi đè tệp trước).
- Các khóa cùng cấp: được hợp nhất sau các includes (ghi đè các giá trị đã include).
- Include lồng nhau: tối đa 10 cấp độ.
- Đường dẫn: tương đối (so với tệp đang include), tuyệt đối hoặc tham chiếu thư mục cha `../`.
- Lỗi: thông báo rõ ràng cho tệp bị thiếu, lỗi phân tích cú pháp và include vòng lặp.

---

_Liên quan: [Configuration](/gateway/configuration) · [Configuration Examples](/gateway/configuration-examples) · [Doctor](/gateway/doctor)_

