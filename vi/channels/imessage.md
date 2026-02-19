---
summary: "Legacy iMessage support via imsg (JSON-RPC over stdio). New setups should use BlueBubbles."
read_when:
  - Thiết lập hỗ trợ iMessage
  - Gỡ lỗi gửi/nhận iMessage
title: "iMessage"
---

# iMessage (legacy: imsg)

<Warning>
Đối với triển khai iMessage mới, hãy sử dụng <a href="/channels/bluebubbles">BlueBubbles</a>.

Tích hợp `imsg` là phiên bản cũ và có thể bị loại bỏ trong bản phát hành tương lai. 
</Warning>

Trạng thái: tích hợp CLI bên ngoài dạng legacy. Gateway khởi chạy `imsg rpc` và giao tiếp qua JSON-RPC trên stdio (không có daemon/cổng riêng).

<CardGroup cols={3}>
  <Card title="BlueBubbles (recommended)" icon="message-circle" href="/channels/bluebubbles">
    Đường dẫn iMessage được khuyến nghị cho thiết lập mới.
  
</Card>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Tin nhắn trực tiếp iMessage mặc định ở chế độ ghép nối.
  
</Card>
  <Card title="Configuration reference" icon="settings" href="/gateway/configuration-reference#imessage">
    Tham chiếu đầy đủ các trường iMessage.
  
</Card>
</CardGroup>

## Thiết lập nhanh

<Tabs>
  <Tab title="Local Mac (fast path)">
    <Steps>
      <Step title="Install and verify imsg">

```bash
brew install steipete/tap/imsg
imsg rpc --help
```

        
</Step>
      
        <Step title="Cấu hình OpenClaw">

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "/usr/local/bin/imsg",
      dbPath: "/Users/<you>/Library/Messages/chat.db",
    },
  },
}
```

        
</Step>
      
        <Step title="Khởi động gateway">

```bash
openclaw gateway
```

      {
        channels: { imessage: { configWrites: false } },
      }

```bash
openclaw pairing list imessage
openclaw pairing approve imessage <CODE>
```

        ```
            Yêu cầu ghép nối sẽ hết hạn sau 1 giờ.
          
</Step>
        
</Steps>
        ```

  
</Tab>

  <Tab title="Remote Mac over SSH">
    OpenClaw chỉ yêu cầu `cliPath` tương thích stdio, vì vậy bạn có thể trỏ `cliPath` đến một script wrapper SSH vào máy Mac từ xa và chạy `imsg`.

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

    ```
    Cấu hình được khuyến nghị khi bật tệp đính kèm:
    ```

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "~/.openclaw/scripts/imsg-ssh",
      remoteHost: "user@gateway-host", // dùng cho việc lấy tệp đính kèm qua SCP
      includeAttachments: true,
    },
  },
}
```

    ```
    Nếu không đặt `remoteHost`, OpenClaw sẽ cố gắng tự động phát hiện bằng cách phân tích script wrapper SSH.
    ```

  
</Tab>
</Tabs>

## Yêu cầu và quyền (macOS)

- Messages phải được đăng nhập trên máy Mac đang chạy `imsg`.
- Cần cấp Full Disk Access cho ngữ cảnh tiến trình đang chạy OpenClaw/`imsg` (truy cập cơ sở dữ liệu Messages).
- Cần có quyền Automation để gửi tin nhắn qua Messages.app.

<Tip>
Quyền được cấp theo từng ngữ cảnh tiến trình. Nếu gateway chạy ở chế độ headless (LaunchAgent/SSH), hãy chạy một lệnh tương tác một lần trong cùng ngữ cảnh đó để kích hoạt các hộp thoại yêu cầu quyền:

```bash
imsg chats --limit 1
# or
imsg send <handle> "test"
```

</Tip>

## Kiểm soát truy cập và định tuyến

<Tabs>
  <Tab title="DM policy">
    `channels.imessage.dmPolicy` kiểm soát tin nhắn trực tiếp (DM):

    ```
    - `pairing` (mặc định)
    - `allowlist`
    - `open` (yêu cầu `allowFrom` bao gồm `"*"`)
    - `disabled`
    
    Trường allowlist: `channels.imessage.allowFrom`.
    
    Các mục trong allowlist có thể là handle hoặc mục tiêu chat (`chat_id:*`, `chat_guid:*`, `chat_identifier:*`).
    ```

  
</Tab>

  <Tab title="Group policy + mentions">
    `channels.imessage.groupPolicy` kiểm soát xử lý nhóm:

    ```
    {
      channels: {
        imessage: {
          enabled: true,
          accounts: {
            bot: {
              name: "Bot",
              enabled: true,
              cliPath: "/path/to/imsg-bot",
              dbPath: "/Users/<bot-macos-user>/Library/Messages/chat.db",
            },
          },
        },
      },
    }
    ```

  
</Tab>

  <Tab title="Sessions and deterministic replies">
    - DM sử dụng định tuyến trực tiếp; nhóm sử dụng định tuyến nhóm.
    - Với mặc định `session.dmScope=main`, iMessage DM sẽ gộp vào phiên chính của agent.
    - Phiên nhóm được tách biệt (`agent:<agentId>:imessage:group:<chat_id>`).
    - Phản hồi sẽ được định tuyến lại về iMessage bằng metadata kênh/mục tiêu ban đầu.

    ```
    Hành vi luồng giống nhóm:
    
    Một số luồng iMessage có nhiều người tham gia có thể đến với `is_group=false`.
    Nếu `chat_id` đó được cấu hình rõ ràng trong `channels.imessage.groups`, OpenClaw sẽ xử lý như lưu lượng nhóm (áp dụng kiểm soát nhóm + tách biệt phiên nhóm).
    ```

  
