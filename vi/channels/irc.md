---
title: IRC
description: Kết nối OpenClaw với các kênh IRC và tin nhắn trực tiếp.
---

Sử dụng IRC khi bạn muốn OpenClaw hoạt động trong các kênh cổ điển (`#room`) và tin nhắn trực tiếp.
IRC được phát hành dưới dạng extension plugin, nhưng được cấu hình trong tệp config chính tại `channels.irc`.

## Bắt đầu nhanh

1. Bật cấu hình IRC trong `~/.openclaw/openclaw.json`.
2. Thiết lập tối thiểu:

```json
{
  "channels": {
    "irc": {
      "enabled": true,
      "host": "irc.libera.chat",
      "port": 6697,
      "tls": true,
      "nick": "openclaw-bot",
      "channels": ["#openclaw"]
    }
  }
}
```

3. Khởi động/khởi động lại gateway:

```bash
openclaw gateway run
```

## Mặc định bảo mật

- `channels.irc.dmPolicy` mặc định là `"pairing"`.
- `channels.irc.groupPolicy` mặc định là `"allowlist"`.
- Với `groupPolicy="allowlist"`, hãy thiết lập `channels.irc.groups` để xác định các kênh được phép.
- Sử dụng TLS (`channels.irc.tls=true`) trừ khi bạn chủ ý chấp nhận truyền tải dạng văn bản thuần (plaintext).

## Kiểm soát truy cập

Có hai “cổng” riêng biệt cho các kênh IRC:

1. **Truy cập kênh** (`groupPolicy` + `groups`): liệu bot có chấp nhận tin nhắn từ một kênh hay không.
2. **Truy cập người gửi** (`groupAllowFrom` / `groups["#channel"].allowFrom` theo từng kênh): ai được phép kích hoạt bot trong kênh đó.

Các khóa cấu hình:

- Danh sách cho phép DM (quyền người gửi DM): `channels.irc.allowFrom`
- Danh sách cho phép người gửi trong nhóm (quyền người gửi trong kênh): `channels.irc.groupAllowFrom`
- Kiểm soát theo từng kênh (kênh + người gửi + quy tắc mention): `channels.irc.groups["#channel"]`
- `channels.irc.groupPolicy="open"` cho phép các kênh chưa được cấu hình (**mặc định vẫn yêu cầu mention**)

Các mục trong allowlist có thể sử dụng dạng nick hoặc `nick!user@host`.

### Lỗi thường gặp: `allowFrom` dành cho DM, không phải cho kênh

Nếu bạn thấy log như:

- `irc: drop group sender alice!ident@host (policy=allowlist)`

…điều đó có nghĩa là người gửi không được phép cho tin nhắn **nhóm/kênh**. Khắc phục bằng cách:

- thiết lập `channels.irc.groupAllowFrom` (áp dụng toàn cục cho tất cả kênh), hoặc
- thiết lập allowlist người gửi theo từng kênh: `channels.irc.groups["#channel"].allowFrom`

Ví dụ (cho phép bất kỳ ai trong `#tuirc-dev` nói chuyện với bot):

```json5
{
  channels: {
    irc: {
      groupPolicy: "allowlist",
      groups: {
        "#tuirc-dev": { allowFrom: ["*"] },
      },
    },
  },
}
```

## Kích hoạt phản hồi (mentions)

Ngay cả khi một kênh được cho phép (thông qua `groupPolicy` + `groups`) và người gửi được phép, OpenClaw mặc định vẫn **yêu cầu mention** trong ngữ cảnh nhóm.

Điều đó có nghĩa là bạn có thể thấy log như `drop channel … (missing-mention)` trừ khi tin nhắn chứa mẫu mention khớp với bot.

Để bot trả lời trong một kênh IRC **mà không cần mention**, hãy tắt yêu cầu mention cho kênh đó:

```json5
{
  channels: {
    irc: {
      groupPolicy: "allowlist",
      groups: {
        "#tuirc-dev": {
          requireMention: false,
          allowFrom: ["*"],
        },
      },
    },
  },
}
```

