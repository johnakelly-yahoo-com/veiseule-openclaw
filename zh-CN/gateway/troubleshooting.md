---
summary: "面向 gateway、频道、自动化、节点和浏览器的深度故障排查运行手册"
read_when:
  - 故障排查中心将你指引到这里以进行更深入的诊断
  - 你需要基于稳定症状的运行手册章节，并包含精确的命令
title: "故障排除"
---

# gateway/troubleshooting.md

本页面是深度运行手册。
如果你想先走快速分诊流程，请从 [/help/troubleshooting](/help/troubleshooting) 开始。

## 命令阶梯

按以下顺序首先运行这些命令：

```bash
openclaw gateway status
openclaw doctor
```

预期的健康信号：

- `openclaw gateway status` 显示 `Runtime: running` 且 `RPC probe: ok`。
- `openclaw doctor` 报告没有阻塞性的配置/服务问题。
- `openclaw channels status --probe`

## 消息未触发

如果频道已启动但没有任何响应，在重新连接任何内容之前，请先检查路由和策略。

```bash
openclaw channels logout
trash "${OPENCLAW_STATE_DIR:-$HOME/.openclaw}/credentials" # 如果 logout 无法完全清除所有内容
openclaw channels login --verbose       # 重新扫描二维码
```

Look for:

- Pairing pending for DM senders.
- Group mention gating (`requireMention`, `mentionPatterns`).
- Channel/group allowlist mismatches.

Common signatures:

- `drop guild message (mention required` → group message ignored until mention.
- `pairing request` → sender needs approval.
- `blocked` / `allowlist` → sender/channel was filtered by policy.

Related:

- 文档：[Discord](/channels/discord)、[渠道故障排除](/channels/troubleshooting)。
- [/channels/pairing](/channels/pairing)
- [/channels/groups](/channels/groups)

## Dashboard control ui connectivity

When dashboard/control UI will not connect, validate URL, auth mode, and secure context assumptions.

```bash
`openclaw gateway status` 和 `openclaw doctor` 在服务看起来正在运行但端口关闭时会显示日志中的**最后 Gateway 网关错误**。
```

Look for:

- Correct probe URL and dashboard URL.
- Auth mode/token mismatch between client and gateway.
- HTTP usage where device identity is required.

Common signatures:

- `device identity required` → non-secure context or missing device auth.
- `unauthorized` / reconnect loop → token/password mismatch.
- `gateway connect failed:` → wrong host/port/url target.

Related:

- [/web/control-ui](/web/control-ui)
- [/gateway/authentication](/gateway/authentication)
- [/gateway/remote](/gateway/remote)

## 服务运行但端口未监听

服务已安装但没有运行

```bash
openclaw gateway status
```

Look for:

- `Runtime: stopped` with exit hints.
- `Config (cli): ...` 和 `Config (service): ...` 通常应该匹配。
- Port/listener conflicts.

Common signatures:

- "Gateway start blocked: set gateway.mode=local" openclaw config set gateway.mode local 如果你使用专用的 `openclaw` 用户通过 Podman 运行 OpenClaw，配置文件位于 `~openclaw/.openclaw/openclaw.json`。
- **如果 `Last gateway error:` 提到"refusing to bind … without auth"**
- `another gateway instance is already listening` / `EADDRINUSE` → port conflict.

Related:

- [/gateway/background-process](/gateway/background-process)
- [/gateway/configuration](/gateway/configuration)
- [/gateway/doctor](/gateway/doctor)

## Channel connected messages not flowing

If channel state is connected but message flow is dead, focus on policy, permissions, and channel specific delivery rules.

```bash
运行 `openclaw channels status --probe` 获取审计提示。
```

Look for:

- DM policy (`pairing`, `allowlist`, `open`, `disabled`).
- Group allowlist and mention requirements.
- Missing channel API permissions/scopes.

Common signatures:

- `mention required` → message ignored by group mention policy.
- `pairing` / pending approval traces → sender is not approved.
- `missing_scope`, `not_in_channel`, `Forbidden`, `401/403` → channel auth/permissions issue.

Related:

- 特定提供商的快捷方式：[/channels/troubleshooting](/channels/troubleshooting)
- 参见 [WhatsApp 设置](/channels/whatsapp)。
- [/channels/telegram](/channels/telegram)
- 参见 [流式传输](/concepts/streaming)。

## Cron and heartbeat delivery

If cron or heartbeat did not run or did not deliver, verify scheduler state first, then delivery target.

```bash
openclaw cron status
openclaw cron list
openclaw cron runs --id <jobId> --limit 20
openclaw system heartbeat last
openclaw logs --follow
```

Look for:

