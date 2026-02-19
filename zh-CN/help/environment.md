---
summary: "OpenClaw 从哪里加载环境变量以及优先级顺序"
read_when:
  - You need to know which env vars are loaded, and in what order
  - 你在调试 Gateway 网关中缺失的 API 密钥
  - 你在编写提供商认证或部署环境的文档
title: "环境变量"
---

# 环境变量

OpenClaw 从多个来源拉取环境变量。规则是**永不覆盖现有值**。 The rule is **never override existing values**.

## 优先级（从高到低）

1. **进程环境**（Gateway 网关进程从父 shell/守护进程已有的内容）。
2. **当前工作目录中的 `.env`**（dotenv 默认；不覆盖）。
3. **全局 `.env`** 位于 `~/.openclaw/.env`（即 `$OPENCLAW_STATE_DIR/.env`；不覆盖）。
4. **配置 `env` 块** 位于 `~/.openclaw/openclaw.json`（仅在缺失时应用）。
5. **可选的登录 shell 导入**（`env.shellEnv.enabled` 或 `OPENCLAW_LOAD_SHELL_ENV=1`），仅对缺失的预期键名应用。

如果配置文件完全缺失，步骤 4 将被跳过；如果启用了 shell 导入，它仍会运行。

## 配置 `env` 块

Two equivalent ways to set inline env vars (both are non-overriding):

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: {
      GROQ_API_KEY: "gsk-...",
    },
  },
}
```

## Shell 环境导入

`env.shellEnv` 运行你的登录 shell 并仅导入**缺失的**预期键名：

```json5
{
  env: {
    shellEnv: {
      enabled: true,
      timeoutMs: 15000,
    },
  },
}
```

Env var equivalents:

- `OPENCLAW_LOAD_SHELL_ENV=1`
- `OPENCLAW_SHELL_ENV_TIMEOUT_MS=15000`

## 配置中的环境变量替换

你可以使用 `${VAR_NAME}` 语法在配置字符串值中直接引用环境变量：

```json5
{
  models: {
    providers: {
      "vercel-gateway": {
        apiKey: "${VERCEL_GATEWAY_API_KEY}",
      },
    },
  },
}
```

完整详情参见[配置：环境变量替换](/gateway/configuration#env-var-substitution-in-config)。

## 与路径相关的环境变量

| 变量                     | 用途                                                                                                                         |
| ---------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| `OPENCLAW_HOME`        | 覆盖用于所有内部路径解析的主目录（`~/.openclaw/`、agent 目录、会话、凭据）。 Useful when running OpenClaw as a dedicated service user. |
| `OPENCLAW_STATE_DIR`   | 覆盖状态目录（默认 `~/.openclaw`）。                                                                                                  |
| `OPENCLAW_CONFIG_PATH` | 覆盖配置文件路径（默认 `~/.openclaw/openclaw.json`）。                                                                                  |

### `OPENCLAW_HOME`

设置后，`OPENCLAW_HOME` 将替换系统主目录（`$HOME` / `os.homedir()`），用于所有内部路径解析。 这使无头服务账户实现完整的文件系统隔离。

**优先级：** `OPENCLAW_HOME` > `$HOME` > `USERPROFILE` > `os.homedir()`

**示例**（macOS LaunchDaemon）：

```xml
<key>EnvironmentVariables</key>
<dict>
  <key>OPENCLAW_HOME</key>
  <string>/Users/kira</string>
</dict>
```

`OPENCLAW_HOME` 也可以设置为波浪号路径（例如 `~/svc`），在使用前会基于 `$HOME` 进行展开。

## 相关内容

- [Gateway 网关配置](/gateway/configuration)
- [常见问题：环境变量和 .env 加载](/help/faq#env-vars-and-env-loading)
- [模型概述](/concepts/models)

