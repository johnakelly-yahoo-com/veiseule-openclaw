---
title: "Firecrawl"
---

# Firecrawl

OpenClaw có thể sử dụng **Firecrawl** làm trình trích xuất dự phòng cho `web_fetch`. Đây là một dịch vụ được lưu trữ trên nền tảng đám mây.
content extraction service that supports bot circumvention and caching, which helps
with JS-heavy sites or pages that block plain HTTP fetches.

## Lấy khóa API

1. Tạo tài khoản Firecrawl và tạo khóa API.
2. Lưu khóa trong cấu hình hoặc đặt `FIRECRAWL_API_KEY` trong môi trường gateway.

## Cấu hình Firecrawl

```json5
{
  tools: {
    web: {
      fetch: {
        firecrawl: {
          apiKey: "FIRECRAWL_API_KEY_HERE",
          baseUrl: "https://api.firecrawl.dev",
          onlyMainContent: true,
          maxAgeMs: 172800000,
          timeoutSeconds: 60,
        },
      },
    },
  },
}
```

Ghi chú:

- `firecrawl.enabled` mặc định là true khi có khóa API.
- `maxAgeMs` kiểm soát độ cũ tối đa của kết quả được lưu trong bộ nhớ đệm (ms). Mặc định là 2 ngày.

## Stealth / vượt qua bot

Firecrawl cung cấp tham số **proxy mode** để vượt qua cơ chế chống bot (`basic`, `stealth` hoặc `auto`).
OpenClaw always uses `proxy: "auto"` plus `storeInCache: true` for Firecrawl requests.
If proxy is omitted, Firecrawl defaults to `auto`. `auto` retries with stealth proxies if a basic attempt fails, which may use more credits
than basic-only scraping.

## Cách `web_fetch` dùng Firecrawl

Thứ tự trích xuất của `web_fetch`:

1. Readability (cục bộ)
2. Firecrawl (nếu đã cấu hình)
3. Dọn dẹp HTML cơ bản (dự phòng cuối cùng)

Xem [Web tools](/tools/web) để biết thiết lập đầy đủ cho công cụ web.


