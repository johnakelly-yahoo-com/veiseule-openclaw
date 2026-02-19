---
summary: "CLI 新手引导向导：引导式配置 Gateway 网关、工作区、渠道和 Skills"
read_when:
  - 运行或配置新手引导向导
  - 设置新机器
title: "22. 入门向导（CLI）"
sidebarTitle: "23. 入门：CLI"
---

# 新手引导向导（CLI）

25. 入门向导是**推荐**在 macOS、Linux 或 Windows（通过 WSL2；强烈推荐）上设置 OpenClaw 的方式。
26. 它在一个引导流程中配置本地网关或远程网关连接，以及频道、技能和工作区默认值。

```bash
openclaw onboard
```

<Info>
28. 最快的首次聊天：打开 Control UI（无需频道设置）。 29. 运行
`openclaw dashboard` 并在浏览器中聊天。 30. 文档：[Dashboard](/web/dashboard)。
</Info>

后续重新配置：

```bash
openclaw agents add <name>
```

<Note>
提示：`--json` **不**意味着非交互模式。脚本中请使用 `--non-interactive`（和 `--workspace`）。 使用 `--non-interactive` 自动化或脚本化新手引导：
</Note>

<Tip>
推荐：设置 Brave Search API 密钥，以便智能体可以使用 `web_search`（`web_fetch` 无需密钥即可使用）。最简单的方式：`openclaw configure --section web`，它会存储 `tools.web.search.apiKey`。文档：[Web 工具](/tools/web)。 36. 最简单的方式：`openclaw configure --section web`
它会存储 `tools.web.search.apiKey`。 37. 文档：[Web tools](/tools/web)。
</Tip>

## 快速开始 vs 高级

向导从**快速开始**（默认值）vs **高级**（完全控制）开始。

<Tabs>
  <Tab title="QuickStart (defaults)">40. 
    - 本地网关（回环）
    - 工作区默认值（或现有工作区）
    - 网关端口 **18789**
    - 网关认证 **Token**（即使在回环模式下也会自动生成）
    - Tailscale 暴露 **关闭**
    - Telegram + WhatsApp 私信默认为 **允许列表**（将提示你输入手机号）
  
</Tab>
  <Tab title="Advanced (full control)">41. 
    - 暴露每一步（模式、工作区、网关、频道、守护进程、技能）。
  
</Tab>
</Tabs>

## 42. 向导会配置的内容

\*\*本地模式（默认）\*\*引导你完成：

1. **模型/认证** — Anthropic API key（推荐）、OpenAI，或 Custom Provider
   （OpenAI-compatible、Anthropic-compatible，或 Unknown 自动检测）。 45. 选择一个默认模型。
2. 46. **工作区** — 代理文件的位置（默认 `~/.openclaw/workspace`）。 47. 初始化引导文件。
3. 48. **网关** — 端口、绑定地址、认证模式、Tailscale 暴露。
4. 49. **频道** — WhatsApp、Telegram、Discord、Google Chat、Mattermost、Signal、BlueBubbles 或 iMessage。
5. 50. **守护进程** — 安装 LaunchAgent（macOS）或 systemd 用户单元（Linux/WSL2）。
6. 1. **健康检查** — 启动 Gateway 并验证其是否正在运行。
7. **技能** — 安装推荐的技能和可选依赖。

<Note>
重新运行向导**不会**清除任何内容，除非你明确选择**重置**（或传递 `--reset`）。
如果配置无效或包含遗留键名，向导会停止并要求你在继续之前运行 `openclaw doctor`。
</Note>

**远程模式**仅配置本地客户端连接到其他位置的 Gateway 网关。
它**不会**在远程主机上安装或更改任何内容。
6. 它**不会**在远程主机上安装或更改任何内容。

## 添加另一个智能体

使用 `openclaw agents add <name>` 创建一个具有独立工作区、会话和认证配置文件的单独智能体。不带 `--workspace` 运行会启动向导。 9. 在未指定 `--workspace` 的情况下运行会启动向导。

它设置的内容：

- `agents.list[].name`
- `agents.list[].workspace`
- `agents.list[].agentDir`

注意事项：

- 默认工作区遵循 `~/.openclaw/workspace-<agentId>`。
- 添加 `bindings` 以路由入站消息（向导可以执行此操作）。
- 非交互标志：`--model`、`--agent-dir`、`--bind`、`--non-interactive`。

## 18. 完整参考

19. 有关详细的分步说明、非交互式脚本、Signal 设置、RPC API 以及向导写入的全部配置字段列表，请参见
    [向导参考](/reference/wizard)。

## 相关文档

- 21. CLI 命令参考：[`openclaw onboard`](/cli/onboard)
- Onboarding 概览： [Onboarding Overview](/start/onboarding-overview)
- macOS 应用新手引导：[新手引导](/start/onboarding)
- 23. 代理首次运行仪式： [Agent Bootstrapping](/start/bootstrapping)
