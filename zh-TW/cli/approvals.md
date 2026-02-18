---
title: "cli/approvals.md"
---

# `openclaw approvals`

管理 **local host**、**gateway host** 或 **node host** 的 exec 核准。
By default, commands target the local approvals file on disk. Use `--gateway` to target the gateway, or `--node` to target a specific node.

相關：

- Exec 核准：[Exec approvals](/tools/exec-approvals)
- 節點：[Nodes](/nodes)

## 常用指令

```bash
openclaw approvals get
openclaw approvals get --node <id|name|ip>
openclaw approvals get --gateway
```

## 從檔案取代核准

```bash
openclaw approvals set --file ./exec-approvals.json
openclaw approvals set --node <id|name|ip> --file ./exec-approvals.json
openclaw approvals set --gateway --file ./exec-approvals.json
```

## 允許清單輔助工具

```bash
openclaw approvals allowlist add "~/Projects/**/bin/rg"
openclaw approvals allowlist add --agent main --node <id|name|ip> "/usr/bin/uptime"
openclaw approvals allowlist add --agent "*" "/usr/bin/uname"

openclaw approvals allowlist remove "~/Projects/**/bin/rg"
```

## 注意事項

- `--node` 使用與 `openclaw nodes` 相同的解析器（id、名稱、ip 或 id 前綴）。
- `--agent` 預設為 `"*"`，此設定會套用至所有代理程式。
- node host 必須宣告 `system.execApprovals.get/set`（macOS 應用程式或無頭 node host）。
- 每個主機的核准檔案會儲存在 `~/.openclaw/exec-approvals.json`。


