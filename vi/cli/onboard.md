---
summary: "Tài liệu tham chiếu CLI cho `openclaw onboard` (trình hướng dẫn onboarding tương tác)"
read_when:
  - Bạn muốn thiết lập có hướng dẫn cho gateway, workspace, xác thực, kênh và skills
title: "onboard"
---

# `openclaw onboard`

Trình hướng dẫn onboarding tương tác (thiết lập Gateway cục bộ hoặc từ xa).

## Hướng dẫn liên quan

- Trung tâm onboarding CLI: [Onboarding Wizard (CLI)](/start/wizard)
- Tham chiếu onboarding CLI: [CLI Onboarding Reference](/start/wizard-cli-reference)
- Tự động hóa CLI: [CLI Automation](/start/wizard-cli-automation)
- Onboarding trên macOS: [Onboarding (macOS App)](/start/onboarding)
- Onboarding trên macOS: [Onboarding (macOS App)](/start/onboarding)

## Ví dụ

```bash
openclaw onboard
openclaw onboard --flow quickstart
openclaw onboard --flow manual
openclaw onboard --mode remote --remote-url ws://gateway-host:18789
```

Ghi chú luồng:

```bash
openclaw onboard --non-interactive \
  --auth-choice custom-api-key \
  --custom-base-url "https://llm.example.com/v1" \
  --custom-model-id "foo-large" \
  --custom-api-key "$CUSTOM_API_KEY" \
  --custom-compatibility openai
```

`--custom-api-key` là tùy chọn trong chế độ không tương tác. Nếu không cung cấp, quá trình onboarding sẽ kiểm tra `CUSTOM_API_KEY`.

Các lựa chọn endpoint Z.AI ở chế độ không tương tác:

Lưu ý: `--auth-choice zai-api-key` hiện tự động phát hiện endpoint Z.AI tốt nhất cho khóa của bạn (ưu tiên API chung với `zai/glm-5`).
Nếu bạn muốn cụ thể các endpoint GLM Coding Plan, hãy chọn `zai-coding-global` hoặc `zai-coding-cn`.

```bash
# Chọn endpoint không cần nhắc
openclaw onboard --non-interactive \
  --auth-choice zai-coding-global \
  --zai-api-key "$ZAI_API_KEY"

# Các lựa chọn endpoint Z.AI khác:
# --auth-choice zai-coding-cn
# --auth-choice zai-global
# --auth-choice zai-cn
```

Ghi chú luồng:

- `quickstart`: lời nhắc tối thiểu, tự động tạo token gateway.
- `manual`: đầy đủ lời nhắc cho cổng/bind/xác thực (bí danh của `advanced`).
- Bắt đầu chat nhanh nhất: `openclaw dashboard` (UI điều khiển, không cần thiết lập kênh).
- Custom Provider: kết nối với bất kỳ endpoint tương thích OpenAI hoặc Anthropic nào,
  bao gồm cả các nhà cung cấp được host nhưng không được liệt kê. Sử dụng Unknown để tự động phát hiện.

## Các lệnh tiếp theo phổ biến

```bash
openclaw configure
openclaw agents add <name>
```

<Note>
`--json` không ngụ ý chế độ không tương tác. Dùng `--non-interactive` cho script.
</Note>
