---
summary: "Hỗ trợ Signal qua signal-cli (JSON-RPC + SSE), thiết lập và mô hình số"
read_when:
  - Thiết lập hỗ trợ Signal
  - Gỡ lỗi gửi/nhận Signal
title: "Signal"
---

# Signal (signal-cli)

Status: external CLI integration. Gateway talks to `signal-cli` over HTTP JSON-RPC + SSE.

## Thiết lập nhanh (cho người mới)

- Dùng **một số Signal riêng** cho bot (khuyến nghị).
- Cài đặt `signal-cli` (cần Java).
- Liên kết thiết bị bot và khởi động daemon:
- Cấu hình OpenClaw và khởi động gateway.

## Thiết lập nhanh (cho người mới)

1. Dùng **một số Signal riêng** cho bot (khuyến nghị).
2. Cài đặt `signal-cli` (yêu cầu Java nếu bạn sử dụng bản build JVM).
3. Chọn một trong các cách thiết lập:
   - **Cách A (liên kết QR):** `signal-cli link -n "OpenClaw"` và quét bằng Signal.
   - **Cách B (đăng ký SMS):** đăng ký một số chuyên dụng với captcha + xác minh SMS.
4. Cấu hình OpenClaw và khởi động lại gateway.
5. Gửi DM đầu tiên và chấp thuận ghép nối (`openclaw pairing approve signal <CODE>`).

Cấu hình tối thiểu:

```json5
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"],
    },
  },
}
```

Tham chiếu trường:

| Trường      | Mô tả                                                                                   |
| ----------- | --------------------------------------------------------------------------------------- |
| `account`   | Số điện thoại bot ở định dạng E.164 (`+15551234567`) |
| `cliPath`   | Đường dẫn tới `signal-cli` (`signal-cli` nếu có trong `PATH`)        |
| `dmPolicy`  | Chính sách truy cập DM (khuyến nghị `pairing`)                       |
| `allowFrom` | Số điện thoại hoặc giá trị `uuid:&lt;id&gt;` được phép gửi DM                                 |

## Nó là gì

- Kênh Signal qua `signal-cli` (không phải libsignal nhúng).
- Định tuyến xác định: phản hồi luôn quay lại Signal.
- DM dùng chung phiên chính của tác tử; nhóm được cô lập (`agent:<agentId>:signal:group:<groupId>`).

## Mô hình số (quan trọng)

Theo mặc định, Signal được phép ghi cập nhật cấu hình do `/config set|unset` kích hoạt (cần `commands.config: true`).

Tắt bằng:

```json5
{
  channels: { signal: { configWrites: false } },
}
```

## Mô hình số (quan trọng)

- Gateway kết nối tới **một thiết bị Signal** (tài khoản `signal-cli`).
- Nếu chạy bot trên **tài khoản Signal cá nhân của bạn**, nó sẽ bỏ qua tin nhắn của chính bạn (bảo vệ vòng lặp).
- Để có hành vi “tôi nhắn bot và nó trả lời”, hãy dùng **một số bot riêng**.

## Thiết lập cách A: liên kết tài khoản Signal hiện có (QR)

1. Cài đặt `signal-cli` (bản JVM hoặc bản native).
2. Liên kết một tài khoản bot:
   - `signal-cli link -n "OpenClaw"` rồi quét QR trong Signal.
3. Cấu hình Signal và khởi động gateway.

Nếu bạn muốn tự quản lý `signal-cli` (khởi động JVM chậm, init container, hoặc CPU dùng chung), hãy chạy daemon riêng và trỏ OpenClaw tới đó:

```json5
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"],
    },
  },
}
```

Multi-account support: use `channels.signal.accounts` with per-account config and optional `name`. See [`gateway/configuration`](/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) for the shared pattern.

## Kiểm soát truy cập (DM + nhóm)

DM:

1. Mặc định: `channels.signal.dmPolicy = "pairing"`.
   - Sử dụng một số bot chuyên dụng để tránh xung đột tài khoản/phiên.
