---
summary: "OpenClaw 沙箱隔离的工作原理：模式、作用域、工作区访问和镜像"
title: 沙箱隔离
read_when: "You want a dedicated explanation of sandboxing or need to tune agents.defaults.sandbox."
status: active
---

# 沙箱隔离

OpenClaw can run **tools inside Docker containers** to reduce blast radius.
This is **optional** and controlled by configuration (`agents.defaults.sandbox` or
`agents.list[].sandbox`). If sandboxing is off, tools run on the host.
The Gateway stays on the host; tool execution runs in an isolated sandbox
when enabled.

这不是完美的安全边界，但当模型做出愚蠢行为时，它实质性地限制了文件系统和进程访问。

## 什么会被沙箱隔离

- 工具执行（`exec`、`read`、`write`、`edit`、`apply_patch`、`process` 等）。
- 可选的沙箱浏览器（`agents.defaults.sandbox.browser`）。
  - By default, the sandbox browser auto-starts (ensures CDP is reachable) when the browser tool needs it.
    默认情况下，当浏览器工具需要时，沙箱浏览器会自动启动（确保 CDP 可达）。
    通过 `agents.defaults.sandbox.browser.autoStart` 和 `agents.defaults.sandbox.browser.autoStartTimeoutMs` 配置。
  - `agents.defaults.sandbox.browser.allowHostControl` 允许沙箱会话显式定位主机浏览器。
  - 可选的允许列表限制 `target: "custom"`：`allowedControlUrls`、`allowedControlHosts`、`allowedControlPorts`。

不被沙箱隔离：

- Gateway 网关进程本身。
- 任何明确允许在主机上运行的工具（例如 `tools.elevated`）。
  - **提权 exec 在主机上运行并绕过沙箱隔离。**
  - 如果沙箱隔离关闭，`tools.elevated` 不会改变执行（已经在主机上）。参见[提权模式](/tools/elevated)。 See [Elevated Mode](/tools/elevated).

## 模式

`agents.defaults.sandbox.mode` 控制**何时**使用沙箱隔离：

- `"off"`：不使用沙箱隔离。
- `"non-main"`：仅沙箱隔离**非主**会话（如果你想让普通聊天在主机上运行，这是默认值）。
- `"all"`：每个会话都在沙箱中运行。
  注意：`"non-main"` 基于 `session.mainKey`（默认 `"main"`），而不是智能体 ID。
  群组/频道会话使用它们自己的键，因此它们算作非主会话并将被沙箱隔离。
  Note: `"non-main"` is based on `session.mainKey` (default `"main"`), not agent id.
  Group/channel sessions use their own keys, so they count as non-main and will be sandboxed.

## 作用域

`agents.defaults.sandbox.scope` 控制**创建多少容器**：

- `"session"`（默认）：每个会话一个容器。
- `"agent"`：每个智能体一个容器。
- `"shared"`：所有沙箱会话共享一个容器。

## 工作区访问

`agents.defaults.sandbox.workspaceAccess` 控制**沙箱可以看到什么**：

- `"none"`（默认）：工具看到 `~/.openclaw/sandboxes` 下的沙箱工作区。
- `"ro"`：以只读方式在 `/agent` 挂载智能体工作区（禁用 `write`/`edit`/`apply_patch`）。
- `"rw"`：以读写方式在 `/workspace` 挂载智能体工作区。

Inbound media is copied into the active sandbox workspace (`media/inbound/*`).
Skills note: the `read` tool is sandbox-rooted. 入站媒体被复制到活动沙箱工作区（`media/inbound/*`）。
Skills 注意事项：`read` 工具以沙箱为根。使用 `workspaceAccess: "none"` 时，OpenClaw 将符合条件的 Skills 镜像到沙箱工作区（`.../skills`）以便可以读取。使用 `"rw"` 时，工作区 Skills 可从 `/workspace/skills` 读取。 With `"rw"`, workspace skills are readable from
`/workspace/skills`.

## 自定义绑定挂载

`agents.defaults.sandbox.docker.binds` mounts additional host directories into the container.
Format: `host:container:mode` (e.g., `"/home/user/source:/source:rw"`).

Global and per-agent binds are **merged** (not replaced). Under `scope: "shared"`, per-agent binds are ignored.

`agents.defaults.sandbox.docker.binds` 将额外的主机目录挂载到容器中。
格式：`host:container:mode`（例如 `"/home/user/source:/source:rw"`）。

- 沙箱 exec **不**继承主机 `process.env`。使用 `agents.defaults.sandbox.docker.env`（或自定义镜像）设置 Skills API 密钥。
- 默认情况下，沙箱容器运行时**没有网络**。
  通过 `agents.defaults.sandbox.docker.network` 覆盖。

