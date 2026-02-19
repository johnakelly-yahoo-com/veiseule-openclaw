---
summary: "Runbook cho dịch vụ Gateway, vòng đời và vận hành"
read_when:
  - Khi chạy hoặc gỡ lỗi tiến trình gateway
title: "Runbook Gateway"
---

# Runbook dịch vụ Gateway

Cập nhật lần cuối: 2025-12-09

<CardGroup cols={2}>
  <Card title="Deep troubleshooting" icon="siren" href="/gateway/troubleshooting">
    Chẩn đoán theo triệu chứng trước, kèm theo các chuỗi lệnh chính xác và dấu hiệu log.
  
</Card>
  <Card title="Configuration" icon="sliders" href="/gateway/configuration">
    Hướng dẫn thiết lập theo tác vụ + tài liệu tham chiếu cấu hình đầy đủ.
  
</Card>
</CardGroup>

## Khởi động cục bộ trong 5 phút

<Steps>
  <Step title="Start the Gateway">

```bash
openclaw gateway --port 18789
# for full debug/trace logs in stdio:
openclaw gateway --port 18789 --verbose
# if the port is busy, terminate listeners then start:
openclaw gateway --force
# dev loop (auto-reload on TS changes):
pnpm gateway:watch
```

  
</Step>

  <Step title="Verify service health">

```bash
openclaw gateway status
openclaw status
openclaw logs --follow
```

Trạng thái chuẩn: `Runtime: running` và `RPC probe: ok`.

  
</Step>

  <Step title="Validate channel readiness">

```bash
openclaw channels status --probe
```

  
</Step>
</Steps>

<Note>
Cơ chế tải lại cấu hình Gateway theo dõi đường dẫn tệp cấu hình đang hoạt động (được phân giải từ mặc định hồ sơ/trạng thái, hoặc `OPENCLAW_CONFIG_PATH` khi được thiết lập).
Chế độ mặc định là `gateway.reload.mode="hybrid"`.
</Note>

## Mô hình runtime

- Một tiến trình luôn hoạt động để định tuyến, control plane và kết nối kênh.
- Một cổng được ghép kênh duy nhất cho:
  - WebSocket control/RPC
  - HTTP APIs (tương thích OpenAI, Responses, gọi tools)
  - Control UI và hooks
- Chế độ bind mặc định: `loopback`.
- Yêu cầu xác thực theo mặc định (`gateway.auth.token` / `gateway.auth.password`, hoặc `OPENCLAW_GATEWAY_TOKEN` / `OPENCLAW_GATEWAY_PASSWORD`).

### Profile dev (`--dev`)

| Thiết lập    | Thứ tự phân giải                                              |
| ------------ | ------------------------------------------------------------- |
| Cổng Gateway | `--port` → `OPENCLAW_GATEWAY_PORT` → `gateway.port` → `18789` |
| Chế độ bind  | CLI/override → `gateway.bind` → `loopback`                    |

### Các chế độ hot reload

| `gateway.reload.mode`                  | Hành vi                                            |
| -------------------------------------- | -------------------------------------------------- |
| `off`                                  | Không tải lại cấu hình                             |
| `hot`                                  | Chỉ áp dụng các thay đổi an toàn cho hot reload    |
| `restart`                              | Khởi động lại khi có thay đổi yêu cầu reload       |
| `hybrid` (mặc định) | Hot-apply khi an toàn, khởi động lại khi cần thiết |

## Bộ lệnh dành cho operator

```bash
openclaw gateway status
openclaw gateway status --deep
openclaw gateway status --json
openclaw gateway install
openclaw gateway restart
openclaw gateway stop
openclaw logs --follow
openclaw doctor
```

## Truy cập từ xa

Ưu tiên: Tailscale/VPN.
Phương án dự phòng: SSH tunnel.

```bash
ssh -N -L 18789:127.0.0.1:18789 user@host
```

Cài đặt dịch vụ theo profile:

<Warning>
Nếu cấu hình xác thực gateway được bật, client vẫn phải gửi thông tin xác thực (`token`/`password`) ngay cả khi kết nối qua SSH tunnel.
</Warning>

Ví dụ:

## Giám sát và vòng đời dịch vụ

Sử dụng chế độ chạy có giám sát để đạt độ tin cậy tương tự môi trường production.

<Tabs>
  <Tab title="macOS (launchd)">

```bash
openclaw gateway install
openclaw gateway status
openclaw gateway restart
openclaw gateway stop
```

Nhãn LaunchAgent là `ai.openclaw.gateway` (mặc định) hoặc `ai.openclaw.<profile>` (profile được đặt tên). `openclaw doctor` kiểm tra và sửa các sai lệch cấu hình dịch vụ.

  
</Tab>

  <Tab title="Linux (systemd user)">

