---
summary: "`openclaw reset`（重置本地状态/配置）的 CLI 参考"
read_when:
  - 你想在保留 CLI 安装的同时清除本地状态
  - You want a dry-run of what would be removed
title: "reset"
---

# `openclaw reset`

重置本地配置/状态（保留 CLI 安装）。

```bash
openclaw reset
openclaw reset --dry-run
openclaw reset --scope config+creds+sessions --yes --non-interactive
```

