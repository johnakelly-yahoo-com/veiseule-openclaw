---
summary: "Ủy quyền xác thực gateway cho một reverse proxy đáng tin cậy (Pomerium, Caddy, nginx + OAuth)"
read_when:
  - Chạy OpenClaw phía sau một proxy nhận diện danh tính
  - Thiết lập Pomerium, Caddy hoặc nginx với OAuth phía trước OpenClaw
  - Khắc phục lỗi WebSocket 1008 unauthorized khi thiết lập reverse proxy
---

# Trusted Proxy Auth

> ⚠️ **Tính năng nhạy cảm về bảo mật.** Chế độ này ủy quyền hoàn toàn việc xác thực cho reverse proxy của bạn. Cấu hình sai có thể khiến Gateway của bạn bị truy cập trái phép. Hãy đọc kỹ trang này trước khi bật.

## Khi nào nên sử dụng

Sử dụng chế độ xác thực `trusted-proxy` khi:

- Bạn chạy OpenClaw phía sau một **proxy nhận diện danh tính** (Pomerium, Caddy + OAuth, nginx + oauth2-proxy, Traefik + forward auth)
- Proxy của bạn xử lý toàn bộ việc xác thực và truyền danh tính người dùng qua header
- Bạn đang ở trong môi trường Kubernetes hoặc container nơi proxy là con đường duy nhất đến Gateway
- Bạn gặp lỗi WebSocket `1008 unauthorized` vì trình duyệt không thể truyền token trong payload WS

## Khi KHÔNG nên sử dụng

- Nếu proxy của bạn không xác thực người dùng (chỉ là bộ kết thúc TLS hoặc load balancer)
- Nếu tồn tại bất kỳ đường truy cập nào đến Gateway bỏ qua proxy (lỗ hổng firewall, truy cập mạng nội bộ)
- Nếu bạn không chắc liệu proxy có loại bỏ/ghi đè đúng các forwarded header hay không
- Nếu bạn chỉ cần truy cập cá nhân cho một người dùng (cân nhắc Tailscale Serve + loopback để thiết lập đơn giản hơn)

## Cách hoạt động

1. Reverse proxy của bạn xác thực người dùng (OAuth, OIDC, SAML, v.v.)
2. Proxy thêm một header chứa danh tính người dùng đã xác thực (ví dụ: `x-forwarded-user: nick@example.com`)
3. OpenClaw kiểm tra xem yêu cầu có đến từ **IP proxy đáng tin cậy** hay không (được cấu hình trong `gateway.trustedProxies`)
4. OpenClaw trích xuất danh tính người dùng từ header đã cấu hình
5. Nếu mọi thứ hợp lệ, yêu cầu sẽ được ủy quyền

## Cấu hình

```json5
{
  gateway: {
    // Phải bind tới network interface (không phải loopback)
    bind: "lan",

    // QUAN TRỌNG: Chỉ thêm IP của proxy của bạn vào đây
    trustedProxies: ["10.0.0.1", "172.17.0.1"],

    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        // Header chứa danh tính người dùng đã xác thực (bắt buộc)
        userHeader: "x-forwarded-user",

        // Tùy chọn: các header BẮT BUỘC phải có (xác minh proxy)
        requiredHeaders: ["x-forwarded-proto", "x-forwarded-host"],

        // Tùy chọn: giới hạn cho người dùng cụ thể (để trống = cho phép tất cả)
        allowUsers: ["nick@example.com", "admin@company.org"],
      },
    },
  },
}
```

### Tham chiếu cấu hình

| Trường                                      | Bắt buộc | Mô tả                                                                                                                                       |
| ------------------------------------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| `gateway.trustedProxies`                    | Có       | Mảng các địa chỉ IP proxy được tin cậy. Các yêu cầu từ IP khác sẽ bị từ chối.                               |
| `gateway.auth.mode`                         | Có       | Phải là `"trusted-proxy"`                                                                                                                   |
| `gateway.auth.trustedProxy.userHeader`      | Có       | Tên header chứa định danh người dùng đã được xác thực                                                                                       |
| `gateway.auth.trustedProxy.requiredHeaders` | Không    | Các header bổ sung bắt buộc phải có để yêu cầu được tin cậy                                                                                 |
| `gateway.auth.trustedProxy.allowUsers`      | Không    | Danh sách cho phép các định danh người dùng. Để trống nghĩa là cho phép tất cả người dùng đã được xác thực. |

## Ví dụ thiết lập Proxy

### Pomerium

Pomerium truyền định danh trong `x-pomerium-claim-email` (hoặc các claim header khác) và một JWT trong `x-pomerium-jwt-assertion`.

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["10.0.0.1"], // IP của Pomerium
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-pomerium-claim-email",
        requiredHeaders: ["x-pomerium-jwt-assertion"],
      },
    },
  },
}
```

Đoạn cấu hình Pomerium:

```yaml
routes:
  - from: https://openclaw.example.com
    to: http://openclaw-gateway:18789
    policy:
      - allow:
          or:
            - email:
                is: nick@example.com
    pass_identity_headers: true
