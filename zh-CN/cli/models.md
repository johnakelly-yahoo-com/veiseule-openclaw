---
summary: "`openclaw models` 的 CLI 参考（status/list/set/scan、别名、回退、认证）"
read_when:
  - 你想更改默认模型或查看提供商认证状态
  - 你想扫描可用的模型/提供商并调试认证配置
title: "models"
---

# `openclaw models`

模型发现、扫描和配置（默认模型、回退、认证配置）。

相关内容：

- 提供商 + 模型：[模型](/providers/models)
- 提供商认证设置：[快速开始](/start/getting-started)

## 常用命令

```bash
openclaw models status
openclaw models list
openclaw models set <model-or-alias>
openclaw models scan
```

`openclaw models status` shows the resolved default/fallbacks plus an auth overview.
When provider usage snapshots are available, the OAuth/token status section includes
provider usage headers.
Add `--probe` to run live auth probes against each configured provider profile.
4. 探针是真实请求（可能消耗令牌并触发速率限制）。
Use `--agent <id>` to inspect a configured agent’s model/auth state. When omitted,
the command uses `OPENCLAW_AGENT_DIR`/`PI_CODING_AGENT_DIR` if set, otherwise the
configured default agent.

注意事项：

- `models set <model-or-alias>` 接受 `provider/model` 或别名。
- Model refs are parsed by splitting on the **first** `/`. 模型引用通过在**第一个** `/` 处拆分来解析。如果模型 ID 包含 `/`（OpenRouter 风格），需包含提供商前缀（示例：`openrouter/moonshotai/kimi-k2`）。
- 如果省略提供商，OpenClaw 会将输入视为别名或**默认提供商**的模型（仅在模型 ID 不包含 `/` 时有效）。

### `models status`

选项：

- `--json`
- `--plain`
- `--check`（退出码 1=已过期/缺失，2=即将过期）
- `--probe`（对已配置的认证配置进行实时探测）
- `--probe-provider <name>`（探测单个提供商）
- `--probe-profile <id>`（可重复或逗号分隔的配置 ID）
- `--probe-timeout <ms>`
- `--probe-concurrency <n>`
- `--probe-max-tokens <n>`
- `--agent <id>`（已配置的智能体 ID；覆盖 `OPENCLAW_AGENT_DIR`/`PI_CODING_AGENT_DIR`）

## Aliases + fallbacks

```bash
openclaw models aliases list
openclaw models fallbacks list
```

## Auth profiles

```bash
openclaw models auth add
openclaw models auth login --provider <id>
openclaw models auth setup-token
openclaw models auth paste-token
```

`models auth login` 运行提供商插件的认证流程（OAuth/API 密钥）。使用
`openclaw plugins list` 查看已安装的提供商。 Use
`openclaw plugins list` to see which providers are installed.

注意事项：

- `setup-token` 会提示输入 setup-token 值（在任意机器上使用 `claude setup-token` 生成）。
- `paste-token` 接受在其他地方或通过自动化生成的令牌字符串。

