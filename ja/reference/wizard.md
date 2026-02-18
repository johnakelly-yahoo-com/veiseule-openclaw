---
title: "オンボーディング ウィザード リファレンス"
sidebarTitle: "ウィザード リファレンス"
---

# オンボーディング ウィザード リファレンス

これは `openclaw onboard` CLI ウィザードの完全リファレンスです。  
概要については、[Onboarding Wizard](/start/wizard) を参照してください。

## フロー詳細（ローカルモード）

<Steps>
  <Step title="Existing config detection">
    - `~/.openclaw/openclaw.json` が存在する場合は、**Keep / Modify / Reset** を選択します。
    - **Reset** を明示的に選択しない限り（または `--reset` を渡さない限り）、ウィザードを再実行しても何も消去されません。
    - 設定が無効、またはレガシーキーが含まれている場合、ウィザードは停止し、続行前に `openclaw doctor` を実行するよう求めます。
    - リセットは `rm` ではなく `trash` を使用し、以下のスコープを提供します:
      - Config のみ
      - Config + credentials + sessions
      - フルリセット（workspace も削除）
  </Step>
  <Step title="Model/Auth">
    - **Anthropic API key (recommended)**: `ANTHROPIC_API_KEY` が存在すれば使用し、なければキー入力を求め、デーモン用に保存します。
    - **Anthropic OAuth (Claude Code CLI)**: macOS ではキーチェーン項目「Claude Code-credentials」を確認します（「Always Allow」を選択すると launchd 起動時にブロックされません）。Linux/Windows では `~/.claude/.credentials.json` が存在すれば再利用します。
    - **Anthropic token (paste setup-token)**: 任意のマシンで `claude setup-token` を実行し、トークンを貼り付けます（名前を付けることも可能。空白の場合は default）。
    - **OpenAI Code (Codex) subscription (Codex CLI)**: `~/.codex/auth.json` が存在する場合、再利用できます。
    - **OpenAI Code (Codex) subscription (OAuth)**: ブラウザフローで `code#state` を貼り付けます。
      - モデルが未設定、または `openai/*` の場合、`agents.defaults.model` を `openai-codex/gpt-5.2` に設定します。
    - **OpenAI API key**: `OPENAI_API_KEY` が存在すれば使用し、なければ入力を求め、`~/.openclaw/.env` に保存します（launchd から読み取り可能にするため）。
    - **xAI (Grok) API key**: `XAI_API_KEY` の入力を求め、xAI をモデルプロバイダとして設定します。
    - **OpenCode Zen (multi-model proxy)**: `OPENCODE_API_KEY`（または `OPENCODE_ZEN_API_KEY`、https://opencode.ai/auth で取得）を求めます。
    - **API key**: API キーを保存します。
    - **Vercel AI Gateway (multi-model proxy)**: `AI_GATEWAY_API_KEY` を求めます。
    - 詳細: [Vercel AI Gateway](/providers/vercel-ai-gateway)
    - **Cloudflare AI Gateway**: Account ID、Gateway ID、`CLOUDFLARE_AI_GATEWAY_API_KEY` を求めます。
    - 詳細: [Cloudflare AI Gateway](/providers/cloudflare-ai-gateway)
    - **MiniMax M2.1**: 設定は自動的に書き込まれます。
    - 詳細: [MiniMax](/providers/minimax)
    - **Synthetic (Anthropic-compatible)**: `SYNTHETIC_API_KEY` を求めます。
    - 詳細: [Synthetic](/providers/synthetic)
    - **Moonshot (Kimi K2)**: 設定は自動的に書き込まれます。
    - **Kimi Coding**: 設定は自動的に書き込まれます。
    - 詳細: [Moonshot AI (Kimi + Kimi Coding)](/providers/moonshot)
    - **Skip**: まだ認証は設定しません。
    - 検出されたオプションからデフォルトモデルを選択します（または provider/model を手動入力）。
    - ウィザードはモデルチェックを実行し、設定モデルが不明または認証不足の場合に警告します。
    - OAuth 資格情報は `~/.openclaw/credentials/oauth.json` に保存されます。認証プロファイルは `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`（API キー + OAuth）に保存されます。
    - 詳細: [/concepts/oauth](/concepts/oauth)
    <Note>
    ヘッドレス / サーバー向けのヒント: ブラウザのあるマシンで OAuth を完了し、  
    `~/.openclaw/credentials/oauth.json`（または `$OPENCLAW_STATE_DIR/credentials/oauth.json`）を  
    Gateway ホストへコピーしてください。
    </Note>
  </Step>
  <Step title="Workspace">
    - デフォルトは `~/.openclaw/workspace`（変更可能）。
    - エージェントのブートストラップ儀式に必要なワークスペースファイルをシードします。
    - 完全なワークスペース構成とバックアップガイド: [Agent workspace](/concepts/agent-workspace)
  </Step>
  <Step title="Gateway">
    - ポート、bind、認証モード、Tailscale 公開を設定します。
    - 認証の推奨: ループバックのみでも **Token** を維持し、ローカル WS クライアントにも認証を必須にします。
    - すべてのローカルプロセスを完全に信頼できる場合のみ、認証を無効化してください。
    - ループバック以外の bind でも認証は必須です。
  </Step>
  <Step title="Channels">
    - [WhatsApp](/channels/whatsapp): オプションの QR ログイン。
    - [Telegram](/channels/telegram): bot token。
    - [Discord](/channels/discord): bot token。
    - [Google Chat](/channels/googlechat): サービスアカウント JSON + webhook audience。
    - [Mattermost](/channels/mattermost) (plugin): bot token + base URL。
    - [Signal](/channels/signal): オプションの `signal-cli` インストール + アカウント設定。
    - [BlueBubbles](/channels/bluebubbles): **iMessage に推奨**; server URL + password + webhook。
    - [iMessage](/channels/imessage): レガシー `imsg` CLI パス + DB アクセス。
    - DM セキュリティ: デフォルトはペアリングです。最初の DM でコードが送信されます。`openclaw pairing approve <channel> <code>` で承認するか、allowlist を使用します。
  </Step>
  <Step title="Daemon install">
    - macOS: LaunchAgent
      - ログイン済みユーザーセッションが必要です。ヘッドレス環境ではカスタム LaunchDaemon（同梱されていません）を使用してください。
    - Linux（および Windows + WSL2）: systemd user unit
      - ウィザードは `loginctl enable-linger <user>` を有効化し、ログアウト後も Gateway が起動し続けるよう試みます。
      - sudo を求められる場合があります（`/var/lib/systemd/linger` に書き込み）。まず sudo なしで試行します。
    - **Runtime selection:** Node（推奨; WhatsApp/Telegram に必須）。Bun は **非推奨**。
  </Step>
  <Step title="Health check">
    - 必要に応じて Gateway を起動し、`openclaw health` を実行します。
    - ヒント: `openclaw status --deep` はステータス出力に gateway ヘルスプローブを追加します（到達可能な gateway が必要）。
  </Step>
  <Step title="Skills (recommended)">
    - 利用可能なスキルを読み取り、要件を確認します。
    - node manager を選択: **npm / pnpm**（bun は非推奨）。
    - 任意の依存関係をインストールします（一部は macOS で Homebrew を使用）。
  </Step>
  <Step title="Finish">
    - サマリーと次のステップ（追加機能のための iOS/Android/macOS アプリを含む）。
  </Step>
