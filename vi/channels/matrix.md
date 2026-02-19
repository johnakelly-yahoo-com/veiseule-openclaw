---
summary: "Trạng thái hỗ trợ Matrix, khả năng và cấu hình"
read_when:
  - Làm việc với các tính năng kênh Matrix
title: "Matrix"
---

# Matrix (plugin)

32. Matrix là một giao thức nhắn tin mở, phi tập trung. 33. OpenClaw kết nối như một **người dùng** Matrix
    trên bất kỳ homeserver nào, vì vậy bạn cần một tài khoản Matrix cho bot. 34. Sau khi đăng nhập, bạn có thể DM
    bot trực tiếp hoặc mời nó vào các phòng ("groups" của Matrix). 35. Beeper cũng là một tùy chọn client hợp lệ,
    nhưng yêu cầu phải bật E2EE.

36. Trạng thái: được hỗ trợ thông qua plugin (@vector-im/matrix-bot-sdk). Tin nhắn trực tiếp, phòng, luồng, media, reaction,
    thăm dò ý kiến (gửi + poll-start dưới dạng văn bản), vị trí và E2EE (có hỗ trợ crypto).

## Yêu cầu plugin

Cài đặt qua CLI (npm registry):

Cài đặt qua CLI (npm registry):

```bash
openclaw plugins install @openclaw/matrix
```

Cài đặt từ bản checkout cục bộ (khi chạy từ repo git):

```bash
openclaw plugins install ./extensions/matrix
```

Chi tiết: [Plugins](/tools/plugin)

Chi tiết: [Plugins](/tools/plugin)

## Thiết lập

1. Cài đặt plugin Matrix:
   - Từ npm: `openclaw plugins install @openclaw/matrix`
   - Từ bản checkout cục bộ: `openclaw plugins install ./extensions/matrix`

