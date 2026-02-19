---
summary: "Gateway 服務的操作手冊，涵蓋生命週期與營運"
read_when:
  - 正在執行或除錯 Gateway 程序時
title: "Gateway 操作手冊"
---

# Gateway 服務操作手冊

本頁用於 Gateway 服務的第 1 天啟動與第 2 天營運。

<CardGroup cols={2}>
  <Card title="Deep troubleshooting" icon="siren" href="/gateway/troubleshooting">    以症狀為優先的診斷流程，提供精確的指令階梯與日誌特徵。
</Card>
  <Card title="Configuration" icon="sliders" href="/gateway/configuration">    以任務為導向的設定指南 + 完整組態參考。
</Card>
</CardGroup>

## 5 分鐘本機啟動

<Steps>
  <Step title="Start the Gateway">

```bash
openclaw gateway --port 18789
# for full debug/trace logs in stdio:
openclaw gateway --port 18789 --verbose
# if the port is busy, terminate listeners then start:
openclaw gateway --force
# dev loop (auto-reload on TS changes):
pnpm gateway:watch
```

  
</Step>

  <Step title="Verify service health">

```bash
openclaw gateway status
openclaw status
openclaw logs --follow
```

健康基準：`Runtime: running` 與 `RPC probe: ok`。

  
</Step>

  <Step title="Validate channel readiness">

```bash
openclaw channels status --probe
```

  
</Step>
</Steps>

<Note>
Gateway 組態重新載入會監看目前啟用的組態檔路徑（從 profile/state 預設值解析，或在設定 `OPENCLAW_CONFIG_PATH` 時使用該值）。
預設模式：`gateway.reload.mode="hybrid"`（安全變更即時套用，關鍵變更則重啟）。
</Note>

## 執行階段模型

- 單一常駐程序，用於路由、控制平面與通道連線。
- 單一多工連接埠，用於：
  - WebSocket 控制/RPC
  - OpenResponses（HTTP）：[`/v1/responses`](/gateway/openresponses-http-api)。
  - 控制 UI 與 hooks
- 預設綁定模式：`loopback`。
- 預設需要 Gateway 驗證：設定 `gateway.auth.token`（或 `OPENCLAW_GATEWAY_TOKEN`）或 `gateway.auth.password`。除非使用 Tailscale Serve 身分，否則客戶端必須送出 `connect.params.auth.token/password`。 17.

### 連接埠與綁定優先順序

| 設定          | 解析順序                                                          |
| ----------- | ------------------------------------------------------------- |
| Gateway 連接埠 | `--port` → `OPENCLAW_GATEWAY_PORT` → `gateway.port` → `18789` |
| 綁定模式        | CLI/override → `gateway.bind` → `loopback`                    |

### 熱重載模式

| 使用 `gateway.reload.mode="off"` 停用。 | 行為               |
| ---------------------------------- | ---------------- |
| `off`                              | 不重新載入組態          |
| `hot`                              | 僅套用可安全熱更新的變更     |
| `restart`                          | 在需要重新啟動的變更時重新啟動  |
| `hybrid`（預設）                       | 安全時進行熱套用，需要時重新啟動 |

## 操作員指令集

```bash
openclaw gateway status
openclaw gateway status --deep
openclaw gateway status --json
openclaw gateway install
openclaw gateway restart
openclaw gateway stop
openclaw logs --follow
openclaw doctor
```

## 49. 遠端存取

首選：Tailscale/VPN。
備用方案：SSH tunnel。

```bash
ssh -N -L 18789:127.0.0.1:18789 user@host
```

然後在本機將用戶端連線至 `ws://127.0.0.1:18789`。

<Warning>
如果已設定 gateway 驗證，即使透過 SSH tunnel，用戶端仍必須傳送驗證資訊（`token`/`password`）。
</Warning>

參見：[Remote Gateway](/gateway/remote)、[Authentication](/gateway/authentication)、[Tailscale](/gateway/tailscale)。

## 監控與服務生命週期

為了達到類生產環境的可靠性，請使用受監管（supervised）的執行方式。

<Tabs>
  <Tab title="macOS (launchd)">

```bash
`openclaw gateway stop|restart` — 停止／重啟受監督的 Gateway 服務（launchd/systemd）。
```