- Cron enabled and next wake present.
- Job run history status (`ok`, `skipped`, `error`).
- Heartbeat skip reasons (`quiet-hours`, `requests-in-flight`, `alerts-disabled`).

Common signatures:

- `cron: scheduler disabled; jobs will not run automatically` → cron disabled.
- `cron: timer tick failed` → scheduler tick failed; check file/log/runtime errors.
- `heartbeat skipped` with `reason=quiet-hours` → outside active hours window.
- `heartbeat: unknown accountId` → invalid account id for heartbeat delivery target.

Related:

- [/automation/troubleshooting](/automation/troubleshooting)
- [/automation/cron-jobs](/automation/cron-jobs)
- [/gateway/heartbeat](/gateway/heartbeat)

## Node paired tool fails

If a node is paired but tools fail, isolate foreground, permission, and approval state.

```bash
openclaw nodes status
openclaw nodes describe --node <idOrNameOrIp>
openclaw approvals get --node <idOrNameOrIp>
openclaw logs --follow
openclaw status
```

Look for:

- Node online with expected capabilities.
- OS permission grants for camera/mic/location/screen.
- Exec approvals and allowlist state.

Common signatures:

- `NODE_BACKGROUND_UNAVAILABLE` → node app must be in foreground.
- `*_PERMISSION_REQUIRED` / `LOCATION_PERMISSION_REQUIRED` → missing OS permission.
- `SYSTEM_RUN_DENIED: approval required` → exec approval pending.
- `SYSTEM_RUN_DENIED: allowlist miss` → command blocked by allowlist.

Related:

- [/nodes/troubleshooting](/nodes/troubleshooting)
- [/nodes/index](/nodes/index)
- [/tools/exec-approvals](/tools/exec-approvals)

## Browser tool fails

Use this when browser tool actions fail even though the gateway itself is healthy.

```bash
openclaw doctor
openclaw doctor --fix
```

Look for:

- Valid browser executable path.
- CDP profile reachability.
- Extension relay tab attachment for `profile="chrome"`.

Common signatures:

- 如果你看到 `"Failed to start Chrome CDP on port 18800"`：
- `browser.executablePath not found` → configured path is invalid.
- `Chrome extension relay is running, but no tab is connected` → 扩展中继未附加。
- `Browser attachOnly is enabled ... not reachable` → 仅附加配置没有可达目标。

Related:

- **完整指南：** 参见 [browser-linux-troubleshooting](/tools/browser-linux-troubleshooting)
- [/tools/chrome-extension](/tools/chrome-extension)
- [/tools/browser](/tools/browser)

## 如果你升级后突然出现故障

大多数升级后的故障是由于配置漂移或现在开始强制执行更严格的默认值。

### 1. 认证和 URL 覆盖行为已更改

```bash
openclaw config set gateway.mode remote
openclaw config set gateway.remote.url "wss://gateway.example.com"
```

需要检查的内容：

- 如果你设置了 `gateway.mode=remote`，**CLI 默认**使用远程 URL。服务可能仍在本地运行，但你的 CLI 可能在探测错误的位置。使用 `openclaw gateway status` 查看服务解析的端口 + 探测目标（或传递 `--url`）。
- Explicit `--url` calls do not fall back to stored credentials.

Common signatures:

- `gateway connect failed:` → URL 目标错误。
- `unauthorized` → 端点可达，但认证错误。

### 2. 绑定和认证防护更严格了

```bash
# 在 Gateway 网关主机上运行（粘贴 setup-token）
openclaw models auth setup-token --provider anthropic
openclaw models status
```

需要检查的内容：

- Non-loopback binds (`lan`, `tailnet`, `custom`) need auth configured.
- `gateway.token` 被忽略；使用 `gateway.auth.token`。

Common signatures:

- Gateway 网关卡在"Starting..." without auth\` → 绑定与认证不匹配。
- `RPC probe: failed` 且运行时仍在运行 → 网关存活，但以当前认证/URL 无法访问。

### 3. 配对和设备身份状态已更改

```bash
openclaw pairing list <channel>
```

需要检查的内容：

- 仪表板/节点存在待批准的设备。
- 策略或身份更改后，存在待批准的 DM 配对。

Common signatures:

- `device identity required` → 设备认证未满足。
- `pairing required` → 发送方/设备必须获批。

如果在检查后服务配置与运行时仍不一致，请从同一配置文件/状态目录重新安装服务元数据：

```bash
openclaw doctor
openclaw gateway restart
```

Related:

- [/gateway/pairing](/gateway/pairing)
- [/gateway/authentication](/gateway/authentication)
- [/gateway/background-process](/gateway/background-process)
