---
title: "提升模式"
---

# 提升模式（/elevated 指令）

## 功能說明

- `/elevated on` 在 Gateway 閘道器主機上執行，並保留 exec 核准（與 `/elevated ask` 相同）。
- `/elevated full` 在 Gateway 閘道器主機上執行 **且** 自動核准 exec（略過 exec 核准）。
- `/elevated ask` 在 Gateway 閘道器主機上執行，但保留 exec 核准（與 `/elevated on` 相同）。
- `on`/`ask` **不會** 強制 `exec.security=full`；已設定的安全性／詢問政策仍然適用。
- 僅在代理程式為 **sandboxed** 時才會改變行為（否則 exec 已在主機上執行）。
- 指令形式：`/elevated on|off|ask|full`、`/elev on|off|ask|full`。
- 僅接受 `on|off|ask|full`；任何其他值都會回傳提示，且不會變更狀態。

## 控制範圍（以及不包含的部分）

- **可用性閘門**：`tools.elevated` 為全域基準。`agents.list[].tools.elevated` 可針對個別代理進一步限制 elevated（兩者皆須允許）。
- **每個工作階段狀態**：`/elevated on|off|ask|full` 為目前的工作階段金鑰設定 elevated 等級。
- **行內指令**：訊息內的 `/elevated on|ask|full` 僅套用於該則訊息。
- **群組**：在群組聊天中，只有在提及代理程式時才會採用提升指令。可略過提及需求的純指令訊息會被視為已提及。 Command-only messages that bypass mention requirements are treated as mentioned.
- **主機執行**：提升會將 `exec` 強制到 Gateway 閘道器主機；`full` 也會設定 `security=full`。
- **核准**：`full` 會略過 exec 核准；`on`/`ask` 在允許清單／詢問規則要求時會遵循核准。
- **未沙箱化的代理**：對位置無影響；僅影響閘門控制、記錄與狀態。
- **工具政策仍然適用**：若 `exec` 被工具政策拒絕，則無法使用提升模式。
- **與 `/exec` 分離**：`/exec` 會為已授權的寄件者調整每個工作階段的預設值，且不需要提升模式。

## 解析順序

1. 訊息中的內嵌指令（僅套用於該則訊息）。
2. 工作階段覆寫（透過傳送僅含指令的訊息來設定）。
3. 全域預設（設定中的 `agents.defaults.elevatedDefault`）。

## 設定工作階段預設值

- 傳送一則 **僅** 包含指令的訊息（允許空白），例如 `/elevated full`。
- 會傳送確認回覆（`Elevated mode set to full...`／`Elevated mode disabled.`）。
- 若已停用 elevated 存取，或傳送者不在核准的允許清單中，該指令會回覆可操作的錯誤訊息，且不會變更工作階段狀態。
- 傳送 `/elevated`（或 `/elevated:`）且不帶參數，可查看目前的提升層級。

## 可用性＋允許清單

- 功能閘門：`tools.elevated.enabled`（即使程式碼支援，預設也可透過設定關閉）。
- 寄件者允許清單：`tools.elevated.allowFrom`，並提供各提供者的允許清單（例如 `discord`、`whatsapp`）。
- 每個代理程式的閘門：`agents.list[].tools.elevated.enabled`（選用；只能進一步限制）。
- 每個代理程式的允許清單：`agents.list[].tools.elevated.allowFrom`（選用；設定後，寄件者必須同時符合 **全域＋每個代理程式** 的允許清單）。
- Discord 後備：若省略 `tools.elevated.allowFrom.discord`，會使用 `channels.discord.dm.allowFrom` 清單作為後備。設定 `tools.elevated.allowFrom.discord`（即使是 `[]`）即可覆寫。每個代理程式的允許清單 **不會** 使用後備。 Set `tools.elevated.allowFrom.discord` (even `[]`) to override. Per-agent allowlists do **not** use the fallback.
- 必須通過所有閘門；否則提升模式會被視為不可用。

## 記錄＋狀態

- 提升的 exec 呼叫會以 info 等級記錄。
- 1. 工作階段狀態包含提升模式（例如 `elevated=ask`、`elevated=full`）。
