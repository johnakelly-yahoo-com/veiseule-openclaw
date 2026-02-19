---
summary: "`openclaw tui` 的 CLI 参考（连接到 Gateway 网关的终端 UI）"
read_when:
  - 你想要一个连接 Gateway 网关的终端 UI（支持远程）
  - 你想从脚本传递 url/token/session
title: "tui"
---

# `openclaw tui`

打开连接到 Gateway 网关的终端 UI。

相关：

- TUI 指南：[TUI](/web/tui)

## 示例

```bash
openclaw tui
openclaw tui --url ws://127.0.0.1:18789 --token <token>
openclaw tui --session main --deliver
```
