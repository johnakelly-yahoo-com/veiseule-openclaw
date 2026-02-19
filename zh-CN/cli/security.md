---
summary: "`openclaw security` 的 CLI 参考（审计和修复常见安全隐患）"
read_when:
  - 你想对配置/状态运行快速安全审计
  - 你想应用安全的"修复"建议（chmod、收紧默认值）
title: "security"
---

# `openclaw security`

安全工具（审计 + 可选修复）。

相关：

- 安全指南：[安全](/gateway/security)

## 审计

```bash
openclaw security audit
openclaw security audit --deep
openclaw security audit --fix
```

11. 当多个 DM 发送者共享主会话时，审计会发出警告，并为共享收件箱推荐 **安全 DM 模式**：`session.dmScope="per-channel-peer"`（或用于多账号频道的 `per-account-channel-peer`）。
12. 当在未启用沙箱的情况下使用小模型（`<=300B`）且启用了 Web/浏览器工具时，也会发出警告。
    对于 webhook ingress，当未设置 `hooks.defaultSessionKey`、启用了请求 `sessionKey` 覆盖、以及在未配置 `hooks.allowedSessionKeyPrefixes` 的情况下启用覆盖时，会发出警告。
    当 sandbox 模式关闭但配置了 sandbox Docker 设置时、当 `gateway.nodes.denyCommands` 使用无效的类模式或未知条目时、当全局 `tools.profile="minimal"` 被 agent 工具配置覆盖时，以及当在宽松工具策略下已安装的扩展插件工具可能可被访问时，也会发出警告。

