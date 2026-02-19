---
summary: "执行审批、允许列表和沙箱逃逸提示"
read_when:
  - 配置执行审批或允许列表
  - 在 macOS 应用中实现执行审批用户体验
  - 审查沙箱逃逸提示及其影响
title: "执行审批"
---

# 执行审批

7. Exec 审批是用于让沙盒化代理在真实主机（`gateway` 或 `node`）上运行命令的 **配套应用 / 节点主机防护栏**。 Think of it like a safety interlock:
   commands are allowed only when policy + allowlist + (optional) user approval all agree.
8. Exec 审批是 **额外** 叠加在工具策略和提升级别门控之上的（除非 elevated 设置为 `full`，此时会跳过审批）。
9. 生效策略取 `tools.exec.*` 与审批默认值中 **更严格** 的那个；如果某个审批字段被省略，则使用 `tools.exec` 的值。

如果配套应用 UI **不可用**，任何需要提示的请求都将由 **ask fallback**（默认：deny）决定。

## 12. 适用范围

执行审批在执行主机上本地强制执行：

- **gateway 主机** → gateway 机器上的 `openclaw` 进程
- **node 主机** → 节点运行器（macOS 配套应用或无头节点主机）

macOS 分工：

- **node 主机服务**通过本地 IPC 将 `system.run` 转发给 **macOS 应用**。
- **macOS 应用**执行审批并在 UI 上下文中执行命令。

## 设置和存储

审批信息存储在执行主机上的本地 JSON 文件中：

`~/.openclaw/exec-approvals.json`

示例结构：

```json
{
  "version": 1,
  "socket": {
    "path": "~/.openclaw/exec-approvals.sock",
    "token": "base64url-token"
  },
  "defaults": {
    "security": "deny",
    "ask": "on-miss",
    "askFallback": "deny",
    "autoAllowSkills": false
  },
  "agents": {
    "main": {
      "security": "allowlist",
      "ask": "on-miss",
      "askFallback": "deny",
      "autoAllowSkills": true,
      "allowlist": [
        {
          "id": "B0C8C0B3-2C2D-4F8A-9A3C-5A4B3C2D1E0F",
          "pattern": "~/Projects/**/bin/rg",
          "lastUsedAt": 1737150000000,
          "lastUsedCommand": "rg -n TODO",
          "lastResolvedPath": "/Users/user/Projects/.../bin/rg"
        }
      ]
    }
  }
}
```

## 24. 策略旋钮

### Security（`exec.security`）

- **deny**：阻止所有主机执行请求。
- **allowlist**：仅允许在允许列表中的命令。
- **full**：允许所有命令（等同于提权模式）。

### Ask（`exec.ask`）

- **off**：从不提示。
- **on-miss**：仅在允许列表未匹配时提示。
- **always**：每次命令都提示。

### Ask fallback（`askFallback`）

如果需要提示但无法访问 UI，fallback 决定：

- **deny**：阻止。
- **allowlist**：仅在允许列表匹配时允许。
- **full**：允许。

## 38. 允许列表（按代理）

39. 允许列表是 **按代理** 的。 40. 如果存在多个代理，请在 macOS 应用中切换你正在编辑的代理。 41. 模式是 **不区分大小写的 glob 匹配**。
40. 模式应解析为 **二进制路径**（仅基名的条目会被忽略）。
41. 旧版 `agents.default` 条目在加载时会迁移到 `agents.main`。

示例：

- `~/Projects/**/bin/bird`
- `~/.local/bin/*`
- `/opt/homebrew/bin/rg`

每个允许列表条目会跟踪：

- **id** 用于 UI 标识的稳定 UUID（可选）
- **last used** 时间戳
- **最近使用的命令**
- **最近解析的路径**

## 自动允许 skill CLI

启用 **Auto-allow skill CLIs** 后，已知 Skills 引用的可执行文件在节点（macOS 节点或无头节点主机）上被视为已列入允许列表。这通过 Gateway RPC 的 `skills.bins` 获取 skill 二进制列表。如果你想要严格的手动允许列表，请禁用此选项。 这通过 Gateway RPC 使用 `skills.bins` 来获取技能二进制列表。 如果你希望严格的手动允许列表，请禁用此项。

## 安全二进制（仅 stdin）

`tools.exec.safeBins` 定义了一小组**仅限标准输入**的二进制文件（例如 `jq`），这些文件可以在允许列表模式下运行，**无需**显式的允许列表条目。安全二进制会拒绝位置文件参数和类路径标记，因此它们只能操作传入的流。
在允许列表模式下，shell 链式命令和重定向不会被自动允许。 安全二进制会拒绝位置参数中的文件参数和类似路径的标记，因此它们只能作用于传入的流。
安全二进制（safe bins）还会强制在执行时将 argv 参数视为**字面文本**（不进行通配符展开
也不进行 `$VARS` 变量展开），适用于仅通过 stdin 传递的片段，因此像 `*` 或 `$HOME/...` 这样的模式
无法被用于偷偷读取文件。
在允许列表模式下，不会自动允许 Shell 链接和重定向。

