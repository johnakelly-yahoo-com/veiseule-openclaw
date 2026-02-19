---
summary: "Skills：托管与工作区、门控规则以及配置/环境变量连接"
read_when:
  - 添加或修改 Skills
  - 更改 Skills 门控或加载规则
title: "Skills"
---

# Skills（OpenClaw）

OpenClaw 使用**兼容 [AgentSkills](https://agentskills.io)** 的 Skills 文件夹来教智能体如何使用工具。每个 Skills 是一个包含带有 YAML frontmatter 和说明的 `SKILL.md` 的目录。OpenClaw 加载**内置 Skills** 以及可选的本地覆盖，并在加载时根据环境、配置和二进制文件存在情况进行过滤。 50. 每个技能都是一个目录，包含带有 YAML 前置元数据和说明的 `SKILL.md`。 1. OpenClaw 会加载 **内置技能** 以及可选的本地覆盖，并在加载时根据环境、配置和二进制文件的存在情况进行过滤。

## 位置和优先级

Skills 从**三个**位置加载：

1. **内置 Skills**：随安装包一起发布（npm 包或 OpenClaw.app）
2. **托管/本地 Skills**：`~/.openclaw/skills`
3. **工作区 Skills**：`<workspace>/skills`

如果 Skills 名称冲突，优先级为：

`<workspace>/skills`（最高）→ `~/.openclaw/skills` → 内置 Skills（最低）

此外，你可以通过 `~/.openclaw/openclaw.json` 中的 `skills.load.extraDirs` 配置额外的 Skills 文件夹（最低优先级）。

## 单智能体 vs 共享 Skills

11. 在 **多代理** 设置中，每个代理都有自己的工作区。 12. 这意味着：

- **单智能体 Skills** 位于 `<workspace>/skills` 中，仅供该智能体使用。
- **共享 Skills** 位于 `~/.openclaw/skills`（托管/本地），对同一机器上的**所有智能体**可见。
- 如果你想要多个智能体使用一个通用的 Skills 包，也可以通过 `skills.load.extraDirs`（最低优先级）添加**共享文件夹**。

如果同一个 Skills 名称存在于多个位置，将应用通常的优先级规则：工作区优先，然后是托管/本地，最后是内置。

## 插件 + Skills

插件可以通过在 `openclaw.plugin.json` 中列出 `skills` 目录（相对于插件根目录的路径）来发布自己的 Skills。插件 Skills 在插件启用时加载，并参与正常的 Skills 优先级规则。你可以通过插件配置条目上的 `metadata.openclaw.requires.config` 对它们进行门控。参见[插件](/tools/plugin)了解发现/配置，以及[工具](/tools)了解这些 Skills 所教授的工具接口。 19. 当插件启用时，插件技能会被加载，并参与正常的技能优先级规则。
20. 你可以通过插件配置项中的 `metadata.openclaw.requires.config` 来对其进行门控。 21. 有关发现/配置请参见 [Plugins](/tools/plugin)，有关这些技能所教授的工具界面请参见 [Tools](/tools)。

## ClawHub（安装 + 同步）

ClawHub 是 OpenClaw 的公共 Skills 注册表。浏览 https://clawhub.com。使用它来发现、安装、更新和备份 Skills。完整指南：[ClawHub](/tools/clawhub)。 24. 浏览地址：
[https://clawhub.com](https://clawhub.com)。 25. 使用它来发现、安装、更新和备份技能。
26. 完整指南：[ClawHub](/tools/clawhub)。

常见流程：

- 将 Skills 安装到你的工作区：
  - `clawhub install <skill-slug>`
- 更新所有已安装的 Skills：
  - `clawhub update --all`
- 同步（扫描 + 发布更新）：
  - `clawhub sync --all`

默认情况下，`clawhub` 安装到当前工作目录下的 `./skills`（或回退到配置的 OpenClaw 工作区）。OpenClaw 在下一个会话中将其识别为 `<workspace>/skills`。 35. OpenClaw 会在下一次会话中
将其识别为 `<workspace>/skills`。

## 安全注意事项

- 将第三方 Skills 视为**不受信任的代码**。启用前请阅读它们。 38. 在启用之前先阅读它们。
- 对于不受信任的输入和高风险工具，优先使用沙箱隔离运行。参见[沙箱隔离](/gateway/sandboxing)。 40. 参见 [Sandboxing](/gateway/sandboxing)。
- `skills.entries.*.env` 和 `skills.entries.*.apiKey` 为该智能体轮次将秘密注入到**宿主机**进程中（而非沙箱）。将秘密保持在提示词和日志之外。 Keep secrets out of prompts and logs.
- 有关更广泛的威胁模型和检查清单，参见[安全性](/gateway/security)。

## 格式（AgentSkills + Pi 兼容）

`SKILL.md` 必须至少包含：

```markdown
---
name: nano-banana-pro
description: Generate or edit images via Gemini 3 Pro Image
---
```

注意事项：

- 我们遵循 AgentSkills 规范的布局/意图。
- 内嵌智能体使用的解析器仅支持**单行** frontmatter 键。
- `metadata` 应该是**单行 JSON 对象**。
- 在说明中使用 `{baseDir}` 来引用 Skills 文件夹路径。
- 可选的 frontmatter 键：
  - `homepage` — 在 macOS Skills UI 中显示为"Website"的 URL（也支持通过 `metadata.openclaw.homepage`）。
  - `user-invocable` — `true|false`（默认：`true`）。当为 `true` 时，Skills 作为用户斜杠命令暴露。 When `true`, the skill is exposed as a user slash command.
  - `disable-model-invocation` — `true|false`（默认：`false`）。当为 `true` 时，Skills 从模型提示词中排除（仍可通过用户调用使用）。 When `true`, the skill is excluded from the model prompt (still available via user invocation).
  - `command-dispatch` — `tool` (optional). `command-dispatch` — `tool`（可选）。当设置为 `tool` 时，斜杠命令绕过模型直接调度到工具。
  - `command-tool` — 当设置 `command-dispatch: tool` 时要调用的工具名称。
  - `command-arg-mode` — `raw`（默认）。对于工具调度，将原始参数字符串转发到工具（无核心解析）。 For tool dispatch, forwards the raw args string to the tool (no core parsing).

    工具使用以下参数调用：
    `{ command: "<raw args>", commandName: "<slash command>", skillName: "<skill name>" }`。

## 门控（加载时过滤）

OpenClaw 使用 `metadata`（单行 JSON）**在加载时过滤 Skills**：

```markdown
---
name: nano-banana-pro
description: Generate or edit images via Gemini 3 Pro Image
metadata:
  {
    "openclaw":
      {
        "requires": { "bins": ["uv"], "env": ["GEMINI_API_KEY"], "config": ["browser.enabled"] },
        "primaryEnv": "GEMINI_API_KEY",
      },
  }
---
```

`metadata.openclaw` 下的字段：

- `always: true` — 始终包含该 Skills（跳过其他门控）。
- `emoji` — macOS Skills UI 使用的可选表情符号。
- `homepage` — 在 macOS Skills UI 中显示为"Website"的可选 URL。
- `os` — 可选的平台列表（`darwin`、`linux`、`win32`）。如果设置，该 Skills 仅在这些操作系统上有资格。 If set, the skill is only eligible on those OSes.
- `requires.bins` — 列表；每个都必须存在于 `PATH` 中。
- `requires.anyBins` — 列表；至少一个必须存在于 `PATH` 中。
- `requires.env` — 列表；环境变量必须存在**或**在配置中提供。
- `requires.config` — `openclaw.json` 路径列表，必须为真值。
- `primaryEnv` — 与 `skills.entries.<name>.apiKey` 关联的环境变量名称。
- `install` — macOS Skills UI 使用的可选安装器规格数组（brew/node/go/uv/download）。

沙箱隔离注意事项：

- `requires.bins` 在 Skills 加载时在**宿主机**上检查。
- If an agent is sandboxed, the binary must also exist **inside the container**.
  Install it via `agents.defaults.sandbox.docker.setupCommand` (or a custom image).
  `setupCommand` runs once after the container is created.
  Package installs also require network egress, a writable root FS, and a root user in the sandbox.
  Example: the `summarize` skill (`skills/summarize/SKILL.md`) needs the `summarize` CLI
  in the sandbox container to run there.

安装器示例：

```markdown
---
name: gemini
description: Use Gemini CLI for coding assistance and Google search lookups.
metadata:
  {
    "openclaw":
      {
        "emoji": "♊️",
        "requires": { "bins": ["gemini"] },
        "install":
          [
            {
              "id": "brew",
              "kind": "brew",
              "formula": "gemini-cli",
              "bins": ["gemini"],
              "label": "Install Gemini CLI (brew)",
            },
          ],
      },
  }
---
```

注意事项：

- 如果列出了多个安装器，Gateway 网关会选择**单个**首选选项（可用时选择 brew，否则选择 node）。
- 如果所有安装器都是 `download`，OpenClaw 会列出每个条目，以便你查看可用的构件。
- 安装器规格可以包含 `os: ["darwin"|"linux"|"win32"]` 按平台过滤选项。
- Node 安装遵循 `openclaw.json` 中的 `skills.install.nodeManager`（默认：npm；选项：npm/pnpm/yarn/bun）。这仅影响 **Skills 安装**；Gateway 网关运行时应仍为 Node（不推荐 Bun 用于 WhatsApp/Telegram）。
  This only affects **skill installs**; the Gateway runtime should still be Node
  (Bun is not recommended for WhatsApp/Telegram).
- Go 安装：如果缺少 `go` 且 `brew` 可用，Gateway 网关会首先通过 Homebrew 安装 Go，并在可能时将 `GOBIN` 设置为 Homebrew 的 `bin`。
- Download 安装：`url`（必填）、`archive`（`tar.gz` | `tar.bz2` | `zip`）、`extract`（默认：检测到归档时自动）、`stripComponents`、`targetDir`（默认：`~/.openclaw/tools/<skillKey>`）。

如果没有 `metadata.openclaw`，该 Skills 始终有资格（除非在配置中禁用或被 `skills.allowBundled` 阻止用于内置 Skills）。

## 配置覆盖（`~/.openclaw/openclaw.json`）

内置/托管 Skills 可以被切换并提供环境变量值：

```json5
{
  skills: {
    entries: {
      "nano-banana-pro": {
        enabled: true,
        apiKey: "GEMINI_KEY_HERE",
        env: {
          GEMINI_API_KEY: "GEMINI_KEY_HERE",
        },
        config: {
          endpoint: "https://example.invalid",
          model: "nano-pro",
        },
      },
      peekaboo: { enabled: true },
      sag: { enabled: false },
    },
  },
}
```

注意：如果 Skills 名称包含连字符，请用引号括起键名（JSON5 允许带引号的键名）。

2. 默认情况下，配置键与 **技能名称** 匹配。 配置键默认匹配 **Skills 名称**。如果 Skills 定义了 `metadata.openclaw.skillKey`，请在 `skills.entries` 下使用该键。

规则：

- `enabled: false` 禁用该 Skills，即使它是内置/已安装的。
- `env`：**仅在**变量在进程中尚未设置时注入。
- `apiKey`：为声明 `metadata.openclaw.primaryEnv` 的 Skills 提供的便捷字段。
- `config`：用于自定义单 Skills 字段的可选容器；自定义键必须放在这里。
- `allowBundled`：可选的仅用于**内置** Skills 的白名单。如果设置，只有列表中的内置 Skills 才有资格（托管/工作区 Skills 不受影响）。 10. 如果设置了该项，则只有列表中的
  捆绑技能才有资格（不影响 managed/workspace 技能）。

## 11. 环境变量注入（每次 agent 运行）

当智能体运行开始时，OpenClaw：

1. 读取 Skills 元数据。
2. 将任何 `skills.entries.<key>.env` 或 `skills.entries.<key>.apiKey` 应用到 `process.env`。
3. 使用**有资格的** Skills 构建系统提示词。
4. 在运行结束后恢复原始环境。

这是**限定于智能体运行范围内的**，不是全局 shell 环境。

## 会话快照（性能）

OpenClaw 在**会话开始时**对有资格的 Skills 进行快照，并在同一会话的后续轮次中重用该列表。对 Skills 或配置的更改在下一个新会话中生效。 22. 对技能或配置的更改会在下一个新会话中生效。

23. 当启用 skills watcher 或出现新的符合条件的远程节点时，技能也可以在会话中途刷新（见下文）。 24. 可将其视为一种**热重载**：刷新后的列表会在下一次 agent 轮次中生效。

## 远程 macOS 节点（Linux Gateway 网关）

如果 Gateway 网关运行在 Linux 上但连接了一个**允许 `system.run` 的 macOS 节点**（Exec 批准安全设置未设为 `deny`），当所需的二进制文件存在于该节点上时，OpenClaw 可以将仅限 macOS 的 Skills 视为有资格。智能体应通过 `nodes` 工具（通常是 `nodes.run`）执行这些 Skills。 27. agent 应通过 `nodes` 工具（通常是 `nodes.run`）来执行这些技能。

28. 这依赖于节点上报其命令支持情况，以及通过 `system.run` 进行的二进制探测。 29. 如果 macOS 节点随后离线，这些技能仍然可见；在节点重新连接之前，调用可能会失败。

## Skills 监视器（自动刷新）

默认情况下，OpenClaw 监视 Skills 文件夹，并在 `SKILL.md` 文件更改时更新 Skills 快照。在 `skills.load` 下配置： 32. 在 `skills.load` 下进行配置：

```json5
{
  skills: {
    load: {
      watch: true,
      watchDebounceMs: 250,
    },
  },
}
```

## Token 影响（Skills 列表）

当 Skills 有资格时，OpenClaw 将可用 Skills 的紧凑 XML 列表注入到系统提示词中（通过 `pi-coding-agent` 中的 `formatSkillsForPrompt`）。成本是确定性的： 36. 成本是确定的：

- **基础开销（仅当 ≥1 个 Skills 时）：** 195 字符。
- **每个 Skills：** 97 字符 + XML 转义的 `<name>`、`<description>` 和 `<location>` 值的长度。

公式（字符）：

```
total = 195 + Σ (97 + len(name_escaped) + len(description_escaped) + len(location_escaped))
```

注意事项：

- XML 转义将 `& < > " '` 扩展为实体（`&amp;`、`&lt;` 等），增加长度。
- 43. 不同模型的分词器会导致 token 数不同。 Token 数量因模型分词器而异。粗略的 OpenAI 风格估计是 ~4 字符/token，所以**每个 Skills 97 字符 ≈ 24 token** 加上你的实际字段长度。

## 托管 Skills 生命周期

OpenClaw 作为安装的一部分（npm 包或 OpenClaw.app）发布一组基线 Skills 作为**内置 Skills**。`~/.openclaw/skills` 用于本地覆盖（例如，在不更改内置副本的情况下固定/修补 Skills）。工作区 Skills 由用户拥有，在名称冲突时覆盖两者。 47. `~/.openclaw/skills` 用于本地
覆盖（例如，在不更改捆绑副本的情况下固定/修补某个技能）。 48. Workspace 技能由用户拥有，并在名称冲突时覆盖前两者。

## 配置参考

参见 [Skills 配置](/tools/skills-config)了解完整的配置 schema。

## 寻找更多 Skills？

浏览 https://clawhub.com。

---
