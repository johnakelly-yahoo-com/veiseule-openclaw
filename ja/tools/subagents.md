---
summary: "サブエージェント: 要求元のチャットに結果を通知する、分離されたエージェント実行の生成"
read_when:
  - エージェントによるバックグラウンド／並列作業が必要な場合
  - sessions_spawn または sub-agent のツールポリシーを変更する場合
title: "Sub-Agents"
---

# サブエージェント

Sub-agents let you run background tasks without blocking the main conversation. When you spawn a sub-agent, it runs in its own isolated session, does its work, and announces the result back to the chat when finished.

## Command

Use the `/subagents` slash command to inspect and control sub-agent runs for the current session:

- `/subagents list`
- \`/subagents stop <id\\
- \`/subagents log <id\\
- \`/subagents info <id\\
- \`/subagents send <id\\

`/subagents info` は実行メタデータ（ステータス、タイムスタンプ、セッション ID、トランスクリプトのパス、クリーンアップ）を表示します。

主な目的:

- メイン実行をブロックせずに、「調査／長時間タスク／低速ツール」の作業を並列化すること。
- サブエージェントをデフォルトで分離する（セッション分離＋オプションのサンドボックス化）。
- ツールの利用範囲を誤用しにくい設計に保つ：サブエージェントにはデフォルトでセッションツールは**付与されません**。
- オーケストレーターパターン向けに、ネストの深さを設定可能にする。

コストに関する注意：各サブエージェントはそれぞれ**独自の**コンテキストとトークン使用量を持ちます。 負荷の高い、または繰り返し発生する
tasks については、サブエージェントにより安価なモデルを設定し、メインエージェントにはより高品質なモデルを維持してください。
Per-agent config: `agents.list[].subagents.model`

## #> [limit] [tools]\`

Explicit `model` parameter in the `sessions_spawn` call

- Global default: `agents.defaults.subagents.thinking`
- その後、announce ステップを実行し、その announce の返信をリクエスト元のチャットチャネルに投稿します
- デフォルトモデル：`agents.defaults.subagents.model`（またはエージェントごとの `agents.list[].subagents.model`）を設定しない限り、呼び出し元を継承します；ただし、明示的に指定された `sessions_spawn.model` が常に優先されます。
- Per-agent config: `agents.list[].subagents.thinking`

ツールパラメータ：

- `task`
- `label`
- Spawn under a different agent id (must be allowed)
- Invalid model values are silently skipped — the sub-agent runs on the next valid default with a warning in the tool result.
- `thinking?`（任意；サブエージェント実行時の thinking レベルを上書きします）
- Abort the sub-agent after N seconds
- `"delete"` \\

許可リスト：

- `agents.list[].subagents.allowAgents`：`agentId` 経由で指定可能なエージェントIDの一覧（任意のエージェントを許可する場合は `["*"]`）。 デフォルト：リクエスト元エージェントのみ。

検出：

- Use the `agents_list` tool to discover which agent ids are currently allowed for `sessions_spawn`.

自動アーカイブ

- サブエージェントのセッションは、設定可能な期間後に自動的にアーカイブされます：
- アーカイブでは、トランスクリプトの名前が \`\*.deleted. (same folder).
- `"delete"` archives immediately after announce
- 自動アーカイブのタイマーはベストエフォートです。ゲートウェイが再起動すると、保留中のタイマーは失われます。
- `runTimeoutSeconds` はセッションを自動アーカイブ **しません**。 22. セッションは、通常のアーカイブタイマーが発火するまで残ります。
- 自動アーカイブは、depth-1 と depth-2 のセッションに同様に適用されます。

## Stop a running sub-agent

By default, sub-agents can only spawn under their own agent id. To allow an agent to spawn sub-agents under other agent ids:

### 有効化方法

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxConcurrent: 4, // default: 8
      },
    },
  },
}
```

### 深さレベル

