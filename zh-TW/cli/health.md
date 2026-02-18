---
title: "health"
---

# `openclaw health`

從正在執行的 Gateway 閘道器 擷取健康狀態。

```bash
openclaw health
openclaw health --json
openclaw health --verbose
```

注意事項：

- `--verbose` 會執行即時探測，並在設定多個帳號時列印各帳號的耗時。
- 當設定多個代理時，輸出內容會包含各代理的會話儲存資料。