</Steps>

<Note>
GUI が検出されない場合、ウィザードはブラウザを開く代わりに Control UI 用の SSH ポートフォワード手順を表示します。  
Control UI アセットが不足している場合、ウィザードはビルドを試みます。フォールバックは `pnpm ui:build`（UI 依存関係は自動インストール）です。
</Note>

## 非対話モード

オンボーディングを自動化・スクリプト化するには `--non-interactive` を使用します:

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice apiKey \
  --anthropic-api-key "$ANTHROPIC_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback \
  --install-daemon \
  --daemon-runtime node \
  --skip-skills
```

機械可読なサマリーを取得するには `--json` を追加します。

<Note>
`--json` は **非対話モードを意味しません**。スクリプトでは `--non-interactive`（および `--workspace`）を使用してください。
</Note>

<AccordionGroup>
  <Accordion title="Gemini example">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice gemini-api-key \
      --gemini-api-key "$GEMINI_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="Z.AI example">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice zai-api-key \
      --zai-api-key "$ZAI_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="Vercel AI Gateway example">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice ai-gateway-api-key \
      --ai-gateway-api-key "$AI_GATEWAY_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="Cloudflare AI Gateway example">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice cloudflare-ai-gateway-api-key \
      --cloudflare-ai-gateway-account-id "your-account-id" \
      --cloudflare-ai-gateway-gateway-id "your-gateway-id" \
      --cloudflare-ai-gateway-api-key "$CLOUDFLARE_AI_GATEWAY_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="Moonshot example">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice moonshot-api-key \
      --moonshot-api-key "$MOONSHOT_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="Synthetic example">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice synthetic-api-key \
      --synthetic-api-key "$SYNTHETIC_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="OpenCode Zen example">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice opencode-zen \
      --opencode-zen-api-key "$OPENCODE_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
</AccordionGroup>

### エージェントの追加（非対話）

```bash
openclaw agents add work \
  --workspace ~/.openclaw/workspace-work \
  --model openai/gpt-5.2 \
  --bind whatsapp:biz \
  --non-interactive \
  --json
