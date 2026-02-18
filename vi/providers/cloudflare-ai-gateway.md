---
title: "Cloudflare AI Gateway"---

# Cloudflare AI Gateway

Cloudflare AI Gateway nằm phía trước các API của nhà cung cấp và cho phép bạn thêm phân tích, lưu bộ đệm và các biện pháp kiểm soát. Đối với Anthropic, OpenClaw sử dụng Anthropic Messages API thông qua endpoint Gateway của bạn.

- Nhà cung cấp: `cloudflare-ai-gateway`
- URL cơ sở: `https://gateway.ai.cloudflare.com/v1/<account_id>/<gateway_id>/anthropic`
- Mô hình mặc định: `cloudflare-ai-gateway/claude-sonnet-4-5`
- API key: `CLOUDFLARE_AI_GATEWAY_API_KEY` (khóa API của nhà cung cấp cho các yêu cầu đi qua Gateway)

Với các mô hình Anthropic, hãy dùng khóa API Anthropic của bạn.

## Bắt đầu nhanh

1. Đặt khóa API của nhà cung cấp và chi tiết Gateway:

```bash
openclaw onboard --auth-choice cloudflare-ai-gateway-api-key
```

2. Đặt mô hình mặc định:

```json5
{
  agents: {
    defaults: {
      model: { primary: "cloudflare-ai-gateway/claude-sonnet-4-5" },
    },
  },
}
```

## Ví dụ không tương tác

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice cloudflare-ai-gateway-api-key \
  --cloudflare-ai-gateway-account-id "your-account-id" \
  --cloudflare-ai-gateway-gateway-id "your-gateway-id" \
  --cloudflare-ai-gateway-api-key "$CLOUDFLARE_AI_GATEWAY_API_KEY"
```

## Gateway có xác thực

Nếu bạn đã bật xác thực Gateway trong Cloudflare, hãy thêm header `cf-aig-authorization` (ngoài khóa API của nhà cung cấp).

```json5
{
  models: {
    providers: {
      "cloudflare-ai-gateway": {
        headers: {
          "cf-aig-authorization": "Bearer <cloudflare-ai-gateway-token>",
        },
      },
    },
  },
}
```

## Lưu ý về môi trường

Nếu Gateway chạy như một daemon (launchd/systemd), hãy đảm bảo `CLOUDFLARE_AI_GATEWAY_API_KEY` khả dụng cho tiến trình đó (ví dụ, trong `~/.openclaw/.env` hoặc qua `env.shellEnv`).

