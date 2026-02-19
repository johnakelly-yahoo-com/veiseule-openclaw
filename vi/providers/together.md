---
summary: "Thiết lập Together AI (xác thực + chọn model)"
read_when:
  - Bạn muốn sử dụng Together AI với OpenClaw
  - Bạn cần biến môi trường API key hoặc lựa chọn xác thực qua CLI
---

# Together AI

[Together AI](https://together.ai) cung cấp quyền truy cập vào các mô hình mã nguồn mở hàng đầu như Llama, DeepSeek, Kimi và nhiều mô hình khác thông qua một API hợp nhất.

- Provider: `together`
- Xác thực: `TOGETHER_API_KEY`
- API: tương thích OpenAI

## Bắt đầu nhanh

1. Thiết lập API key (khuyến nghị: lưu cho Gateway):

```bash
openclaw onboard --auth-choice together-api-key
```

2. Thiết lập model mặc định:

```json5
{
  agents: {
    defaults: {
      model: { primary: "together/moonshotai/Kimi-K2.5" },
    },
  },
}
```

## Ví dụ không tương tác

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice together-api-key \
  --together-api-key "$TOGETHER_API_KEY"
```

Thao tác này sẽ đặt `together/moonshotai/Kimi-K2.5` làm mô hình mặc định.

## Lưu ý về môi trường

Nếu Gateway chạy dưới dạng daemon (launchd/systemd), hãy đảm bảo `TOGETHER_API_KEY`
khả dụng cho tiến trình đó (ví dụ: trong `~/.clawdbot/.env` hoặc thông qua
`env.shellEnv`).

## Các mô hình khả dụng

Together AI cung cấp quyền truy cập vào nhiều mô hình mã nguồn mở phổ biến:

- **GLM 4.7 Fp8** - Mô hình mặc định với cửa sổ ngữ cảnh 200K
- **Llama 3.3 70B Instruct Turbo** - Tuân theo chỉ dẫn nhanh và hiệu quả
- **Llama 4 Scout** - Mô hình thị giác với khả năng hiểu hình ảnh
- **Llama 4 Maverick** - Thị giác và suy luận nâng cao
- **DeepSeek V3.1** - Mô hình mạnh mẽ cho lập trình và suy luận
- **DeepSeek R1** - Mô hình suy luận nâng cao
- **Kimi K2 Instruct** - Mô hình hiệu năng cao với cửa sổ ngữ cảnh 262K

Tất cả các mô hình đều hỗ trợ chat completions tiêu chuẩn và tương thích OpenAI API.