2. Người gửi chưa biết sẽ nhận mã ghép cặp; tin nhắn bị bỏ qua cho đến khi được duyệt (mã hết hạn sau 1 giờ).

```bash
VERSION=$(curl -Ls -o /dev/null -w %{url_effective} https://github.com/AsamK/signal-cli/releases/latest | sed -e 's/^.*\/v//')
curl -L -O "https://github.com/AsamK/signal-cli/releases/download/v${VERSION}/signal-cli-${VERSION}-Linux-native.tar.gz"
sudo tar xf "signal-cli-${VERSION}-Linux-native.tar.gz" -C /opt
sudo ln -sf /opt/signal-cli /usr/local/bin/
signal-cli --version
```

Nếu bạn sử dụng bản JVM (`signal-cli-${VERSION}.tar.gz`), hãy cài đặt JRE 25+ trước.
Luôn cập nhật `signal-cli`; theo upstream, các bản phát hành cũ có thể ngừng hoạt động khi API máy chủ Signal thay đổi.

3. Đăng ký và xác minh số điện thoại:

```bash
signal-cli -a +<BOT_PHONE_NUMBER> register
```

Nếu yêu cầu captcha:

1. Văn bản gửi đi được chia khối theo `channels.signal.textChunkLimit` (mặc định 4000).
2. Tùy chọn chia theo dòng mới: đặt `channels.signal.chunkMode="newline"` để tách theo dòng trống (ranh giới đoạn) trước khi chia theo độ dài.
3. Hỗ trợ tệp đính kèm (base64 lấy từ `signal-cli`).
4. Giới hạn media mặc định: `channels.signal.mediaMaxMb` (mặc định 8).

```bash
signal-cli -a +<BOT_PHONE_NUMBER> register --captcha '<SIGNALCAPTCHA_URL>'
signal-cli -a +<BOT_PHONE_NUMBER> verify <VERIFICATION_CODE>
```

4. **Chỉ báo đang gõ**: OpenClaw gửi tín hiệu đang gõ qua `signal-cli sendTyping` và làm mới trong khi đang tạo phản hồi.

```bash
# Nếu bạn chạy gateway dưới dạng dịch vụ systemd của người dùng:
systemctl --user restart openclaw-gateway

# Sau đó xác minh:
openclaw doctor
openclaw channels status --probe
```

5. Dùng `message action=react` với `channel=signal`.
   - Gửi bất kỳ tin nhắn nào đến số bot.
   - Phê duyệt mã trên máy chủ: `openclaw pairing approve signal <PAIRING_CODE>`.
   - Lưu số bot vào danh bạ trên điện thoại để tránh hiển thị "Unknown contact".

Quan trọng: đăng ký một tài khoản số điện thoại bằng `signal-cli` có thể hủy xác thực phiên ứng dụng Signal chính cho số đó. Ưu tiên sử dụng một số bot chuyên dụng, hoặc dùng chế độ liên kết QR nếu bạn cần giữ nguyên thiết lập ứng dụng trên điện thoại hiện tại.

Tài liệu tham khảo upstream:

- `signal-cli` README: `https://github.com/AsamK/signal-cli`
- Luồng captcha: `https://github.com/AsamK/signal-cli/wiki/Registration-with-captcha`
- Luồng liên kết: `https://github.com/AsamK/signal-cli/wiki/Linking-other-devices-(Provisioning)`

## Chế độ daemon bên ngoài (httpUrl)

Nếu bạn muốn tự quản lý `signal-cli` (khởi động JVM chậm, init container, hoặc CPU dùng chung), hãy chạy daemon riêng và trỏ OpenClaw tới đó:

```json5
{
  channels: {
    signal: {
      httpUrl: "http://127.0.0.1:8080",
      autoStart: false,
    },
  },
}
```

This skips auto-spawn and the startup wait inside OpenClaw. For slow starts when auto-spawning, set `channels.signal.startupTimeoutMs`.

## Kiểm soát truy cập (DM + nhóm)

DM:

