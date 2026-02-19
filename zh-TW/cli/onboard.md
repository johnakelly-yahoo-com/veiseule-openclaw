---
summary: "CLI 參考文件：`openclaw onboard`（互動式入門引導精靈）"
read_when:
  - 你想要為 Gateway 閘道器、工作區、身分驗證、頻道與 Skills 進行引導式設定
title: "onboard"
---

# `openclaw onboard`

互動式入門引導精靈（本機或遠端 Gateway 閘道器設定）。

## 相關指南

- CLI 入門引導中樞：[Onboarding Wizard (CLI)](/start/wizard)
- Onboarding 概覽：[Onboarding Overview](/start/onboarding-overview)
- CLI 自動化：[CLI Automation](/start/wizard-cli-automation)
- CLI 入門引導參考：[CLI Onboarding Reference](/start/wizard-cli-reference)
- macOS 入門引導：[Onboarding (macOS App)](/start/onboarding)

## 範例

```bash
openclaw onboard
openclaw onboard --flow quickstart
openclaw onboard --flow manual
openclaw onboard --mode remote --remote-url ws://gateway-host:18789
```

非互動式自訂 provider：

```bash
openclaw onboard --non-interactive \
  --auth-choice custom-api-key \
  --custom-base-url "https://llm.example.com/v1" \
  --custom-model-id "foo-large" \
  --custom-api-key "$CUSTOM_API_KEY" \
  --custom-compatibility openai
```

在非互動模式下，`--custom-api-key` 為選填。 若省略，onboarding 會檢查 `CUSTOM_API_KEY`。

非互動式 Z.AI 端點選項：

注意：`--auth-choice zai-api-key` 現在會為您的金鑰自動偵測最佳的 Z.AI 端點（優先使用搭配 `zai/glm-5` 的一般 API）。
若您特別需要 GLM Coding Plan 端點，請選擇 `zai-coding-global` 或 `zai-coding-cn`。

```bash
# Promptless endpoint selection
openclaw onboard --non-interactive \
  --auth-choice zai-coding-global \
  --zai-api-key "$ZAI_API_KEY"

# Other Z.AI endpoint choices:
# --auth-choice zai-coding-cn
# --auth-choice zai-global
# --auth-choice zai-cn
```

Flow notes:

- `quickstart`：最少提示，自動產生 Gateway 閘道器權杖。
- `manual`：包含連接埠／綁定／身分驗證的完整提示（`advanced` 的別名）。
- 最快開始第一個聊天：`openclaw dashboard`（控制介面，不進行頻道設定）。
- 自訂 Provider：可連接任何相容於 OpenAI 或 Anthropic 的端點，
  包含未列出的託管 provider。 使用 Unknown 以自動偵測。

## 常見後續指令

```bash
openclaw configure
openclaw agents add <name>
```

<Note>
`--json` 並不代表非互動模式。用於腳本請使用 `--non-interactive`。
 Use `--non-interactive` for scripts. Use `--non-interactive` for scripts.
</Note>

