---
summary: "Trạng thái hỗ trợ bot Discord, khả năng và cấu hình"
read_when:
  - Làm việc trên các tính năng kênh Discord
title: "Discord"
---

# Discord (Bot API)

Trạng thái: sẵn sàng cho DM và kênh văn bản guild thông qua gateway bot Discord chính thức.

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Hành vi lệnh native và danh mục lệnh.
  
</Card>
  <Card title="Slash commands" icon="terminal" href="/tools/slash-commands">
    Chẩn đoán và quy trình sửa lỗi liên kênh.
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">Thiết lập nhanh
</Card>
</CardGroup>

## ```
Tạo một ứng dụng trong Discord Developer Portal, thêm bot, sau đó bật:
```

<Steps>
  <Step title="Create a Discord bot and enable intents">- **Message Content Intent**
- **Server Members Intent** (bắt buộc cho allowlist vai trò và định tuyến dựa trên vai trò; khuyến nghị cho việc khớp allowlist tên-sang-ID)

    ```
    Biến môi trường dự phòng cho tài khoản mặc định:
    ```

  
</Step>

  <Step title="Configure token">

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "YOUR_BOT_TOKEN",
    },
  },
}
```

    ```
    DISCORD_BOT_TOKEN=...
    ```

```bash

    Mời bot vào server của bạn với quyền gửi tin nhắn.
```

  
</Step>

  <Step title="Invite the bot and start gateway">
    Invite the bot to your server with message permissions.

```bash
openclaw gateway
```

  
</Step>

  <Step title="Approve first DM pairing">

```bash
openclaw pairing list discord
openclaw pairing approve discord <CODE>
```

    ```
    Mã ghép nối sẽ hết hạn sau 1 giờ.
    ```

  
</Step>
</Steps>

<Note>
Phân giải token theo từng tài khoản. Giá trị token trong config sẽ được ưu tiên hơn biến môi trường dự phòng. `DISCORD_BOT_TOKEN` chỉ được sử dụng cho tài khoản mặc định.
</Note>

## Mô hình runtime

- Gateway quản lý kết nối Discord.
- Định tuyến phản hồi có tính xác định: tin nhắn đến từ Discord sẽ được trả lời lại trên Discord.
- Theo mặc định (`session.dmScope=main`), các cuộc trò chuyện trực tiếp sẽ dùng chung phiên chính của agent (`agent:main:main`).
- Các kênh Guild được tách biệt bằng khóa phiên riêng (`agent:<agentId>:discord:channel:<channelId>`).
- Group DM bị bỏ qua theo mặc định (`channels.discord.dm.groupEnabled=false`).
- Các slash command gốc chạy trong các phiên lệnh tách biệt (`agent:<agentId>:discord:slash:<userId>`), đồng thời vẫn mang theo `CommandTargetSessionKey` tới phiên hội thoại được định tuyến.

## Kiểm soát truy cập và định tuyến

<Tabs>
  <Tab title="DM policy">
    `channels.discord.dmPolicy` kiểm soát quyền truy cập DM (legacy: `channels.discord.dm.policy`):

    ```
    - `pairing` (mặc định)
    - `allowlist`
    - `open` (yêu cầu `channels.discord.allowFrom` bao gồm `"*"`; legacy: `channels.discord.dm.allowFrom`)
    - `disabled`
    
    Nếu DM policy không phải `open`, người dùng chưa xác định sẽ bị chặn (hoặc được nhắc ghép nối trong chế độ `pairing`).
    
    Định dạng DM target để gửi:
    
    - `user:<id>`
    - đề cập `<@id>`
    
    ID số thuần túy là không rõ ràng và sẽ bị từ chối trừ khi cung cấp rõ loại target là user/channel.
    ```

  
</Tab>

  <Tab title="Guild policy">
    Xử lý Guild được kiểm soát bởi `channels.discord.groupPolicy`:

    ```
    - `open`
    - `allowlist`
    - `disabled`
    
    Thiết lập bảo mật cơ bản khi tồn tại `channels.discord` là `allowlist`.
    
    Hành vi của `allowlist`:
    
    - guild phải khớp với `channels.discord.guilds` (ưu tiên `id`, chấp nhận slug)
    - danh sách cho phép người gửi tùy chọn: `users` (ID hoặc tên) và `roles` (chỉ ID role); nếu cấu hình một trong hai, người gửi được chấp nhận khi khớp `users` HOẶC `roles`
    - nếu một guild có cấu hình `channels`, các kênh không được liệt kê sẽ bị từ chối
    - nếu một guild không có khối `channels`, tất cả kênh trong guild được allowlist sẽ được phép
    
    Ví dụ:
    ```

