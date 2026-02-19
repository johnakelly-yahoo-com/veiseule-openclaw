---
summary: "Get OpenClaw installed and run your first chat in minutes."
read_when:
  - 从零开始首次设置
  - You want the fastest path to a working chat
title: "Getting Started"
---

# Getting Started

目标：尽快从**零**到**第一个可用聊天**（使用合理的默认值）。

<Info>
Fastest chat: open the Control UI (no channel setup needed). 最快聊天：打开 Control UI（无需渠道设置）。运行 `openclaw dashboard` 并在浏览器中聊天，或在 Gateway 网关主机上打开 `http://127.0.0.1:18789/`。文档：[Dashboard](/web/dashboard) 和 [Control UI](/web/control-ui)。
<Tooltip headline="Gateway host" tip="The machine running the OpenClaw gateway service.">gateway host</Tooltip>.
Docs: [Dashboard](/web/dashboard) and [Control UI](/web/control-ui).
</Info>

## Prereqs

- Node 22 or newer

<Tip>
Check your Node version with `node --version` if you are unsure.
</Tip>

## Quick setup (CLI)

<Steps>
  <Step title="Install OpenClaw (recommended)">
    <Tabs>
      <Tab title="macOS/Linux">curl -fsSL https://openclaw.ai/install.sh | bash<img
  src="/assets/install-script.svg"
  alt="Install Script Process"
  className="rounded-lg"></img>
      
</Tab>
      <Tab title="Windows (PowerShell)">iwr -useb https://openclaw.ai/install.ps1 | iex
</Tab>
    
</Tabs>

    ````
    ```
    <Note>
    Other install methods and requirements: [Install](/install).
    
</Note>
    ```
    ````

  
</Step>
  <Step title="Run the onboarding wizard">openclaw onboard --install-daemon

    ```
    如果你想要更深入的参考页面，跳转到：[向导](/start/wizard)、[设置](/start/setup)、[配对](/channels/pairing)、[安全](/gateway/security)。
    ```

  
</Step>
  <Step title="Check the Gateway">如果你在新手引导期间安装了服务，Gateway 网关应该已经在运行：

    ```
    openclaw gateway status
    ```

  
</Step>
  <Step title="Open the Control UI">
    ```bash
    openclaw dashboard
    ```
  
</Step>
</Steps>

<Check>
If the Control UI loads, your Gateway is ready for use.
</Check>

## Optional checks and extras

<AccordionGroup>
  <Accordion title="Run the Gateway in the foreground">
    Useful for quick tests or troubleshooting.

    ```
    openclaw gateway --port 18789 --verbose
    ```

  
</Accordion>
  <Accordion title="Send a test message">
    Requires a configured channel.

    ```
    openclaw message send --target +15555550123 --message "Hello from OpenClaw"
    ```

  
</Accordion>
</AccordionGroup>

## Useful environment variables

If you run OpenClaw as a service account or want custom config/state locations:

- `OPENCLAW_HOME` sets the home directory used for internal path resolution.
- `OPENCLAW_STATE_DIR` 覆盖状态目录。
- `OPENCLAW_CONFIG_PATH` 覆盖配置文件路径。

完整的环境变量参考：[Environment vars](/help/environment)。

## ```
完整的 CLI 向导参考和高级选项。
```

<Columns>
  <Card title="Onboarding Wizard (details)" href="/start/wizard">完整的 CLI 向导参考和高级选项。
</Card>
  <Card title="macOS app onboarding" href="/start/onboarding">你将获得
</Card>
</Columns>

## 一个正在运行的 Gateway

- 已配置认证
- 控制 UI 访问或已连接的渠道
- 下一步

## 私信安全与审批：[配对](/channels/pairing)

- 连接更多渠道：[Channels](/channels)
- Mattermost（插件）：[Mattermost](/channels/mattermost)
- 链接到所有 OpenClaw 文档的枢纽