LaunchAgent 標籤為 `ai.openclaw.gateway`（預設）或 `ai.openclaw.<profile>`（具名設定檔）。 `openclaw doctor` 會稽核並修復服務設定漂移問題。

  
</Tab>

  <Tab title="Linux (systemd user)">

```bash
systemctl --user enable --now openclaw-gateway[-<profile>].service
```

啟用 lingering（必要，讓使用者服務在登出／閒置後仍存活）：

```bash
sudo loginctl enable-linger youruser
```

  
</Tab>

  <Tab title="Linux (system service)">

對於多使用者／常駐主機，請使用 system unit。

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now openclaw-gateway[-<profile>].service
```

  
</Tab>
</Tabs>

## 多個 Gateway（同一主機）

大多數設定應只執行**一個** Gateway。
僅在需要嚴格隔離／備援時才使用多個（例如救援設定檔）。

每個實例的檢查清單：

- 唯一的 `gateway.port`
- 唯一的 `OPENCLAW_CONFIG_PATH`
- 唯一的 `OPENCLAW_STATE_DIR`
- 唯一的 `agents.defaults.workspace`

範例：

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json OPENCLAW_STATE_DIR=~/.openclaw-a openclaw gateway --port 19001
OPENCLAW_CONFIG_PATH=~/.openclaw/b.json OPENCLAW_STATE_DIR=~/.openclaw-b openclaw gateway --port 19002
```

完整指南：[Multiple gateways](/gateway/multiple-gateways)。

### Dev 設定檔快速路徑

```bash
openclaw gateway status
openclaw gateway install
openclaw gateway stop
openclaw gateway restart
openclaw logs --follow
```

預設包含隔離的 state/config，以及基礎 gateway 連接埠 `19001`。

## 協定快速參考（操作人員視角）

- 第一個用戶端 frame 必須是 `connect`。
- Gateway 會回傳 `hello-ok` 快照（`presence`、`health`、`stateVersion`、`uptimeMs`、limits/policy）。
- 請求：`{type:"req", id, method, params}` → `{type:"res", id, ok, payload|error}`
- 常見事件：`connect.challenge`、`agent`、`chat`、`presence`、`tick`、`health`、`heartbeat`、`shutdown`。

Agent 執行分為兩個階段：

1. 立即回傳 accepted ack（`status:"accepted"`）
2. `agent` 回應為兩階段：先 `res` ack `{runId,status:"accepted"}`，完成後再送出最終的 `res` `{runId,status:"ok"|"error",summary}`；串流輸出以 `event:"agent"` 抵達。

完整文件：[Gateway protocol](/gateway/protocol) 與 [Bridge protocol（舊版）](/gateway/bridge-protocol)。

## Operational checks

### 存活性

- 開啟 WS 並傳送 `connect`。
- 預期收到包含快照的 `hello-ok` 回應。

### 就緒狀態

```bash
`openclaw gateway health|status` — 透過 Gateway WS 請求健康／狀態。
```

### 間隙復原

事件不會被重播。 32. 當出現序列間隙時，請在繼續之前重新整理狀態（`health`、`system-presence`）。

## 常見失敗特徵

| 特徵                                                             | 可能問題                                     |
| -------------------------------------------------------------- | ---------------------------------------- |
| `refusing to bind gateway ... without auth`                    | 在未提供 token/password 的情況下綁定至非 loopback 位址 |
| `another gateway instance is already listening` / `EADDRINUSE` | 連接埠衝突                                    |
| `Gateway start blocked: set gateway.mode=local`                | 設定為 remote 模式                            |
| 在 connect 過程中出現 `unauthorized`                                 | 用戶端與 gateway 之間的驗證不一致                    |

完整的診斷流程請參閱 [Gateway Troubleshooting](/gateway/troubleshooting)。

## 安全保證

- 當 Gateway 無法使用時，Gateway 協定用戶端會快速失敗（不會隱式回退到直接通道）。
- 無效／非 connect 的首個 frame 會被拒絕並關閉連線。
- 在 socket 關閉前，會先發送 `shutdown` 事件以進行優雅關閉。

---

相關：

- [Troubleshooting](/gateway/troubleshooting)
- [Background Process](/gateway/background-process)
- [設定](/gateway/configuration)
- [健康狀態](/gateway/health)
- [Doctor](/gateway/doctor)
- [驗證](/gateway/authentication)
