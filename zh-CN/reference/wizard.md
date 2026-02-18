---
title: 向导参考
sidebarTitle: 向导参考
---

# 向导参考

这是 `openclaw onboard` CLI 向导的完整参考文档。  
如需高层概览，请参阅 [Onboarding Wizard](/start/wizard)。

## 流程详情（本地模式）

<Steps>
  <Step title="现有配置检测">
    - 如果存在 `~/.openclaw/openclaw.json`，可选择 **保留 / 修改 / 重置**。
    - 重新运行向导**不会**清除任何内容，除非你明确选择 **重置**
      （或传入 `--reset`）。
    - 如果配置无效或包含旧版键名，向导会停止并提示
      先运行 `openclaw doctor` 再继续。
    - 重置使用 `trash`（绝不使用 `rm`），并提供以下范围：
      - 仅配置
      - 配置 + 凭证 + 会话
      - 完全重置（同时移除工作区）
  </Step>
  <Step title="模型 / 认证">
    - **Anthropic API key（推荐）**：如果存在则使用 `ANTHROPIC_API_KEY`，否则提示输入密钥，然后保存以供守护进程使用。
    - **Anthropic OAuth（Claude Code CLI）**：在 macOS 上向导会检查钥匙串项目 “Claude Code-credentials”（请选择“始终允许”，以免 launchd 启动被阻止）；在 Linux/Windows 上如果存在则复用 `~/.claude/.credentials.json`。
    - **Anthropic token（粘贴 setup-token）**：在任意机器上运行 `claude setup-token`，然后粘贴 token（可以命名；留空 = 默认）。
    - **OpenAI Code（Codex）订阅（Codex CLI）**：如果存在 `~/.codex/auth.json`，向导可以复用。
    - **OpenAI Code（Codex）订阅（OAuth）**：浏览器流程；粘贴 `code#state`。
      - 当模型未设置或为 `openai/*` 时，将 `agents.defaults.model` 设置为 `openai-codex/gpt-5.2`。
    - **OpenAI API key**：如果存在则使用 `OPENAI_API_KEY`，否则提示输入密钥，然后将其保存到 `~/.openclaw/.env`，以便 launchd 读取。
    - **xAI（Grok）API key**：提示输入 `XAI_API_KEY` 并将 xAI 配置为模型提供商。
    - **OpenCode Zen（多模型代理）**：提示输入 `OPENCODE_API_KEY`（或 `OPENCODE_ZEN_API_KEY`，可在 https://opencode.ai/auth 获取）。
    - **API key**：为你存储该密钥。
    - **Vercel AI Gateway（多模型代理）**：提示输入 `AI_GATEWAY_API_KEY`。  
      - 更多详情：[Vercel AI Gateway](/providers/vercel-ai-gateway)
    - **Cloudflare AI Gateway**：提示输入 Account ID、Gateway ID 和 `CLOUDFLARE_AI_GATEWAY_API_KEY`。  
      - 更多详情：[Cloudflare AI Gateway](/providers/cloudflare-ai-gateway)
    - **MiniMax M2.1**：自动写入配置。  
      - 更多详情：[MiniMax](/providers/minimax)
    - **Synthetic（兼容 Anthropic）**：提示输入 `SYNTHETIC_API_KEY`。  
      - 更多详情：[Synthetic](/providers/synthetic)
    - **Moonshot（Kimi K2）**：自动写入配置。
    - **Kimi Coding**：自动写入配置。  
      - 更多详情：[Moonshot AI（Kimi + Kimi Coding）](/providers/moonshot)
    - **跳过**：暂不配置认证。
    - 从检测到的选项中选择默认模型（或手动输入 provider/model）。
    - 向导会运行模型检查；如果配置的模型未知或缺少认证，会发出警告。
    - OAuth 凭证存储在 `~/.openclaw/credentials/oauth.json`；认证配置文件存储在 `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`（API keys + OAuth）。
    - 更多详情：[/concepts/oauth](/concepts/oauth)
    <Note>
    无头 / 服务器提示：请在有浏览器的机器上完成 OAuth，然后将  
    `~/.openclaw/credentials/oauth.json`（或 `$OPENCLAW_STATE_DIR/credentials/oauth.json`）复制到网关主机。
    </Note>
  </Step>
  <Step title="工作区">
    - 默认 `~/.openclaw/workspace`（可配置）。
    - 为代理引导流程初始化所需的工作区文件。
    - 完整工作区结构与备份指南：[Agent workspace](/concepts/agent-workspace)
  </Step>
  <Step title="网关">
    - 端口、绑定地址、认证模式、Tailscale 暴露。
    - 认证建议：即使仅 loopback 也保持 **Token**，这样本地 WS 客户端必须进行认证。
    - 仅在你完全信任所有本地进程时才禁用认证。
    - 非 loopback 绑定仍然需要认证。
  </Step>
  <Step title="渠道">
    - [WhatsApp](/channels/whatsapp)：可选二维码登录。
    - [Telegram](/channels/telegram)：机器人 token。
    - [Discord](/channels/discord)：机器人 token。
    - [Google Chat](/channels/googlechat)：服务账号 JSON + webhook audience。
    - [Mattermost](/channels/mattermost)（插件）：机器人 token + 基础 URL。
    - [Signal](/channels/signal)：可选安装 `signal-cli` + 账号配置。
    - [BlueBubbles](/channels/bluebubbles)：**iMessage 推荐方案**；服务器 URL + 密码 + webhook。
    - [iMessage](/channels/imessage)：传统 `imsg` CLI 路径 + 数据库访问。
    - 私信安全：默认使用配对。首次私信会发送验证码；通过 `openclaw pairing approve <channel> <code>` 批准，或使用允许列表。
  </Step>
  <Step title="安装守护进程">
    - macOS：LaunchAgent  
      - 需要已登录的用户会话；无头环境需使用自定义 LaunchDaemon（未随附）。
    - Linux（以及通过 WSL2 的 Windows）：systemd 用户单元  
      - 向导会尝试通过 `loginctl enable-linger <user>` 启用 lingering，使网关在注销后仍保持运行。
      - 可能提示输入 sudo（写入 `/var/lib/systemd/linger`）；会先尝试无 sudo。
    - **运行时选择：**Node（推荐；WhatsApp/Telegram 必需）。不推荐 Bun。
  </Step>
  <Step title="健康检查">
    - 启动网关（如需要）并运行 `openclaw health`。
    - 提示：`openclaw status --deep` 会在状态输出中添加网关健康探测（需要可访问的网关）。
  </Step>
  <Step title="技能（推荐）">
    - 读取可用技能并检查依赖要求。
    - 选择 node 管理器：**npm / pnpm**（不推荐 bun）。
    - 安装可选依赖（部分在 macOS 上使用 Homebrew）。
  </Step>
  <Step title="完成">
    - 总结 + 后续步骤，包括 iOS / Android / macOS 应用以启用更多功能。
  </Step>
