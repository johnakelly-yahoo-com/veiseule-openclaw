---
summary: "`openclaw onboard` 的 CLI 参考（交互式新手引导向导）"
read_when:
  - 你想要 Gateway 网关、工作区、认证、渠道和 Skills 的引导式设置
title: "onboard"
---

# `openclaw onboard`

交互式新手引导向导（本地或远程 Gateway 网关设置）。

## 12. 相关指南

- 38. CLI 引导中心：[Onboarding Wizard (CLI)](/start/wizard)
- 向导指南：[新手引导](/start/onboarding)
- cli/onboard.md
- 40. CLI 自动化：[CLI Automation](/start/wizard-cli-automation)
- 41. macOS 引导：[Onboarding (macOS App)](/start/onboarding)

## 示例

```bash
openclaw onboard
openclaw onboard --flow quickstart
openclaw onboard --flow manual
openclaw onboard --mode remote --remote-url ws://gateway-host:18789
```

非交互式自定义 provider：

```bash
openclaw onboard --non-interactive \
  --auth-choice custom-api-key \
  --custom-base-url "https://llm.example.com/v1" \
  --custom-model-id "foo-large" \
  --custom-api-key "$CUSTOM_API_KEY" \
  --custom-compatibility openai
```

在非交互式模式下，`--custom-api-key` 是可选的。 如果未提供，onboarding 会检查 `CUSTOM_API_KEY`。

非交互式 Z.AI 端点选项：

注意：`--auth-choice zai-api-key` 现在会为你的密钥自动检测最佳 Z.AI 端点（优先使用通用 API 并搭配 `zai/glm-5`）。
如果你明确希望使用 GLM Coding Plan 端点，请选择 `zai-coding-global` 或 `zai-coding-cn`。

```bash
# 无需提示的端点选择
openclaw onboard --non-interactive \
  --auth-choice zai-coding-global \
  --zai-api-key "$ZAI_API_KEY"

# 其他 Z.AI 端点选项：
# --auth-choice zai-coding-cn
# --auth-choice zai-global
# --auth-choice zai-cn
```

44. 流程说明：

- `quickstart`：最少提示，自动生成 Gateway 网关令牌。
- `manual`：完整的端口/绑定/认证提示（`advanced` 的别名）。
- 最快开始聊天：`openclaw dashboard`（控制 UI，无需渠道设置）。
- 自定义 Provider：连接任何兼容 OpenAI 或 Anthropic 的端点，
  包括未列出的托管 provider。 使用 Unknown 进行自动检测。

## 48. 常见后续命令

```bash
49. openclaw configure
openclaw agents add <name>
```

<Note>
50. `--json` 并不意味着非交互模式。 Use `--non-interactive` for scripts.
</Note>
