---
summary: "OpenClaw 系统提示词包含的内容及其组装方式"
read_when:
  - 编辑系统提示词文本、工具列表或时间/心跳部分
  - 更改工作区引导或 Skills 注入行为
title: "系统提示词"
---

# 系统提示词

OpenClaw builds a custom system prompt for every agent run. OpenClaw 为每次智能体运行构建自定义系统提示词。该提示词由 **OpenClaw 拥有**，不使用 pi-coding-agent 默认提示词。

The prompt is assembled by OpenClaw and injected into each agent run.

## 结构

该提示词设计紧凑，使用固定部分：

- **Tooling**：当前工具列表 + 简短描述。
- **Safety**：简短的防护提醒，避免追求权力的行为或绕过监督。
- **Skills**（如果可用）：告诉模型如何按需加载 Skill 指令。
- **OpenClaw Self-Update**：如何运行 `config.apply` 和 `update.run`。
- **Workspace**：工作目录（`agents.defaults.workspace`）。
- **Documentation**：OpenClaw 文档的本地路径（仓库或 npm 包）以及何时阅读它们。
- **Workspace Files (injected)**：表示下方包含引导文件。
- **Sandbox**（启用时）：表示沙箱隔离运行时、沙箱路径，以及是否可用提权执行。
- **Current Date & Time**：用户本地时间、时区和时间格式。
- **Reply Tags**：支持的提供商的可选回复标签语法。
- **Heartbeats**：心跳提示词和确认行为。
- **Runtime**：主机、操作系统、node、模型、仓库根目录（检测到时）、思考级别（一行）。
- **Reasoning**：当前可见性级别 + /reasoning 切换提示。

49. 系统提示中的安全防护栏是建议性的。 They guide model behavior but do not enforce policy. Use tool policy, exec approvals, sandboxing, and channel allowlists for hard enforcement; operators can disable these by design.

## 提示词模式

OpenClaw can render smaller system prompts for sub-agents. The runtime sets a
`promptMode` for each run (not a user-facing config):

- `full`（默认）：包含上述所有部分。
- `minimal`：用于子智能体；省略 **Skills**、**Memory Recall**、**OpenClaw Self-Update**、**Model Aliases**、**User Identity**、**Reply Tags**、**Messaging**、**Silent Replies** 和 **Heartbeats**。Tooling、**Safety**、Workspace、Sandbox、Current Date & Time（已知时）、Runtime 和注入的上下文仍然可用。 Tooling, **Safety**,
  Workspace, Sandbox, Current Date & Time (when known), Runtime, and injected
  context stay available.
- `none`：仅返回基本身份行。

当 `promptMode=minimal` 时，额外注入的提示词标记为 **Subagent Context** 而不是 **Group Chat Context**。

## 工作区引导注入

引导文件被修剪后附加在 **Project Context** 下，这样模型无需显式读取即可看到身份和配置上下文：

- `AGENTS.md`
- `SOUL.md`
- `TOOLS.md`
- `IDENTITY.md`
- `USER.md`
- `HEARTBEAT.md`
- `BOOTSTRAP.md`（仅在全新工作区上）
- `MEMORY.md` 和/或 `memory.md`（当工作区中存在时；可以注入其中一个或两个）

所有这些文件都会在每一轮**注入到上下文窗口中**，这意味着它们会消耗 tokens。 请保持其简洁——尤其是 `MEMORY.md`，它
可能会随着时间增长，导致上下文使用量意外升高以及更频繁的
压缩。

> **注意：** `memory/*.md` 每日文件**不会**自动注入。 它们
> 会通过 `memory_search` 和 `memory_get` 工具按需访问，因此
> 除非模型显式读取，否则不会计入上下文窗口。

Large files are truncated with a marker. 大文件会带截断标记被截断。每个文件的最大大小由 `agents.defaults.bootstrapMaxChars` 控制（默认：20000）。缺失的文件会注入一个简短的缺失文件标记。 跨文件注入的 bootstrap
内容总量由 `agents.defaults.bootstrapTotalMaxChars`
（默认：24000）限制。 缺失的文件会注入一个简短的缺失文件标记。

子代理会话仅注入 `AGENTS.md` 和 `TOOLS.md`（其他 bootstrap 文件
会被过滤，以保持子代理上下文较小）。

内部钩子可以通过 `agent:bootstrap` 拦截此步骤以修改或替换注入的引导文件（例如将 `SOUL.md` 替换为其他角色）。

要检查每个注入文件贡献了多少（原始 vs 注入、截断，加上工具 schema 开销），使用 `/context list` 或 `/context detail`。参见[上下文](/concepts/context)。 See [Context](/concepts/context).

## 时间处理

当用户时区已知时，系统提示词包含专用的 **Current Date & Time** 部分。为保持提示词缓存稳定，它现在只包含**时区**（没有动态时钟或时间格式）。 To keep the prompt cache-stable, it now only includes
the **time zone** (no dynamic clock or time format).

当智能体需要当前时间时使用 `session_status`；状态卡片包含时间戳行。

配置方式：

- `agents.defaults.userTimezone`
- `agents.defaults.timeFormat`（`auto` | `12` | `24`）

完整行为详情参见[日期和时间](/date-time)。

## Skills

当存在符合条件的 Skills 时，OpenClaw 注入一个紧凑的**可用 Skills 列表**（`formatSkillsForPrompt`），其中包含每个 Skill 的**文件路径**。提示词指示模型使用 `read` 加载列出位置（工作区、托管或内置）的 SKILL.md。如果没有符合条件的 Skills，则省略 Skills 部分。 The
prompt instructs the model to use `read` to load the SKILL.md at the listed
location (workspace, managed, or bundled). If no skills are eligible, the
Skills section is omitted.

```
<available_skills>
  <skill>
    <name>...</name>
    <description>...</description>
    <location>...</location>
  </skill>
</available_skills>
```

这使基础提示词保持小巧，同时仍然支持有针对性的 Skill 使用。

## 文档

如果可用，系统提示词包含一个 **Documentation** 部分，指向本地 OpenClaw 文档目录（仓库工作区中的 `docs/` 或打包的 npm 包文档），并注明公共镜像、源仓库、社区 Discord 和 ClawHub (https://clawhub.com) 用于 Skills 发现。提示词指示模型首先查阅本地文档了解 OpenClaw 行为、命令、配置或架构，并尽可能自己运行 `openclaw status`（仅在无法访问时询问用户）。 The prompt instructs the model to consult local docs first
for OpenClaw behavior, commands, configuration, or architecture, and to run
`openclaw status` itself when possible (asking the user only when it lacks access).
