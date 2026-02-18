---
title: "サブエージェント"
---

# Sub-agents

サブエージェントは、既存のエージェント実行から生成されるバックグラウンド実行です。独自のセッション（`agent:<agentId>:subagent:<uuid>`）で実行され、完了すると依頼元のチャットチャンネルに結果を**announce（通知）**します。

## Slash command

**現在のセッション**のサブエージェント実行を確認・操作するには `/subagents` を使用します：

- `/subagents list`
- `/subagents kill <id|#|all>`
- `/subagents log <id|#> [limit] [tools]`
- `/subagents info <id|#>`
- `/subagents send <id|#> <message>`

`/subagents info` は実行メタデータ（ステータス、タイムスタンプ、セッションID、トランスクリプトパス、クリーンアップ）を表示します。

主な目的：

- メイン実行をブロックせずに「調査／長時間タスク／低速ツール」作業を並列化する。
- デフォルトでサブエージェントを分離する（セッション分離＋オプションのサンドボックス化）。
- ツールの誤用を防ぐ：サブエージェントはデフォルトでセッションツールを持ちません。
- オーケストレーターパターンのためのネスト深度設定をサポートする。

コストに関する注意：各サブエージェントは**独自の**コンテキストとトークン使用量を持ちます。重い、または繰り返しの多いタスクには、サブエージェントにより安価なモデルを設定し、メインエージェントは高品質モデルのままにすることを推奨します。これは `agents.defaults.subagents.model` またはエージェントごとの上書きで設定できます。

## Tool

`sessions_spawn` を使用します：

- サブエージェント実行を開始します（`deliver: false`、グローバルレーン: `subagent`）
- その後 announce ステップを実行し、announce の返信を依頼元チャットチャンネルに投稿します
- デフォルトモデル：呼び出し元を継承しますが、`agents.defaults.subagents.model`（または `agents.list[].subagents.model`）を設定している場合はそれが使用されます。`sessions_spawn.model` を明示した場合はそれが最優先されます。
- デフォルト思考レベル：呼び出し元を継承しますが、`agents.defaults.subagents.thinking`（または `agents.list[].subagents.thinking`）を設定している場合はそれが使用されます。`sessions_spawn.thinking` を明示した場合はそれが最優先されます。

ツールパラメータ：

- `task`（必須）
- `label?`（任意）
- `agentId?`（任意；許可されている場合は別のエージェントID配下で生成）
- `model?`（任意；サブエージェントのモデルを上書き。無効な値はスキップされ、警告付きでデフォルトモデルで実行されます）
- `thinking?`（任意；サブエージェント実行の思考レベルを上書き）
- `runTimeoutSeconds?`（デフォルト `0`；設定した場合、N秒後に実行を中止）
- `cleanup?`（`delete|keep`、デフォルト `keep`）

許可リスト：

- `agents.list[].subagents.allowAgents`：`agentId` で指定可能なエージェントIDのリスト（`["*"]` ですべて許可）。デフォルトは依頼元エージェントのみ。

検出：

- `agents_list` を使用すると、`sessions_spawn` で現在許可されているエージェントIDを確認できます。

自動アーカイブ：

- サブエージェントのセッションは `agents.defaults.subagents.archiveAfterMinutes`（デフォルト: 60）後に自動アーカイブされます。
- アーカイブは `sessions.delete` を使用し、トランスクリプトを `*.deleted.<timestamp>` にリネームします（同じフォルダ）。
- `cleanup: "delete"` を指定すると、announce 後に即時アーカイブされます（リネームによりトランスクリプトは保持されます）。
- 自動アーカイブはベストエフォートです。ゲートウェイが再起動すると保留中タイマーは失われます。
- `runTimeoutSeconds` は自動アーカイブしません。実行のみ停止し、セッションは自動アーカイブまで残ります。
- 自動アーカイブは深度1および深度2セッションの両方に適用されます。

## Nested Sub-Agents

デフォルトでは、サブエージェントは自身のサブエージェントを生成できません（`maxSpawnDepth: 1`）。`maxSpawnDepth: 2` を設定すると1段階のネストが可能になり、**オーケストレーターパターン**（main → オーケストレーターサブエージェント → ワーカーサブサブエージェント）を実現できます。

