---
summary: "Tham chiếu CLI cho `openclaw security` (kiểm tra và khắc phục các lỗi bảo mật thường gặp)"
read_when:
  - Bạn muốn chạy kiểm tra bảo mật nhanh cho cấu hình/trạng thái
  - Bạn muốn áp dụng các gợi ý “sửa” an toàn (chmod, siết chặt mặc định)
title: "security"
---

# `openclaw security`

Công cụ bảo mật (kiểm tra + tùy chọn sửa).

Liên quan:

- Hướng dẫn bảo mật: [Security](/gateway/security)

## Kiểm toán

```bash
openclaw security audit
openclaw security audit --deep
openclaw security audit --fix
```

Bản audit cảnh báo khi nhiều người gửi DM chia sẻ cùng phiên chính và khuyến nghị **chế độ DM an toàn**: `session.dmScope="per-channel-peer"` (hoặc `per-account-channel-peer` cho các kênh đa tài khoản) cho các hộp thư đến dùng chung.
Xếp hàng một sự kiện hệ thống vào phiên **main**.
Đối với webhook ingress, hệ thống sẽ cảnh báo khi `hooks.defaultSessionKey` chưa được thiết lập, khi cho phép ghi đè `sessionKey` trong request và khi cho phép ghi đè nhưng không có `hooks.allowedSessionKeyPrefixes`.
Hệ thống cũng cảnh báo khi cấu hình Docker sandbox được thiết lập nhưng chế độ sandbox đang tắt, khi `gateway.nodes.denyCommands` sử dụng các mẫu không hiệu quả/không xác định, khi `tools.profile="minimal"` toàn cục bị ghi đè bởi hồ sơ công cụ của agent và khi các công cụ của extension plugin đã cài đặt có thể truy cập được dưới chính sách công cụ quá permissive.