当每个顶级片段都满足允许列表（包括安全二进制或技能自动允许）时，允许使用 Shell 链接（`&&`、`||`、`;`）。 在允许列表模式下仍然不支持重定向。
当每个顶级段都满足允许列表（包括安全二进制或 skill 自动允许）时，允许 shell 链式命令（`&&`、`||`、`;`）。重定向在允许列表模式下仍不受支持。
命令替换（`$()` / 反引号）在允许列表解析期间会被拒绝，包括在双引号内；如果你需要字面的 `$()` 文本，请使用单引号。

默认安全二进制：`jq`、`grep`、`cut`、`sort`、`uniq`、`head`、`tail`、`tr`、`wc`。

## Control UI 编辑

使用 **Control UI → Nodes → Exec approvals** 卡片来编辑默认值、按智能体的覆盖设置和允许列表。选择一个作用域（Defaults 或某个智能体），调整策略，添加/删除允许列表模式，然后点击 **Save**。UI 会显示每个模式的 **last used** 元数据，以便你保持列表整洁。 选择一个作用域（默认值或某个代理），调整策略，添加/移除允许列表模式，然后点击 **保存**。 UI 会按模式显示 **上次使用** 的元数据，便于你保持列表整洁。

目标选择器可选择 **Gateway**（本地审批）或 **Node**。 节点必须声明 `system.execApprovals.get/set`（macOS 应用或无头节点主机）。
如果某个节点尚未声明 exec 审批，请直接编辑其本地的 `~/.openclaw/exec-approvals.json`。

CLI：`openclaw approvals` 支持 gateway 或 node 编辑（参见 [Approvals CLI](/cli/approvals)）。

## 审批流程

当需要提示时，Gateway 会向操作员客户端广播 `exec.approval.requested`。
当需要提示时，gateway 向操作员客户端广播 `exec.approval.requested`。
Control UI 和 macOS 应用通过 `exec.approval.resolve` 进行处理，然后 gateway 将已批准的请求转发给节点主机。

当需要审批时，exec 工具会立即返回一个审批 id。 当需要审批时，exec 工具会立即返回一个审批 id。使用该 id 来关联后续的系统事件（`Exec finished` / `Exec denied`）。如果在超时前没有收到决定，请求将被视为审批超时，并作为拒绝原因显示。 如果在超时之前没有收到决定，请求将被视为审批超时，并以拒绝原因呈现。

确认对话框包括：

- 命令 + 参数
- cwd
- 代理 id
- 解析后的可执行文件路径
- 主机 + 策略元数据

操作：

- **Allow once** → 立即运行
- **Always allow** → 添加到允许列表 + 运行
- **Deny** → 阻止

## 审批转发到聊天渠道

你可以将执行审批提示转发到任何聊天渠道（包括插件渠道），并使用 `/approve` 进行批准。这使用正常的出站投递管道。 这使用的是正常的出站投递流水线。

配置：

```json5
{
  approvals: {
    exec: {
      enabled: true,
      mode: "session", // "session" | "targets" | "both"
      agentFilter: ["main"],
      sessionFilter: ["discord"], // substring or regex
      targets: [
        { channel: "slack", to: "U12345678" },
        { channel: "telegram", to: "123456789" },
      ],
    },
  },
}
```

在聊天中回复：

```
/approve <id> allow-once
/approve <id> allow-always
/approve <id> deny
```

### macOS IPC 流程

```
Gateway -> Node Service (WS)
                 |  IPC (UDS + token + HMAC + TTL)
                 v
             Mac App (UI + approvals + system.run)
```

安全注意事项：

- Unix socket 模式 `0600`，token 存储在 `exec-approvals.json` 中。
- 同 UID 对端检查。
- 挑战/响应（nonce + HMAC token + 请求哈希）+ 短 TTL。

## 系统事件

执行生命周期以系统消息的形式呈现：

- `Exec running`（仅当命令超过运行通知阈值时）
- `Exec finished`
- `Exec denied`

These are posted to the agent’s session after the node reports the event.
Gateway-host exec approvals emit the same lifecycle events when the command finishes (and optionally when running longer than the threshold).
Approval-gated execs reuse the approval id as the `runId` in these messages for easy correlation.

## Implications

- **full** 权限很大；尽可能优先使用允许列表。
- **ask** 让你保持知情，同时仍允许快速审批。
- Per-agent allowlists prevent one agent’s approvals from leaking into others.
- 审批仅适用于来自**授权发送者**的主机执行请求。未授权的发送者无法发出 `/exec`。 Unauthorized senders cannot issue `/exec`.
- `/exec security=full` is a session-level convenience for authorized operators and skips approvals by design.
  To hard-block host exec, set approvals security to `deny` or deny the `exec` tool via tool policy.

相关内容：

- [Exec 工具](/tools/exec)
- [提权模式](/tools/elevated)
- [技能](/tools/skills)
