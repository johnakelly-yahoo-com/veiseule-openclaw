---
summary: "`openclaw plugins` 的 CLI 参考（列出、安装、启用/禁用、诊断）"
read_when:
  - 你想安装或管理进程内 Gateway 网关插件
  - 你想调试插件加载失败问题
title: "plugins"
---

# `openclaw plugins`

管理 Gateway 网关插件/扩展（进程内加载）。

相关内容：

- 插件系统：[插件](/tools/plugin)
- 插件清单 + 模式：[插件清单](/plugins/manifest)
- 安全加固：[安全](/gateway/security)

## 命令

```bash
openclaw plugins list
openclaw plugins info <id>
openclaw plugins enable <id>
openclaw plugins disable <id>
openclaw plugins doctor
openclaw plugins update <id>
openclaw plugins update --all
```

内置插件随 OpenClaw 一起发布，但默认禁用。使用 `plugins enable` 来激活它们。 Use `plugins enable` to
activate them.

所有插件必须提供 `openclaw.plugin.json` 文件，其中包含内联 JSON Schema（`configSchema`，即使为空）。缺少或无效的清单或模式会阻止插件加载并导致配置验证失败。 Missing/invalid manifests or schemas prevent
the plugin from loading and fail config validation.

### 安装

```bash
openclaw plugins install <path-or-spec>
```

安全提示：将插件安装视为运行代码。优先使用固定版本。 Prefer pinned versions.

Npm 规范仅支持 **registry-only**（包名 + 可选版本/标签）。 Git/URL/file
规范将被拒绝。 为确保安全，依赖安装使用 `--ignore-scripts` 运行。

支持的归档格式：`.zip`、`.tgz`、`.tar.gz`、`.tar`。

使用 `--link` 避免复制本地目录（添加到 `plugins.load.paths`）：

```bash
openclaw plugins install -l ./my-plugin
```

### 卸载

```bash
openclaw plugins uninstall <id>
openclaw plugins uninstall <id> --dry-run
openclaw plugins uninstall <id> --keep-files
```

`uninstall` 会从 `plugins.entries`、`plugins.installs`、插件 allowlist，以及相关的 `plugins.load.paths` 条目中移除插件记录（如适用）。
对于活跃的 memory 插件，memory 槽会重置为 `memory-core`。

默认情况下，卸载还会移除位于当前 state 目录 extensions 根路径下的插件安装目录（`$OPENCLAW_STATE_DIR/extensions/<id>`）。 使用 `--keep-files` 以保留磁盘上的文件。

`--keep-config` 作为 `--keep-files` 的已弃用别名仍受支持。

### 更新

```bash
openclaw plugins update <id>
openclaw plugins update --all
openclaw plugins update <id> --dry-run
```

更新仅适用于从 npm 安装的插件（在 `plugins.installs` 中跟踪）。

