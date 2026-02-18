---
title: "CLI 入門引導參考"
sidebarTitle: "CLI 參考"
---

# CLI 入門引導參考

本頁為 `openclaw onboard` 的完整參考文件。  
簡短指南請見 [Onboarding Wizard (CLI)](/start/wizard)。

## 精靈會執行的內容

本機模式（預設）會引導你完成：

- 模型與身分驗證設定（OpenAI Code 訂閱 OAuth、Anthropic API 金鑰或 setup token，另含 MiniMax、GLM、Moonshot 與 AI Gateway 選項）
- 工作區位置與啟動用檔案
- Gateway 設定（連接埠、繫結、驗證、tailscale）
- 頻道與提供者（Telegram、WhatsApp、Discord、Google Chat、Mattermost 外掛、Signal）
- 常駐程式安裝（LaunchAgent 或 systemd 使用者單位）
- 健康檢查
- Skills 設定

遠端模式會設定此機器以連線到其他位置的 Gateway。  
它不會在遠端主機上安裝或修改任何內容。

## Local flow details

<Steps>
  <Step title="Existing config detection">
    - 若 `~/.openclaw/openclaw.json` 已存在，請選擇 Keep、Modify 或 Reset。  
    - 重新執行精靈不會清除任何內容，除非你明確選擇 Reset（或傳入 `--reset`）。  
    - 若設定無效或包含舊版金鑰，精靈會停止並要求你先執行 `openclaw doctor`。  
    - Reset 會使用 `trash`，並提供以下範圍選項：
      - 僅設定
      - 設定 + 憑證 + sessions
      - 完整重設（同時移除 workspace）
  </Step>
  <Step title="Model and auth">
    - 完整選項矩陣請見 [驗證與模型選項](#auth-and-model-options)。
  </Step>
  <Step title="Workspace">
    - 預設為 `~/.openclaw/workspace`（可設定）。  
    - 會建立首次啟動 bootstrap 所需的 workspace 檔案。  
    - 工作區配置請見：[Agent workspace](/concepts/agent-workspace)。
  </Step>
  <Step title="Gateway">
    - 會提示設定連接埠、bind、驗證模式與 tailscale 對外暴露。  
    - 建議：即使是 loopback 也保留 token 驗證，讓本機 WS 用戶端仍需驗證。  
    - 僅在完全信任所有本機程序時才停用驗證。  
    - 非 loopback 綁定仍然需要驗證。
  </Step>
  <Step title="Channels">
    - [WhatsApp](/channels/whatsapp)：選用的 QR 登入  
    - [Telegram](/channels/telegram)：機器人權杖  
    - [Discord](/channels/discord)：機器人權杖  
    - [Google Chat](/channels/googlechat)：服務帳戶 JSON + webhook 受眾  
    - [Mattermost](/channels/mattermost) 外掛：機器人權杖 + 基底 URL  
    - [Signal](/channels/signal)：選用的 `signal-cli` 安裝 + 帳戶設定  
    - [BlueBubbles](/channels/bluebubbles)：建議用於 iMessage；伺服器 URL + 密碼 + webhook  
    - [iMessage](/channels/imessage)：舊版 `imsg` CLI 路徑 + DB 存取  
    - 私訊安全性：預設為配對。第一則私訊會傳送代碼；透過  
      `openclaw pairing approve <channel> <code>` 核准，或使用允許清單。
  </Step>
  <Step title="Daemon install">
    - macOS：LaunchAgent  
      - 需要已登入的使用者工作階段；無頭環境請使用自訂 LaunchDaemon（未隨附）。  
    - Linux 與 Windows（透過 WSL2）：systemd 使用者單位  
      - 精靈會嘗試執行 `loginctl enable-linger <user>`，使 gateway 在登出後仍持續運行。  
      - 可能會要求 sudo（寫入 `/var/lib/systemd/linger`）；會先嘗試不使用 sudo。  
    - 執行環境選擇：Node（建議；WhatsApp 與 Telegram 需要）。不建議使用 Bun。
  </Step>
  <Step title="Health check">
    - 啟動 gateway（如有需要）並執行 `openclaw health`。  
    - `openclaw status --deep` 會在狀態輸出中加入 gateway 健康探測。
  </Step>
  <Step title="Skills">
    - 讀取可用的 skills 並檢查需求。  
    - 可選擇 node manager：npm 或 pnpm（不建議 bun）。  
    - 安裝選用相依套件（部分在 macOS 上使用 Homebrew）。
  </Step>
  <Step title="Finish">
    - 顯示摘要與後續步驟，包括 iOS、Android 與 macOS 應用程式選項。
  </Step>
</Steps>

<Note>
若未偵測到 GUI，精靈會顯示 Control UI 的 SSH 連接埠轉發說明，而不會開啟瀏覽器。  
若缺少 Control UI 資產，精靈會嘗試建置；備援指令為 `pnpm ui:build`（會自動安裝 UI 相依套件）。
</Note>

## Remote mode details

遠端模式會設定此機器以連線到其他位置的 Gateway。

<Info>
遠端模式不會在遠端主機上安裝或修改任何內容。
</Info>

你需要設定：

- 遠端 Gateway URL（`ws://...`）
- 若遠端 Gateway 需要驗證，則設定權杖（建議）

<Note>
- 若 Gateway 僅限 loopback，請使用 SSH 通道或 tailnet。  
- 探索提示：
  - macOS：Bonjour（`dns-sd`）  
  - Linux：Avahi（`avahi-browse`）
</Note>

## Auth and model options

<AccordionGroup>
  <Accordion title="Anthropic API key (recommended)">
    若存在 `ANTHROPIC_API_KEY` 則使用，否則提示輸入金鑰，並儲存供常駐程式使用。
  </Accordion>
  <Accordion title="Anthropic OAuth (Claude Code CLI)">
    - macOS：檢查鑰匙圈項目「Claude Code-credentials」  
    - Linux 與 Windows：若存在則重用 `~/.claude/.credentials.json`  

    在 macOS 上，請選擇「Always Allow」，以避免 launchd 啟動被阻擋。
  </Accordion>
  <Accordion title="Anthropic token (setup-token paste)">
    在任一機器上執行 `claude setup-token`，然後貼上該 token。  
    你可以為其命名；留空則使用預設名稱。
  </Accordion>
  <Accordion title="OpenAI Code subscription (Codex CLI reuse)">
    若存在 `~/.codex/auth.json`，精靈可重用它。
  </Accordion>
  <Accordion title="OpenAI Code subscription (OAuth)">
    瀏覽器流程；貼上 `code#state`。  

    當模型未設定或為 `openai/*` 時，會將 `agents.defaults.model` 設為 `openai-codex/gpt-5.3-codex`。
  </Accordion>
  <Accordion title="OpenAI API key">
    若存在 `OPENAI_API_KEY` 則使用，否則提示輸入金鑰，並將其儲存至  
    `~/.openclaw/.env` 以供 launchd 讀取。  

    當模型未設定、為 `openai/*`，或為 `openai-codex/*` 時，會將 `agents.defaults.model` 設為 `openai/gpt-5.1-codex`。
  </Accordion>
  <Accordion title="xAI (Grok) API key">
    提示輸入 `XAI_API_KEY`，並將 xAI 設定為模型提供者。
  </Accordion>
  <Accordion title="OpenCode Zen">
    提示輸入 `OPENCODE_API_KEY`（或 `OPENCODE_ZEN_API_KEY`）。  
    設定 URL：[opencode.ai/auth](https://opencode.ai/auth)。
  </Accordion>
  <Accordion title="API key (generic)">
    為你儲存金鑰。
  </Accordion>
  <Accordion title="Vercel AI Gateway">
    提示輸入 `AI_GATEWAY_API_KEY`。  
    更多說明：[Vercel AI Gateway](/providers/vercel-ai-gateway)。
  </Accordion>
  <Accordion title="Cloudflare AI Gateway">
    提示輸入帳戶 ID、Gateway ID，以及 `CLOUDFLARE_AI_GATEWAY_API_KEY`。  
    更多說明：[Cloudflare AI Gateway](/providers/cloudflare-ai-gateway)。
  </Accordion>
  <Accordion title="MiniMax M2.1">
    設定會自動寫入。  
    更多說明：[MiniMax](/providers/minimax)。
  </Accordion>
  <Accordion title="Synthetic (Anthropic-compatible)">
    提示輸入 `SYNTHETIC_API_KEY`。  
    更多說明：[Synthetic](/providers/synthetic)。
  </Accordion>
  <Accordion title="Moonshot and Kimi Coding">
    Moonshot（Kimi K2）與 Kimi Coding 的設定會自動寫入。  
    更多說明：[Moonshot AI (Kimi + Kimi Coding)](/providers/moonshot)。
  </Accordion>
  <Accordion title="Custom provider">
    可搭配 OpenAI-compatible 與 Anthropic-compatible 端點使用。  

    非互動式旗標：
    - `--auth-choice custom-api-key`
    - `--custom-base-url`
    - `--custom-model-id`
    - `--custom-api-key`（選用；預設為 `CUSTOM_API_KEY`）
    - `--custom-provider-id`（選用）
    - `--custom-compatibility <openai|anthropic>`（選用；預設為 `openai`）
  </Accordion>
  <Accordion title="Skip">
    保持未設定驗證。
  </Accordion>
</AccordionGroup>

模型行為：

- 從偵測到的選項中選擇預設模型，或手動輸入 provider 與模型。  
- 精靈會執行模型檢查，若模型未知或缺少驗證會顯示警告。

憑證與設定檔路徑：

- OAuth 憑證：`~/.openclaw/credentials/oauth.json`  
- 驗證設定檔（API 金鑰 + OAuth）：`~/.openclaw/agents/<agentId>/agent/auth-profiles.json`

<Note>
無頭與伺服器提示：請在有瀏覽器的機器上完成 OAuth，然後將  
`~/.openclaw/credentials/oauth.json`（或 `$OPENCLAW_STATE_DIR/credentials/oauth.json`）  
複製到 gateway 主機。
</Note>

## Outputs and internals

`~/.openclaw/openclaw.json` 中的常見欄位：

- `agents.defaults.workspace`
- `agents.defaults.model` / `models.providers`（若選擇 Minimax）
- `gateway.*`（mode、bind、auth、tailscale）
- `channels.telegram.botToken`、`channels.discord.token`、`channels.signal.*`、`channels.imessage.*`
- 在提示時選擇加入的頻道允許清單（Slack、Discord、Matrix、Microsoft Teams）（名稱會在可能時解析為 ID）
- `skills.install.nodeManager`
- `wizard.lastRunAt`
- `wizard.lastRunVersion`
- `wizard.lastRunCommit`
- `wizard.lastRunCommand`
- `wizard.lastRunMode`

`openclaw agents add` 會寫入 `agents.list[]` 與選用的 `bindings`。

WhatsApp 憑證位於 `~/.openclaw/credentials/whatsapp/<accountId>/`。  
Sessions 會儲存在 `~/.openclaw/agents/<agentId>/sessions/`。

<Note>
部分頻道以外掛形式提供。於入門設定期間選取時，精靈會在頻道設定前提示安裝外掛（npm 或本機路徑）。
</Note>

Gateway 精靈 RPC：

- `wizard.start`
- `wizard.next`
- `wizard.cancel`
- `wizard.status`

用戶端（macOS 應用程式與 Control UI）可在不重新實作入門引導邏輯的情況下呈現步驟。

Signal 設定行為：

- 下載對應的 release asset  
- 儲存至 `~/.openclaw/tools/signal-cli/<version>/`  
- 在設定中寫入 `channels.signal.cliPath`  
- JVM 版本需要 Java 21  
- 可用時會使用 Native builds  
- Windows 透過 WSL2，並在 WSL 內依循 Linux 的 signal-cli 流程

## Related docs

- 入門引導樞紐：[入門引導精靈（CLI）](/start/wizard)  
- 自動化與腳本：[CLI 自動化](/start/wizard-cli-automation)  
- 指令參考：[`openclaw onboard`](/cli/onboard)

