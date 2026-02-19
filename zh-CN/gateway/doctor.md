---
summary: "Doctor 命令：健康检查、配置迁移和修复步骤"
read_when:
  - 添加或修改 doctor 迁移
  - 引入破坏性配置更改
title: "医生"
---

# 医生

`openclaw doctor` 是 OpenClaw 的修复 + 迁移工具。它修复过时的配置/状态，检查健康状况，并提供可操作的修复步骤。 5. 它可修复过期的
配置/状态、进行健康检查，并提供可执行的修复步骤。

## 快速开始

```bash
openclaw doctor
```

### 无头/自动化

```bash
openclaw doctor --yes
```

无需提示接受默认值（包括适用时的重启/服务/沙箱修复步骤）。

```bash
openclaw doctor --repair
```

无需提示应用推荐的修复（安全时进行修复 + 重启）。

```bash
openclaw doctor --repair --force
```

也应用激进的修复（覆盖自定义 supervisor 配置）。

```bash
openclaw doctor --non-interactive
```

16. 在无提示的情况下运行，仅应用安全的迁移（配置规范化 + 磁盘上的状态迁移）。 17. 跳过需要人工确认的重启/服务/沙箱操作。
17. 检测到旧版状态迁移时会自动运行。

```bash
openclaw doctor --deep
```

扫描系统服务以查找额外的 Gateway 网关安装（launchd/systemd/schtasks）。

如果你想在写入前查看更改，请先打开配置文件：

```bash
cat ~/.openclaw/openclaw.json
```

## 23. 它的作用（摘要）

- git 安装的可选预检更新（仅交互模式）。
- UI 协议新鲜度检查（当协议 schema 较新时重建 Control UI）。
- 健康检查 + 重启提示。
- Skills 状态摘要（符合条件/缺失/被阻止）。
- 遗留值的配置规范化。
- OpenCode Zen 提供商覆盖警告（`models.providers.opencode`）。
- 遗留磁盘状态迁移（会话/智能体目录/WhatsApp 认证）。
- 状态完整性和权限检查（会话、记录、状态目录）。
- 本地运行时的配置文件权限检查（chmod 600）。
- 模型认证健康：检查 OAuth 过期，可刷新即将过期的 token，并报告认证配置文件冷却/禁用状态。
- 额外工作区目录检测（`~/openclaw`）。
- 启用沙箱隔离时的沙箱镜像修复。
- 遗留服务迁移和额外 Gateway 网关检测。
- Gateway 网关运行时检查（服务已安装但未运行；缓存的 launchd 标签）。
- 渠道状态警告（从运行中的 Gateway 网关探测）。
- Supervisor 配置审计（launchd/systemd/schtasks）及可选修复。
- Gateway 网关运行时最佳实践检查（Node vs Bun，版本管理器路径）。
- Gateway 网关端口冲突诊断（默认 `18789`）。
- 开放私信策略的安全警告。
- 未设置 `gateway.auth.token` 时的 Gateway 网关认证警告（本地模式；提供 token 生成）。
- Linux 上的 systemd linger 检查。
- 源码安装检查（pnpm workspace 不匹配、缺失 UI 资产、缺失 tsx 二进制文件）。
- 写入更新后的配置 + 向导元数据。

## 详细行为和原理

### 0）可选更新（git 安装）

如果这是 git 检出且 doctor 以交互模式运行，它会在运行 doctor 之前提供更新（fetch/rebase/build）。

### 1）配置规范化

如果配置包含遗留值形式（例如没有渠道特定覆盖的 `messages.ackReaction`），doctor 会将它们规范化为当前 schema。

### 2）遗留配置键迁移

当配置包含已弃用的键时，其他命令会拒绝运行并要求你运行 `openclaw doctor`。

Doctor 将：

- 解释找到了哪些遗留键。
- 显示它应用的迁移。
- 使用更新后的 schema 重写 `~/.openclaw/openclaw.json`。

Gateway 网关在检测到遗留配置格式时也会在启动时自动运行 doctor 迁移，因此过时的配置无需手动干预即可修复。

当前迁移：

