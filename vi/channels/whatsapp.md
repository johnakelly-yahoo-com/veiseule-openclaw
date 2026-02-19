---
summary: "Tích hợp WhatsApp (kênh web): đăng nhập, hộp thư, trả lời, media và vận hành"
read_when:
  - Làm việc với hành vi kênh WhatsApp/web hoặc định tuyến hộp thư
title: "WhatsApp"
---

# WhatsApp (kênh web)

Trạng thái: sẵn sàng cho production qua WhatsApp Web (Baileys). Gateway quản lý (các) phiên đã liên kết.

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Chính sách DM mặc định là ghép đôi đối với người gửi chưa xác định.
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    Chẩn đoán liên kênh và hướng dẫn khắc phục.
  
</Card>
  <Card title="Gateway configuration" icon="settings" href="/gateway/configuration">
    Mẫu và ví dụ cấu hình kênh đầy đủ.
  
</Card>
</CardGroup>

## Thiết lập nhanh

<Steps>
  <Step title="Configure WhatsApp access policy">

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551234567"],
    },
  },
}
```

  
</Step>

  <Step title="Link WhatsApp (QR)">

```bash
openclaw channels login --channel whatsapp
```

    ```
    Cho một tài khoản cụ thể:
    ```

```bash
openclaw channels login --channel whatsapp --account work
```

  
</Step>

  <Step title="Start the gateway">

```bash
openclaw gateway
```

  
</Step>

  <Step title="Approve first pairing request (if using pairing mode)">

```bash
openclaw pairing list whatsapp
openclaw pairing approve whatsapp <CODE>
```

    ```
    Yêu cầu ghép đôi sẽ hết hạn sau 1 giờ. Số yêu cầu đang chờ được giới hạn tối đa 3 mỗi kênh.
    ```

  
</Step>
</Steps>

<Note>
OpenClaw khuyến nghị chạy WhatsApp bằng một số điện thoại riêng khi có thể. (Metadata của kênh và quy trình onboarding được tối ưu cho thiết lập đó, nhưng vẫn hỗ trợ sử dụng số cá nhân.)
</Note>

## Các mô hình triển khai

<AccordionGroup>
  <Accordion title="Dedicated number (recommended)">
    Đây là chế độ vận hành gọn gàng nhất:
  

    ````
    - danh tính WhatsApp riêng biệt cho OpenClaw
    - ranh giới định tuyến và allowlist DM rõ ràng hơn
    - giảm khả năng nhầm lẫn khi tự nhắn tin
    
    Mẫu chính sách tối thiểu:
    
    ```json5
    {
      channels: {
        whatsapp: {
          dmPolicy: "allowlist",
          allowFrom: ["+15551234567"],
        },
      },
    }
    ```
    ````

  
</Accordion>

  <Accordion title="Personal-number fallback">Onboarding hỗ trợ chế độ dùng số cá nhân và ghi cấu hình cơ sở thân thiện với tự nhắn tin:

    ```
    {
      "whatsapp": {
        "selfChatMode": true,
        "dmPolicy": "allowlist",
        "allowFrom": ["+15551234567"]
      }
    }
    ```

  
</Accordion>

  <Accordion title="WhatsApp Web-only channel scope">Kênh nền tảng nhắn tin sử dụng WhatsApp Web (`Baileys`) trong kiến trúc kênh hiện tại của OpenClaw.

    ```
    Không có kênh nhắn tin WhatsApp Twilio riêng biệt trong registry chat-channel tích hợp sẵn.
    ```

  
</Accordion>
</AccordionGroup>

## Mô hình runtime

- Gateway sở hữu socket WhatsApp và vòng lặp kết nối lại.
- Các tin nhắn gửi đi yêu cầu có listener WhatsApp đang hoạt động cho tài khoản đích.
- Các cuộc trò chuyện trạng thái và broadcast bị bỏ qua (`@status`, `@broadcast`).
- Chat trực tiếp sử dụng quy tắc phiên DM (`session.dmScope`; mặc định `main` sẽ gộp DM vào phiên chính của agent).
- Phiên nhóm được tách biệt (`agent:<agentId>:whatsapp:group:<jid>`).

## Kiểm soát truy cập và kích hoạt

<Tabs>
  <Tab title="DM policy">`channels.whatsapp.dmPolicy` kiểm soát quyền truy cập chat trực tiếp:

    ```
    - `pairing` (mặc định)
    - `allowlist`
    - `open` (yêu cầu `allowFrom` bao gồm `"*"`)
    - `disabled`
    
    `allowFrom` chấp nhận số theo chuẩn E.164 (được chuẩn hóa nội bộ).
    
    Ghi đè đa tài khoản: `channels.whatsapp.accounts.<id>.dmPolicy` (và `allowFrom`) sẽ được ưu tiên hơn cấu hình mặc định cấp kênh cho tài khoản đó.
    
    Chi tiết hành vi runtime:
    
    - pairing được lưu trong allow-store của kênh và được hợp nhất với `allowFrom` đã cấu hình
    - nếu không có allowlist được cấu hình, số tự liên kết sẽ được cho phép theo mặc định
    - DM gửi đi `fromMe` sẽ không bao giờ tự động được pairing
    ```

  
</Tab>

  <Tab title="Group policy + allowlists">Truy cập nhóm có hai lớp:

    ```
    1. **Allowlist thành viên nhóm** (`channels.whatsapp.groups`)
       - nếu bỏ qua `groups`, tất cả các nhóm đều hợp lệ
       - nếu có `groups`, nó hoạt động như một allowlist nhóm (`"*"` được chấp nhận)
    
    2. **Chính sách người gửi trong nhóm** (`channels.whatsapp.groupPolicy` + `groupAllowFrom`)
       - `open`: bỏ qua allowlist người gửi
       - `allowlist`: người gửi phải khớp `groupAllowFrom` (hoặc `*`)
       - `disabled`: chặn toàn bộ inbound từ nhóm
    
    Cơ chế dự phòng allowlist người gửi:
    
    - nếu `groupAllowFrom` chưa được thiết lập, runtime sẽ dùng `allowFrom` khi có sẵn
    
    Lưu ý: nếu hoàn toàn không có block `channels.whatsapp`, cơ chế dự phòng group-policy ở runtime sẽ mặc định là `open`.
    ```

  
</Tab>

  <Tab title="Mentions + /activation">Phản hồi trong nhóm mặc định yêu cầu được mention.

    ```
    Phát hiện mention bao gồm:
    
    - mention WhatsApp rõ ràng tới danh tính của bot
    - các regex mention đã cấu hình (`agents.list[].groupChat.mentionPatterns`, dự phòng `messages.groupChat.mentionPatterns`)
    - phát hiện ngầm khi trả lời bot (người gửi reply trùng với danh tính bot)
    
    Lệnh kích hoạt ở cấp phiên:
    
    - `/activation mention`
    - `/activation always`
    
    `activation` cập nhật trạng thái phiên (không phải cấu hình toàn cục). Yêu cầu quyền owner.
    ```

  
</Tab>
</Tabs>

## Hành vi số cá nhân và tự nhắn tin

Tắt toàn cục:

- bỏ qua read receipts cho các lượt tự nhắn tin
- bỏ qua hành vi tự kích hoạt bằng mention-JID vốn có thể tự ping chính bạn
- nếu `messages.responsePrefix` chưa được thiết lập, phản hồi tự nhắn tin mặc định sẽ là `[{identity.name}]` hoặc `[openclaw]`

## Chuẩn hóa tin nhắn và ngữ cảnh

<AccordionGroup>
  <Accordion title="Inbound envelope + reply context">Tin nhắn WhatsApp inbound được bọc trong inbound envelope dùng chung.

    ````
    Nếu có reply trích dẫn, ngữ cảnh sẽ được thêm vào theo dạng sau:
    
    ```text
    [Replying to <sender> id:<stanzaId>]
    <quoted body or media placeholder>
    [/Replying]
    ```
    
    Các trường metadata reply cũng được điền khi có sẵn (`ReplyToId`, `ReplyToBody`, `ReplyToSender`, sender JID/E.164).
    ````

  
</Accordion>

  <Accordion title="Media placeholders and location/contact extraction">Tin nhắn inbound chỉ có media được chuẩn hóa bằng các placeholder như:

    ```
    - `<media:image>`
    - `<media:video>`
    - `<media:audio>`
    - `<media:document>`
    - `<media:sticker>`
    
    Payload vị trí và danh bạ được chuẩn hóa thành ngữ cảnh văn bản trước khi định tuyến.
    ```

  
</Accordion>

  <Accordion title="Pending group history injection">Đối với nhóm, các tin nhắn chưa được xử lý có thể được đệm và chèn vào làm ngữ cảnh khi bot được kích hoạt.

    ```
    - giới hạn mặc định: `50`
    - cấu hình: `channels.whatsapp.historyLimit`
    - dự phòng: `messages.groupChat.historyLimit`
    - `0` để tắt
    
    Các marker chèn:
    
    - `[Chat messages since your last reply - for context]`
    - `[Current message - respond to this]`
    ```

  
</Accordion>

  <Accordion title="Read receipts">Read receipts được bật mặc định cho các tin nhắn WhatsApp inbound được chấp nhận.

    ````
    Tắt toàn cục:
    
    ```json5
    {
      channels: {
        whatsapp: {
          sendReadReceipts: false,
        },
      },
    }
    ```
    
    Ghi đè theo tài khoản:
    
    ```json5
    {
      channels: {
        whatsapp: {
          accounts: {
            work: {
              sendReadReceipts: false,
            },
          },
        },
      },
    }
    ```
    
    Các lượt tự nhắn tin sẽ bỏ qua read receipts ngay cả khi bật toàn cục.
    ````

  
</Accordion>
</AccordionGroup>

## Phân phối, chia đoạn và media

<AccordionGroup>
  <Accordion title="Text chunking">- giới hạn chia đoạn mặc định: `channels.whatsapp.textChunkLimit = 4000`
- `channels.whatsapp.chunkMode = "length" | "newline"`
- chế độ `newline` ưu tiên ranh giới đoạn văn (dòng trống), sau đó mới dự phòng chia đoạn an toàn theo độ dài
</Accordion>

  <Accordion title="Outbound media behavior">- hỗ trợ payload image, video, audio (voice-note PTT) và document
- `audio/ogg` được chuyển thành `audio/ogg; codecs=opus` để tương thích voice-note
- hỗ trợ phát GIF động qua `gifPlayback: true` khi gửi video
- caption được áp dụng cho mục media đầu tiên khi gửi phản hồi nhiều media
- nguồn media có thể là HTTP(S), `file://`, hoặc đường dẫn cục bộ
</Accordion>

  <Accordion title="Media size limits and fallback behavior">- giới hạn lưu media inbound: `channels.whatsapp.mediaMaxMb` (mặc định `50`)
