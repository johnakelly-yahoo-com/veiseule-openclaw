---
title: "CLI オンボーディング リファレンス"
sidebarTitle: "CLI リファレンス"
---

# CLI オンボーディング リファレンス

このページは `openclaw onboard` の完全なリファレンスです。  
ショートガイドについては、[オンボーディングウィザード (CLI)](/start/wizard)を参照してください。

## ウィザードが行うこと

ローカルモード（デフォルト）では、次の内容を順に案内します。

- モデルおよび認証のセットアップ（OpenAI Code サブスクリプション OAuth、Anthropic API キーまたはセットアップトークン、加えて MiniMax、GLM、Moonshot、AI Gateway の各オプション）
- ワークスペースの場所とブートストラップファイル
- Gateway 設定（ポート、バインド、認証、Tailscale）
- チャンネルとプロバイダー（Telegram、WhatsApp、Discord、Google Chat、Mattermost プラグイン、Signal）
- デーモンのインストール（LaunchAgent または systemd ユーザーユニット）
- ヘルスチェック
- Skills のセットアップ

リモートモードでは、このマシンを別の場所にあるゲートウェイへ接続するよう構成します。  
リモートホストへのインストールや変更は行いません。

## ローカルフローの詳細

<Steps>
  <Step title="Existing config detection">
    - `~/.openclaw/openclaw.json` が存在する場合は、Keep、Modify、Reset を選択します。  
    - ウィザードを再実行しても、明示的に Reset（または `--reset` を指定）しない限り、何も削除されません。  
    - 設定が無効、またはレガシーキーが含まれている場合、ウィザードは停止し、続行前に `openclaw doctor` の実行を求めます。  
    - Reset は `trash` を使用し、次のスコープを選択できます:  
      - 設定のみ  
      - 設定 + 認証情報 + セッション  
      - 完全リセット（ワークスペースも削除）
  
