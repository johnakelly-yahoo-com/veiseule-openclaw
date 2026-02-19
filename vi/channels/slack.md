---
summary: "Thiết lập Slack cho chế độ socket hoặc HTTP webhook"
read_when:
  - Thiết lập Slack hoặc gỡ lỗi chế độ Slack socket/HTTP
title: "Slack"
---

# Slack

Trạng thái: sẵn sàng cho môi trường production với DM + kênh thông qua tích hợp ứng dụng Slack. Chế độ mặc định là Socket Mode; cũng hỗ trợ chế độ HTTP Events API.

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Slack DMs mặc định ở chế độ pairing.
  
</Card>
  <Card title="Slash commands" icon="terminal" href="/tools/slash-commands">
    Hành vi lệnh gốc và danh mục lệnh.
   
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">Chẩn đoán và quy trình khắc phục sự cố đa kênh.
</Card>
</CardGroup>

## Thiết lập nhanh

<Tabs>
  <Tab title="Socket Mode (default)">
    <Steps>
      <Step title="Create Slack app and tokens">        Trong cài đặt ứng dụng Slack:

        ```
        {
          channels: {
            slack: {
              enabled: true,
              appToken: "xapp-...",
              botToken: "xoxb-...",
            },
          },
        }
        ```

```json5
{
  channels: {
    slack: {
      enabled: true,
      mode: "socket",
      appToken: "xapp-...",
      botToken: "xoxb-...",
    },
  },
}
```

        ```
            Dự phòng qua biến môi trường (chỉ tài khoản mặc định):
        ```

```bash
SLACK_APP_TOKEN=xapp-...
SLACK_BOT_TOKEN=xoxb-...
```

        
</Step>
      
        <Step title="Đăng ký sự kiện ứng dụng">
          Đăng ký các sự kiện bot sau:
      
          - `app_mention`
          - `message.channels`, `message.groups`, `message.im`, `message.mpim`
          - `reaction_added`, `reaction_removed`
          - `member_joined_channel`, `member_left_channel`
          - `channel_rename`
          - `pin_added`, `pin_removed`
      
          Đồng thời bật **Messages Tab** trong App Home cho DM.
        
</Step>
      
        <Step title="Khởi động gateway">

```bash
openclaw gateway
```

        
</Step>
      
</Steps>

  
</Tab>

  <Tab title="HTTP Events API mode">
    <Steps>
      <Step title="Configure Slack app for HTTP">

        ```
        {
          channels: {
            slack: {
              enabled: true,
              appToken: "xapp-...",
              botToken: "xoxb-...",
            },
          },
        }
        ```

```json5
{
  channels: {
    slack: {
      enabled: true,
      mode: "http",
      botToken: "xoxb-...",
      signingSecret: "your-signing-secret",
      webhookPath: "/slack/events",
    },
  },
}
```

        
</Step>
      
        <Step title="Sử dụng đường dẫn webhook riêng biệt cho HTTP đa tài khoản">
          Chế độ HTTP theo từng tài khoản được hỗ trợ.
      
          Cung cấp `webhookPath` riêng cho mỗi tài khoản để tránh xung đột khi đăng ký.
        
</Step>
      
</Steps>

  
</Tab>
</Tabs>

## Mô hình token

- `botToken` + `appToken` là bắt buộc cho Socket Mode.
- Chế độ HTTP yêu cầu `botToken` + `signingSecret`.
- Token trong cấu hình sẽ ghi đè dự phòng từ biến môi trường.
- Dự phòng biến môi trường `SLACK_BOT_TOKEN` / `SLACK_APP_TOKEN` chỉ áp dụng cho tài khoản mặc định.
- `userToken` (`xoxp-...`) chỉ cấu hình trong config (không có dự phòng biến môi trường) và mặc định ở chế độ chỉ đọc (`userTokenReadOnly: true`).
- Tùy chọn: thêm `chat:write.customize` nếu bạn muốn tin nhắn gửi đi sử dụng danh tính agent đang hoạt động (tùy chỉnh `username` và icon). `icon_emoji` sử dụng cú pháp `:emoji_name:`.

<Tip>
Đối với các thao tác/đọc danh bạ, có thể ưu tiên user token khi được cấu hình. Đối với thao tác ghi, bot token vẫn được ưu tiên; chỉ cho phép ghi bằng user token khi `userTokenReadOnly: false` và bot token không khả dụng.
</Tip>

## Kiểm soát truy cập và định tuyến