- giới hạn media outbound cho auto-replies: `agents.defaults.mediaMaxMb` (mặc định `5MB`)
- image được tự động tối ưu (resize/điều chỉnh chất lượng) để phù hợp giới hạn
- khi gửi media thất bại, cơ chế dự phòng cho mục đầu tiên sẽ gửi cảnh báo dạng text thay vì im lặng bỏ qua phản hồi
</Accordion>
</AccordionGroup>

## Phản ứng xác nhận

**Cấu hình:**

```json5
{
  "whatsapp": {
    "ackReaction": {
      "emoji": "👀",
      "direct": true,
      "group": "mentions"
    }
  }
}
```

**Tùy chọn:**

- được gửi ngay sau khi inbound được chấp nhận (trước khi trả lời)
- `direct` (boolean, mặc định: `true`): Gửi phản ứng trong chat trực tiếp/DM.
- `group` (string, mặc định: `"mentions"`): Hành vi chat nhóm:
- WhatsApp sử dụng `channels.whatsapp.ackReaction` (không dùng `messages.ackReaction` cũ ở đây)

## Nhiều tài khoản và thông tin xác thực

<AccordionGroup>
  <Accordion title="Account selection and defaults">
    - id tài khoản lấy từ `channels.whatsapp.accounts`
    - chọn tài khoản mặc định: `default` nếu có, nếu không thì lấy id tài khoản được cấu hình đầu tiên (đã sắp xếp)
    - id tài khoản được chuẩn hóa nội bộ để tra cứu
  