Hoặc để cho phép **tất cả** các kênh IRC (không dùng allowlist theo từng kênh) và vẫn trả lời mà không cần mention:

```json5
{
  channels: {
    irc: {
      groupPolicy: "open",
      groups: {
        "*": { requireMention: false, allowFrom: ["*"] },
      },
    },
  },
}
```

## Lưu ý bảo mật (khuyến nghị cho kênh công khai)

Nếu bạn đặt `allowFrom: ["*"]` trong một kênh công khai, bất kỳ ai cũng có thể gửi prompt cho bot.
Để giảm rủi ro, hãy hạn chế các công cụ cho kênh đó.

### Cùng một bộ công cụ cho mọi người trong kênh

```json5
{
  channels: {
    irc: {
      groups: {
        "#tuirc-dev": {
          allowFrom: ["*"],
          tools: {
            deny: ["group:runtime", "group:fs", "gateway", "nodes", "cron", "browser"],
          },
        },
      },
    },
  },
}
```

### Công cụ khác nhau theo từng người gửi (owner có nhiều quyền hơn)

Sử dụng `toolsBySender` để áp dụng chính sách nghiêm ngặt hơn cho `"*"` và chính sách nới lỏng hơn cho nick của bạn:

```json5
{
  channels: {
    irc: {
      groups: {
        "#tuirc-dev": {
          allowFrom: ["*"],
          toolsBySender: {
            "*": {
              deny: ["group:runtime", "group:fs", "gateway", "nodes", "cron", "browser"],
            },
            eigen: {
              deny: ["gateway", "nodes", "cron"],
            },
          },
        },
      },
    },
  },
}
```

Ghi chú:

- Các khóa trong `toolsBySender` có thể là một nick (ví dụ: `"eigen"`) hoặc một hostmask đầy đủ (`"eigen!~eigen@174.127.248.171"`) để khớp danh tính chính xác hơn.
- Chính sách người gửi khớp đầu tiên sẽ được áp dụng; `"*"` là wildcard dự phòng.

Để tìm hiểu thêm về quyền truy cập nhóm so với cơ chế yêu cầu mention (và cách chúng tương tác), xem: [/channels/groups](/channels/groups).

## NickServ

Để xác thực với NickServ sau khi kết nối:

```json
{
  "channels": {
    "irc": {
      "nickserv": {
        "enabled": true,
        "service": "NickServ",
        "password": "your-nickserv-password"
      }
    }
  }
}
```

Tùy chọn đăng ký một lần khi kết nối:

```json
{
  "channels": {
    "irc": {
      "nickserv": {
        "register": true,
        "registerEmail": "bot@example.com"
      }
    }
  }
}
```

Tắt `register` sau khi nick đã được đăng ký để tránh lặp lại các lần thử REGISTER.

## Biến môi trường

Tài khoản mặc định hỗ trợ:

- `IRC_HOST`
- `IRC_PORT`
- `IRC_TLS`
- `IRC_NICK`
- `IRC_USERNAME`
- `IRC_REALNAME`
- `IRC_PASSWORD`
- `IRC_CHANNELS` (phân tách bằng dấu phẩy)
- `IRC_NICKSERV_PASSWORD`
- `IRC_NICKSERV_REGISTER_EMAIL`

## Khắc phục sự cố

- Nếu bot kết nối nhưng không bao giờ trả lời trong kênh, hãy kiểm tra `channels.irc.groups` **và** xem cơ chế yêu cầu mention có đang loại bỏ tin nhắn (`missing-mention`) hay không. Nếu bạn muốn bot trả lời mà không cần ping, hãy đặt `requireMention:false` cho kênh.
- Nếu đăng nhập thất bại, hãy kiểm tra tính khả dụng của nick và mật khẩu máy chủ.
- Nếu TLS thất bại trên một mạng tùy chỉnh, hãy kiểm tra host/port và cấu hình chứng chỉ.
