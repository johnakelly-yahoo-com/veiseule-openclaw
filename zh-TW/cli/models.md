---
title: "模型"
---

# `openclaw models`

模型探索、掃描與設定（預設模型、備援、驗證設定檔）。

相關：

- 提供者＋模型：[Models](/providers/models)
- 提供者身分驗證設定：[Getting started](/start/getting-started)

## 常用指令

```bash
openclaw models status
openclaw models list
openclaw models set <model-or-alias>
openclaw models scan
```

`openclaw models status` 會顯示解析後的預設/備援設定，以及驗證總覽。
When provider usage snapshots are available, the OAuth/token status section includes
provider usage headers.
Add `--probe` to run live auth probes against each configured provider profile.
Probes are real requests (may consume tokens and trigger rate limits).
Use `--agent <id>` to inspect a configured agent’s model/auth state. When omitted,
the command uses `OPENCLAW_AGENT_DIR`/`PI_CODING_AGENT_DIR` if set, otherwise the
configured default agent.

注意事項：

- `models set <model-or-alias>` 接受 `provider/model` 或別名。
- Model refs are parsed by splitting on the **first** `/`. 模型參照是以**第一個** `/` 進行分割。若模型 ID 包含 `/`（OpenRouter 風格），請包含提供者前綴（例如：`openrouter/moonshotai/kimi-k2`）。
- 若你省略提供者，OpenClaw 會將輸入視為別名或**預設提供者**的模型（僅在模型 ID 中沒有 `/` 時適用）。

### `models status`

選項：

- `--json`
- `--plain`
- `--check`（結束碼 1＝過期／缺失，2＝即將過期）
- `--probe`（對已設定的身分驗證設定檔進行即時探測）
- `--probe-provider <name>`（探測單一提供者）
- `--probe-profile <id>`（重複或以逗號分隔的設定檔 ID）
- `--probe-timeout <ms>`
- `--probe-concurrency <n>`
- `--probe-max-tokens <n>`
- `--agent <id>`（已設定的代理程式 ID；會覆寫 `OPENCLAW_AGENT_DIR`/`PI_CODING_AGENT_DIR`）

## 別名 + 備援

```bash
openclaw models aliases list
openclaw models fallbacks list
```

## 驗證設定檔

```bash
openclaw models auth add
openclaw models auth login --provider <id>
openclaw models auth setup-token
openclaw models auth paste-token
```

`models auth login` 會執行提供者外掛的身分驗證流程（OAuth／API 金鑰）。使用
`openclaw plugins list` 以查看已安裝的提供者。 Use
`openclaw plugins list` to see which providers are installed.

注意事項：

- `setup-token` 會提示輸入 setup-token 值（可在任何機器上使用 `claude setup-token` 產生）。
- `paste-token` 接受在其他地方或由自動化產生的權杖字串。