- `routing.allowFrom` → `channels.whatsapp.allowFrom`
- `routing.groupChat.requireMention` → `channels.whatsapp/telegram/imessage.groups."*".requireMention`
- `routing.groupChat.historyLimit` → `messages.groupChat.historyLimit`
- `routing.groupChat.mentionPatterns` → `messages.groupChat.mentionPatterns`
- `routing.queue` → `messages.queue`
- `routing.bindings` → 顶级 `bindings`
- `routing.agents`/`routing.defaultAgentId` → `agents.list` + `agents.list[].default`
- `routing.agentToAgent` → `tools.agentToAgent`
- `routing.transcribeAudio` → `tools.media.audio.models`
- `bindings[].match.accountID` → `bindings[].match.accountId`
- `identity` → `agents.list[].identity`
- `agent.*` → `agents.defaults` + `tools.*`（tools/elevated/exec/sandbox/subagents）
- `agent.model`/`allowedModels`/`modelAliases`/`modelFallbacks`/`imageModelFallbacks`
  → `agents.defaults.models` + `agents.defaults.model.primary/fallbacks` + `agents.defaults.imageModel.primary/fallbacks`

### 2b）OpenCode Zen 提供商覆盖

如果你手动添加了 `models.providers.opencode`（或 `opencode-zen`），它会覆盖 `@mariozechner/pi-ai` 中内置的 OpenCode Zen 目录。这可能会强制将每个模型放到单个 API 上或将成本归零。Doctor 会发出警告，以便你可以移除覆盖并恢复每模型 API 路由 + 成本。 25. 这可能会
强制所有模型使用单一 API，或将成本清零。 26. Doctor 会发出警告，以便你
移除该覆盖并恢复按模型划分的 API 路由和成本。

### 3）遗留状态迁移（磁盘布局）

Doctor 可以将旧的磁盘布局迁移到当前结构：

- 会话存储 + 记录：
  - 从 `~/.openclaw/sessions/` 到 `~/.openclaw/agents/<agentId>/sessions/`
- 31. Agent 目录：
  - 从 `~/.openclaw/agent/` 到 `~/.openclaw/agents/<agentId>/agent/`
- WhatsApp 认证状态（Baileys）：
  - 从遗留的 `~/.openclaw/credentials/*.json`（除 `oauth.json` 外）
  - 到 `~/.openclaw/credentials/whatsapp/<accountId>/...`（默认账户 id：`default`）

36. 这些迁移是尽力而为且幂等的；当它将任何旧版文件夹作为备份保留下来时，doctor 会发出警告。 37. Gateway/CLI 也会在启动时自动迁移
    旧版会话和 agent 目录，使历史记录/认证/模型落在
    按 agent 划分的路径中，而无需手动运行 doctor。 38. WhatsApp 认证有意只通过
    `openclaw doctor` 进行迁移。

### 4）状态完整性检查（会话持久化、路由和安全）

40. 状态目录是运行时的中枢神经。 41. 如果它消失，你将丢失
    会话、凭据、日志和配置（除非你在其他地方有备份）。

Doctor 检查：

- **状态目录缺失**：警告灾难性状态丢失，提示重新创建目录，并提醒你它无法恢复丢失的数据。
- **状态目录权限**：验证可写性；提供修复权限（并在检测到所有者/组不匹配时发出 `chown` 提示）。
- **会话目录缺失**：`sessions/` 和会话存储目录是持久化历史和避免 `ENOENT` 崩溃所必需的。
- **记录不匹配**：当最近的会话条目缺少记录文件时发出警告。
- **主会话"1 行 JSONL"**：当主记录只有一行时标记（历史未累积）。
- **多个状态目录**：当多个 `~/.openclaw` 文件夹存在于不同 home 目录或当 `OPENCLAW_STATE_DIR` 指向别处时发出警告（历史可能在安装之间分裂）。
- **远程模式提醒**：如果 `gateway.mode=remote`，doctor 会提醒你在远程主机上运行它（状态在那里）。
- **配置文件权限**：当 `~/.openclaw/openclaw.json` 对组/其他用户可读时发出警告，并提供收紧到 `600` 的选项。

### 5）模型认证健康（OAuth 过期）

2. Doctor 会检查鉴权存储中的 OAuth 配置文件，在令牌即将过期或已过期时发出警告，并在安全时可刷新它们。 3. 如果 Anthropic Claude Code 配置文件已过期，它会建议运行 `claude setup-token`（或粘贴一个 setup-token）。
3. 刷新提示仅在交互式运行（TTY）时出现；`--non-interactive` 会跳过刷新尝试。

