---
title: "Đăng nhập trình duyệt"
---

# Đăng nhập trình duyệt + đăng bài lên X/Twitter

## Đăng nhập thủ công (khuyến nghị)

Khi một trang yêu cầu đăng nhập, hãy **đăng nhập thủ công** trong hồ sơ trình duyệt **host** (trình duyệt openclaw).

**Không** cung cấp thông tin đăng nhập của bạn cho model. Việc đăng nhập tự động thường kích hoạt cơ chế chống bot và có thể khiến tài khoản bị khóa.

Quay lại tài liệu trình duyệt chính: [Browser](/tools/browser).

## Dùng hồ sơ Chrome nào?

OpenClaw kiểm soát một **hồ sơ Chrome chuyên dụng** (tên là `openclaw`, giao diện tông màu cam). Hồ sơ này tách biệt với hồ sơ trình duyệt bạn sử dụng hằng ngày.

Hai cách đơn giản để truy cập:

1. **Yêu cầu tác tử mở trình duyệt** rồi tự bạn đăng nhập.
2. **Mở qua CLI**:

```bash
openclaw browser start
openclaw browser open https://x.com
```

Nếu bạn có nhiều hồ sơ, truyền `--browser-profile <name>` (mặc định là `openclaw`).

## X/Twitter: quy trình khuyến nghị

- **Đọc/tìm kiếm/chuỗi bài:** dùng trình duyệt **host** (đăng nhập thủ công).
- **Đăng cập nhật:** dùng trình duyệt **host** (đăng nhập thủ công).

## Sandboxing + truy cập trình duyệt host

Các phiên trình duyệt trong môi trường sandbox **có nhiều khả năng hơn** bị phát hiện là bot. Đối với X/Twitter (và các trang nghiêm ngặt khác), hãy ưu tiên trình duyệt **host**.

Nếu agent đang chạy trong sandbox, công cụ trình duyệt sẽ mặc định sử dụng sandbox. Để cho phép điều khiển trên host:

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        browser: {
          allowHostControl: true,
        },
      },
    },
  },
}
```

Sau đó nhắm tới trình duyệt host:

```bash
openclaw browser open https://x.com --browser-profile openclaw --target host
```

Hoặc tắt sandboxing cho tác tử đăng bài cập nhật.