```

## Gateway ウィザード RPC

Gateway は RPC（`wizard.start`、`wizard.next`、`wizard.cancel`、`wizard.status`）を通じてウィザードフローを公開します。  
クライアント（macOS アプリ、Control UI）はオンボーディングロジックを再実装せずにステップをレンダリングできます。

## Signal セットアップ（signal-cli）

ウィザードは GitHub リリースから `signal-cli` をインストールできます:

- 適切なリリースアセットをダウンロードします。
- `~/.openclaw/tools/signal-cli/<version>/` 配下に保存します。
- 設定に `channels.signal.cliPath` を書き込みます。

注記:

- JVM ビルドには **Java 21** が必要です。
- 利用可能な場合はネイティブビルドを使用します。
- Windows は WSL2 を使用し、signal-cli のインストールは WSL 内で Linux フローに従います。

## ウィザードが書き込む内容

`~/.openclaw/openclaw.json` に含まれる代表的なフィールド:

- `agents.defaults.workspace`
- `agents.defaults.model` / `models.providers`（Minimax を選択した場合）
- `gateway.*`（mode、bind、auth、tailscale）
- `channels.telegram.botToken`、`channels.discord.token`、`channels.signal.*`、`channels.imessage.*`
- プロンプト中にオプトインした場合のチャンネル allowlist（Slack/Discord/Matrix/Microsoft Teams）。可能な場合、名前は ID に解決されます。
- `skills.install.nodeManager`
- `wizard.lastRunAt`
- `wizard.lastRunVersion`
- `wizard.lastRunCommit`
- `wizard.lastRunCommand`
- `wizard.lastRunMode`

`openclaw agents add` は `agents.list[]` と、任意で `bindings` を書き込みます。

WhatsApp の認証情報は `~/.openclaw/credentials/whatsapp/<accountId>/` 配下に保存されます。  
セッションは `~/.openclaw/agents/<agentId>/sessions/` 配下に保存されます。

一部のチャンネルはプラグインとして提供されます。オンボーディング中に選択すると、設定前にインストール（npm またはローカルパス）が求められます。

## 関連ドキュメント

- ウィザード概要: [Onboarding Wizard](/start/wizard)
- macOS アプリのオンボーディング: [Onboarding](/start/onboarding)
- 設定リファレンス: [Gateway configuration](/gateway/configuration)
- プロバイダー: [WhatsApp](/channels/whatsapp)、[Telegram](/channels/telegram)、[Discord](/channels/discord)、[Google Chat](/channels/googlechat)、[Signal](/channels/signal)、[BlueBubbles](/channels/bluebubbles)（iMessage）、[iMessage](/channels/imessage)（レガシー）
- Skills: [Skills](/tools/skills)、[Skills config](/tools/skills-config)
