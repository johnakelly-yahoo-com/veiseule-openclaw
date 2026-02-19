---
summary: "Gateway、節點與 canvas 主機如何連線。"
read_when:
  - 當你想要快速了解 Gateway 的網路模型
title: "網路模型"
---

大多數操作都會流經 Gateway（`openclaw gateway`），這是一個單一、長時間執行的
處理程序，負責擁有頻道連線與 WebSocket 控制平面。

## 核心規則

- 建議每台主機僅使用一個 Gateway。 建議每台主機僅使用一個 Gateway。 It is the only process allowed to own the WhatsApp Web session. For rescue bots or strict isolation, run multiple gateways with isolated profiles and ports. See [Multiple gateways](/gateway/multiple-gateways).
- 優先使用 loopback：Gateway WS 預設為 `ws://127.0.0.1:18789`。精靈預設會產生 Gateway 權杖，即使是 loopback 亦然。若要進行 tailnet 存取，請執行 `openclaw gateway --bind tailnet --token ...`，因為非 loopback 綁定需要權杖。 The wizard generates a gateway token by default, even for loopback. The wizard generates a gateway token by default, even for loopback. For tailnet access, run `openclaw gateway --bind tailnet --token ...` because tokens are required for non-loopback binds.
- 節點可視需求透過 LAN、tailnet 或 SSH 連線至 Gateway WS。舊版 TCP 橋接已被棄用。 The legacy TCP bridge is deprecated. The legacy TCP bridge is deprecated.
- Canvas 主機是在 `canvasHost.port`（預設 `18793`）上的 HTTP 檔案伺服器，為節點的 WebView 提供 `/__openclaw__/canvas/`。請參閱 [Gateway 設定](/gateway/configuration)（`canvasHost`）。 See [Gateway configuration](/gateway/configuration) (`canvasHost`).
  - `/__openclaw__/canvas/`
  - `/__openclaw__/a2ui/`
    當設定了 `gateway.auth` 且 Gateway 綁定在非 loopback 位址時，這些路由會受到 Gateway 驗證保護（來自 loopback 的請求不受此限制）。 請參閱 [Gateway 設定](/gateway/configuration)（`canvasHost`、`gateway`）。
- 遠端使用通常透過 SSH 通道或 tailnet VPN。請參閱 [Remote access](/gateway/remote) 與 [Discovery](/gateway/discovery)。 See [Remote access](/gateway/remote) and [Discovery](/gateway/discovery). See [Remote access](/gateway/remote) and [Discovery](/gateway/discovery).