</Step>
  <Step title="Model and auth">
    - 完全な選択肢一覧は [認証とモデルのオプション](#auth-and-model-options) を参照してください。
  
</Step>
  <Step title="Workspace">
    - デフォルトは `~/.openclaw/workspace`（設定可能）です。  
    - 初回実行時のブートストラップに必要なワークスペースファイルを生成します。  
    - ワークスペース構成: [Agent ワークスペース](/concepts/agent-workspace)。
  
</Step>
  <Step title="Gateway">
    - ポート、バインド、認証モード、Tailscale 公開設定を入力します。  
    - 推奨: ループバックのみでもトークン認証を有効にし、ローカル WS クライアントが認証を必要とするようにします。  
    - すべてのローカルプロセスを完全に信頼している場合のみ認証を無効にしてください。  
    - ループバック以外のバインドでも認証は必須です。
  
</Step>
  <Step title="Channels">
    - [WhatsApp](/channels/whatsapp): 任意の QR ログイン  
    - [Telegram](/channels/telegram): ボットトークン  
    - [Discord](/channels/discord): ボットトークン  
    - [Google Chat](/channels/googlechat): サービスアカウント JSON + Webhook オーディエンス  
    - [Mattermost](/channels/mattermost) プラグイン: ボットトークン + ベース URL  
    - [Signal](/channels/signal): 任意の `signal-cli` インストール + アカウント設定  
    - [BlueBubbles](/channels/bluebubbles): iMessage 用に推奨；サーバー URL + パスワード + Webhook  
    - [iMessage](/channels/imessage): レガシー `imsg` CLI パス + DB アクセス  
    - DM セキュリティ: デフォルトはペアリングです。最初の DM でコードが送信されます。  
      `openclaw pairing approve <channel> <code>` で承認するか、許可リストを使用します。
  
</Step>
  <Step title="Daemon install">
    - macOS: LaunchAgent  
      - ログイン中のユーザーセッションが必要です。ヘッドレス環境ではカスタム LaunchDaemon（同梱されていません）を使用してください。  
    - Linux および Windows（WSL2 経由）: systemd ユーザーユニット  
      - ウィザードは `loginctl enable-linger <user>` を試み、ログアウト後もゲートウェイが起動し続けるようにします。  
      - sudo を求められる場合があります（`/var/lib/systemd/linger` へ書き込み）。まず sudo なしで試行します。  
    - ランタイム選択: Node（推奨。WhatsApp と Telegram に必須）。Bun は推奨されません。
  
</Step>
  <Step title="Health check">
    - 必要に応じてゲートウェイを起動し、`openclaw health` を実行します。  
    - `openclaw status --deep` はゲートウェイのヘルスプローブをステータス出力に追加します。
  
</Step>
  <Step title="Skills">
    - 利用可能なスキルを読み取り、要件を確認します。  
    - Node マネージャーを選択できます: npm または pnpm（bun は推奨されません）。  
    - 任意の依存関係をインストールします（一部は macOS で Homebrew を使用します）。
  
</Step>
  <Step title="Finish">
    - iOS、Android、macOS アプリのオプションを含む概要と次のステップを表示します。
  
</Step>
</Steps>

<Note>
GUI が検出されない場合、ウィザードはブラウザを開く代わりに Control UI 用の SSH ポートフォワード手順を表示します。  
Control UI アセットが不足している場合、ウィザードはビルドを試みます。フォールバックは `pnpm ui:build`（UI 依存関係を自動インストール）です。
</Note>

## リモートモードの詳細

リモートモードでは、このマシンを別の場所にあるゲートウェイへ接続するよう構成します。

<Info>
リモートモードでは、リモートホストへのインストールや変更は行いません。
</Info>

設定する内容:

- リモートゲートウェイ URL（`ws://...`）  
- リモートゲートウェイで認証が必要な場合のトークン（推奨）

<Note>
- ゲートウェイがループバック専用の場合は、SSH トンネルまたは tailnet を使用してください。  
- ディスカバリーヒント:  
  - macOS: Bonjour（`dns-sd`）  
  - Linux: Avahi（`avahi-browse`）
</Note>

## 認証とモデルのオプション

<AccordionGroup>
  <Accordion title="Anthropic API key (recommended)">
    `ANTHROPIC_API_KEY` が存在する場合はそれを使用し、なければキーの入力を求め、デーモン利用のために保存します。
  
</Accordion>
  <Accordion title="Anthropic OAuth (Claude Code CLI)">
    - macOS: キーチェーン項目「Claude Code-credentials」を確認します  
    - Linux および Windows: `~/.claude/.credentials.json` が存在すれば再利用します  

    macOS では「常に許可」を選択し、launchd 起動時にブロックされないようにしてください。
  
</Accordion>
  <Accordion title="Anthropic token (setup-token paste)">
    任意のマシンで `claude setup-token` を実行し、そのトークンを貼り付けます。  
    名前を付けることができます。空欄の場合はデフォルトを使用します。
  
</Accordion>
  <Accordion title="OpenAI Code subscription (Codex CLI reuse)">
    `~/.codex/auth.json` が存在する場合、ウィザードで再利用できます。
  
</Accordion>
  <Accordion title="OpenAI Code subscription (OAuth)">
    ブラウザーフローを実行し、`code#state` を貼り付けます。

    モデルが未設定、または `openai/*` の場合、`agents.defaults.model` を `openai-codex/gpt-5.3-codex` に設定します。
  
</Accordion>
  <Accordion title="OpenAI API key">
    `OPENAI_API_KEY` が存在する場合はそれを使用し、なければキーの入力を求め、  
    launchd が読み取れるよう `~/.openclaw/.env` に保存します。

    モデルが未設定、`openai/*`、または `openai-codex/*` の場合、  
    `agents.defaults.model` を `openai/gpt-5.1-codex` に設定します。
  
</Accordion>
  <Accordion title="xAI (Grok) API key">
    `XAI_API_KEY` の入力を求め、xAI をモデルプロバイダーとして構成します。
  
</Accordion>
  <Accordion title="OpenCode Zen">
    `OPENCODE_API_KEY`（または `OPENCODE_ZEN_API_KEY`）の入力を求めます。  
    セットアップ URL: [opencode.ai/auth](https://opencode.ai/auth)。
  
</Accordion>
  <Accordion title="API key (generic)">
    キーを保存します。
  
</Accordion>
  <Accordion title="Vercel AI Gateway">
    `AI_GATEWAY_API_KEY` の入力を求めます。  
    詳細: [Vercel AI Gateway](/providers/vercel-ai-gateway)。
  
</Accordion>
  <Accordion title="Cloudflare AI Gateway">
    アカウント ID、ゲートウェイ ID、`CLOUDFLARE_AI_GATEWAY_API_KEY` の入力を求めます。  
    詳細: [Cloudflare AI Gateway](/providers/cloudflare-ai-gateway)。
  
</Accordion>
  <Accordion title="MiniMax M2.1">
    設定は自動的に書き込まれます。  
    詳細: [MiniMax](/providers/minimax)。
  
</Accordion>
  <Accordion title="Synthetic (Anthropic-compatible)">
    `SYNTHETIC_API_KEY` の入力を求めます。  
    詳細: [Synthetic](/providers/synthetic)。
  
</Accordion>
  <Accordion title="Moonshot and Kimi Coding">
    Moonshot（Kimi K2）および Kimi Coding の設定は自動的に書き込まれます。  
    詳細: [Moonshot AI (Kimi + Kimi Coding)](/providers/moonshot)。
  
</Accordion>
  <Accordion title="Skip">
    認証は未設定のままにします。
  
</Accordion>
</AccordionGroup>

モデルの挙動:

- 検出された選択肢からデフォルトモデルを選択するか、プロバイダーとモデルを手動で入力します。  
- ウィザードはモデルチェックを実行し、設定されたモデルが不明、または認証が不足している場合に警告します。

資格情報およびプロファイルのパス:

- OAuth 資格情報: `~/.openclaw/credentials/oauth.json`  
- 認証プロファイル（API キー + OAuth）: `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`

<Note>
ヘッドレスおよびサーバー向けのヒント: ブラウザーのあるマシンで OAuth を完了し、  
`~/.openclaw/credentials/oauth.json`（または `$OPENCLAW_STATE_DIR/credentials/oauth.json`）  
をゲートウェイホストへコピーしてください。
</Note>

## 出力と内部仕様

`~/.openclaw/openclaw.json` に含まれる代表的なフィールド:

- `agents.defaults.workspace`  
- `agents.defaults.model` / `models.providers`（MiniMax を選択した場合）  
- `gateway.*`（モード、バインド、認証、Tailscale）  
- `channels.telegram.botToken`, `channels.discord.token`, `channels.signal.*`, `channels.imessage.*`  
- プロンプト中に同意した場合のチャンネル許可リスト（Slack、Discord、Matrix、Microsoft Teams）。可能な場合は名前が ID に解決されます。  
- `skills.install.nodeManager`  
- `wizard.lastRunAt`  
- `wizard.lastRunVersion`  
- `wizard.lastRunCommit`  
- `wizard.lastRunCommand`  
- `wizard.lastRunMode`

`openclaw agents add` は `agents.list[]` と、任意で `bindings` を書き込みます。

WhatsApp の資格情報は `~/.openclaw/credentials/whatsapp/<accountId>/` 配下に保存されます。  
セッションは `~/.openclaw/agents/<agentId>/sessions/` 配下に保存されます。

<Note>
一部のチャンネルはプラグインとして提供されます。オンボーディング中に選択された場合、  
ウィザードはチャンネル設定の前にプラグイン（npm またはローカルパス）のインストールを促します。
</Note>

Gateway ウィザード RPC:

- `wizard.start`  
- `wizard.next`  
- `wizard.cancel`  
- `wizard.status`

クライアント（macOS アプリおよび Control UI）は、オンボーディングロジックを再実装せずに手順を描画できます。

Signal セットアップの挙動:

- 適切なリリースアセットをダウンロードします  
- `~/.openclaw/tools/signal-cli/<version>/` 配下に保存します  
- 設定に `channels.signal.cliPath` を書き込みます  
- JVM ビルドには Java 21 が必要です  
- 利用可能な場合はネイティブビルドが使用されます  
- Windows では WSL2 を使用し、WSL 内で Linux の signal-cli フローに従います

## 関連ドキュメント

- オンボーディング ハブ: [Onboarding Wizard (CLI)](/start/wizard)  
- 自動化とスクリプト: [CLI Automation](/start/wizard-cli-automation)  
- コマンド リファレンス: [`openclaw onboard`](/cli/onboard)