</Tab>
</Tabs>

## Mô hình triển khai

<AccordionGroup>
  <Accordion title="Dedicated bot macOS user (separate iMessage identity)">
    Sử dụng một Apple ID và người dùng macOS riêng để lưu lượng bot được tách biệt khỏi hồ sơ Messages cá nhân của bạn.

    ```
    {
      channels: {
        imessage: {
          cliPath: "~/imsg-ssh", // SSH wrapper to remote Mac
          remoteHost: "user@gateway-host", // for SCP file transfer
          includeAttachments: true,
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="Remote Mac over Tailscale (example)">
    Mô hình phổ biến:

    ```
    - gateway chạy trên Linux/VM
    - iMessage + `imsg` chạy trên máy Mac trong tailnet của bạn
    - wrapper `cliPath` sử dụng SSH để chạy `imsg`
    - `remoteHost` cho phép lấy tệp đính kèm qua SCP
    
    Ví dụ:
    ```

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "~/.openclaw/scripts/imsg-ssh",
      remoteHost: "bot@mac-mini.tailnet-1234.ts.net",
      includeAttachments: true,
      dbPath: "/Users/bot/Library/Messages/chat.db",
    },
  },
}
```

```bash
#!/usr/bin/env bash
exec ssh -T bot@mac-mini.tailnet-1234.ts.net imsg "$@"
```

    ```
    Sử dụng SSH keys để cả SSH và SCP đều không cần tương tác.
    ```

  
</Accordion>

  <Accordion title="Multi-account pattern">
    iMessage hỗ trợ cấu hình theo từng tài khoản trong `channels.imessage.accounts`.

    ```
    Mỗi tài khoản có thể ghi đè các trường như `cliPath`, `dbPath`, `allowFrom`, `groupPolicy`, `mediaMaxMb` và các thiết lập lịch sử.
    ```

  
</Accordion>
</AccordionGroup>

## Media, chia nhỏ và mục tiêu gửi

<AccordionGroup>
  <Accordion title="Attachments and media">
    - việc nhận tệp đính kèm đầu vào là tùy chọn: `channels.imessage.includeAttachments`
    - đường dẫn tệp đính kèm từ xa có thể được tải qua SCP khi thiết lập `remoteHost`
    - kích thước media gửi đi sử dụng `channels.imessage.mediaMaxMb` (mặc định 16 MB)
  
</Accordion>

  <Accordion title="Outbound chunking">
    - giới hạn độ dài văn bản: `channels.imessage.textChunkLimit` (mặc định 4000)
    - chế độ chia nhỏ: `channels.imessage.chunkMode`
      - `length` (mặc định)
      - `newline` (ưu tiên tách theo đoạn)
  
</Accordion>

  <Accordion title="Addressing formats">
    Mục tiêu tường minh được ưu tiên:

    ```
    - `chat_id:123` (khuyến nghị để định tuyến ổn định)
    - `chat_guid:...`
    - `chat_identifier:...`
    
    Mục tiêu theo handle cũng được hỗ trợ:
    
    - `imessage:+1555...`
    - `sms:+1555...`
    - `user@example.com`
    ```

```bash
imsg chats --limit 20
```

  
</Accordion>
</AccordionGroup>

## Cách hoạt động (hành vi)

iMessage mặc định cho phép ghi cấu hình do kênh khởi tạo (cho `/config set|unset` khi `commands.config: true`).

Vô hiệu hóa:

```json5
{
  channels: {
    imessage: {
      configWrites: false,
    },
  },
}
```

## Khắc phục sự cố

<AccordionGroup>
  <Accordion title="imsg not found or RPC unsupported">
    Xác thực binary và hỗ trợ RPC:

```bash
imsg rpc --help
openclaw channels status --probe
```

    ```
    {
      channels: {
        imessage: {
          groupPolicy: "allowlist",
          groupAllowFrom: ["+15555550123"],
          groups: {
            "42": { requireMention: false },
          },
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="DMs are ignored">
    Kiểm tra:

    ```
    - `channels.imessage.dmPolicy`
    - `channels.imessage.allowFrom`
    - phê duyệt pairing (`openclaw pairing list imessage`)
    ```

  
</Accordion>

  <Accordion title="Group messages are ignored">
    Kiểm tra:

    ```
    - `channels.imessage.groupPolicy`
    - `channels.imessage.groupAllowFrom`
    - hành vi allowlist của `channels.imessage.groups`
    - cấu hình mẫu mention (`agents.list[].groupChat.mentionPatterns`)
    ```

  
</Accordion>

  <Accordion title="Remote attachments fail">
    Kiểm tra:

    ```
    - `channels.imessage.remoteHost`
    - Xác thực khóa SSH/SCP từ máy chủ gateway
    - Đảm bảo đường dẫn từ xa có thể đọc được trên máy Mac chạy Messages
    ```

  
</Accordion>

  <Accordion title="macOS permission prompts were missed">    Chạy lại trong một terminal GUI tương tác trong cùng ngữ cảnh người dùng/phiên và chấp thuận các yêu cầu:

```bash
imsg chats --limit 1
imsg send <handle> "test"
```

    ```
    Xác nhận rằng Full Disk Access + Automation đã được cấp cho ngữ cảnh tiến trình đang chạy OpenClaw/`imsg`.
    ```

  
</Accordion>
</AccordionGroup>

## Các tham chiếu cấu hình

- `agents.list[].groupChat.mentionPatterns` (hoặc `messages.groupChat.mentionPatterns`).
- `messages.responsePrefix`.
- [Pairing](/channels/pairing)
- [BlueBubbles](/channels/bluebubbles)