| 深さ | セッションキーの形式                                                                                                                                                                                                                                                                                                                                                                                                                   | 役割                                                    | spawn 可能か？                 |
| -- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------- | -------------------------- |
| 0  | `agentId`                                                                                                                                                                                                                                                                                                                                                                                                                    | _(caller's agent)_                 | 常に                         |
| 1  | {&#xA;agents: {&#xA;defaults: {&#xA;subagents: {&#xA;thinking: "low",&#xA;},&#xA;},&#xA;},&#xA;}                                                                                                                                                                                                                                                             | サブエージェント（depth 2 が許可されている場合はオーケストレーター）                | `maxSpawnDepth >= 2` の場合のみ |
| 2  | {&#xA;agents: {&#xA;list: [&#xA;{&#xA;id: "orchestrator",&#xA;subagents: {&#xA;allowAgents: ["researcher", "coder"], // or ["\*"] to allow any&#xA;},&#xA;},&#xA;],&#xA;},&#xA;} | Managing Sub-Agents (`/subagents`) | 不可                         |

### Announce チェーン

結果はチェーンを遡って流れます：

1. Depth-2 ワーカーが完了 → 親（depth-1 オーケストレーター）に announce
2. Depth-1 オーケストレーターが announce を受信し、結果を統合して完了 → メインに announce
3. メインエージェントが announce を受信し、ユーザーに提供します

各レベルは直下の子からの announce のみを参照できます。

### Tool Policy

- **Depth 1（オーケストレーター、`maxSpawnDepth >= 2` の場合）**：子を管理できるように、`sessions_spawn`、`subagents`、`sessions_list`、`sessions_history` を取得します。 その他のセッション／システムツールは引き続き拒否されます。
- **Depth 1（リーフ、`maxSpawnDepth == 1` の場合）**：セッションツールはありません（現在のデフォルト動作）。
- **Depth 2（リーフワーカー）**：セッションツールはありません — depth 2 では `sessions_spawn` は常に拒否されます。 それ以上の子を spawn することはできません。

### Cross-Agent Spawning

各エージェントセッション（任意の深さ）は、同時に最大 `maxChildrenPerAgent`（デフォルト：5）個までのアクティブな子を持つことができます。 単一のオーケストレーターからの制御不能なファンアウトを防ぎます。

### カスケード停止

深さ1のオーケストレーターを停止すると、その配下にある深さ2の子も自動的に停止します。

- メインチャットで `/stop` を実行すると、すべての深さ1エージェントが停止し、その深さ2の子にもカスケードします。
- {
  tools: {
  subagents: {
  tools: {
  // deny always wins over allow
  deny: ["browser", "firecrawl"],
  },
  },
  },
  }
- `/subagents kill all` は、リクエスターに紐づくすべてのサブエージェントを停止し、カスケードします。

## 認証

サブエージェントの認証は、セッション種別ではなく**エージェント ID** により解決されます。

- ```
  新しい分離されたセッションが作成されます（`agent:
  :subagent:
  `）。専用の `subagent` キューレーン上で実行されます。
  ```
- auth ストアは、対象エージェントの `agentDir` から読み込まれます。
- メインエージェントの auth プロファイルは **フォールバック** としてマージされます（競合時はエージェント側のプロファイルが優先されます）。

注: マージは加算的に行われるため、メインのプロファイルは常にフォールバックとして利用可能です。 現在、サブエージェントごとに完全に分離された auth はサポートされていません。
</Note>

## Announce Status

サブエージェントは announce ステップを通じて結果を報告します。

- announce ステップはサブエージェントのセッション内で実行されます（リクエスターのセッション内ではありません）。
- If no user-facing announcement is needed, the main-agent summarize step can return `NO_REPLY` and nothing is posted. This is different from `ANNOUNCE_SKIP`, which is used in agent-to-agent announce flow (`sessions_send`).
- それ以外の場合、announce の返信はフォローアップの `agent` 呼び出し（`deliver=true`）を通じてリクエスターのチャットチャンネルに投稿されます。
- 利用可能な場合、アナウンス返信はスレッド／トピックのルーティングを保持します（Slack スレッド、Telegram トピック、Matrix スレッド）。
- announce メッセージは安定したテンプレートに正規化されます。
  - `Status:` は実行結果（`success`、`error`、`timeout`、または `unknown`）から導出されます。
  - `Result:` は announce ステップのサマリー内容（存在しない場合は `(not available)`）。
  - `Notes:` エラーの詳細やその他の有用なコンテキスト。
- The announce message includes a status derived from the runtime outcome (not from model output):

Each announce includes a stats line with:

- 実行時間（例: `runtime 5m12s`）
- トークン使用量（入力／出力／合計）
- Estimated cost (when model pricing is configured via `models.providers.*.models[].cost`)
- `sessionKey`、`sessionId`、およびトランスクリプトのパス（メインエージェントが `sessions_history` を介して履歴を取得したり、ディスク上のファイルを確認したりできるようにするため）

## Customizing Sub-Agent Tools

By default, sub-agents get **all tools except** a set of denied tools that are unsafe or unnecessary for background tasks:

- Explicit `thinking` parameter in the `sessions_spawn` call
- `runTimeoutSeconds`
- `sessions_send`
- The `sessions_spawn` Tool

`maxSpawnDepth >= 2` の場合、深さ1のオーケストレーターのサブエージェントには、子を管理できるように `sessions_spawn`、`subagents`、`sessions_list`、および `sessions_history` も追加で付与されます。

config によるオーバーライド:

```json5
{
  tools: {
    subagents: {
      tools: {
        allow: ["read", "exec", "process", "write", "edit", "apply_patch"],
        // deny still wins if set
      },
    },
  },
}
```

## 同時実行

サブエージェントは専用のインプロセスキューレーンを使用します。

- レーン名: `subagent`
- Global default: `agents.defaults.subagents.model`

## 停止

- リクエスターのチャットで `/stop` を送信すると、リクエスターのセッションが中断され、そこから起動されたアクティブなサブエージェント実行も停止し、ネストされた子にもカスケードします。
- To restrict sub-agents to **only** specific tools:

## 制限事項

- サブエージェントの announce は**ベストエフォート**です。 gateway が再起動すると、保留中の「announce back」処理は失われます。
- サブエージェントは同じ gateway プロセスリソースを共有します。`maxConcurrent` はセーフティバルブとして扱ってください。
- ```
  メインエージェントは、タスク内容とともに `sessions_spawn` を呼び出します。 この呼び出しは **ノンブロッキング** です — メインエージェントは即座に `{ status: "accepted", runId, childSessionKey }` を受け取ります。
  ```
- サブエージェントのコンテキストには `AGENTS.md` と `TOOLS.md` のみが注入されます（`SOUL.md`、`IDENTITY.md`、`USER.md`、`HEARTBEAT.md`、`BOOTSTRAP.md` は含まれません）。
- 最大ネスト深度は 5 です（`maxSpawnDepth` の範囲: 1–5）。 ほとんどのユースケースでは深さ2が推奨されます。
- `maxChildrenPerAgent` はセッションごとのアクティブな子の数を制限します（デフォルト: 5、範囲: 1–20）。
