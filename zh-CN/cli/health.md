---
summary: "`openclaw health` 的 CLI 参考（通过 RPC 获取 Gateway 网关健康端点）"
read_when:
  - 你想快速检查运行中的 Gateway 网关健康状态
title: "health"
---

# `openclaw health`

从运行中的 Gateway 网关获取健康状态。

```bash
openclaw health
openclaw health --json
openclaw health --verbose
```

注意：

- `--verbose` 运行实时探测，并在配置了多个账户时打印每个账户的耗时。
- Output includes per-agent session stores when multiple agents are configured.