```

### Caddy với OAuth

Caddy với plugin `caddy-security` có thể xác thực người dùng và truyền các header định danh.

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["127.0.0.1"], // IP của Caddy (nếu chạy trên cùng máy chủ)
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-forwarded-user",
      },
    },
  },
}
```

Đoạn Caddyfile:

```
openclaw.example.com {
    authenticate with oauth2_provider
    authorize with policy1

    reverse_proxy openclaw:18789 {
        header_up X-Forwarded-User {http.auth.user.email}
    }
}
```

### nginx + oauth2-proxy

oauth2-proxy xác thực người dùng và truyền định danh trong `x-auth-request-email`.

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["10.0.0.1"], // IP của nginx/oauth2-proxy
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-auth-request-email",
      },
    },
  },
}
```

Đoạn cấu hình nginx:

```nginx
location / {
    auth_request /oauth2/auth;
    auth_request_set $user $upstream_http_x_auth_request_email;

    proxy_pass http://openclaw:18789;
    proxy_set_header X-Auth-Request-Email $user;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
}
```

### Traefik với Forward Auth

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["172.17.0.1"], // IP container của Traefik
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-forwarded-user",
      },
    },
  },
}
```

## Danh sách kiểm tra bảo mật

Trước khi bật xác thực trusted-proxy, hãy xác minh:

- [ ] **Proxy là đường truy cập duy nhất**: Cổng Gateway được tường lửa chặn với tất cả ngoại trừ proxy của bạn
- [ ] **trustedProxies ở mức tối thiểu**: Chỉ bao gồm các IP proxy thực tế của bạn, không phải toàn bộ subnet
- [ ] **Proxy loại bỏ header**: Proxy của bạn ghi đè (không phải thêm vào) các header `x-forwarded-*` từ client
- [ ] **TLS termination**: Proxy của bạn xử lý TLS; người dùng kết nối qua HTTPS
- [ ] **allowUsers đã được thiết lập** (khuyến nghị): Giới hạn cho những người dùng đã biết thay vì cho phép bất kỳ ai đã được xác thực

## Kiểm tra bảo mật

`openclaw security audit` sẽ đánh dấu cơ chế xác thực trusted-proxy với mức độ nghiêm trọng **critical**. Điều này là có chủ đích — nhằm nhắc nhở rằng bạn đang ủy quyền bảo mật cho cấu hình proxy của mình.

Quá trình kiểm tra sẽ kiểm tra các mục sau:

- Thiếu cấu hình `trustedProxies`
- Thiếu cấu hình `userHeader`
- `allowUsers` trống (cho phép bất kỳ người dùng đã xác thực nào)

## Khắc phục sự cố

### "trusted_proxy_untrusted_source"

Yêu cầu không đến từ một IP nằm trong `gateway.trustedProxies`. Kiểm tra:

- IP của proxy có chính xác không? (IP của container Docker có thể thay đổi)
- Có load balancer đứng trước proxy của bạn không?
- Sử dụng `docker inspect` hoặc `kubectl get pods -o wide` để tìm các IP thực tế

### "trusted_proxy_user_missing"

Header người dùng bị trống hoặc thiếu. Kiểm tra:

- Proxy của bạn có được cấu hình để truyền các header định danh không?
- Tên header có chính xác không? (không phân biệt chữ hoa/thường, nhưng phải đúng chính tả)
- Người dùng có thực sự đã được xác thực tại proxy không?

### "trusted_proxy_missing_header_\*"

Một header bắt buộc không xuất hiện. Kiểm tra:

- Cấu hình proxy của bạn cho các header cụ thể đó
- Các header có bị loại bỏ ở đâu đó trong chuỗi xử lý hay không

### "trusted_proxy_user_not_allowed"

Người dùng đã được xác thực nhưng không có trong `allowUsers`. Hoặc thêm họ vào hoặc xóa allowlist.

### WebSocket vẫn bị lỗi

Đảm bảo proxy của bạn:

- Hỗ trợ nâng cấp WebSocket (`Upgrade: websocket`, `Connection: upgrade`)
- Truyền các header định danh trong yêu cầu nâng cấp WebSocket (không chỉ HTTP)
- Không có đường dẫn xác thực riêng cho kết nối WebSocket

## Di chuyển từ Token Auth

Nếu bạn đang chuyển từ xác thực bằng token sang trusted-proxy:

1. Cấu hình proxy của bạn để xác thực người dùng và truyền các header
2. Kiểm tra cấu hình proxy một cách độc lập (curl kèm header)
3. Cập nhật cấu hình OpenClaw với xác thực trusted-proxy
4. Khởi động lại Gateway
5. Kiểm tra kết nối WebSocket từ Control UI
6. Chạy `openclaw security audit` và xem lại các phát hiện

## Liên quan

- [Security](/gateway/security) — hướng dẫn bảo mật đầy đủ
- [Configuration](/gateway/configuration) — tài liệu tham chiếu cấu hình
- [Remote Access](/gateway/remote) — các mô hình truy cập từ xa khác
- [Tailscale](/gateway/tailscale) — giải pháp thay thế đơn giản hơn cho truy cập chỉ trong tailnet

