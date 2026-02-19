---
summary: "`openclaw logs` 的 CLI 参考（通过 RPC 跟踪 Gateway 网关日志）"
read_when:
  - 你需要远程跟踪 Gateway 网关日志（无需 SSH）
  - 你需要 JSON 日志行用于工具处理
title: "logs"
---

# `openclaw logs`

通过 RPC 跟踪 Gateway 网关文件日志（在远程模式下可用）。

相关内容：

- 日志概述：[日志](/logging)

## 示例

```bash
openclaw logs
openclaw logs --follow
openclaw logs --json
openclaw logs --limit 500
```

使用 `--local-time` 以本地时区显示时间戳。