- Mặc định: `channels.signal.dmPolicy = "pairing"`.
- Người gửi chưa biết sẽ nhận mã ghép cặp; tin nhắn bị bỏ qua cho đến khi được duyệt (mã hết hạn sau 1 giờ).
- Duyệt qua:
  - `openclaw pairing list signal`
  - `openclaw pairing approve signal <CODE>`
- Pairing is the default token exchange for Signal DMs. Details: [Pairing](/channels/pairing)
- Người gửi chỉ có UUID (từ `sourceUuid`) được lưu dưới dạng `uuid:<id>` trong `channels.signal.allowFrom`.

Nhóm:

- `channels.signal.groupPolicy = open | allowlist | disabled`.
- `channels.signal.groupAllowFrom` kiểm soát ai có thể kích hoạt trong nhóm khi đặt `allowlist`.

## Cách hoạt động (hành vi)

- `signal-cli` chạy như một daemon; gateway đọc sự kiện qua SSE.
- Tin nhắn vào được chuẩn hóa vào phong bì kênh dùng chung.
- Phản hồi luôn được định tuyến về cùng số hoặc nhóm.

## Tham chiếu cấu hình (Signal)

- Văn bản gửi đi được chia khối theo `channels.signal.textChunkLimit` (mặc định 4000).
- Tùy chọn chia theo dòng mới: đặt `channels.signal.chunkMode="newline"` để tách theo dòng trống (ranh giới đoạn) trước khi chia theo độ dài.
- Hỗ trợ tệp đính kèm (base64 lấy từ `signal-cli`).
- Giới hạn media mặc định: `channels.signal.mediaMaxMb` (mặc định 8).
- Dùng `channels.signal.ignoreAttachments` để bỏ qua tải media.
- Group history context uses `channels.signal.historyLimit` (or `channels.signal.accounts.*.historyLimit`), falling back to `messages.groupChat.historyLimit`. Set `0` to disable (default 50).

## Đang gõ + biên nhận đã đọc

- `channels.signal.enabled`: bật/tắt khởi động kênh.
- `channels.signal.account`: E.164 cho tài khoản bot.
- `channels.signal.cliPath`: đường dẫn tới `signal-cli`.

## Phản ứng (công cụ tin nhắn)

- `agents.list[].groupChat.mentionPatterns` (Signal không hỗ trợ nhắc tên gốc).
- `messages.groupChat.mentionPatterns` (dự phòng toàn cục).
- `messageId` là dấu thời gian Signal của tin nhắn bạn đang phản ứng.
- Phản ứng trong nhóm cần `targetAuthor` hoặc `targetAuthorUuid`.

Ví dụ:

```
message action=react channel=signal target=uuid:123e4567-e89b-12d3-a456-426614174000 messageId=1737630212345 emoji=🔥
message action=react channel=signal target=+15551234567 messageId=1737630212345 emoji=🔥 remove=true
message action=react channel=signal target=signal:group:<groupId> targetAuthor=uuid:<sender-uuid> messageId=1737630212345 emoji=✅
```

Cấu hình:

- `channels.signal.actions.reactions`: bật/tắt hành động phản ứng (mặc định true).
- `channels.signal.reactionLevel`: `off | ack | minimal | extensive`.
  - `off`/`ack` tắt phản ứng của tác tử (công cụ tin nhắn `react` sẽ báo lỗi).
  - `minimal`/`extensive` bật phản ứng của tác tử và đặt mức hướng dẫn.