### How to enable

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxSpawnDepth: 2, // allow sub-agents to spawn children (default: 1)
        maxChildrenPerAgent: 5, // max active children per agent session (default: 5)
        maxConcurrent: 8, // global concurrency lane cap (default: 8)
      },
    },
  },
}
```

### Depth levels

| Depth | Session key shape                            | Role                                          | Can spawn?                   |
| ----- | -------------------------------------------- | --------------------------------------------- | ---------------------------- |
| 0     | `agent:<id>:main`                            | メインエージェント                            | 常に可能                     |
| 1     | `agent:<id>:subagent:<uuid>`                 | サブエージェント（depth 2 許可時はオーケストレーター） | `maxSpawnDepth >= 2` の場合のみ |
| 2     | `agent:<id>:subagent:<uuid>:subagent:<uuid>` | サブサブエージェント（リーフワーカー）        | 不可                         |

### Announce chain

結果はチェーンを遡って伝播します：

1. 深度2ワーカーが完了 → 親（深度1オーケストレーター）に announce
2. 深度1オーケストレーターが受信・統合して完了 → メインに announce
3. メインエージェントが受信し、ユーザーへ配信

各レベルは直接の子からの announce のみを受け取ります。

### Tool policy by depth

- **Depth 1（オーケストレーター、`maxSpawnDepth >= 2` の場合）**：`sessions_spawn`、`subagents`、`sessions_list`、`sessions_history` を利用可能。他のセッション／システムツールは引き続き拒否。
- **Depth 1（リーフ、`maxSpawnDepth == 1` の場合）**：セッションツールなし（現在のデフォルト動作）。
- **Depth 2（リーフワーカー）**：セッションツールなし — 深度2では `sessions_spawn` は常に拒否。さらに子を生成できません。

### Per-agent spawn limit

各エージェントセッション（任意の深度）は、同時に最大 `maxChildrenPerAgent`（デフォルト: 5）のアクティブな子のみを持てます。単一のオーケストレーターからの暴走的なファンアウトを防ぎます。

### Cascade stop

深度1オーケストレーターを停止すると、その深度2の子も自動的に停止します：

- メインチャットで `/stop` を送信すると、すべての深度1エージェントが停止し、その深度2の子にも波及します。
- `/subagents kill <id>` は特定のサブエージェントを停止し、その子にも波及します。
- `/subagents kill all` は依頼元のすべてのサブエージェントを停止し、波及します。

## Authentication

サブエージェントの認証は、セッション種別ではなく**エージェントID**で解決されます：

- サブエージェントのセッションキーは `agent:<agentId>:subagent:<uuid>` です。
- auth ストアは対象エージェントの `agentDir` から読み込まれます。
- メインエージェントの auth プロファイルは**フォールバック**としてマージされ、競合時はエージェント側が優先されます。

注意：マージは加算的であり、メインのプロファイルは常にフォールバックとして利用可能です。エージェントごとの完全分離された認証は現在サポートされていません。

## Announce

サブエージェントは announce ステップを通じて結果を報告します：

- announce ステップはサブエージェントのセッション内で実行されます（依頼元セッションではありません）。
- サブエージェントが正確に `ANNOUNCE_SKIP` と返信した場合、何も投稿されません。
- それ以外の場合、announce の返信はフォローアップの `agent` 呼び出し（`deliver=true`）を通じて依頼元チャットに投稿されます。
- announce 返信は利用可能な場合、スレッド／トピックのルーティングを保持します（Slack スレッド、Telegram トピック、Matrix スレッド）。
- announce メッセージは安定したテンプレートに正規化されます：
  - `Status:` 実行結果（`success`、`error`、`timeout`、`unknown`）に基づく。
  - `Result:` announce ステップの要約内容（ない場合は `(not available)`）。
  - `Notes:` エラー詳細やその他有用な情報。
- `Status` はモデル出力から推測されず、ランタイム結果シグナルに基づきます。

announce ペイロードの末尾には（ラップされていても）統計行が含まれます：

- 実行時間（例：`runtime 5m12s`）
- トークン使用量（input/output/total）
- モデル価格設定が構成されている場合の推定コスト（`models.providers.*.models[].cost`）
- `sessionKey`、`sessionId`、トランスクリプトパス（メインエージェントが `sessions_history` で履歴取得またはディスク上のファイル確認に使用可能）

## Tool Policy (sub-agent tools)

デフォルトでは、サブエージェントは**すべてのツール（セッションツールとシステムツールを除く）**を利用できます：

- `sessions_list`
- `sessions_history`
- `sessions_send`
- `sessions_spawn`

`maxSpawnDepth >= 2` の場合、深度1オーケストレーターサブエージェントは追加で `sessions_spawn`、`subagents`、`sessions_list`、`sessions_history` を受け取り、子を管理できます。

設定で上書き可能：

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxConcurrent: 1,
      },
    },
  },
  tools: {
    subagents: {
      tools: {
        // deny wins
        deny: ["gateway", "cron"],
        // if allow is set, it becomes allow-only (deny still wins)
        // allow: ["read", "exec", "process"]
      },
    },
  },
}
```

## Concurrency

サブエージェントは専用のインプロセスキューレーンを使用します：

- レーン名：`subagent`
- 同時実行数：`agents.defaults.subagents.maxConcurrent`（デフォルト `8`）

## Stopping

- 依頼元チャットで `/stop` を送信すると、依頼元セッションが中止され、そこから生成されたアクティブなサブエージェント実行も停止し、ネストした子にも波及します。
- `/subagents kill <id>` は特定のサブエージェントを停止し、その子にも波及します。

## Limitations

- サブエージェントの announce は**ベストエフォート**です。ゲートウェイが再起動すると、保留中の「announce 戻し」処理は失われます。
- サブエージェントは同じゲートウェイプロセスのリソースを共有します；`maxConcurrent` を安全弁として扱ってください。
- `sessions_spawn` は常にノンブロッキングです：即座に `{ status: "accepted", runId, childSessionKey }` を返します。
- サブエージェントのコンテキストには `AGENTS.md` と `TOOLS.md` のみが注入されます（`SOUL.md`、`IDENTITY.md`、`USER.md`、`HEARTBEAT.md`、`BOOTSTRAP.md` は含まれません）。
- 最大ネスト深度は5（`maxSpawnDepth` の範囲：1–5）。ほとんどのユースケースでは深度2が推奨されます。
- `maxChildrenPerAgent` はセッションごとのアクティブな子数を制限します（デフォルト: 5、範囲: 1–20）。

