---
title: "WhatsApp"
---

# WhatsApp (kênh web)

Status: WhatsApp Web via Baileys only. Gateway sở hữu (các) phiên làm việc.

## Khởi động nhanh (cho người mới)

1. Dùng **một số điện thoại riêng** nếu có thể (khuyến nghị).
2. Cấu hình WhatsApp trong `~/.openclaw/openclaw.json`.
3. Chạy `openclaw channels login` để quét mã QR (Thiết bị đã liên kết).
4. Khởi động gateway.

Cấu hình tối thiểu:

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

## Mục tiêu

- Nhiều tài khoản WhatsApp (đa tài khoản) trong một tiến trình Gateway.
- Định tuyến xác định: trả lời quay lại WhatsApp, không định tuyến theo mô hình.
- Mô hình có đủ ngữ cảnh để hiểu trả lời trích dẫn.

## Ghi cấu hình

Theo mặc định, WhatsApp được phép ghi cập nhật cấu hình do `/config set|unset` kích hoạt (yêu cầu `commands.config: true`).

Tắt bằng:

```json5
{
  channels: { whatsapp: { configWrites: false } },
}
```

## Kiến trúc (ai sở hữu cái gì)

- **Gateway** sở hữu socket Baileys và vòng lặp hộp thư.
- **CLI / ứng dụng macOS** giao tiếp với gateway; không dùng Baileys trực tiếp.
- **Active listener** là bắt buộc cho gửi ra; nếu không, gửi sẽ thất bại ngay.

## Lấy số điện thoại (hai chế độ)

WhatsApp yêu cầu một số điện thoại di động thật để xác minh. Các số VoIP và số ảo thường bị chặn. Có hai cách được hỗ trợ để chạy OpenClaw trên WhatsApp:

### Số chuyên dụng (khuyến nghị)

Sử dụng **một số điện thoại riêng** cho OpenClaw. Trải nghiệm người dùng (UX) tốt nhất, định tuyến rõ ràng, không có các vấn đề tự nhắn tin. Ideal setup: **spare/old Android phone + eSIM**. Để thiết bị kết nối Wi‑Fi và nguồn điện, rồi liên kết qua QR.

**WhatsApp Business:** Bạn có thể dùng WhatsApp Business trên cùng một thiết bị với một số khác. Great for keeping your personal WhatsApp separate — install WhatsApp Business and register the OpenClaw number there.

**Cấu hình mẫu (số chuyên dụng, allowlist một người dùng):**

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

**Chế độ ghép cặp (tùy chọn):**
Nếu bạn muốn ghép cặp thay vì allowlist, hãy đặt `channels.whatsapp.dmPolicy` thành `pairing`. Người gửi không xác định sẽ nhận được mã ghép cặp; phê duyệt bằng:
`openclaw pairing approve whatsapp <code>`

### Số cá nhân (dự phòng)

Giải pháp nhanh: chạy OpenClaw trên **chính số của bạn**. Message yourself (WhatsApp “Message yourself”) for testing so you don’t spam contacts. Dự kiến sẽ phải đọc mã xác minh trên điện thoại chính của bạn trong quá trình thiết lập và thử nghiệm. **Bắt buộc bật chế độ tự chat.**
Khi trình hướng dẫn hỏi số WhatsApp cá nhân của bạn, hãy nhập số điện thoại mà bạn sẽ nhắn từ đó (chủ sở hữu/người gửi), không phải số của trợ lý.

**Cấu hình mẫu (số cá nhân, self-chat):**

```json
{
  "whatsapp": {
    "selfChatMode": true,
    "dmPolicy": "allowlist",
    "allowFrom": ["+15551234567"]
  }
}
```

Self-chat replies default to `[{identity.name}]` when set (otherwise `[openclaw]`)
if `messages.responsePrefix` is unset. Set it explicitly to customize or disable
the prefix (use `""` to remove it).

### Mẹo nguồn số

