---
title: "CLI 引导参考"
sidebarTitle: "CLI 参考"
---

# CLI 引导参考

本页面是 `openclaw onboard` 的完整参考文档。  
简要指南请参阅 [Onboarding Wizard (CLI)](/start/wizard)。

## 向导的功能

本地模式（默认）将引导你完成：

- 模型和认证设置（OpenAI Code 订阅 OAuth、Anthropic API key 或 setup token，以及 MiniMax、GLM、Moonshot 和 AI Gateway 选项）
- 工作区位置与引导初始化文件
- Gateway 设置（端口、绑定地址、认证、tailscale）
- 渠道与提供商（Telegram、WhatsApp、Discord、Google Chat、Mattermost 插件、Signal）
- 守护进程安装（LaunchAgent 或 systemd 用户单元）
- 健康检查
- 技能设置

远程模式将此机器配置为连接到其他位置的 gateway。  
它不会在远程主机上安装或修改任何内容。

## 本地流程详情

<Steps>
  <Step title="现有配置检测">
    - 如果存在 `~/.openclaw/openclaw.json`，可选择 Keep、Modify 或 Reset。
    - 重新运行向导不会清除任何内容，除非你明确选择 Reset（或传入 `--reset`）。
    - 如果配置无效或包含旧版键，向导会停止并要求你先运行 `openclaw doctor`。
    - Reset 使用 `trash`，并提供以下范围选项：
      - 仅配置
      - 配置 + 凭证 + 会话
      - 完全重置（同时移除工作区）
  </Step>
  <Step title="模型与认证">
    - 完整选项矩阵见 [Auth and model options](#auth-and-model-options)。
  </Step>
  <Step title="工作区">
    - 默认 `~/.openclaw/workspace`（可配置）。
    - 初始化首运行所需的工作区文件。
    - 工作区结构： [Agent workspace](/concepts/agent-workspace)。
  </Step>
  <Step title="Gateway">
    - 提示输入端口、绑定地址、认证模式和 tailscale 暴露方式。
    - 推荐：即使仅绑定到 loopback，也保持启用 token 认证，以确保本地 WS 客户端必须进行身份验证。
    - 仅在完全信任所有本地进程时才禁用认证。
    - 非 loopback 绑定仍然需要认证。
  </Step>
  <Step title="渠道">
    - [WhatsApp](/channels/whatsapp)：可选 QR 登录
    - [Telegram](/channels/telegram)：bot token
    - [Discord](/channels/discord)：bot token
    - [Google Chat](/channels/googlechat)：service account JSON + webhook audience
    - [Mattermost](/channels/mattermost) 插件：bot token + base URL
    - [Signal](/channels/signal)：可选安装 `signal-cli` + 账户配置
    - [BlueBubbles](/channels/bluebubbles)：推荐用于 iMessage；需要 server URL + password + webhook
    - [iMessage](/channels/imessage)：传统 `imsg` CLI 路径 + 数据库访问
    - DM 安全：默认启用配对。首次 DM 会发送验证码；通过  
      `openclaw pairing approve <channel> <code>` 批准，或使用 allowlist。
  </Step>
  <Step title="守护进程安装">
    - macOS：LaunchAgent  
      - 需要用户已登录会话；如需无头模式，请使用自定义 LaunchDaemon（未随软件提供）。
    - Linux 和 Windows（通过 WSL2）：systemd 用户单元  
      - 向导会尝试执行 `loginctl enable-linger <user>`，以便在注销后 gateway 仍保持运行。  
      - 可能会提示 sudo（写入 `/var/lib/systemd/linger`）；会先尝试无 sudo 执行。
    - 运行时选择：Node（推荐；WhatsApp 和 Telegram 必需）。不推荐使用 Bun。
  </Step>
  <Step title="健康检查">
    - 启动 gateway（如有需要）并运行 `openclaw health`。
    - `openclaw status --deep` 会在状态输出中添加 gateway 健康探测。
  </Step>
  <Step title="技能">
    - 读取可用技能并检查其要求。
    - 选择 node 管理器：npm 或 pnpm（不推荐 bun）。
    - 安装可选依赖（部分在 macOS 上使用 Homebrew）。
  </Step>
  <Step title="完成">
    - 显示摘要和后续步骤，包括 iOS、Android 和 macOS 应用选项。
  </Step>
</Steps>

<Note>
如果未检测到 GUI，向导会打印 Control UI 的 SSH 端口转发说明，而不是打开浏览器。  
如果缺少 Control UI 资源，向导会尝试构建；备用方式为 `pnpm ui:build`（会自动安装 UI 依赖）。
</Note>

## 远程模式详情

远程模式将此机器配置为连接到其他位置的 gateway。

<Info>
远程模式不会在远程主机上安装或修改任何内容。
</Info>

需要设置：

- 远程 gateway URL（`ws://...`）
- 若远程 gateway 需要认证，则提供 token（推荐）

<Note>
- 如果 gateway 仅绑定 loopback，请使用 SSH 隧道或 tailnet。
- 发现提示：
  - macOS：Bonjour（`dns-sd`）
  - Linux：Avahi（`avahi-browse`）
</Note>

## 认证与模型选项

<AccordionGroup>
  <Accordion title="Anthropic API key（推荐）">
    如果存在 `ANTHROPIC_API_KEY` 则使用，否则提示输入 key，然后保存以供守护进程使用。
  </Accordion>
  <Accordion title="Anthropic OAuth（Claude Code CLI）">
    - macOS：检查 Keychain 项目 "Claude Code-credentials"
    - Linux 和 Windows：如果存在则复用 `~/.claude/.credentials.json`

    在 macOS 上请选择“Always Allow”，以避免 launchd 启动时被阻塞。

  </Accordion>
  <Accordion title="Anthropic token（setup-token 粘贴）">
    在任意机器上运行 `claude setup-token`，然后粘贴 token。  
    你可以为其命名；留空则使用默认名称。
  </Accordion>
  <Accordion title="OpenAI Code subscription（复用 Codex CLI）">
    如果存在 `~/.codex/auth.json`，向导可以复用。
  </Accordion>
  <Accordion title="OpenAI Code subscription（OAuth）">
    浏览器流程；粘贴 `code#state`。

    当模型未设置或为 `openai/*` 时，将 `agents.defaults.model` 设置为 `openai-codex/gpt-5.3-codex`。

  </Accordion>
  <Accordion title="OpenAI API key">
    如果存在 `OPENAI_API_KEY` 则使用，否则提示输入 key，然后保存到  
    `~/.openclaw/.env` 以便 launchd 读取。

    当模型未设置、为 `openai/*` 或 `openai-codex/*` 时，将 `agents.defaults.model` 设置为 `openai/gpt-5.1-codex`。

  </Accordion>
  <Accordion title="xAI（Grok）API key">
    提示输入 `XAI_API_KEY` 并将 xAI 配置为模型提供商。
  </Accordion>
  <Accordion title="OpenCode Zen">
    提示输入 `OPENCODE_API_KEY`（或 `OPENCODE_ZEN_API_KEY`）。  
    设置地址：[opencode.ai/auth](https://opencode.ai/auth)。
  </Accordion>
  <Accordion title="API key（通用）">
    为你保存该 key。
  </Accordion>
  <Accordion title="Vercel AI Gateway">
    提示输入 `AI_GATEWAY_API_KEY`。  
    更多信息：[Vercel AI Gateway](/providers/vercel-ai-gateway)。
  </Accordion>
  <Accordion title="Cloudflare AI Gateway">
    提示输入 account ID、gateway ID 和 `CLOUDFLARE_AI_GATEWAY_API_KEY`。  
    更多信息：[Cloudflare AI Gateway](/providers/cloudflare-ai-gateway)。
  </Accordion>
  <Accordion title="MiniMax M2.1">
    自动写入配置。  
    更多信息：[MiniMax](/providers/minimax)。
  </Accordion>
  <Accordion title="Synthetic（Anthropic 兼容）">
    提示输入 `SYNTHETIC_API_KEY`。  
    更多信息：[Synthetic](/providers/synthetic)。
  </Accordion>
  <Accordion title="Moonshot 和 Kimi Coding">
    自动写入 Moonshot（Kimi K2）和 Kimi Coding 配置。  
    更多信息：[Moonshot AI (Kimi + Kimi Coding)](/providers/moonshot)。
  </Accordion>
  <Accordion title="自定义提供商">
    适用于 OpenAI 兼容和 Anthropic 兼容端点。

    非交互式参数：
    - `--auth-choice custom-api-key`
    - `--custom-base-url`
    - `--custom-model-id`
    - `--custom-api-key`（可选；默认使用 `CUSTOM_API_KEY`）
    - `--custom-provider-id`（可选）
    - `--custom-compatibility <openai|anthropic>`（可选；默认 `openai`）

  </Accordion>
  <Accordion title="跳过">
    不配置认证。
  </Accordion>
</AccordionGroup>

模型行为：

- 从检测到的选项中选择默认模型，或手动输入 provider 和 model。
- 向导会执行模型检查，如果配置的模型未知或缺少认证会发出警告。

凭证与配置文件路径：

- OAuth 凭证：`~/.openclaw/credentials/oauth.json`
- 认证配置（API key + OAuth）：`~/.openclaw/agents/<agentId>/agent/auth-profiles.json`

<Note>
无头或服务器提示：在有浏览器的机器上完成 OAuth，然后将  
`~/.openclaw/credentials/oauth.json`（或 `$OPENCLAW_STATE_DIR/credentials/oauth.json`）  
复制到 gateway 主机。
</Note>

## 输出与内部机制

`~/.openclaw/openclaw.json` 中的常见字段：

- `agents.defaults.workspace`
- `agents.defaults.model` / `models.providers`（如果选择 Minimax）
- `gateway.*`（mode、bind、auth、tailscale）
- `channels.telegram.botToken`、`channels.discord.token`、`channels.signal.*`、`channels.imessage.*`
- 当在提示中选择启用时的渠道 allowlist（Slack、Discord、Matrix、Microsoft Teams）（名称会尽可能解析为 ID）
- `skills.install.nodeManager`
- `wizard.lastRunAt`
- `wizard.lastRunVersion`
- `wizard.lastRunCommit`
- `wizard.lastRunCommand`
- `wizard.lastRunMode`

`openclaw agents add` 会写入 `agents.list[]` 以及可选的 `bindings`。

WhatsApp 凭证存储在 `~/.openclaw/credentials/whatsapp/<accountId>/`。  
会话存储在 `~/.openclaw/agents/<agentId>/sessions/`。

<Note>
某些渠道以插件形式提供。在引导过程中选择后，向导会在进行渠道配置之前提示安装插件（npm 或本地路径）。
</Note>

Gateway 向导 RPC：

- `wizard.start`
- `wizard.next`
- `wizard.cancel`
- `wizard.status`

客户端（macOS 应用和 Control UI）可以渲染步骤，而无需重新实现引导逻辑。

Signal 设置行为：

- 下载相应的发布资源
- 存储在 `~/.openclaw/tools/signal-cli/<version>/`
- 在配置中写入 `channels.signal.cliPath`
- JVM 构建需要 Java 21
- 如可用则使用原生构建版本
- Windows 使用 WSL2，并在 WSL 内遵循 Linux signal-cli 流程

## 相关文档

- 引导中心：[Onboarding Wizard (CLI)](/start/wizard)
- 自动化与脚本：[CLI Automation](/start/wizard-cli-automation)
- 命令参考：[`openclaw onboard`](/cli/onboard)