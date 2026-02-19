---
title: "ショーケース"
description: "コミュニティによる実世界のOpenClawプロジェクト"
---

{/* v2 */}
# ショーケース

コミュニティによる実際のプロジェクト。OpenClawで人々がどのようなものを構築しているかをご覧ください。

<Info>
**掲載されたいですか？** [Discordの#showcase](https://discord.gg/clawd)でプロジェクトを共有するか、[Xで@openclawをタグ付け](https://x.com/openclaw)してください。
</Info>

## 🎥 OpenClawの実演

VelvetSharkによる完全セットアップ解説（28分）。

<div
  style={{
    position: "relative",
    paddingBottom: "56.25%",
    height: 0,
    overflow: "hidden",
    borderRadius: 16,
  }}
>
  <iframe
    src="https://www.youtube-nocookie.com/embed/SaWSPZoPX34"
    title="OpenClaw: The self-hosted AI that Siri should have been (Full setup)"
    style={{ position: "absolute", top: 0, left: 0, width: "100%", height: "100%" }}
    frameBorder="0"
    loading="lazy"
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
    allowFullScreen></iframe>
</div>

[YouTubeで視聴](https://www.youtube.com/watch?v=SaWSPZoPX34)

<div
  style={{
    position: "relative",
    paddingBottom: "56.25%",
    height: 0,
    overflow: "hidden",
    borderRadius: 16,
  }}
>
  <iframe
    src="https://www.youtube-nocookie.com/embed/mMSKQvlmFuQ"
    title="OpenClaw showcase video"
    style={{ position: "absolute", top: 0, left: 0, width: "100%", height: "100%" }}
    frameBorder="0"
    loading="lazy"
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
    allowFullScreen></iframe>
</div>

[YouTubeで視聴](https://www.youtube.com/watch?v=mMSKQvlmFuQ)

<div
  style={{
    position: "relative",
    paddingBottom: "56.25%",
    height: 0,
    overflow: "hidden",
    borderRadius: 16,
  }}
>
  <iframe
    src="https://www.youtube-nocookie.com/embed/5kkIJNUGFho"
    title="OpenClaw community showcase"
    style={{ position: "absolute", top: 0, left: 0, width: "100%", height: "100%" }}
    frameBorder="0"
    loading="lazy"
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
    allowFullScreen></iframe>
</div>

[YouTubeで視聴](https://www.youtube.com/watch?v=5kkIJNUGFho)

## 🆕 Discordの最新情報

<CardGroup cols={2}>

<Card title="PRレビュー → Telegramフィードバック" icon="code-pull-request" href="https://x.com/i/status/2010878524543131691">
  **@bangnokia** • `review` `github` `telegram`

OpenCodeが変更を完了 → PRを作成 → OpenClawが差分をレビューし、「軽微な提案」と明確なマージ可否（先に適用すべき重要な修正を含む）をTelegramで返信。

  <img src="/assets/showcase/pr-review-telegram.jpg" alt="Telegramで配信されたOpenClawのPRレビュー結果"></img>
</Card>

<Card title="数分で作るワインセラースキル" icon="wine-glass" href="https://x.com/i/status/2010916352454791216">
  **@prades_maxime** • `skills` `local` `csv`

ローカルのワインセラースキルを「Robby」（@openclaw）に依頼。サンプルのCSVエクスポートと保存場所を確認し、その後すばやくスキルを構築・テスト（例では962本）。

  <img src="/assets/showcase/wine-cellar-skill.jpg" alt="CSVからローカルのワインセラースキルを構築するOpenClaw"></img>
</Card>

<Card title="Tesco買い物オートパイロット" icon="cart-shopping" href="https://x.com/i/status/2009724862470689131">
  **@marchattonhere** • `automation` `browser` `shopping`

週間ミールプラン → 定番商品 → 配送枠を予約 → 注文確定。APIは使わず、ブラウザ操作のみ。

  <img src="/assets/showcase/tesco-shop.jpg" alt="チャットによるTesco買い物自動化"></img>
</Card>

<Card title="SNAG スクリーンショット→Markdown" icon="scissors" href="https://github.com/am-will/snag">
  **@am-will** • `devtools` `screenshots` `markdown`

画面範囲をホットキーで選択 → Gemini Vision → クリップボードに即座にMarkdown生成。

  <img src="/assets/showcase/snag.png" alt="SNAGスクリーンショットからMarkdownへのツール"></img>
</Card>

<Card title="Agents UI" icon="window-maximize" href="https://releaseflow.net/kitze/agents-ui">
  **@kitze** • `ui` `skills` `sync`

Agents、Claude、Codex、OpenClaw間でスキル／コマンドを管理するデスクトップアプリ。

  <img src="/assets/showcase/agents-ui.jpg" alt="Agents UIアプリ"></img>
</Card>

<Card title="Telegram音声ノート（papla.media）" icon="microphone" href="https://papla.media/docs">
  **Community** • `voice` `tts` `telegram`

papla.mediaのTTSをラップし、結果をTelegramの音声ノートとして送信（煩わしい自動再生なし）。

  <img src="/assets/showcase/papla-tts.jpg" alt="TTSからのTelegram音声ノート出力"></img>
</Card>

<Card title="CodexMonitor" icon="eye" href="https://clawhub.com/odrobnik/codexmonitor">
  **@odrobnik** • `devtools` `codex` `brew`

Homebrewでインストールできるヘルパー。ローカルのOpenAI Codexセッションを一覧表示／検査／監視（CLI + VS Code）。

  <img src="/assets/showcase/codexmonitor.png" alt="ClawHub上のCodexMonitor"></img>
</Card>

<Card title="Bambu 3Dプリンター制御" icon="print" href="https://clawhub.com/tobiasbischoff/bambu-cli">
  **@tobiasbischoff** • `hardware` `3d-printing` `skill`

BambuLabプリンターの制御とトラブルシュート：ステータス、ジョブ、カメラ、AMS、キャリブレーションなど。

  <img src="/assets/showcase/bambu-cli.png" alt="ClawHub上のBambu CLIスキル"></img>
</Card>

<Card title="ウィーン交通（Wiener Linien）" icon="train" href="https://clawhub.com/hjanuschka/wienerlinien">
  **@hjanuschka** • `travel` `transport` `skill`

ウィーンの公共交通機関のリアルタイム出発情報、運行障害、エレベーター状況、経路案内。

  <img src="/assets/showcase/wienerlinien.png" alt="ClawHub上のWiener Linienスキル"></img>
</Card>

<Card title="ParentPay学校給食" icon="utensils" href="#">
  **@George5562** • `automation` `browser` `parenting`

ParentPayを使った英国の学校給食予約を自動化。確実に表のセルをクリックするためマウス座標を使用。
</Card>

<Card title="R2アップロード（Send Me My Files）" icon="cloud-arrow-up" href="https://clawhub.com/skills/r2-upload">
  **@julianengel** • `files` `r2` `presigned-urls`

Cloudflare R2/S3へアップロードし、安全な署名付きダウンロードリンクを生成。リモートのOpenClawインスタンスに最適。
</Card>

<Card title="Telegram経由でiOSアプリ開発" icon="mobile" href="#">
  **@coard** • `ios` `xcode` `testflight`

マップと音声録音機能を備えた完全なiOSアプリを構築し、すべてTelegramチャット経由でTestFlightにデプロイ。

  <img src="/assets/showcase/ios-testflight.jpg" alt="TestFlight上のiOSアプリ"></img>
</Card>

<Card title="Oura Ringヘルスアシスタント" icon="heart-pulse" href="#">
  **@AS** • `health` `oura` `calendar`

Ouraリングのデータをカレンダー、予定、ジムのスケジュールと統合するパーソナルAIヘルスアシスタント。

  <img src="/assets/showcase/oura-health.png" alt="Ouraリングのヘルスアシスタント"></img>
</Card>
<Card title="Kevのドリームチーム（14以上のエージェント）" icon="robot" href="https://github.com/adam91holt/orchestrated-ai-articles">
  **@adam91holt** • `multi-agent` `orchestration` `architecture` `manifesto`

Opus 4.5オーケストレーターがCodexワーカーへ委任する、1つのゲートウェイ配下に14以上のエージェントを構成。ドリームチームの構成、モデル選定、サンドボックス化、webhooks、ハートビート、委任フローを網羅した包括的な[技術解説](https://github.com/adam91holt/orchestrated-ai-articles)。エージェントのサンドボックス用[Clawdspace](https://github.com/adam91holt/clawdspace)。[ブログ記事](https://adams-ai-journey.ghost.io/2026-the-year-of-the-orchestrator/)。
</Card>

<Card title="Linear CLI" icon="terminal" href="https://github.com/Finesssee/linear-cli">
  **@NessZerra** • `devtools` `linear` `cli` `issues`

エージェントワークフロー（Claude Code、OpenClaw）と統合するLinear用CLI。ターミナルから課題、プロジェクト、ワークフローを管理。初の外部PRがマージ！
</Card>

<Card title="Beeper CLI" icon="message" href="https://github.com/blqke/beepcli">
  **@jules** • `messaging` `beeper` `cli` `automation`

Beeper Desktop経由でメッセージの閲覧、送信、アーカイブを実行。BeeperのローカルMCP APIを使用し、エージェントがiMessageやWhatsAppなどすべてのチャットを一元管理。
</Card>

</CardGroup>

## 🤖 自動化＆ワークフロー

<CardGroup cols={2}>

<Card title="Winix空気清浄機制御" icon="wind" href="https://x.com/antonplex/status/2010518442471006253">
  **@antonplex** • `automation` `hardware` `air-quality`

Claude Codeが清浄機の制御方法を発見・確認し、その後OpenClawが引き継いで室内の空気質を管理。

  <img src="/assets/showcase/winix-air-purifier.jpg" alt="OpenClawによるWinix空気清浄機の制御"></img>
</Card>

<Card title="美しい空のカメラショット" icon="camera" href="https://x.com/signalgaining/status/2010523120604746151">
  **@signalgaining** • `automation` `camera` `skill` `images`

屋根のカメラをトリガーに、空がきれいなときに写真を撮るようOpenClawに依頼——スキルを設計し、撮影を実行。

  <img src="/assets/showcase/roof-camera-sky.jpg" alt="OpenClawが撮影した屋根カメラの空のスナップショット"></img>
</Card>

<Card title="ビジュアル朝のブリーフィングシーン" icon="robot" href="https://x.com/buddyhadry/status/2010005331925954739">
  **@buddyhadry** • `automation` `briefing` `images` `telegram`

スケジュールされたプロンプトが、毎朝ひとつの「シーン」画像（天気、タスク、日付、お気に入りの投稿／引用）をOpenClawペルソナ経由で生成。
</Card>

<Card title="パデルコート予約" icon="calendar-check" href="https://github.com/joshp123/padel-cli">
  **@joshp123** • `automation` `booking` `cli`
  
  Playtomicの空き状況チェック＋予約CLI。空きコートを見逃さない。
  
  <img src="/assets/showcase/padel-screenshot.jpg" alt="padel-cliのスクリーンショット"></img>
</Card>

<Card title="会計書類取り込み" icon="file-invoice-dollar">
  **Community** • `automation` `email` `pdf`
  
  メールからPDFを収集し、税理士向けに書類を準備。毎月の会計処理を自動化。
</Card>

<Card title="カウチポテト開発モード" icon="couch" href="https://davekiss.com">
  **@davekiss** • `telegram` `website` `migration` `astro`

Netflixを見ながらTelegram経由で個人サイト全体を再構築——Notion → Astro、18記事を移行、DNSをCloudflareへ。ノートPCは一度も開かず。
</Card>

<Card title="求人検索エージェント" icon="briefcase">
  **@attol8** • `automation` `api` `skill`

求人情報を検索し、CVキーワードと照合して関連求人をリンク付きで返却。JSearch APIを使い30分で構築。
</Card>

<Card title="Jiraスキルビルダー" icon="diagram-project" href="https://x.com/jdrhyne/status/2008336434827002232">
  **@jdrhyne** • `automation` `jira` `skill` `devtools`

OpenClawがJiraに接続し、その場で新しいスキルを生成（ClawHubに存在する前に）。
</Card>

<Card title="Telegram経由Todoistスキル" icon="list-check" href="https://x.com/iamsubhrajyoti/status/2009949389884920153">
  **@iamsubhrajyoti** • `automation` `todoist` `skill` `telegram`

Todoistタスクを自動化し、OpenClawがTelegramチャット内で直接スキルを生成。
</Card>

<Card title="TradingView分析" icon="chart-line">
  **@bheem1798** • `finance` `browser` `automation`

ブラウザ自動化でTradingViewにログインし、チャートをスクリーンショット、必要に応じてテクニカル分析を実行。API不要——ブラウザ操作のみ。
</Card>

<Card title="Slack自動サポート" icon="slack">
  **@henrymascot** • `slack` `automation` `support`

社内Slackチャンネルを監視し、適切に返信し、通知をTelegramへ転送。依頼されることなく本番環境のバグを自律的に修正。
</Card>

</CardGroup>

## 🧠 ナレッジ＆メモリ

<CardGroup cols={2}>

<Card title="xuezh中国語学習" icon="language" href="https://github.com/joshp123/xuezh">
  **@joshp123** • `learning` `voice` `skill`
  
  発音フィードバックと学習フローを備えた、中国語学習エンジン（OpenClaw経由）。
  
  <img src="/assets/showcase/xuezh-pronunciation.jpeg" alt="xuezhの発音フィードバック"></img>
</Card>

<Card title="WhatsAppメモリーヴォールト" icon="vault">
  **Community** • `memory` `transcription` `indexing`
  
  WhatsAppのエクスポート全体を取り込み、1,000件以上の音声ノートを文字起こしし、gitログと照合してリンク付きMarkdownレポートを出力。
</Card>

<Card title="Karakeepセマンティック検索" icon="magnifying-glass" href="https://github.com/jamesbrooksco/karakeep-semantic-search">
  **@jamesbrooksco** • `search` `vector` `bookmarks`
  
  Qdrant + OpenAI/Ollamaの埋め込みを使用し、Karakeepブックマークにベクトル検索を追加。
</Card>

<Card title="Inside-Out-2メモリ" icon="brain">
  **Community** • `memory` `beliefs` `self-model`
  
  セッションファイルをメモリへ→信念へ→進化する自己モデルへと変換する独立したメモリマネージャー。
</Card>

</CardGroup>

## 🎙️ 音声＆電話

<CardGroup cols={2}>

<Card title="Clawdia電話ブリッジ" icon="phone" href="https://github.com/alejandroOPI/clawdia-bridge">
  **@alejandroOPI** • `voice` `vapi` `bridge`
  
  Vapi音声アシスタント ↔ OpenClaw HTTPブリッジ。エージェントとのほぼリアルタイム通話。
</Card>

<Card title="OpenRouter文字起こし" icon="microphone" href="https://clawhub.com/obviyus/openrouter-transcribe">
  **@obviyus** • `transcription` `multilingual` `skill`

OpenRouter（Geminiなど）経由の多言語音声文字起こし。ClawHubで利用可能。
</Card>

</CardGroup>

## 🏗️ インフラ＆デプロイ

<CardGroup cols={2}>

<Card title="Home Assistantアドオン" icon="home" href="https://github.com/ngutman/openclaw-ha-addon">
  **@ngutman** • `homeassistant` `docker` `raspberry-pi`
  
  SSHトンネル対応と永続状態を備えた、Home Assistant OS上で動作するOpenClawゲートウェイ。
</Card>

<Card title="Home Assistantスキル" icon="toggle-on" href="https://clawhub.com/skills/homeassistant">
  **ClawHub** • `homeassistant` `skill` `automation`
  
  自然言語でHome Assistantデバイスを制御・自動化。
</Card>

<Card title="Nixパッケージング" icon="snowflake" href="https://github.com/openclaw/nix-openclaw">
  **@openclaw** • `nix` `packaging` `deployment`
  
  再現可能なデプロイのための、必要なものをすべて含んだnix化OpenClaw設定。
</Card>

<Card title="CalDAVカレンダー" icon="calendar" href="https://clawhub.com/skills/caldav-calendar">
  **ClawHub** • `calendar` `caldav` `skill`
  
  khal/vdirsyncerを使用したカレンダースキル。セルフホスト型カレンダー統合。
</Card>

</CardGroup>

## 🏠 ホーム＆ハードウェア

<CardGroup cols={2}>

<Card title="GoHome自動化" icon="house-signal" href="https://github.com/joshp123/gohome">
  **@joshp123** • `home` `nix` `grafana`
  
  OpenClawをインターフェースとするNixネイティブのホームオートメーションに、美しいGrafanaダッシュボードを追加。
  
  <img src="/assets/showcase/gohome-grafana.png" alt="GoHomeのGrafanaダッシュボード"></img>
</Card>

<Card title="Roborock掃除機" icon="robot" href="https://github.com/joshp123/gohome/tree/main/plugins/roborock">
  **@joshp123** • `vacuum` `iot` `plugin`
  
  自然な会話を通じてRoborockロボット掃除機を操作。
  
  <img src="/assets/showcase/roborock-screenshot.jpg" alt="Roborockのステータス"></img>
</Card>

</CardGroup>

## 🌟 コミュニティプロジェクト

<CardGroup cols={2}>

<Card title="StarSwapマーケットプレイス" icon="star" href="https://star-swap.com/">
  **Community** • `marketplace` `astronomy` `webapp`
  
  本格的な天文機材マーケットプレイス。OpenClawエコシステムを活用して構築。
</Card>

</CardGroup>

---

## プロジェクトを投稿する

共有したいものがありますか？ぜひ掲載させてください！

<Steps>
  <Step title="共有する">
    [Discordの#showcase](https://discord.gg/clawd)に投稿、または[@openclawにツイート](https://x.com/openclaw)
  
</Step>
  <Step title="詳細を含める">
    内容の説明、リポジトリ／デモへのリンク、可能であればスクリーンショットを共有
  
</Step>
  <Step title="掲載される">
    優れたプロジェクトをこのページに追加します
  
</Step>
</Steps>