示例（只读源码 + docker 套接字）：

```json5
{
  agents: {
    defaults: {
      sandbox: {
        docker: {
          binds: ["/home/user/source:/source:ro", "/var/run/docker.sock:/var/run/docker.sock"],
        },
      },
    },
    list: [
      {
        id: "build",
        sandbox: {
          docker: {
            binds: ["/mnt/cache:/cache:rw"],
          },
        },
      },
    ],
  },
}
```

安全注意事项：

- 绑定绕过沙箱文件系统：它们以你设置的任何模式（`:ro` 或 `:rw`）暴露主机路径。
- 敏感挂载（例如 `docker.sock`、密钥、SSH 密钥）应该是 `:ro`，除非绝对必要。
- 如果你只需要对工作区的读取访问，请结合 `workspaceAccess: "ro"`；绑定模式保持独立。
- 参见[沙箱 vs 工具策略 vs 提权](/gateway/sandbox-vs-tool-policy-vs-elevated)了解绑定如何与工具策略和提权 exec 交互。

## 镜像 + 设置

默认镜像：`openclaw-sandbox:bookworm-slim`

构建一次：

```bash
scripts/sandbox-setup.sh
```

10. 注意：默认镜像**不**包含 Node。 注意：默认镜像**不**包含 Node。如果 Skills 需要 Node（或其他运行时），要么构建自定义镜像，要么通过 `sandbox.docker.setupCommand` 安装（需要网络出口 + 可写根 + root 用户）。

沙箱浏览器镜像：

```bash
scripts/sandbox-browser-setup.sh
```

14. 默认情况下，沙箱容器**没有网络**。
15. 可通过 `agents.defaults.sandbox.docker.network` 覆盖。

Docker 安装和容器化 Gateway 网关在此：
[Docker](/install/docker)

## setupCommand（一次性容器设置）

`setupCommand` 在沙箱容器创建后**运行一次**（不是每次运行）。
它通过 `sh -lc` 在容器内执行。
It executes inside the container via `sh -lc`.

路径：

- 全局：`agents.defaults.sandbox.docker.setupCommand`
- 每智能体：`agents.list[].sandbox.docker.setupCommand`

常见陷阱：

- 默认 `docker.network` 是 `"none"`（无出口），因此包安装会失败。
- `readOnlyRoot: true` 阻止写入；设置 `readOnlyRoot: false` 或构建自定义镜像。
- `user` 必须是 root 才能安装包（省略 `user` 或设置 `user: "0:0"`）。
- Sandbox exec does **not** inherit host `process.env`. 28. 使用
  `agents.defaults.sandbox.docker.env`（或自定义镜像）来提供技能的 API 密钥。

## 工具策略 + 逃逸通道

30. 在沙箱规则之前，工具的允许/拒绝策略仍然适用。 If a tool is denied
    globally or per-agent, sandboxing doesn’t bring it back.

32. `tools.elevated` 是一个显式的逃生舱，会在宿主机上运行 `exec`。
    `tools.elevated` 是一个显式的逃逸通道，在主机上运行 `exec`。
    `/exec` 指令仅适用于授权发送者并按会话持久化；要硬禁用 `exec`，使用工具策略拒绝（参见[沙箱 vs 工具策略 vs 提权](/gateway/sandbox-vs-tool-policy-vs-elevated)）。

调试：

- 使用 `openclaw sandbox explain` 检查生效的沙箱模式、工具策略和修复配置键。
- 参见[沙箱 vs 工具策略 vs 提权](/gateway/sandbox-vs-tool-policy-vs-elevated)了解"为什么被阻止？"的心智模型。
  保持锁定。
  37. 保持严格锁定。

## 38. 多代理覆盖

每个智能体可以覆盖沙箱 + 工具：
`agents.list[].sandbox` 和 `agents.list[].tools`（加上 `agents.list[].tools.sandbox.tools` 用于沙箱工具策略）。
参见[多智能体沙箱与工具](/tools/multi-agent-sandbox-tools)了解优先级。
40. 参见 [Multi-Agent Sandbox & Tools](/tools/multi-agent-sandbox-tools) 了解优先级。

## 最小启用示例

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        scope: "session",
        workspaceAccess: "none",
      },
    },
  },
}
```

## 相关文档

- [沙箱配置](/gateway/configuration#agentsdefaults-sandbox)
- [多智能体沙箱与工具](/tools/multi-agent-sandbox-tools)
- [安全](/gateway/security)