2. Tạo một tài khoản Matrix trên homeserver:
   - Xem các tùy chọn lưu trữ tại [https://matrix.org/ecosystem/hosting/](https://matrix.org/ecosystem/hosting/)
   - Hoặc tự host.

3. Lấy access token cho tài khoản bot:

   - Dùng Matrix login API với `curl` tại homeserver của bạn:

   ```bash
   curl --request POST \
     --url https://matrix.example.org/_matrix/client/v3/login \
     --header 'Content-Type: application/json' \
     --data '{
     "type": "m.login.password",
     "identifier": {
       "type": "m.id.user",
       "user": "your-user-name"
     },
     "password": "your-password"
   }'
   ```

   - Thay `matrix.example.org` bằng URL homeserver của bạn.
   - Hoặc đặt `channels.matrix.userId` + `channels.matrix.password`: OpenClaw gọi cùng endpoint đăng nhập,
     lưu access token vào `~/.openclaw/credentials/matrix/credentials.json`,
     và tái sử dụng ở lần khởi động tiếp theo.

4. Cấu hình thông tin xác thực:
   - Env: `MATRIX_HOMESERVER`, `MATRIX_ACCESS_TOKEN` (hoặc `MATRIX_USER_ID` + `MATRIX_PASSWORD`)
   - Hoặc config: `channels.matrix.*`
   - Nếu cả hai cùng được đặt, config có ưu tiên cao hơn.
   - Khi dùng access token: user ID được lấy tự động qua `/whoami`.
   - Khi đặt, `channels.matrix.userId` phải là Matrix ID đầy đủ (ví dụ: `@bot:example.org`).

5. Khởi động lại gateway (hoặc hoàn tất onboarding).

6. 38. Bắt đầu DM với bot hoặc mời nó vào một phòng từ bất kỳ client Matrix nào
       (Element, Beeper, v.v.; xem [https://matrix.org/ecosystem/clients/](https://matrix.org/ecosystem/clients/)). 39. Beeper yêu cầu E2EE,
       vì vậy hãy đặt `channels.matrix.encryption: true` và xác minh thiết bị.

Cấu hình tối thiểu (access token, user ID tự động lấy):

```json5
{
  channels: {
    matrix: {
      enabled: true,
      homeserver: "https://matrix.example.org",
      accessToken: "syt_***",
      dm: { policy: "pairing" },
    },
  },
}
```

Cấu hình E2EE (bật mã hóa đầu-cuối):

```json5
{
  channels: {
    matrix: {
      enabled: true,
      homeserver: "https://matrix.example.org",
      accessToken: "syt_***",
      encryption: true,
      dm: { policy: "pairing" },
    },
  },
}
```

## Mã hóa (E2EE)

Bật bằng `channels.matrix.encryption: true`:

Bật bằng `channels.matrix.encryption: true`:

- Nếu mô-đun crypto tải thành công, các phòng được mã hóa sẽ tự động được giải mã.
- Nếu access token (thiết bị) thay đổi, một kho mới sẽ được tạo và bot phải được
  xác minh lại cho các phòng được mã hóa.
- **Xác minh thiết bị:**
  Khi E2EE được bật, bot sẽ yêu cầu xác minh từ các phiên khác của bạn khi khởi động.
- Xác minh thiết bị trong một client Matrix khác (Element, v.v.). 41. để bật chia sẻ khóa.
- Nếu mô-đun crypto không thể tải, E2EE sẽ bị tắt và các phòng được mã hóa sẽ không được giải mã;
  OpenClaw ghi log cảnh báo.
- Nếu bạn thấy lỗi thiếu mô-đun crypto (ví dụ, `@matrix-org/matrix-sdk-crypto-nodejs-*`),
  hãy cho phép build scripts cho `@matrix-org/matrix-sdk-crypto-nodejs` và chạy
  `pnpm rebuild @matrix-org/matrix-sdk-crypto-nodejs` hoặc tải binary bằng
  `node node_modules/@matrix-org/matrix-sdk-crypto-nodejs/download-lib.js`.

42. Trạng thái crypto được lưu theo từng tài khoản + access token trong
    `~/.openclaw/matrix/accounts/<account>/<homeserver>__<user>/<token-hash>/crypto/`
    (cơ sở dữ liệu SQLite). 43. Trạng thái đồng bộ được lưu cùng với nó trong `bot-storage.json`.
43. Nếu access token (thiết bị) thay đổi, một kho mới sẽ được tạo và bot phải được
    xác minh lại cho các phòng được mã hóa.

45. **Xác minh thiết bị:**
    Khi E2EE được bật, bot sẽ yêu cầu xác minh từ các phiên khác của bạn khi khởi động.
46. Mở Element (hoặc client khác) và chấp thuận yêu cầu xác minh để thiết lập độ tin cậy.
    Sau khi được xác minh, bot có thể giải mã tin nhắn trong các phòng được mã hóa.

## Kiểm soát truy cập (DM)

Hỗ trợ nhiều tài khoản: sử dụng `channels.matrix.accounts` với thông tin xác thực riêng cho từng tài khoản và tùy chọn `name`. Xem [`gateway/configuration`](/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) để biết mẫu dùng chung.

Mỗi tài khoản chạy như một người dùng Matrix riêng biệt trên bất kỳ homeserver nào. Cấu hình theo từng tài khoản
kế thừa từ các thiết lập `channels.matrix` cấp cao nhất và có thể ghi đè bất kỳ tùy chọn nào
(chính sách DM, nhóm, mã hóa, v.v.).

```json5
{
  channels: {
    matrix: {
      enabled: true,
      dm: { policy: "pairing" },
      accounts: {
        assistant: {
          name: "Main assistant",
          homeserver: "https://matrix.example.org",
          accessToken: "syt_assistant_***",
          encryption: true,
        },
        alerts: {
          name: "Alerts bot",
          homeserver: "https://matrix.example.org",
          accessToken: "syt_alerts_***",
          dm: { policy: "allowlist", allowFrom: ["@admin:example.org"] },
        },
      },
    },
  },
}
```

Lưu ý:

- `requireMention: false` bật tự động trả lời trong phòng đó.
- Biến môi trường (`MATRIX_HOMESERVER`, `MATRIX_ACCESS_TOKEN`, v.v.) chỉ áp dụng cho tài khoản **mặc định**.
- Thiết lập kênh cơ bản (chính sách DM, chính sách nhóm, lọc mention, v.v.) áp dụng cho tất cả tài khoản trừ khi được ghi đè theo từng tài khoản.
- Danh sách cho phép theo từng phòng `users` có thể hạn chế thêm người gửi trong một phòng cụ thể (dùng Matrix user ID đầy đủ).
- Trình cấu hình sẽ hỏi danh sách phòng cho phép (room ID, alias hoặc tên) và chỉ phân giải tên khi có khớp chính xác, duy nhất.

## Luồng

- Hỗ trợ trả lời theo luồng.
- `channels.matrix.threadReplies` điều khiển việc phản hồi có ở trong luồng hay không:

## Khả năng

- Mặc định: `channels.matrix.dm.policy = "pairing"`. 48. Người gửi không xác định sẽ nhận được mã ghép cặp.
- Phê duyệt qua:
  - `openclaw pairing list matrix`
  - `openclaw pairing approve matrix <CODE>`
- DM công khai: `channels.matrix.dm.policy="open"` cùng với `channels.matrix.dm.allowFrom=["*"]`.
- `channels.matrix.dm.allowFrom` chấp nhận ID người dùng Matrix đầy đủ (ví dụ: `@user:server`). Trình hướng dẫn sẽ phân giải tên hiển thị thành ID người dùng khi tìm kiếm thư mục tìm thấy một khớp chính xác duy nhất.
- Không sử dụng display name hoặc localpart trần (ví dụ: `"Alice"` hoặc `"alice"`). Chúng gây mơ hồ và sẽ bị bỏ qua khi khớp allowlist. Sử dụng ID đầy đủ dạng `@user:server`.

## Xử lý sự cố

- Mặc định: `channels.matrix.groupPolicy = "allowlist"` (yêu cầu mention). Use `channels.defaults.groupPolicy` to override the default when unset.
- Cho phép phòng theo danh sách cho phép bằng `channels.matrix.groups` (room ID hoặc alias; tên sẽ được phân giải sang ID khi tìm kiếm thư mục cho ra một kết quả khớp chính xác duy nhất):

```json5
{
  channels: {
    matrix: {
      groupPolicy: "allowlist",
      groups: {
        "!roomId:example.org": { allow: true },
        "#alias:example.org": { allow: true },
      },
      groupAllowFrom: ["@owner:example.org"],
    },
  },
}
```

- `requireMention: false` bật tự động trả lời trong phòng đó.
- `groups."*"` có thể đặt mặc định cho việc giới hạn theo mention trên các phòng.
- `groupAllowFrom` hạn chế người gửi nào có thể kích hoạt bot trong phòng (Matrix user ID đầy đủ).
- Danh sách cho phép theo từng phòng `users` có thể hạn chế thêm người gửi trong một phòng cụ thể (dùng Matrix user ID đầy đủ).
- Trình cấu hình sẽ hỏi danh sách phòng cho phép (room ID, alias hoặc tên) và chỉ phân giải tên khi có khớp chính xác, duy nhất.
- Khi khởi động, OpenClaw phân giải tên phòng/người dùng trong danh sách cho phép sang ID và ghi log ánh xạ; các mục không phân giải được sẽ bị bỏ qua khi so khớp danh sách cho phép.
- Lời mời sẽ tự động tham gia theo mặc định; điều khiển bằng `channels.matrix.autoJoin` và `channels.matrix.autoJoinAllowlist`.
- Để **không cho phép phòng nào**, đặt `channels.matrix.groupPolicy: "disabled"` (hoặc giữ danh sách cho phép trống).
- Khóa cũ: `channels.matrix.rooms` (cùng cấu trúc với `groups`).

## Luồng

- Hỗ trợ trả lời theo luồng.
- `channels.matrix.threadReplies` điều khiển việc phản hồi có ở trong luồng hay không:
  - `off`, `inbound` (mặc định), `always`
- `channels.matrix.replyToMode` điều khiển metadata trả lời khi không trả lời trong luồng:
  - `off` (mặc định), `first`, `all`

## Khả năng

| Tính năng          | Trạng thái                                                                                                 |
| ------------------ | ---------------------------------------------------------------------------------------------------------- |
| Tin nhắn trực tiếp | ✅ Hỗ trợ                                                                                                   |
| Phòng              | ✅ Hỗ trợ                                                                                                   |
| Luồng              | ✅ Hỗ trợ                                                                                                   |
| Media              | ✅ Hỗ trợ                                                                                                   |
| E2EE               | ✅ Hỗ trợ (cần mô-đun crypto)                                                            |
| Phản ứng           | ✅ Hỗ trợ (gửi/đọc qua công cụ)                                                          |
| Thăm dò            | ✅ Hỗ trợ gửi; poll bắt đầu gửi vào được chuyển thành văn bản (bỏ qua phản hồi/kết thúc) |
| Vị trí             | ✅ Hỗ trợ (URI địa lý; bỏ qua độ cao)                                                    |
| Lệnh gốc           | ✅ Hỗ trợ                                                                                                   |

## Tham chiếu cấu hình (Matrix)

Cấu hình đầy đủ: [Cấu hình](/gateway/configuration)

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

Sau đó, nếu cần, xác nhận trạng thái ghép cặp DM:

```bash
openclaw pairing list matrix
```

Các lỗi thường gặp:

- Đã đăng nhập nhưng tin nhắn phòng bị bỏ qua: phòng bị chặn bởi `groupPolicy` hoặc danh sách cho phép phòng.
- DM bị bỏ qua: người gửi đang chờ phê duyệt khi `channels.matrix.dm.policy="pairing"`.
- Phòng mã hóa thất bại: thiếu hỗ trợ crypto hoặc cấu hình mã hóa không khớp.

Luồng phân tích sự cố: [/channels/troubleshooting](/channels/troubleshooting).

## Tham chiếu cấu hình (Matrix)

Cấu hình đầy đủ: [Cấu hình](/gateway/configuration)

Tùy chọn nhà cung cấp:

- `channels.matrix.enabled`: bật/tắt khởi động kênh.
- `channels.matrix.homeserver`: URL homeserver.
- `channels.matrix.userId`: Matrix user ID (tùy chọn khi có access token).
- `channels.matrix.accessToken`: access token.
- `channels.matrix.password`: mật khẩu đăng nhập (token được lưu).
- `channels.matrix.deviceName`: tên hiển thị của thiết bị.
- `channels.matrix.encryption`: bật E2EE (mặc định: false).
- `channels.matrix.initialSyncLimit`: giới hạn đồng bộ ban đầu.
- `channels.matrix.threadReplies`: `off | inbound | always` (mặc định: inbound).
- `channels.matrix.textChunkLimit`: kích thước chia đoạn văn bản gửi đi (ký tự).
- `channels.matrix.chunkMode`: `length` (mặc định) hoặc `newline` để chia theo dòng trống (ranh giới đoạn) trước khi chia theo độ dài.
- `channels.matrix.dm.policy`: `pairing | allowlist | open | disabled` (mặc định: ghép cặp).
- `channels.matrix.dm.allowFrom`: DM allowlist (full Matrix user IDs). `open` requires `"*"`. Trình hướng dẫn phân giải tên thành ID khi có thể.
- `channels.matrix.groupPolicy`: `allowlist | open | disabled` (mặc định: allowlist).
- `channels.matrix.groupAllowFrom`: người gửi được cho phép cho tin nhắn nhóm (Matrix user ID đầy đủ).
- `channels.matrix.allowlistOnly`: ép buộc quy tắc allowlist cho DM + phòng.
- `channels.matrix.groups`: allowlist nhóm + bản đồ cấu hình theo phòng.
- `channels.matrix.rooms`: allowlist/cấu hình nhóm kiểu cũ.
- `channels.matrix.replyToMode`: chế độ reply-to cho luồng/thẻ.
- `channels.matrix.mediaMaxMb`: giới hạn media vào/ra (MB).
- `channels.matrix.autoJoin`: xử lý lời mời (`always | allowlist | off`, mặc định: luôn).
- `channels.matrix.autoJoinAllowlist`: room ID/alias được phép để tự động tham gia.
- `channels.matrix.accounts`: cấu hình đa tài khoản được định danh theo account ID (mỗi tài khoản kế thừa các thiết lập cấp cao nhất).
- `channels.matrix.actions`: kiểm soát công cụ theo từng hành động (reactions/messages/pins/memberInfo/channelInfo).