```json5
{
  channels: {
    discord: {
      groupPolicy: "allowlist",
      guilds: {
        "123456789012345678": {
          requireMention: true,
          users: ["987654321098765432"],
          roles: ["123456789012345678"],
          channels: {
            general: { allow: true },
            help: { allow: true, requireMention: true },
          },
        },
      },
    },
  },
}
```

    ```
    Nếu bạn chỉ thiết lập `DISCORD_BOT_TOKEN` và không tạo khối `channels.discord`, cơ chế fallback runtime sẽ là `groupPolicy="open"` (kèm cảnh báo trong log).
    ```

  
</Tab>

  <Tab title="Mentions and group DMs">
    Tin nhắn Guild mặc định yêu cầu phải có mention.

    ```
    Phát hiện mention bao gồm:
    
    - mention bot tường minh
    - các mẫu mention đã cấu hình (`agents.list[].groupChat.mentionPatterns`, fallback `messages.groupChat.mentionPatterns`)
    - hành vi trả lời bot ngầm định trong các trường hợp được hỗ trợ
    
    `requireMention` được cấu hình theo từng guild/kênh (`channels.discord.guilds...`).
    
    Group DM:
    
    - mặc định: bị bỏ qua (`dm.groupEnabled=false`)
    - tùy chọn allowlist qua `dm.groupChannels` (ID hoặc slug của kênh)
    ```

  
</Tab>
</Tabs>

### Định tuyến agent dựa trên role

Sử dụng `bindings[].match.roles` để định tuyến thành viên Discord guild tới các agent khác nhau theo ID role. Binding dựa trên role chỉ chấp nhận ID role và được đánh giá sau binding peer hoặc parent-peer và trước binding chỉ theo guild. Nếu một binding cũng thiết lập các trường match khác (ví dụ `peer` + `guildId` + `roles`), tất cả các trường đã cấu hình phải khớp.

```json5
{
  bindings: [
    {
      agentId: "opus",
      match: {
        channel: "discord",
        guildId: "123456789012345678",
        roles: ["111111111111111111"],
      },
    },
    {
      agentId: "sonnet",
      match: {
        channel: "discord",
        guildId: "123456789012345678",
      },
    },
  ],
}
```

## Thiết lập Developer Portal

<AccordionGroup>
  <Accordion title="Create app and bot">

    ```
    1. Discord Developer Portal -> **Applications** -> **New Application**
    2. **Bot** -> **Add Bot**
    3. Sao chép bot token
    ```

  
</Accordion>

  <Accordion title="Privileged intents">
    Trong **Bot -> Privileged Gateway Intents**, bật:

    ```
    - Message Content Intent
    - Server Members Intent (khuyến nghị)
    
    Presence intent là tùy chọn và chỉ cần nếu bạn muốn nhận cập nhật trạng thái hiện diện. Việc thiết lập trạng thái bot (`setPresence`) không yêu cầu bật cập nhật presence cho thành viên.
    ```

  
</Accordion>

  <Accordion title="OAuth scopes and baseline permissions">
    Trình tạo URL OAuth:

    ```
    - scopes: `bot`, `applications.commands`
    
    Quyền cơ bản thường dùng:
    
    - View Channels
    - Send Messages
    - Read Message History
    - Embed Links
    - Attach Files
    - Add Reactions (tùy chọn)
    
    Tránh quyền `Administrator` trừ khi thực sự cần thiết.
    ```

  
</Accordion>

  <Accordion title="Copy IDs">
    Bật Discord Developer Mode, sau đó sao chép:

    ```
    - server ID
    - channel ID
    - user ID
    
    Ưu tiên sử dụng ID số trong cấu hình OpenClaw để đảm bảo kiểm tra và dò tìm đáng tin cậy.
    ```

  
</Accordion>
</AccordionGroup>

## Lệnh gốc và xác thực lệnh

- `commands.native` mặc định là `"auto"` và được bật cho Discord.
- Ghi đè theo từng kênh: `channels.discord.commands.native`.
- `commands.native=false` sẽ xóa rõ ràng các lệnh Discord native đã được đăng ký trước đó.
- Xác thực lệnh native sử dụng cùng allowlist/chính sách Discord như xử lý tin nhắn thông thường.
- Các lệnh vẫn có thể hiển thị trong giao diện Discord cho người dùng không được ủy quyền; tuy nhiên khi thực thi vẫn áp dụng xác thực OpenClaw và trả về "not authorized".