</Steps>

<Note>
如果未检测到图形界面，向导会打印 SSH 端口转发说明以访问 Control UI，而不是打开浏览器。  
如果 Control UI 资源缺失，向导会尝试构建；回退命令为 `pnpm ui:build`（会自动安装 UI 依赖）。
</Note>

## 非交互模式

使用 `--non-interactive` 以自动化或脚本方式完成引导：

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice apiKey \
  --anthropic-api-key "$ANTHROPIC_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback \
  --install-daemon \
  --daemon-runtime node \
  --skip-skills
```

添加 `--json` 可获得机器可读的摘要。

<Note>
`--json` **不会**隐含非交互模式。脚本中请使用 `--non-interactive`（以及 `--workspace`）。
</Note>

<AccordionGroup>
  <Accordion title="Gemini 示例">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice gemini-api-key \
      --gemini-api-key "$GEMINI_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="Z.AI 示例">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice zai-api-key \
      --zai-api-key "$ZAI_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="Vercel AI Gateway 示例">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice ai-gateway-api-key \
      --ai-gateway-api-key "$AI_GATEWAY_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="Cloudflare AI Gateway 示例">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice cloudflare-ai-gateway-api-key \
      --cloudflare-ai-gateway-account-id "your-account-id" \
      --cloudflare-ai-gateway-gateway-id "your-gateway-id" \
      --cloudflare-ai-gateway-api-key "$CLOUDFLARE_AI_GATEWAY_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="Moonshot 示例">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice moonshot-api-key \
      --moonshot-api-key "$MOONSHOT_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="Synthetic 示例">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice synthetic-api-key \
      --synthetic-api-key "$SYNTHETIC_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="OpenCode Zen 示例">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice opencode-zen \
      --opencode-zen-api-key "$OPENCODE_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
</AccordionGroup>

### 添加代理（非交互）

```bash
openclaw agents add work \
  --workspace ~/.openclaw/workspace-work \
  --model openai/gpt-5.2 \
  --bind whatsapp:biz \
  --non-interactive \
  --json
```

## Gateway 向导 RPC

Gateway 通过 RPC 暴露向导流程（`wizard.start`、`wizard.next`、`wizard.cancel`、`wizard.status`）。  
客户端（macOS 应用、Control UI）可以渲染步骤，而无需重新实现引导逻辑。

## Signal 设置（signal-cli）

向导可以从 GitHub releases 安装 `signal-cli`：

- 下载对应的 release 资源文件。
- 存储到 `~/.openclaw/tools/signal-cli/<version>/`。
- 将 `channels.signal.cliPath` 写入配置。

注意：

- JVM 构建需要 **Java 21**。
- 如有可用，将使用原生构建版本。
- Windows 使用 WSL2；signal-cli 安装会在 WSL 内遵循 Linux 流程。

## 向导写入的内容

`~/.openclaw/openclaw.json` 中的典型字段：

- `agents.defaults.workspace`
- `agents.defaults.model` / `models.providers`（如果选择 Minimax）
- `gateway.*`（mode、bind、auth、tailscale）
- `channels.telegram.botToken`、`channels.discord.token`、`channels.signal.*`、`channels.imessage.*`
- 当你在提示中选择加入时的渠道允许列表（Slack/Discord/Matrix/Microsoft Teams）（名称在可能时解析为 ID）。
- `skills.install.nodeManager`
- `wizard.lastRunAt`
- `wizard.lastRunVersion`
- `wizard.lastRunCommit`
- `wizard.lastRunCommand`
- `wizard.lastRunMode`

`openclaw agents add` 会写入 `agents.list[]` 和可选的 `bindings`。

WhatsApp 凭证位于 `~/.openclaw/credentials/whatsapp/<accountId>/`。  
会话存储在 `~/.openclaw/agents/<agentId>/sessions/`。

部分渠道以插件形式提供。当你在引导过程中选择其中之一时，向导会提示先安装（npm 或本地路径），然后才能进行配置。

## 相关文档

- 向导概览：[Onboarding Wizard](/start/wizard)
- macOS 应用引导：[Onboarding](/start/onboarding)
- 配置参考：[Gateway configuration](/gateway/configuration)
- 渠道提供方：[WhatsApp](/channels/whatsapp)、[Telegram](/channels/telegram)、[Discord](/channels/discord)、[Google Chat](/channels/googlechat)、[Signal](/channels/signal)、[BlueBubbles](/channels/bluebubbles)（iMessage）、[iMessage](/channels/imessage)（传统）
- 技能：[Skills](/tools/skills)、[Skills config](/tools/skills-config)