</Accordion>

  <Accordion title="Credential paths and legacy compatibility">
    - đường dẫn xác thực hiện tại: `~/.openclaw/credentials/whatsapp/<accountId>/creds.json`
    - tệp sao lưu: `creds.json.bak`
    - xác thực mặc định cũ trong `~/.openclaw/credentials/` vẫn được nhận diện/di chuyển cho các luồng tài khoản mặc định
  
</Accordion>

  <Accordion title="Logout behavior">
    `openclaw channels logout --channel whatsapp [--account <id>]` xóa trạng thái xác thực WhatsApp cho tài khoản đó.

    ```
    Trong các thư mục xác thực cũ, `oauth.json` được giữ nguyên trong khi các tệp xác thực Baileys sẽ bị xóa.
    ```

  
</Accordion>
</AccordionGroup>

## Giới hạn

- Văn bản gửi ra được chia khối tới `channels.whatsapp.textChunkLimit` (mặc định 4000).
- Chia theo dòng trống tùy chọn: đặt `channels.whatsapp.chunkMode="newline"` để tách theo dòng trống (ranh giới đoạn) trước khi chia theo độ dài.
  - `channels.whatsapp.actions.reactions`
  - `channels.whatsapp.actions.polls`
- Lưu media đến bị giới hạn bởi `channels.whatsapp.mediaMaxMb` (mặc định 50 MB).