Xem [Slash commands](/tools/slash-commands) để biết danh mục và hành vi của lệnh.

## Chính sách retry

<AccordionGroup>
  <Accordion title="Reply tags and native replies">
    Discord hỗ trợ thẻ phản hồi trong đầu ra của agent:

    ```
    - `[[reply_to_current]]`
    - `[[reply_to:<id>]]`
    
    Được kiểm soát bởi `channels.discord.replyToMode`:
    
    - `off` (mặc định)
    - `first`
    - `all`
    
    Lưu ý: `off` sẽ vô hiệu hóa luồng phản hồi ngầm định. Các thẻ `[[reply_to_*]]` tường minh vẫn được tôn trọng.
    
    ID tin nhắn được đưa vào context/history để agent có thể nhắm mục tiêu các tin nhắn cụ thể.
    ```

  
</Accordion>

  <Accordion title="History, context, and thread behavior">
    Ngữ cảnh lịch sử Guild:

    ```
    - `channels.discord.historyLimit` mặc định `20`
    - dự phòng: `messages.groupChat.historyLimit`
    - `0` để vô hiệu hóa
    
    Điều khiển lịch sử DM:
    
    - `channels.discord.dmHistoryLimit`
    - `channels.discord.dms["<user_id>"].historyLimit`
    
    Hành vi thread:
    
    - Thread Discord được định tuyến như các phiên theo kênh
    - metadata của thread cha có thể được dùng để liên kết phiên cha
    - cấu hình thread kế thừa cấu hình kênh cha trừ khi có cấu hình riêng cho thread
    
    Chủ đề kênh được đưa vào dưới dạng ngữ cảnh **không đáng tin cậy** (không phải system prompt).
    ```

  
</Accordion>

  <Accordion title="Reaction notifications">
    Chế độ thông báo reaction theo từng guild:

    ```
    - `off`
    - `own` (mặc định)
    - `all`
    - `allowlist` (sử dụng `guilds.<id>.users`)
    
    Các sự kiện reaction được chuyển thành sự kiện hệ thống và gắn vào phiên Discord đã được định tuyến.
    ```

  
</Accordion>

  <Accordion title="Ack reactions">
    `ackReaction` gửi một emoji xác nhận trong khi OpenClaw đang xử lý tin nhắn đến.

    ```
    Thứ tự phân giải:
    
    - `channels.discord.accounts.<accountId>.ackReaction`
    - `channels.discord.ackReaction`
    - `messages.ackReaction`
    - emoji định danh agent dự phòng (`agents.list[].identity.emoji`, nếu không có thì dùng "👀")
    
    Lưu ý:
    
    - Discord chấp nhận emoji unicode hoặc tên emoji tùy chỉnh.
    - Dùng `""` để vô hiệu hóa reaction cho một kênh hoặc tài khoản.
    ```

  
</Accordion>

  <Accordion title="Config writes">
    Ghi cấu hình được khởi tạo từ kênh được bật theo mặc định.

    ```
    Điều này ảnh hưởng đến luồng `/config set|unset` (khi các tính năng lệnh được bật).
    
    Vô hiệu hóa:
    ```

```json5
{
  channels: {
    discord: {
      configWrites: false,
    },
  },
}
```

  
</Accordion>

  <Accordion title="Gateway proxy">
    Định tuyến lưu lượng WebSocket của Discord gateway qua proxy HTTP(S) với `channels.discord.proxy`.

```json5
{
  channels: {
    discord: {
      proxy: "http://proxy.example:8080",
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
    discord: {
      accounts: {
        primary: {
          proxy: "http://proxy.example:8080",
        },
      },
    },
  },
}
```

  
</Accordion>

  <Accordion title="PluralKit support">
    Bật phân giải PluralKit để ánh xạ các tin nhắn được proxy tới danh tính thành viên hệ thống:

```json5
{
  channels: {
    discord: {
      pluralkit: {
        enabled: true,
        token: "pk_live_...", // tùy chọn; cần cho hệ thống riêng tư
      },
    },
  },
}
```

    ```
    Lưu ý:
    
    - allowlist có thể sử dụng `pk:<memberId>`
    - tên hiển thị của thành viên được khớp theo name/slug
    - tra cứu sử dụng ID tin nhắn gốc và bị giới hạn trong một khoảng thời gian
    - nếu tra cứu thất bại, tin nhắn proxy sẽ được xem là tin nhắn bot và bị loại bỏ trừ khi `allowBots=true`
    ```

  
