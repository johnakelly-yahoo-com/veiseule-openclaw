---
title: "RPC 介接器"
---

# RPC 介接器

OpenClaw 透過 JSON-RPC 整合外部 CLI。目前使用兩種模式。 Two patterns are used today.

## 模式 A：HTTP 常駐程式（signal-cli）

- `signal-cli` 以常駐程式形式執行，透過 HTTP 提供 JSON-RPC。
- 事件串流為 SSE（`/api/v1/events`）。
- 健康檢查：`/api/v1/check`。
- 當 `channels.signal.autoStart=true` 時，OpenClaw 會負責生命週期。

設定與端點請參閱 [Signal](/channels/signal)。

## 模式 B：stdio 子行程（legacy：imsg）

> **注意事項：** 新的 iMessage 設定請改用 [BlueBubbles](/channels/bluebubbles)。

- OpenClaw 會將 `imsg rpc` 作為子行程啟動（legacy iMessage 整合）。
- JSON-RPC 透過 stdin/stdout 以逐行方式傳輸（每行一個 JSON 物件）。
- 無需 TCP 連接埠，也不需要常駐程式。

使用的核心方法：

- `watch.subscribe` → 通知（`method: "message"`）
- `watch.unsubscribe`
- `send`
- `chats.list`（探測／診斷）

legacy 設定與位址指定請參閱 [iMessage](/channels/imessage)（偏好 `chat_id`）。

## 介接器指引

- Gateway 負責該進程（啟動/停止與提供者生命週期綁定）。
- 保持 RPC 用戶端具備韌性：設定逾時，於退出時重新啟動。
- 優先使用穩定的 ID（例如 `chat_id`），避免使用顯示字串。