<Tabs>
  <Tab title="DM policy">    `channels.slack.dmPolicy` kiểm soát quyền truy cập DM (phiên bản cũ: `channels.slack.dm.policy`):

    ```
    - `pairing` (mặc định)
    - `allowlist`
    - `open` (yêu cầu `channels.slack.allowFrom` bao gồm `"*"`; phiên bản cũ: `channels.slack.dm.allowFrom`)
    - `disabled`
    
    Cờ DM:
    
    - `dm.enabled` (mặc định true)
    - `channels.slack.allowFrom` (ưu tiên)
    - `dm.allowFrom` (phiên bản cũ)
    - `dm.groupEnabled` (DM nhóm mặc định false)
    - `dm.groupChannels` (danh sách cho phép MPIM tùy chọn)
    
    Ghép đôi trong DM sử dụng `openclaw pairing approve slack <code>`.
    ```

  
</Tab>

  <Tab title="Channel policy">    `channels.slack.groupPolicy` kiểm soát cách xử lý kênh:

    ```
    - `open`
    - `allowlist`
    - `disabled`
    
    Danh sách cho phép kênh nằm trong `channels.slack.channels`.
    
    Lưu ý khi chạy: nếu `channels.slack` hoàn toàn отсутств (thiết lập chỉ qua biến môi trường) và `channels.defaults.groupPolicy` chưa được đặt, hệ thống sẽ mặc định `groupPolicy="open"` và ghi log cảnh báo.
    
    Phân giải Tên/ID:
    
    - các mục trong danh sách cho phép kênh và DM được phân giải khi khởi động nếu quyền truy cập token cho phép
    - các mục không phân giải được sẽ giữ nguyên như đã cấu hình
    ```

  
</Tab>

  <Tab title="Mentions and channel users">    Tin nhắn trong kênh mặc định yêu cầu phải nhắc đến bot.

    ```
    Nguồn nhắc đến:
    
    - nhắc đến ứng dụng trực tiếp (`<@botId>`)
    - mẫu regex nhắc đến (`agents.list[].groupChat.mentionPatterns`, dự phòng `messages.groupChat.mentionPatterns`)
    - hành vi ngầm định trả lời trong luồng của bot
    
    Kiểm soát theo từng kênh (`channels.slack.channels.<id|name>`):
    
    - `requireMention`
    - `users` (allowlist)
    - `allowBots`
    - `skills`
    - `systemPrompt`
    - `tools`, `toolsBySender`
    ```

  
</Tab>
</Tabs>

## Cấu hình OpenClaw (tối thiểu)

- Chế độ tự động cho lệnh gốc (native command auto-mode) **tắt** đối với Slack (`commands.native: "auto"` không kích hoạt lệnh native của Slack).
- Bật trình xử lý lệnh Slack native với `channels.slack.commands.native: true` (hoặc toàn cục `commands.native: true`).
- Khi lệnh native được bật, hãy đăng ký các slash command tương ứng trong Slack (tên `/<command>`).
- Nếu lệnh native không được bật, bạn có thể chạy một slash command đã cấu hình thông qua `channels.slack.slashCommand`.

Thiết lập slash command mặc định:

- `enabled: false`
- `name: "openclaw"`
- `sessionPrefix: "slack:slash"`
- `ephemeral: true`

Phiên slash sử dụng khóa tách biệt:

- `agent:<agentId>:slack:slash:<userId>`

và vẫn định tuyến việc thực thi lệnh tới phiên hội thoại mục tiêu (`CommandTargetSessionKey`).

## Scopes (hiện tại vs tùy chọn)

- DM được định tuyến là `direct`; channel là `channel`; MPIM là `group`.
- Với mặc định `session.dmScope=main`, Slack DM sẽ được gộp vào phiên chính của agent.
- Phiên channel: `agent:<agentId>:slack:channel:<channelId>`.
- Trả lời theo thread có thể tạo hậu tố phiên thread (`:thread:<threadTs>`) khi áp dụng.
- `channels.slack.thread.historyScope` mặc định là `thread`; `thread.inheritParent` mặc định là `false`.
- `channels.slack.thread.initialHistoryLimit` kiểm soát số lượng tin nhắn hiện có trong thread được tải khi một phiên thread mới bắt đầu (mặc định `20`; đặt `0` để tắt).

Điều khiển trả lời theo thread:

- `chat:write` (gửi/cập nhật/xóa tin nhắn qua `chat.postMessage`)
  [https://docs.slack.dev/reference/methods/chat.postMessage](https://docs.slack.dev/reference/methods/chat.postMessage)
- `im:write` (mở DM qua `conversations.open` cho DM người dùng)
  [https://docs.slack.dev/reference/methods/conversations.open](https://docs.slack.dev/reference/methods/conversations.open)
- `channels:history`, `groups:history`, `im:history`, `mpim:history`
  [https://docs.slack.dev/reference/methods/conversations.history](https://docs.slack.dev/reference/methods/conversations.history)

Thẻ trả lời thủ công được hỗ trợ:

- `[[reply_to_current]]`
- `[[reply_to:<id>]]`

Lưu ý: `replyToMode="off"` sẽ vô hiệu hóa cơ chế tự động trả lời theo thread. Các thẻ `[[reply_to_*]]` tường minh vẫn được tôn trọng.

## Chưa cần hiện tại (nhưng có thể trong tương lai)

<AccordionGroup>
  <Accordion title="Inbound attachments">
    Tệp đính kèm Slack được tải xuống từ các URL riêng do Slack lưu trữ (luồng yêu cầu xác thực bằng token) và được ghi vào kho media khi tải thành công và cho phép theo giới hạn kích thước.

    ```
    Giới hạn kích thước inbound khi chạy mặc định là `20MB` trừ khi được ghi đè bởi `channels.slack.mediaMaxMb`.
    ```

  
</Accordion>

  <Accordion title="Outbound text and files">
    - các đoạn văn bản sử dụng `channels.slack.textChunkLimit` (mặc định 4000)
    - `channels.slack.chunkMode="newline"` bật cơ chế tách ưu tiên theo đoạn
    - gửi tệp sử dụng Slack upload APIs và có thể bao gồm trả lời theo thread (`thread_ts`)
    - giới hạn media outbound tuân theo `channels.slack.mediaMaxMb` khi được cấu hình; nếu không, việc gửi qua channel sẽ dùng mặc định theo loại MIME từ media pipeline
  
</Accordion>

  <Accordion title="Delivery targets">
    Mục tiêu tường minh được khuyến nghị:

    ```
    - `user:<id>` cho DM
    - `channel:<id>` cho channel
    
    Slack DM được mở thông qua Slack conversation APIs khi gửi tới mục tiêu user.
    ```

  
</Accordion>
</AccordionGroup>

## Giới hạn

Các hành động Slack được kiểm soát bởi `channels.slack.actions.*`.

Các nhóm hành động khả dụng trong bộ công cụ Slack hiện tại:

| Nhóm       | Mặc định |
| ---------- | -------- |
| messages   | bật      |
| reactions  | bật      |
| pins       | bật      |
| memberInfo | bật      |
| emojiList  | bật      |

## Sự kiện và hành vi vận hành

- Chỉnh sửa/xóa tin nhắn/phát thread được ánh xạ thành các sự kiện hệ thống.
- Sự kiện thêm/xóa reaction được ánh xạ thành các sự kiện hệ thống.
- Sự kiện thành viên tham gia/rời khỏi, channel được tạo/đổi tên và thêm/xóa ghim được ánh xạ thành các sự kiện hệ thống.
- `channel_id_changed` có thể di chuyển các khóa cấu hình channel khi `configWrites` được bật.
- Metadata topic/purpose của channel được xem là ngữ cảnh không đáng tin cậy và có thể được đưa vào ngữ cảnh định tuyến.

## Thread theo từng loại chat

Bạn có thể cấu hình hành vi threading khác nhau cho từng loại chat bằng cách đặt `channels.slack.replyToModeByChatType`:

Thứ tự phân giải:

- `channels.slack.accounts.<accountId>.ackReaction`
- `channels.slack.ackReaction`
- `messages.ackReaction`
- emoji định danh của agent làm phương án dự phòng (`agents.list[].identity.emoji`, nếu không thì "👀")

Lưu ý:

- Slack yêu cầu shortcode (ví dụ: `"eyes"`).
- Sử dụng `""` để tắt reaction cho một channel hoặc tài khoản.

## Danh sách kiểm tra manifest và scope

<AccordionGroup>
  <Accordion title="Slack app manifest example">

```json
{
  "display_information": {
    "name": "OpenClaw",
    "description": "Slack connector cho OpenClaw"
  },
  "features": {
    "bot_user": {
      "display_name": "OpenClaw",
      "always_online": false
    },
    "app_home": {
      "messages_tab_enabled": true,
      "messages_tab_read_only_enabled": false
    },
    "slash_commands": [
      {
        "command": "/openclaw",
        "description": "Gửi tin nhắn tới OpenClaw",
        "should_escape": false
      }
    ]
  },
  "oauth_config": {
    "scopes": {
      "bot": [
        "chat:write",
        "channels:history",
        "channels:read",
        "groups:history",
        "im:history",
        "mpim:history",
        "users:read",
        "app_mentions:read",
        "reactions:read",
        "reactions:write",
        "pins:read",
        "pins:write",
        "emoji:read",
        "commands",
        "files:read",
        "files:write"
      ]
    }
  },
  "settings": {
    "socket_mode_enabled": true,
    "event_subscriptions": {
      "bot_events": [
        "app_mention",
        "message.channels",
        "message.groups",
        "message.im",
        "message.mpim",
        "reaction_added",
        "reaction_removed",
        "member_joined_channel",
        "member_left_channel",
        "channel_rename",
        "pin_added",
        "pin_removed"
      ]
    }
  }
}
```

  
</Accordion>

  <Accordion title="Optional user-token scopes (read operations)">
    Nếu bạn cấu hình `channels.slack.userToken`, các scope đọc điển hình là:

    ```
    - `channels:history`, `groups:history`, `im:history`, `mpim:history`
    - `channels:read`, `groups:read`, `im:read`, `mpim:read`
    - `users:read`
    - `reactions:read`
    - `pins:read`
    - `emoji:read`
    - `search:read` (nếu bạn phụ thuộc vào việc đọc kết quả tìm kiếm của Slack)
    ```

  
</Accordion>
</AccordionGroup>

## Xử lý sự cố

<AccordionGroup>
  <Accordion title="No replies in channels">
    Kiểm tra theo thứ tự:

    ```
    - `groupPolicy`
    - danh sách cho phép kênh (`channels.slack.channels`)
    - `requireMention`
    - danh sách cho phép `users` theo từng kênh
    
    Các lệnh hữu ích:
    ```

```bash
openclaw channels status --probe
openclaw logs --follow
openclaw doctor
```

  
</Accordion>

  <Accordion title="DM messages ignored">
    Kiểm tra:

    ```
    - `channels.slack.dm.enabled`
    - `channels.slack.dmPolicy` (hoặc bản cũ `channels.slack.dm.policy`)
    - phê duyệt ghép nối / các mục trong danh sách cho phép
    ```

```bash
openclaw pairing list slack
```

  
</Accordion>

  <Accordion title="Socket mode not connecting">
    Xác thực bot token + app token và việc bật Socket Mode trong cài đặt ứng dụng Slack.
  
</Accordion>

  <Accordion title="HTTP mode not receiving events">
    Xác thực:

    ```
    - signing secret
    - đường dẫn webhook
    - Slack Request URLs (Events + Interactivity + Slash Commands)
    - `webhookPath` duy nhất cho mỗi tài khoản HTTP
    ```

  
</Accordion>

  <Accordion title="Native/slash commands not firing">
    Xác minh xem bạn có chủ đích:

    ```
    - chế độ lệnh native (`channels.slack.commands.native: true`) với các slash command tương ứng đã đăng ký trong Slack
    - hoặc chế độ một slash command duy nhất (`channels.slack.slashCommand.enabled: true`)
    
    Ngoài ra, kiểm tra `commands.useAccessGroups` và danh sách cho phép kênh/người dùng.
    ```

  
</Accordion>
</AccordionGroup>

## Hành động công cụ

Hành động công cụ Slack có thể bị giới hạn bằng `channels.slack.actions.*`:

- [Configuration reference - Slack](/gateway/configuration-reference#slack)

  Các trường Slack quan trọng:

  - mode/auth: `mode`, `botToken`, `appToken`, `signingSecret`, `webhookPath`, `accounts.*`
  - truy cập DM: `dm.enabled`, `dmPolicy`, `allowFrom` (bản cũ: `dm.policy`, `dm.allowFrom`), `dm.groupEnabled`, `dm.groupChannels`
  - truy cập kênh: `groupPolicy`, `channels.*`, `channels.*.users`, `channels.*.requireMention`
  - luồng/thông tin lịch sử: `replyToMode`, `replyToModeByChatType`, `thread.*`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`
  - phân phối: `textChunkLimit`, `chunkMode`, `mediaMaxMb`
  - vận hành/tính năng: `configWrites`, `commands.native`, `slashCommand.*`, `actions.*`, `userToken`, `userTokenReadOnly`

## Ghi chú bảo mật

- Thao tác ghi mặc định dùng bot token để các hành động thay đổi trạng thái
  được giới hạn trong quyền và danh tính của bot.
- [Channel routing](/channels/channel-routing)
- Nếu bật ghi bằng user token, hãy đảm bảo user token có các scope ghi
  tương ứng (`chat:write`, `reactions:write`, `pins:write`,
  `files:write`) nếu không các thao tác đó sẽ thất bại.
- [Configuration](/gateway/configuration)
- [Slash commands](/tools/slash-commands)
