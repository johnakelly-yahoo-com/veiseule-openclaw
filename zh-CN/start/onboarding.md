---
summary: "OpenClaw 的首次运行新手引导流程（macOS 应用）"
read_when:
  - 设计 macOS 新手引导助手
  - 实现认证或身份设置
title: "28. 引导（macOS 应用）"
sidebarTitle: "29. 引导：macOS 应用"
---

# 新手引导（macOS 应用）

31. 本文档描述 **当前** 的首次运行引导流程。 32. 目标是一个
    流畅的“第 0 天”体验：选择 Gateway 的运行位置，连接认证，运行向导，并让代理自行完成引导启动。
    有关 onboarding 路径的总体说明，请参见 [Onboarding Overview](/start/onboarding-overview)。

<Steps>
<Step title="Approve macOS warning">33. 
<Frame>
<img src="/assets/macos-onboarding/01-macos-warning.jpeg" alt=""></img>34. 
</Frame>
</Step>
<Step title="Approve find local networks">35. 
<Frame>
<img src="/assets/macos-onboarding/02-local-networks.jpeg" alt=""></img>36. 
</Frame>
</Step>
<Step title="Welcome and security notice">阅读显示的安全提示并相应决定。<img src="/assets/macos-onboarding/03-security-notice.png" alt=""></img>38. 
</Frame>
</Step>
<Step title="Local vs Remote">39. 
<Frame>
<img src="/assets/macos-onboarding/04-choose-gateway.png" alt=""></img>
</Frame>

**Gateway 网关**在哪里运行？

- **本地（此 Mac）：** 新手引导可以在本地运行 OAuth 流程并写入凭证。
- **远程（通过 SSH/Tailnet）：** 新手引导**不会**在本地运行 OAuth；凭证必须存在于 Gateway 网关主机上。
- **稍后配置：** 跳过设置并保持应用未配置状态。

<Tip>
向导现在即使对于 loopback 也会生成**令牌**，因此本地 WS 客户端必须认证。
如果你禁用认证，任何本地进程都可以连接；仅在完全受信任的机器上使用。
对于多机器访问或非 loopback 绑定，使用**令牌**。
</Tip>
</Step>
<Step title="Permissions">48. 
<Frame caption="选择你希望授予 OpenClaw 的权限"><img src="/assets/macos-onboarding/05-permissions.png" alt=""></img>
</Frame>

新手引导请求以下所需的 TCC 权限：

- 自动化（AppleScript）
- 通知
- 辅助功能
- 屏幕录制
- 麦克风 / 语音识别
- Speech Recognition
- Camera
- 25. 位置

</Step>
<Step title="CLI">
  <Info>
This step is optional
</Info>应用可以通过 npm/pnpm 安装全局 `openclaw` CLI，以便终端工作流和 launchd 任务开箱即用。
</Step>
<Step title="Onboarding Chat (dedicated session)">
  After setup, the app opens a dedicated onboarding chat session so the agent can
  introduce itself and guide next steps. This keeps first‑run guidance separate
  from your normal conversation. See [Bootstrapping](/start/bootstrapping) for
  what happens on the gateway host during the first agent run.
</Step>
</Steps>