```bash
openclaw gateway install
systemctl --user enable --now openclaw-gateway[-<profile>].service
openclaw gateway status
```

Để duy trì hoạt động sau khi đăng xuất, hãy bật lingering:

```bash
sudo loginctl enable-linger <user>
```

  
</Tab>

  <Tab title="Linux (system service)">

Sử dụng system unit cho các máy chủ nhiều người dùng/luôn bật.

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now openclaw-gateway[-<profile>].service
```

  
</Tab>
</Tabs>

## Nhiều gateway trên một máy chủ

Hầu hết các thiết lập chỉ nên chạy **một** Gateway.
Chỉ sử dụng nhiều gateway khi cần cô lập/dự phòng nghiêm ngặt (ví dụ: profile cứu hộ).

Danh sách kiểm tra cho mỗi instance:

- `gateway.port` duy nhất
- `OPENCLAW_CONFIG_PATH` duy nhất
- `OPENCLAW_STATE_DIR` duy nhất
- `agents.defaults.workspace` duy nhất

Ví dụ:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json OPENCLAW_STATE_DIR=~/.openclaw-a openclaw gateway --port 19001
OPENCLAW_CONFIG_PATH=~/.openclaw/b.json OPENCLAW_STATE_DIR=~/.openclaw-b openclaw gateway --port 19002
```

Xem: [Multiple gateways](/gateway/multiple-gateways).

### Quản lý dịch vụ Gateway (CLI)

```bash
openclaw --dev setup
openclaw --dev gateway --allow-unconfigured
openclaw --dev status
```

Mặc định bao gồm state/config được cô lập và cổng gateway cơ sở `19001`.

## Tham khảo nhanh giao thức (góc nhìn vận hành)

- `gateway status` thăm dò RPC của Gateway theo mặc định bằng cổng/cấu hình đã resolve của dịch vụ (ghi đè bằng `--url`).
- `gateway status --deep` thêm quét cấp hệ thống (LaunchDaemons/system units).
- `gateway status --no-probe` bỏ qua thăm dò RPC (hữu ích khi mạng bị down).
- `gateway status --json` ổn định cho script.

Ứng dụng mac đóng gói:

1. Phản hồi xác nhận ngay lập tức (`status:"accepted"`)
2. Để dừng sạch, dùng `openclaw gateway stop` (hoặc `launchctl bootout gui/$UID/bot.molt.gateway`).

Xem tài liệu giao thức đầy đủ: [Gateway Protocol](/gateway/protocol).

## Kiểm tra vận hành

### Kiểm tra hoạt động (Liveness)

- Mở WS và gửi `connect`.
- Mong đợi phản hồi `hello-ok` kèm snapshot.

### Kiểm tra sẵn sàng (Readiness)

```bash
openclaw gateway status
openclaw channels status --probe
openclaw health
```

### Khôi phục khi mất đồng bộ (Gap recovery)

Sự kiện không được phát lại. Khi phát hiện thiếu sequence, làm mới trạng thái (`health`, `system-presence`) trước khi tiếp tục.

## Các dấu hiệu lỗi phổ biến

| Dấu hiệu                                                       | Vấn đề có thể xảy ra                                            |
| -------------------------------------------------------------- | --------------------------------------------------------------- |
| `refusing to bind gateway ... without auth`                    | Bind tới địa chỉ không phải loopback mà không có token/password |
| `another gateway instance is already listening` / `EADDRINUSE` | Xung đột cổng                                                   |
| `Gateway start blocked: set gateway.mode=local`                | Cấu hình đã được đặt sang chế độ remote                         |
| `unauthorized` trong quá trình kết nối                         | Không khớp xác thực giữa client và gateway                      |

Để chẩn đoán đầy đủ theo từng bước, xem [Gateway Troubleshooting](/gateway/troubleshooting).

## Windows (WSL2)

- Các client sử dụng giao thức Gateway sẽ thất bại ngay khi Gateway không khả dụng (không có cơ chế tự động chuyển sang kênh trực tiếp).
- Các frame đầu tiên không hợp lệ/không phải kết nối sẽ bị từ chối và đóng lại.
- Quá trình tắt mềm sẽ phát sự kiện `shutdown` trước khi đóng socket.

---

Liên quan:

- [Troubleshooting](/gateway/troubleshooting)
- [Background Process](/gateway/background-process)
- [Configuration](/gateway/configuration)
- [Health](/gateway/health)
- [Doctor](/gateway/doctor)
- [Authentication](/gateway/authentication)