- **eSIM nội địa** từ nhà mạng di động trong nước (đáng tin cậy nhất)
  - Áo: [hot.at](https://www.hot.at)
  - Anh: [giffgaff](https://www.giffgaff.com) — SIM miễn phí, không hợp đồng
- **SIM trả trước** — rẻ, chỉ cần nhận một SMS để xác minh

**Tránh:** TextNow, Google Voice, hầu hết dịch vụ “SMS miễn phí” — WhatsApp chặn rất gắt.

**Tip:** The number only needs to receive one verification SMS. Sau đó, các phiên WhatsApp Web sẽ được duy trì qua `creds.json`.

## Vì sao không dùng Twilio?

- Các bản OpenClaw sớm có hỗ trợ tích hợp WhatsApp Business của Twilio.
- Số WhatsApp Business không phù hợp cho trợ lý cá nhân.
- Meta áp dụng cửa sổ trả lời 24 giờ; nếu bạn chưa phản hồi trong 24 giờ qua, số doanh nghiệp không thể khởi tạo tin nhắn mới.
- Lưu lượng cao hoặc sử dụng “chatty” kích hoạt chặn mạnh, vì tài khoản doanh nghiệp không предназнач để gửi hàng chục tin nhắn trợ lý cá nhân.
- Kết quả: giao hàng không ổn định và bị chặn thường xuyên, nên đã loại bỏ hỗ trợ.

## Đăng nhập + thông tin xác thực

- Lệnh đăng nhập: `openclaw channels login` (QR qua Thiết bị đã liên kết).
- Đăng nhập đa tài khoản: `openclaw channels login --account <id>` (`<id>` = `accountId`).
- Tài khoản mặc định (khi bỏ `--account`): `default` nếu có, nếu không thì id tài khoản cấu hình đầu tiên (theo thứ tự).
- Thông tin xác thực lưu tại `~/.openclaw/credentials/whatsapp/<accountId>/creds.json`.
- Bản sao lưu tại `creds.json.bak` (khôi phục khi hỏng).
- Tương thích ngược: cài đặt cũ lưu tệp Baileys trực tiếp trong `~/.openclaw/credentials/`.
- Đăng xuất: `openclaw channels logout` (hoặc `--account <id>`) xóa trạng thái xác thực WhatsApp (nhưng giữ `oauth.json` dùng chung).
- Socket đã đăng xuất ⇒ lỗi hướng dẫn liên kết lại.

## Luồng vào (DM + nhóm)

- Sự kiện WhatsApp đến từ `messages.upsert` (Baileys).
- Listener hộp thư được tháo khi tắt để tránh tích lũy handler trong test/khởi động lại.
- Chat trạng thái/phát sóng bị bỏ qua.
- Chat trực tiếp dùng E.164; nhóm dùng JID nhóm.
- **Chính sách DM**: `channels.whatsapp.dmPolicy` kiểm soát truy cập chat trực tiếp (mặc định: `pairing`).
  - Ghép cặp: người gửi không xác định nhận mã ghép cặp (phê duyệt qua `openclaw pairing approve whatsapp <code>`; mã hết hạn sau 1 giờ).
  - Mở: yêu cầu `channels.whatsapp.allowFrom` bao gồm `"*"`.
  - Số WhatsApp đã liên kết của bạn được tin cậy ngầm, nên tin nhắn tự gửi bỏ qua kiểm tra `channels.whatsapp.dmPolicy` và `channels.whatsapp.allowFrom`.

### Chế độ số cá nhân (dự phòng)

Nếu bạn chạy OpenClaw trên **số WhatsApp cá nhân**, bật `channels.whatsapp.selfChatMode` (xem cấu hình mẫu ở trên).

Hành vi:

- DM gửi ra không bao giờ kích hoạt trả lời ghép cặp (tránh spam danh bạ).
- DM đến từ người lạ vẫn tuân theo `channels.whatsapp.dmPolicy`.
- Chế độ self-chat (allowFrom bao gồm số của bạn) tránh gửi read receipt tự động và bỏ qua JID mention.
- Gửi read receipt cho DM không phải self-chat.

## Read receipts

Theo mặc định, gateway đánh dấu đã đọc tin nhắn WhatsApp đến (dấu tích xanh) khi được chấp nhận.

Tắt toàn cục:

```json5
{
  channels: { whatsapp: { sendReadReceipts: false } },
}
```

Tắt theo tài khoản:

```json5
{
  channels: {
    whatsapp: {
      accounts: {
        personal: { sendReadReceipts: false },
      },
    },
  },
}
```

Ghi chú:

- Chế độ self-chat luôn bỏ qua read receipts.

## WhatsApp FAQ: gửi tin nhắn + ghép cặp

**Will OpenClaw message random contacts when I link WhatsApp?**  
No. Chính sách DM mặc định là **pairing**, vì vậy người gửi không xác định chỉ nhận được mã ghép cặp và tin nhắn của họ **không được xử lý**. OpenClaw chỉ trả lời các cuộc trò chuyện mà nó nhận được, hoặc các lần gửi mà bạn kích hoạt rõ ràng (agent/CLI).

**Ghép cặp hoạt động thế nào trên WhatsApp?**  
Ghép cặp là cổng DM cho người gửi không xác định:

- DM đầu tiên từ người gửi mới trả về một mã ngắn (tin nhắn không được xử lý).
- Phê duyệt bằng: `openclaw pairing approve whatsapp <code>` (liệt kê với `openclaw pairing list whatsapp`).
- Mã hết hạn sau 1 giờ; yêu cầu chờ xử lý bị giới hạn 3 mỗi kênh.

**Can multiple people use different OpenClaw instances on one WhatsApp number?**  
Yes, by routing each sender to a different agent via `bindings` (peer `kind: "direct"`, sender E.164 like `+15551234567`). Replies still come from the **same WhatsApp account**, and direct chats collapse to each agent's main session, so use **one agent per person**. DM access control (`dmPolicy`/`allowFrom`) is global per WhatsApp account. Xem [Multi-Agent Routing](/concepts/multi-agent).

**Vì sao bạn hỏi số điện thoại của tôi trong trình hướng dẫn?**  
Trình hướng dẫn dùng nó để thiết lập **allowlist/owner** để các DM của chính bạn được cho phép. Nó không được dùng cho việc gửi tự động. If you run on your personal WhatsApp number, use that same number and enable `channels.whatsapp.selfChatMode`.

## Chuẩn hóa tin nhắn (mô hình thấy gì)

- `Body` là nội dung tin nhắn hiện tại kèm phong bì.

- Ngữ cảnh trả lời trích dẫn **luôn được nối thêm**:

  ```
  [Replying to +1555 id:ABC123]
  <quoted text or <media:...>>
  [/Replying]
  ```

- Metadata trả lời cũng được đặt:
  - `ReplyToId` = stanzaId
  - `ReplyToBody` = nội dung trích dẫn hoặc placeholder media
  - `ReplyToSender` = E.164 khi biết

- Tin nhắn vào chỉ có media dùng placeholder:
  - `<media:image|video|audio|document|sticker>`

## Nhóm

- Nhóm ánh xạ tới phiên `agent:<agentId>:whatsapp:group:<jid>`.
- Chính sách nhóm: `channels.whatsapp.groupPolicy = open|disabled|allowlist` (mặc định `allowlist`).
- Chế độ kích hoạt:
  - `mention` (mặc định): yêu cầu @mention hoặc khớp regex.
  - `always`: luôn kích hoạt.
- `/activation mention|always` chỉ dành cho chủ sở hữu và phải gửi như một tin nhắn độc lập.
- Chủ sở hữu = `channels.whatsapp.allowFrom` (hoặc E.164 của self nếu chưa đặt).
- **Chèn lịch sử** (chỉ pending):
  - Các tin nhắn _chưa xử lý_ gần đây (mặc định 50) được chèn dưới:
    `[Chat messages since your last reply - for context]` (tin nhắn đã có trong phiên sẽ không được chèn lại)
  - Tin nhắn hiện tại dưới:
    `[Current message - respond to this]`
  - Hậu tố người gửi được nối thêm: `[from: Name (+E164)]`
- Metadata nhóm được cache 5 phút (chủ đề + người tham gia).

## Gửi trả lời (xâu chuỗi)

- WhatsApp Web gửi tin nhắn tiêu chuẩn (không có xâu chuỗi trả lời trích dẫn trong gateway hiện tại).
- Thẻ trả lời bị bỏ qua trên kênh này.

## Phản ứng xác nhận (tự động react khi nhận)

WhatsApp can automatically send emoji reactions to incoming messages immediately upon receipt, before the bot generates a reply. Điều này cung cấp phản hồi tức thì cho người dùng rằng tin nhắn của họ đã được nhận.

**Cấu hình:**

```json
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

- `emoji` (string): Emoji dùng để xác nhận (ví dụ: "👀", "✅", "📨"). Để trống hoặc bỏ qua = tính năng bị vô hiệu hóa.
- `direct` (boolean, mặc định: `true`): Gửi phản ứng trong chat trực tiếp/DM.
- `group` (string, mặc định: `"mentions"`): Hành vi chat nhóm:
  - `"always"`: React mọi tin nhắn nhóm (kể cả không @mention)
  - `"mentions"`: Chỉ react khi bot được @mention
  - `"never"`: Không bao giờ react trong nhóm

**Ghi đè theo tài khoản:**

```json
{
  "whatsapp": {
    "accounts": {
      "work": {
        "ackReaction": {
          "emoji": "✅",
          "direct": false,
          "group": "always"
        }
      }
    }
  }
}
```

**Ghi chú hành vi:**

- Phản ứng được gửi **ngay lập tức** khi nhận tin nhắn, trước chỉ báo đang gõ hoặc trả lời của bot.
- Trong nhóm với `requireMention: false` (kích hoạt: luôn), `group: "mentions"` sẽ react mọi tin nhắn (không chỉ @mention).
- Gửi kiểu fire-and-forget: lỗi phản ứng được ghi log nhưng không ngăn bot trả lời.
- JID người tham gia được tự động bao gồm cho phản ứng nhóm.
- WhatsApp bỏ qua `messages.ackReaction`; hãy dùng `channels.whatsapp.ackReaction` thay thế.

## Công cụ agent (phản ứng)

- Công cụ: `whatsapp` với hành động `react` (`chatJid`, `messageId`, `emoji`, tùy chọn `remove`).
- Tùy chọn: `participant` (người gửi nhóm), `fromMe` (react vào tin nhắn của chính bạn), `accountId` (đa tài khoản).
- Ngữ nghĩa gỡ phản ứng: xem [/tools/reactions](/tools/reactions).
- Chặn công cụ: `channels.whatsapp.actions.reactions` (mặc định: bật).

## Giới hạn

- Văn bản gửi ra được chia khối tới `channels.whatsapp.textChunkLimit` (mặc định 4000).
- Chia theo dòng trống tùy chọn: đặt `channels.whatsapp.chunkMode="newline"` để tách theo dòng trống (ranh giới đoạn) trước khi chia theo độ dài.
- Lưu media đến bị giới hạn bởi `channels.whatsapp.mediaMaxMb` (mặc định 50 MB).
- Media gửi ra bị giới hạn bởi `agents.defaults.mediaMaxMb` (mặc định 5 MB).

## Gửi ra (văn bản + media)

- Dùng web listener đang hoạt động; lỗi nếu gateway không chạy.
- Chia văn bản: tối đa 4k mỗi tin (cấu hình qua `channels.whatsapp.textChunkLimit`, tùy chọn `channels.whatsapp.chunkMode`).
- Media:
  - Hỗ trợ hình ảnh/video/âm thanh/tài liệu.
  - Âm thanh gửi dạng PTT; `audio/ogg` => `audio/ogg; codecs=opus`.
  - Chú thích chỉ ở media đầu tiên.
  - Lấy media hỗ trợ HTTP(S) và đường dẫn cục bộ.
  - GIF động: WhatsApp yêu cầu MP4 với `gifPlayback: true` để lặp nội tuyến.
    - CLI: `openclaw message send --media <mp4> --gif-playback`
    - Gateway: tham số `send` bao gồm `gifPlayback: true`

## Ghi âm giọng nói (âm thanh PTT)

WhatsApp gửi âm thanh dưới dạng **voice notes** (bong bóng PTT).

- Kết quả tốt nhất: OGG/Opus. OpenClaw rewrites `audio/ogg` to `audio/ogg; codecs=opus`.
- `[[audio_as_voice]]` bị bỏ qua cho WhatsApp (âm thanh đã là voice note).

## Giới hạn media + tối ưu

- Trần gửi ra mặc định: 5 MB (mỗi media).
- Ghi đè: `agents.defaults.mediaMaxMb`.
- Hình ảnh được tự động tối ưu thành JPEG dưới trần (đổi kích thước + quét chất lượng).
- Media quá cỡ ⇒ lỗi; trả lời media rơi về cảnh báo văn bản.

## Heartbeats

- **Gateway heartbeat** ghi log tình trạng kết nối (`web.heartbeatSeconds`, mặc định 60s).
- **Agent heartbeat** có thể cấu hình theo agent (`agents.list[].heartbeat`) hoặc toàn cục
  qua `agents.defaults.heartbeat` (dùng khi không có mục theo agent).
  - Sử dụng prompt heartbeat đã cấu hình (mặc định: `Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`) + `HEARTBEAT_OK` skip behavior.
  - Gửi mặc định tới kênh dùng gần nhất (hoặc đích cấu hình).

## Hành vi kết nối lại

- Chính sách backoff: `web.reconnect`:
  - `initialMs`, `maxMs`, `factor`, `jitter`, `maxAttempts`.
- Nếu đạt maxAttempts, giám sát web dừng (suy giảm).
- Đăng xuất ⇒ dừng và yêu cầu liên kết lại.

## Bản đồ cấu hình nhanh

- `channels.whatsapp.dmPolicy` (chính sách DM: pairing/allowlist/open/disabled).
- `channels.whatsapp.selfChatMode` (thiết lập cùng điện thoại; bot dùng số WhatsApp cá nhân của bạn).
- `channels.whatsapp.allowFrom` (DM allowlist). WhatsApp uses E.164 phone numbers (no usernames).
- `channels.whatsapp.mediaMaxMb` (trần lưu media đến).
- `channels.whatsapp.ackReaction` (tự phản ứng khi nhận tin: `{emoji, direct, group}`).
- `channels.whatsapp.accounts.<accountId>.*` (per-account settings + optional `authDir`).
- `channels.whatsapp.accounts.<accountId>.mediaMaxMb` (per-account inbound media cap).
- `channels.whatsapp.accounts.<accountId>.ackReaction` (per-account ack reaction override).
- `channels.whatsapp.groupAllowFrom` (allowlist người gửi nhóm).
- `channels.whatsapp.groupPolicy` (chính sách nhóm).
- `channels.whatsapp.historyLimit` / `channels.whatsapp.accounts.<accountId>.historyLimit` (group history context; `0` disables).
- `channels.whatsapp.dmHistoryLimit` (DM history limit in user turns). Per-user overrides: `channels.whatsapp.dms["<phone>"].historyLimit`.
- `channels.whatsapp.groups` (allowlist nhóm + mặc định chặn theo mention; dùng `"*"` để cho phép tất cả)
- `channels.whatsapp.actions.reactions` (chặn công cụ phản ứng WhatsApp).
- `agents.list[].groupChat.mentionPatterns` (hoặc `messages.groupChat.mentionPatterns`)
- `messages.groupChat.historyLimit`
- `channels.whatsapp.messagePrefix` (inbound prefix; per-account: `channels.whatsapp.accounts.<accountId>.messagePrefix`; deprecated: `messages.messagePrefix`)
- `messages.responsePrefix` (tiền tố đi)
- `agents.defaults.mediaMaxMb`
- `agents.defaults.heartbeat.every`
- `agents.defaults.heartbeat.model` (ghi đè tùy chọn)
- `agents.defaults.heartbeat.target`
- `agents.defaults.heartbeat.to`
- `agents.defaults.heartbeat.session`
- `agents.list[].heartbeat.*` (ghi đè theo agent)
- `session.*` (scope, idle, store, mainKey)
- `web.enabled` (tắt khởi động kênh khi false)
- `web.heartbeatSeconds`
- `web.reconnect.*`

## Log + xử lý sự cố

- Phân hệ: `whatsapp/inbound`, `whatsapp/outbound`, `web-heartbeat`, `web-reconnect`.
- Tệp log: `/tmp/openclaw/openclaw-YYYY-MM-DD.log` (có thể cấu hình).
- Hướng dẫn xử lý sự cố: [Gateway troubleshooting](/gateway/troubleshooting).

## Xử lý sự cố (nhanh)

**Chưa liên kết / cần đăng nhập QR**

- Triệu chứng: `channels status` hiển thị `linked: false` hoặc cảnh báo “Not linked”.
- Cách khắc phục: chạy `openclaw channels login` trên máy chủ gateway và quét QR (WhatsApp → Cài đặt → Thiết bị đã liên kết).

**Đã liên kết nhưng mất kết nối / vòng lặp kết nối lại**

- Triệu chứng: `channels status` hiển thị `running, disconnected` hoặc cảnh báo “Linked but disconnected”.
- Fix: `openclaw doctor` (or restart the gateway). Nếu vẫn tiếp diễn, hãy liên kết lại qua `channels login` và kiểm tra `openclaw logs --follow`.

**Runtime Bun**

- Bun is **not recommended**. WhatsApp (Baileys) and Telegram are unreliable on Bun.
  Chạy gateway bằng **Node**. (See Getting Started runtime note.)