</Accordion>

  <Accordion title="Presence configuration">
    Cập nhật trạng thái hiện diện chỉ được áp dụng khi bạn đặt trường status hoặc activity.

    ```
    Ví dụ chỉ đặt trạng thái:
    ```

```json5
{
  channels: {
    discord: {
      status: "idle",
    },
  },
}
```

    ```
    Ví dụ activity (custom status là loại activity mặc định):
    ```

```json5
{
  channels: {
    discord: {
      activity: "Focus time",
      activityType: 4,
    },
  },
}
```

    ```
    Ví dụ Streaming:
    ```

```json5
{
  channels: {
    discord: {
      activity: "Live coding",
      activityType: 1,
      activityUrl: "https://twitch.tv/openclaw",
    },
  },
}
```

    ```
    Bảng ánh xạ loại activity:
    
    - 0: Playing
    - 1: Streaming (yêu cầu `activityUrl`)
    - 2: Listening
    - 3: Watching
    - 4: Custom (sử dụng văn bản activity làm trạng thái; emoji là tùy chọn)
    - 5: Competing
    ```

  
</Accordion>

  <Accordion title="Exec approvals in Discord">
    Discord hỗ trợ phê duyệt thực thi dựa trên nút trong DM và có thể tùy chọn đăng lời nhắc phê duyệt trong kênh gốc.

    ```
    Đường dẫn cấu hình:
    
    - `channels.discord.execApprovals.enabled`
    - `channels.discord.execApprovals.approvers`
    - `channels.discord.execApprovals.target` (`dm` | `channel` | `both`, mặc định: `dm`)
    - `agentFilter`, `sessionFilter`, `cleanupAfterResolve`
    
    Khi `target` là `channel` hoặc `both`, lời nhắc phê duyệt sẽ hiển thị trong kênh. Chỉ những approver được cấu hình mới có thể sử dụng các nút; người dùng khác sẽ nhận thông báo từ chối dạng ephemeral. Lời nhắc phê duyệt bao gồm nội dung lệnh, vì vậy chỉ nên bật gửi trong kênh ở các kênh đáng tin cậy. Nếu không thể suy ra ID kênh từ khóa phiên, OpenClaw sẽ quay về gửi qua DM.
    
    Nếu phê duyệt thất bại với ID phê duyệt không xác định, hãy kiểm tra danh sách approver và việc bật tính năng.
    
    Tài liệu liên quan: [Exec approvals](/tools/exec-approvals)
    ```

  
</Accordion>
</AccordionGroup>

## Công cụ và action gate

Các hành động tin nhắn Discord bao gồm nhắn tin, quản trị kênh, kiểm duyệt, hiện diện và các hành động metadata.

Ví dụ cốt lõi:

- messaging: `sendMessage`, `readMessages`, `editMessage`, `deleteMessage`, `threadReply`
- reactions: `react`, `reactions`, `emojiList`
- moderation: `timeout`, `kick`, `ban`
- presence: `setPresence`

Các action gate nằm dưới `channels.discord.actions.*`.

Hành vi gate mặc định:

| Nhóm hành động                                                                                                                                                           | Mặc định |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------- |
| reactions, messages, threads, pins, polls, search, memberInfo, roleInfo, channelInfo, channels, voiceStatus, events, stickers, emojiUploads, stickerUploads, permissions | enabled  |
| roles                                                                                                                                                                    | disabled |
| moderation                                                                                                                                                               | disabled |
| presence                                                                                                                                                                 | disabled |

## UI Components v2

OpenClaw sử dụng Discord components v2 cho phê duyệt exec và các dấu đánh dấu đa ngữ cảnh. Các action tin nhắn Discord cũng có thể nhận `components` cho UI tùy chỉnh (nâng cao; yêu cầu các instance component của Carbon), trong khi `embeds` legacy vẫn khả dụng nhưng không được khuyến nghị.

- `channels.discord.ui.components.accentColor` thiết lập màu nhấn được sử dụng bởi các container component của Discord (hex).
- Thiết lập theo từng tài khoản với `channels.discord.accounts.<id>.ui.components.accentColor`.
- `embeds` sẽ bị bỏ qua khi có components v2.

Ví dụ:

```json5
{
  channels: {
    discord: {
      ui: {
        components: {
          accentColor: "#5865F2",
        },
      },
    },
  },
}
```

## Tin nhắn thoại

Tin nhắn thoại Discord hiển thị bản xem trước dạng sóng và yêu cầu âm thanh OGG/Opus kèm metadata. OpenClaw tự động tạo dạng sóng, nhưng cần `ffmpeg` và `ffprobe` có sẵn trên gateway host để phân tích và chuyển đổi tệp âm thanh.