Doctor 还会报告由于以下原因暂时不可用的认证配置文件：

- 短冷却（速率限制/超时/认证失败）
- 长禁用（账单/信用失败）

### 6）Hooks 模型验证

如果设置了 `hooks.gmail.model`，doctor 会根据目录和允许列表验证模型引用，并在无法解析或不允许时发出警告。

### 7）沙箱镜像修复

当启用沙箱隔离时，doctor 检查 Docker 镜像，并在当前镜像缺失时提供构建或切换到遗留名称的选项。

### 8）Gateway 网关服务迁移和清理提示

Doctor 检测遗留的 Gateway 网关服务（launchd/systemd/schtasks），并提供删除它们并使用当前 Gateway 网关端口安装 OpenClaw 服务的选项。它还可以扫描额外的类 Gateway 网关服务并打印清理提示。
配置文件命名的 OpenClaw Gateway 网关服务被视为一等公民，不会被标记为"额外的"。 14. 它还可以扫描额外的类网关服务并输出清理提示。
15. 以配置文件命名的 OpenClaw 网关服务被视为一等公民，不会被标记为“额外”。

### 9）安全警告

当提供商对私信开放而没有允许列表，或当策略以危险方式配置时，Doctor 会发出警告。

### 10）systemd linger（Linux）

如果作为 systemd 用户服务运行，doctor 确保启用 lingering，以便 Gateway 网关在注销后保持活动。

### 11）Skills 状态

Doctor 打印当前工作区符合条件/缺失/被阻止的 Skills 的快速摘要。

### 12）Gateway 网关认证检查（本地 token）

当本地 Gateway 网关缺少 `gateway.auth` 时，Doctor 会发出警告并提供生成 token 的选项。使用 `openclaw doctor --generate-gateway-token` 在自动化中强制创建 token。 24. 使用 `openclaw doctor --generate-gateway-token` 可在自动化中强制创建令牌。

### 13）Gateway 网关健康检查 + 重启

Doctor 运行健康检查，并在 Gateway 网关看起来不健康时提供重启选项。

### 14）渠道状态警告

如果 Gateway 网关健康，doctor 运行渠道状态探测并报告警告及建议的修复。

### 15）Supervisor 配置审计 + 修复

Doctor 检查已安装的 supervisor 配置（launchd/systemd/schtasks）是否有缺失或过时的默认值（例如 systemd network-online 依赖和重启延迟）。当发现不匹配时，它会推荐更新，并可以将服务文件/任务重写为当前默认值。 31. 发现不匹配时，它会推荐更新，并可将服务文件/任务重写为当前默认值。

注意事项：

- `openclaw doctor` 在重写 supervisor 配置前提示。
- `openclaw doctor --yes` 接受默认修复提示。
- `openclaw doctor --repair` 无需提示应用推荐的修复。
- `openclaw doctor --repair --force` 覆盖自定义 supervisor 配置。
- 你始终可以通过 `openclaw gateway install --force` 强制完全重写。

### 16）Gateway 网关运行时 + 端口诊断

39. Doctor 会检查服务运行时（PID、上次退出状态），并在服务已安装但实际上未运行时发出警告。 40. 它还会检查网关端口（默认 `18789`）的端口冲突，并报告可能原因（网关已在运行、SSH 隧道）。

### 17）Gateway 网关运行时最佳实践

42. 当网关服务运行在 Bun 或版本管理的 Node 路径（`nvm`、`fnm`、`volta`、`asdf` 等）上时，Doctor 会发出警告。 43. WhatsApp + Telegram 通道需要 Node，而版本管理器路径可能在升级后失效，因为服务不会加载你的 shell 初始化。 44. 在可用时（Homebrew/apt/choco），Doctor 会提供迁移到系统 Node 安装的选项。

### 18）配置写入 + 向导元数据

Doctor 持久化任何配置更改，并标记向导元数据以记录 doctor 运行。

### 19）工作区提示（备份 + 记忆系统）

当缺失时，Doctor 建议使用工作区记忆系统，并在工作区尚未在 git 下时打印备份提示。

参见 [/concepts/agent-workspace](/concepts/agent-workspace) 了解工作区结构和 git 备份的完整指南（推荐私有 GitHub 或 GitLab）。