## Gửi ra (văn bản + media)

<AccordionGroup>
  <Accordion title="Not linked (QR required)">
    Triệu chứng: trạng thái kênh báo chưa được liên kết.


    ````
    Cách khắc phục:
    
    ```bash
    openclaw channels login --channel whatsapp
    openclaw channels status
    ```
    ````

  
</Accordion>

  <Accordion title="Linked but disconnected / reconnect loop">
    Triệu chứng: tài khoản đã liên kết nhưng liên tục bị ngắt kết nối hoặc cố gắng kết nối lại.


    ````
    Cách khắc phục:
    
    ```bash
    openclaw doctor
    openclaw logs --follow
    ```
    
    Nếu cần, liên kết lại bằng `channels login`.
    ````

  
</Accordion>

  <Accordion title="No active listener when sending">
    Các lần gửi đi sẽ thất bại ngay khi không có gateway listener đang hoạt động cho tài khoản mục tiêu.


    ```
    Đảm bảo gateway đang chạy và tài khoản đã được liên kết.
    ```

  
</Accordion>

  <Accordion title="Group messages unexpectedly ignored">
    Kiểm tra theo thứ tự sau:


    ```
    - `groupPolicy`
    - `groupAllowFrom` / `allowFrom`
    - các mục allowlist trong `groups`
    - kiểm soát mention (`requireMention` + các mẫu mention)
    ```

  
</Accordion>

  <Accordion title="Bun runtime warning">
    Runtime của WhatsApp gateway nên sử dụng Node. Bun được đánh dấu là không tương thích để vận hành WhatsApp/Telegram gateway ổn định.
  
</Accordion>
</AccordionGroup>

## Tham chiếu cấu hình

Tham chiếu chính:

- [Configuration reference - WhatsApp](/gateway/configuration-reference#whatsapp)

Các trường WhatsApp quan trọng:

- truy cập: `dmPolicy`, `allowFrom`, `groupPolicy`, `groupAllowFrom`, `groups`
- phân phối: `textChunkLimit`, `chunkMode`, `mediaMaxMb`, `sendReadReceipts`, `ackReaction`
- đa tài khoản: `accounts.<id>.enabled`, `accounts.<id>.authDir`, ghi đè ở cấp tài khoản
- vận hành: `configWrites`, `debounceMs`, `web.enabled`, `web.heartbeatSeconds`, `web.reconnect.*`
- hành vi phiên: `session.dmScope`, `historyLimit`, `dmHistoryLimit`, `dms.<id>.historyLimit`

## Liên quan

- [Pairing](/channels/pairing)
- [Channel routing](/channels/channel-routing)
- [Troubleshooting](/channels/troubleshooting)