- Per-account overrides: `channels.signal.accounts.<id>.actions.reactions`, `channels.signal.accounts.<id>`.reactionLevel\`.

## Đích gửi (CLI/cron)

- DM: `signal:+15551234567` (hoặc E.164 trần).
- DM bằng UUID: `uuid:<id>` (hoặc UUID trần).
- Nhóm: `signal:group:<groupId>`.
- Tên người dùng: `username:<name>` (nếu tài khoản Signal của bạn hỗ trợ).

## Xử lý sự cố

Chạy thang kiểm tra này trước:

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

Sau đó xác nhận trạng thái ghép cặp DM nếu cần:

```bash
openclaw pairing list signal
```

Lỗi thường gặp:

- Daemon truy cập được nhưng không có phản hồi: kiểm tra cài đặt tài khoản/daemon (`httpUrl`, `account`) và chế độ nhận.
- DM bị bỏ qua: người gửi đang chờ duyệt ghép cặp.
- Tin nhắn nhóm bị bỏ qua: chặn do kiểm soát người gửi/nhắc tên trong nhóm.
- Lỗi xác thực cấu hình sau khi chỉnh sửa: chạy `openclaw doctor --fix`.
- Signal không xuất hiện trong chẩn đoán: xác nhận `channels.signal.enabled: true`.

Kiểm tra bổ sung:

```bash
openclaw pairing list signal
pgrep -af signal-cli
grep -i "signal" "/tmp/openclaw/openclaw-$(date +%Y-%m-%d).log" | tail -20
```

Luồng phân tích sự cố: [/channels/troubleshooting](/channels/troubleshooting).

## Lưu ý bảo mật

- `signal-cli` lưu trữ khóa tài khoản cục bộ (thường tại `~/.local/share/signal-cli/data/`).
- Sao lưu trạng thái tài khoản Signal trước khi di chuyển hoặc xây dựng lại máy chủ.
- Giữ `channels.signal.dmPolicy: "pairing"` trừ khi bạn thực sự muốn mở rộng quyền truy cập DM.
- Xác minh SMS chỉ cần cho quy trình đăng ký hoặc khôi phục, nhưng việc mất quyền kiểm soát số/tài khoản có thể làm phức tạp quá trình đăng ký lại.

## Tham chiếu cấu hình (Signal)

Cấu hình đầy đủ: [Configuration](/gateway/configuration)

Tùy chọn nhà cung cấp:

- `channels.signal.enabled`: bật/tắt khởi động kênh.
- `channels.signal.account`: E.164 cho tài khoản bot.
- `channels.signal.cliPath`: đường dẫn tới `signal-cli`.
- `channels.signal.httpUrl`: URL daemon đầy đủ (ghi đè host/port).
- `channels.signal.httpHost`, `channels.signal.httpPort`: bind daemon (mặc định 127.0.0.1:8080).
- `channels.signal.autoStart`: tự khởi chạy daemon (mặc định true nếu `httpUrl` chưa đặt).
- `channels.signal.startupTimeoutMs`: thời gian chờ khởi động tính bằng ms (giới hạn 120000).
- `channels.signal.receiveMode`: `on-start | manual`.
- `channels.signal.ignoreAttachments`: bỏ qua tải tệp đính kèm.
- `channels.signal.ignoreStories`: bỏ qua stories từ daemon.
- `channels.signal.sendReadReceipts`: chuyển tiếp biên nhận đã đọc.
- `channels.signal.dmPolicy`: `pairing | allowlist | open | disabled` (mặc định: ghép cặp).
- `channels.signal.allowFrom`: DM allowlist (E.164 or `uuid:<id>`). `open` yêu cầu `"*"`. Signal has no usernames; use phone/UUID ids.
- `channels.signal.groupPolicy`: `open | allowlist | disabled` (mặc định: danh sách cho phép).
- `channels.signal.groupAllowFrom`: danh sách cho phép người gửi trong nhóm.
- `channels.signal.historyLimit`: số tin nhắn nhóm tối đa để đưa vào ngữ cảnh (0 để tắt).
- `channels.signal.dmHistoryLimit`: giới hạn lịch sử DM theo lượt người dùng. Per-user overrides: `channels.signal.dms["<phone_or_uuid>"].historyLimit`.
- `channels.signal.textChunkLimit`: kích thước chia khối gửi đi (ký tự).
- `channels.signal.chunkMode`: `length` (mặc định) hoặc `newline` để tách theo dòng trống (ranh giới đoạn) trước khi chia theo độ dài.
- `channels.signal.mediaMaxMb`: giới hạn media vào/ra (MB).

Tùy chọn toàn cục liên quan:

- `agents.list[].groupChat.mentionPatterns` (Signal không hỗ trợ nhắc tên gốc).
- `messages.groupChat.mentionPatterns` (dự phòng toàn cục).
- `messages.responsePrefix`.