Yêu cầu và ràng buộc:

- Cung cấp **đường dẫn tệp cục bộ** (URL sẽ bị từ chối).
- Không kèm nội dung văn bản (Discord không cho phép văn bản + tin nhắn thoại trong cùng một payload).
- Chấp nhận mọi định dạng âm thanh; OpenClaw sẽ chuyển đổi sang OGG/Opus khi cần.

Ví dụ:

```bash
message(action="send", channel="discord", target="channel:123", path="/path/to/audio.mp3", asVoice=true)
```

## Xử lý sự cố

<AccordionGroup>
  <Accordion title="Used disallowed intents or bot sees no guild messages">

    ```
    - bật Message Content Intent
    - bật Server Members Intent khi bạn phụ thuộc vào phân giải user/member
    - khởi động lại gateway sau khi thay đổi intents
    ```

  
</Accordion>

  <Accordion title="Guild messages blocked unexpectedly">

    ```
    - xác minh `groupPolicy`
    - xác minh danh sách cho phép guild dưới `channels.discord.guilds`
    - nếu tồn tại map `channels` của guild, chỉ các channel được liệt kê mới được phép
    - xác minh hành vi `requireMention` và các mẫu mention
    
    Useful checks:
    ```

```bash
openclaw doctor
openclaw channels status --probe
openclaw logs --follow
```

  
</Accordion>

  <Accordion title="Require mention false but still blocked">
    Nguyên nhân phổ biến:

    ```
    - `groupPolicy="allowlist"` nhưng không có guild/channel tương ứng trong allowlist
    - cấu hình `requireMention` sai vị trí (phải nằm dưới `channels.discord.guilds` hoặc mục channel)
    - người gửi bị chặn bởi allowlist `users` của guild/channel
    ```

  
</Accordion>

  <Accordion title="Permissions audit mismatches">
    Kiểm tra quyền bằng `channels status --probe` chỉ hoạt động với channel ID dạng số.

    ```
    Nếu bạn sử dụng khóa slug, việc khớp khi runtime vẫn có thể hoạt động, nhưng probe không thể xác minh đầy đủ quyền.
    ```

  
</Accordion>

  <Accordion title="DM and pairing issues">

    ```
    - DM bị tắt: `channels.discord.dm.enabled=false`
    - chính sách DM bị tắt: `channels.discord.dmPolicy="disabled"` (legacy: `channels.discord.dm.policy`)
    - đang chờ phê duyệt ghép nối trong chế độ `pairing`
    ```

  
</Accordion>

  <Accordion title="Bot to bot loops">
    Theo mặc định, các tin nhắn do bot tạo sẽ bị bỏ qua.

    ```
    Nếu bạn đặt `channels.discord.allowBots=true`, hãy sử dụng quy tắc mention và allowlist nghiêm ngặt để tránh hành vi lặp vô hạn.
    ```

  
</Accordion>
</AccordionGroup>

## Tham chiếu cấu hình

Tham chiếu chính:

- [Configuration reference - Discord](/gateway/configuration-reference#discord)

Các trường Discord quan trọng:

- startup/auth: `enabled`, `token`, `accounts.*`, `allowBots`
- policy: `groupPolicy`, `dm.*`, `guilds.*`, `guilds.*.channels.*`
- command: `commands.native`, `commands.useAccessGroups`, `configWrites`
- reply/history: `replyToMode`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`
- delivery: `textChunkLimit`, `chunkMode`, `maxLinesPerMessage`
- media/retry: `mediaMaxMb`, `retry`
- actions: `actions.*`
- presence: `activity`, `status`, `activityType`, `activityUrl`
- UI: `ui.components.accentColor`
- features: `pluralkit`, `execApprovals`, `intents`, `agentComponents`, `heartbeat`, `responsePrefix`

## An toàn và vận hành

- Xem token bot như thông tin bí mật (`DISCORD_BOT_TOKEN` được ưu tiên trong môi trường được giám sát).
- Chỉ cấp quyền Discord ở mức tối thiểu cần thiết.
- Nếu trạng thái deploy/lệnh bị lỗi thời, hãy khởi động lại gateway và kiểm tra lại bằng `openclaw channels status --probe`.

## Liên quan

- [Ghép nối](/channels/pairing)
- [Định tuyến kênh](/channels/channel-routing)
- [Khắc phục sự cố](/channels/troubleshooting)
- [Lệnh slash](/tools/slash-commands)

