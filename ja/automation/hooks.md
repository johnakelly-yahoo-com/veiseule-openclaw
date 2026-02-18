---
summary: "Hooks：コマンドおよびライフサイクルイベント向けのイベント駆動型自動化"
read_when:
  - /new、/reset、/stop、およびエージェントのライフサイクルイベントに対してイベント駆動型の自動化を行いたい場合
  - フックを構築、インストール、またはデバッグしたい場合
title: "Hooks"
---

# Hooks

フックは、エージェントコマンドやイベントに応じてアクションを自動化するための拡張可能なイベント駆動システムを提供します。 フックはディレクトリから自動的に検出され、CLIコマンドを介して管理することができます。

## 概要の把握

Hooks は、何かが起きたときに実行される小さなスクリプトです。種類は 2 つあります。 2種類あります。

- **Hooks**（このページ）: `/new`、`/reset`、`/stop`、またはライフサイクルイベントなどのエージェントイベントが発火した際に、Gateway（ゲートウェイ）内部で実行されます。
- **Webhooks**: 外部のHTTPWebhookはOpenClawで他のシステムが動作するようにします。 **Webhooks**: 外部の HTTP Webhook で、他のシステムから OpenClaw での処理をトリガーできます。[Webhook Hooks](/automation/webhook) を参照するか、Gmail ヘルパーコマンドとして `openclaw webhooks` を使用してください。

Hooks はプラグイン内にバンドルすることもできます。詳細は [Plugins](/tools/plugin#plugin-hooks) を参照してください。

一般的な用途:

- セッションをリセットした際にメモリスナップショットを保存する
- トラブルシューティングやコンプライアンスのためにコマンドの監査ログを保持する
- セッションの開始や終了時に後続の自動化をトリガーする
- イベント発火時にエージェントのワークスペースへファイルを書き込んだり、外部 API を呼び出したりする

小さなTypeScript関数を書けば、フックを書くことができます。 フックは自動的に検出され、CLI経由でフックを有効または無効にします。

## 概要

フックシステムは次のことを可能にします:

- `/new` が発行されたときに、セッションコンテキストをメモリに保存する
- 監査目的で全コマンドをログに記録する
- エージェントのライフサイクルイベントでカスタム自動化をトリガーする
- コアコードを変更せずに OpenClaw の挙動を拡張する

## はじめに

### バンドルされたフック

OpenClaw には、自動的に検出される 4 つのバンドル済みフックが同梱されています。

- **💾 session-memory**: `/new` を発行したときに、セッションコンテキストをエージェントワークスペース（デフォルトは `~/.openclaw/workspace/memory/`）へ保存します
- **📝 command-logger**: すべてのコマンドイベントを `~/.openclaw/logs/commands.log` にログします
- **🚀 boot-md**: ゲートウェイ起動時に `BOOT.md` を実行します（内部フックを有効化する必要があります）
- **😈 soul-evil**: パージウィンドウ中、またはランダムな確率で、注入された `SOUL.md` コンテンツを `SOUL_EVIL.md` と差し替えます

利用可能なフックを一覧表示:

```bash
openclaw hooks list
```

フックを有効化:

```bash
openclaw hooks enable session-memory
```

フックの状態を確認:

```bash
openclaw hooks check
```

詳細情報を取得:

```bash
openclaw hooks info session-memory
```

### オンボーディング

オンボーディング (`openclawオンボード` ) 中に、推奨フックを有効にするよう求められます。 ウィザードは対象フックを自動的に検出し、選択対象フックを表示します。

## フックの検出

Hooks は、次の 3 つのディレクトリから自動的に検出されます（優先順位順）。

1. **Workspace hooks**: `<workspace>/hooks/`（エージェントごと、最優先）
2. **Managed hooks**: `~/.openclaw/hooks/`（ユーザーがインストール、ワークスペース間で共有）
3. **Bundled hooks**: `<openclaw>/dist/hooks/bundled/`（OpenClaw に同梱）

管理フックのディレクトリは、**単一フック** または **フックパック**（パッケージディレクトリ）のいずれかです。

各フックは、次を含むディレクトリです。

```
my-hook/
├── HOOK.md          # Metadata + documentation
└── handler.ts       # Handler implementation
```

## Hook Packs（npm/archives）

フックパックは標準的な npm パッケージで、`package.json` 内の `openclaw.hooks` を通じて 1 つ以上のフックをエクスポートします。インストール方法: 以下でインストールします。

```bash
openclaw hooks install <path-or-spec>
```

`package.json` の例:

```json
{
  "name": "@acme/my-hooks",
  "version": "0.1.0",
  "openclaw": {
    "hooks": ["./hooks/my-hook", "./hooks/other-hook"]
  }
}
```

各エントリは、`HOOK.md` と `handler.ts`（または `index.ts`）を含むフックディレクトリを指します。フックパックは依存関係を同梱でき、それらは `~/.openclaw/hooks/<id>` 配下にインストールされます。
フックパックは `~/.openclaw/hooks/<id> ` の下にインストールされます。

## Hook Structure

### HOOK.md Format

`HOOK.md` ファイルには、YAML フロントマターのメタデータと Markdown ドキュメントが含まれます。

```markdown
---
name: my-hook
description: "Short description of what this hook does"
homepage: https://docs.openclaw.ai/hooks#my-hook
metadata:
  { "openclaw": { "emoji": "🔗", "events": ["command:new"], "requires": { "bins": ["node"] } } }
---

# My Hook

Detailed documentation goes here...

## What It Does

- Listens for `/new` commands
- Performs some action
- Logs the result

## Requirements

- Node.js must be installed

## Configuration

No configuration needed.
```

### Metadata Fields

`metadata.openclaw` オブジェクトは次をサポートします。

- **`emoji`**: CLI 用の表示絵文字（例: `"💾"`）
- **`events`**: リッスンするイベントの配列（例: `["command:new", "command:reset"]`）
- **`export`**: 使用する名前付きエクスポート（デフォルトは `"default"`）
- **`homepage`**: ドキュメント URL
- **`requires`**: オプションの要件
  - **`bins`**: PATH 上に必要なバイナリ（例: `["git", "node"]`）
  - **`anyBins`**: これらのバイナリのうち少なくとも 1 つが必要
  - **`env`**: 必要な環境変数
  - **`config`**: 必要な設定パス（例: `["workspace.dir"]`）
  - **`os`**: 必要なプラットフォーム（例: `["darwin", "linux"]`）
- **`always`**: 適格性チェックをバイパス（boolean）
- **`install`**: インストール方法（バンドル済みフックの場合: `[{"id":"bundled","kind":"bundled"}]`）

### Handler Implementation

`handler.ts` ファイルは、`HookHandler` 関数をエクスポートします。

```typescript
import type { HookHandler } from "../../src/hooks/hooks.js";

const myHandler: HookHandler = async (event) => {
  // Only trigger on 'new' command
  if (event.type !== "command" || event.action !== "new") {
    return;
  }

  console.log(`[my-hook] New command triggered`);
  console.log(`  Session: ${event.sessionKey}`);
  console.log(`  Timestamp: ${event.timestamp.toISOString()}`);

  // Your custom logic here

  // Optionally send message to user
  event.messages.push("✨ My hook executed!");
};

export default myHandler;
```

#### Event Context

各イベントには次が含まれます。

```typescript
{
  type: 'command' | 'session' | 'agent' | 'gateway',
  action: string,              // e.g., 'new', 'reset', 'stop'
  sessionKey: string,          // Session identifier
  timestamp: Date,             // When the event occurred
  messages: string[],          // Push messages here to send to user
  context: {
    sessionEntry?: SessionEntry,
    sessionId?: string,
    sessionFile?: string,
    commandSource?: string,    // e.g., 'whatsapp', 'telegram'
    senderId?: string,
    workspaceDir?: string,
    bootstrapFiles?: WorkspaceBootstrapFile[],
    cfg?: OpenClawConfig
  }
}
```

## Event Types

### Command Events

エージェントコマンドが発行されたときにトリガーされます。

- **`command`**: すべてのコマンドイベント（汎用リスナー）
- **`command:new`**: `/new` コマンドが発行されたとき
- **`command:reset`**: `/reset` コマンドが発行されたとき
- **`command:stop`**: `/stop` コマンドが発行されたとき

### Agent Events

- **`agent:bootstrap`**: ワークスペースのブートストラップファイルが注入される前（フックは `context.bootstrapFiles` を変更できます）

### Gateway Events

ゲートウェイ起動時にトリガーされます。

- **`gateway:startup`**: チャンネル起動後、フックがロードされた後

### Tool Result Hooks（Plugin API）

これらのフックはイベントストリームのリスナーではありません。OpenClaw がツール結果を永続化する前に、プラグインが同期的に結果を調整できます。

- **`tool_result_persist`**: セッショントランスクリプトに書き込む前にツールの結果を変換します。 同期する必要があります。更新されたツール結果ペイロードを返すか、そのままにするために `undefined` を返します。 [Agent Loop](/concepts/agent-loop)を参照してください。

### Future Events

将来予定されているイベントタイプ:

- **`session:start`**: 新しいセッションが開始されたとき
- **`session:end`**: セッションが終了したとき
- **`agent:error`**: エージェントがエラーに遭遇したとき
- **`message:sent`**: メッセージが送信されたとき
- **`message:received`**: メッセージが受信されたとき

## Creating Custom Hooks

### 1. Choose Location

- **Workspace hooks**（`<workspace>/hooks/`）: エージェントごと、最優先
- **Managed hooks**（`~/.openclaw/hooks/`）: ワークスペース間で共有

### 2. Create Directory Structure

```bash
mkdir -p ~/.openclaw/hooks/my-hook
cd ~/.openclaw/hooks/my-hook
```

### 3. Create HOOK.md

```markdown
---
name: my-hook
description: "Does something useful"
metadata: { "openclaw": { "emoji": "🎯", "events": ["command:new"] } }
---

# My Custom Hook

This hook does something useful when you issue `/new`.
```

### 4. Create handler.ts

```typescript
import type { HookHandler } from "../../src/hooks/hooks.js";

const handler: HookHandler = async (event) => {
  if (event.type !== "command" || event.action !== "new") {
    return;
  }

  console.log("[my-hook] Running!");
  // Your logic here
};

export default handler;
```

### 5. Enable and Test

```bash
# Verify hook is discovered
openclaw hooks list

# Enable it
openclaw hooks enable my-hook

# Restart your gateway process (menu bar app restart on macOS, or restart your dev process)

# Trigger the event
# Send /new via your messaging channel
```

## Configuration

### New Config Format（推奨）

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "session-memory": { "enabled": true },
        "command-logger": { "enabled": false }
      }
    }
  }
}
```

### Per-Hook Configuration

Hooks にはカスタム設定を持たせることができます。

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "my-hook": {
          "enabled": true,
          "env": {
            "MY_CUSTOM_VAR": "value"
          }
        }
      }
    }
  }
}
```

### Extra Directories

追加のディレクトリからフックを読み込みます。

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "load": {
        "extraDirs": ["/path/to/more/hooks"]
      }
    }
  }
}
```

### Legacy Config Format（引き続きサポート）

後方互換性のため、旧設定フォーマットも引き続き動作します。

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "handlers": [
        {
          "event": "command:new",
          "module": "./hooks/handlers/my-handler.ts",
          "export": "default"
        }
      ]
    }
  }
}
```

**Migration**: 新しいフックには、検出ベースの新しいシステムを使用してください。レガシーハンドラーは、ディレクトリベースのフックの後にロードされます。 従来のハンドラは、ディレクトリベースのフックの後にロードされます。

## CLI Commands

### List Hooks

```bash
# List all hooks
openclaw hooks list

# Show only eligible hooks
openclaw hooks list --eligible

# Verbose output (show missing requirements)
openclaw hooks list --verbose

# JSON output
openclaw hooks list --json
```

### Hook Information

```bash
# Show detailed info about a hook
openclaw hooks info session-memory

# JSON output
openclaw hooks info session-memory --json
```

### Check Eligibility

```bash
# Show eligibility summary
openclaw hooks check

# JSON output
openclaw hooks check --json
```

### Enable/Disable

```bash
# Enable a hook
openclaw hooks enable session-memory

# Disable a hook
openclaw hooks disable command-logger
```

## Bundled hook reference

### session-memory

`/new` を発行したときに、セッションコンテキストをメモリに保存します。

**Events**: `command:new`

**Requirements**: `workspace.dir` が設定されている必要があります

**Output**: `<workspace>/memory/YYYY-MM-DD-slug.md`（デフォルトは `~/.openclaw/workspace`）

**What it does**:

1. リセット前のセッションエントリを使用して、正しいトランスクリプトを特定します
2. 会話の最後の 15 行を抽出します
3. LLM を使用して、説明的なファイル名スラッグを生成します
4. セッションメタデータを日付付きのメモリファイルに保存します

**Example output**:

```markdown
# Session: 2026-01-16 14:30:00 UTC

- **Session Key**: agent:main:main
- **Session ID**: abc123def456
- **Source**: telegram
```

**Filename examples**:

- `2026-01-16-vendor-pitch.md`
- `2026-01-16-api-design.md`
- `2026-01-16-1430.md`（スラッグ生成に失敗した場合のフォールバックタイムスタンプ）

**Enable**:

```bash
openclaw hooks enable session-memory
```

### command-logger

すべてのコマンドイベントを集中管理された監査ファイルにログします。

**Events**: `command`

**Requirements**: なし

**Output**: `~/.openclaw/logs/commands.log`

**What it does**:

1. イベント詳細（コマンドアクション、タイムスタンプ、セッションキー、送信者 ID、ソース）を取得します
2. JSONL 形式でログファイルに追記します
3. バックグラウンドで静かに実行されます

**Example log entries**:

```jsonl
{"timestamp":"2026-01-16T14:30:00.000Z","action":"new","sessionKey":"agent:main:main","senderId":"+1234567890","source":"telegram"}
{"timestamp":"2026-01-16T15:45:22.000Z","action":"stop","sessionKey":"agent:main:main","senderId":"user@example.com","source":"whatsapp"}
```

**View logs**:

```bash
# View recent commands
tail -n 20 ~/.openclaw/logs/commands.log

# Pretty-print with jq
cat ~/.openclaw/logs/commands.log | jq .

# Filter by action
grep '"action":"new"' ~/.openclaw/logs/commands.log | jq .
```

**Enable**:

```bash
openclaw hooks enable command-logger
```

### soul-evil

パージウィンドウ中、またはランダムな確率で、注入された `SOUL.md` コンテンツを `SOUL_EVIL.md` と差し替えます。

**Events**: `agent:bootstrap`

**Docs**: [SOUL Evil Hook](/hooks/soul-evil)

**Output**: ファイルは書き込まれません。差し替えはメモリ内のみで行われます。

**Enable**:

```bash
openclaw hooks enable soul-evil
```

**Config**:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "soul-evil": {
          "enabled": true,
          "file": "SOUL_EVIL.md",
          "chance": 0.1,
          "purge": { "at": "21:00", "duration": "15m" }
        }
      }
    }
  }
}
```

### boot-md

ゲートウェイ起動時（チャンネル起動後）に `BOOT.md` を実行します。
これを実行するには、内部フックを有効化する必要があります。
これを実行するには内部フックを有効にする必要があります。

**Events**: `gateway:startup`

**Requirements**: `workspace.dir` が設定されている必要があります

**What it does**:

1. ワークスペースから `BOOT.md` を読み込みます
2. エージェントランナー経由で指示を実行します
3. 要求された送信メッセージを message ツール経由で送信します

**Enable**:

```bash
openclaw hooks enable boot-md
```

## Best Practices

### Keep Handlers Fast

フックはコマンド処理中に実行されます。 体重を軽く保つ:

```typescript
// ✓ Good - async work, returns immediately
const handler: HookHandler = async (event) => {
  void processInBackground(event); // Fire and forget
};

// ✗ Bad - blocks command processing
const handler: HookHandler = async (event) => {
  await slowDatabaseQuery(event);
  await evenSlowerAPICall(event);
};
```

### Handle Errors Gracefully

リスクのある操作は必ずラップしてください。

```typescript
const handler: HookHandler = async (event) => {
  try {
    await riskyOperation(event);
  } catch (err) {
    console.error("[my-handler] Failed:", err instanceof Error ? err.message : String(err));
    // Don't throw - let other handlers run
  }
};
```

### Filter Events Early

イベントが関係ない場合は、早期に return してください。

```typescript
const handler: HookHandler = async (event) => {
  // Only handle 'new' commands
  if (event.type !== "command" || event.action !== "new") {
    return;
  }

  // Your logic here
};
```

### Use Specific Event Keys

可能な限り、メタデータで具体的なイベントを指定してください。

```yaml
metadata: { "openclaw": { "events": ["command:new"] } } # Specific
```

次のようにするのではなく:

```yaml
metadata: { "openclaw": { "events": ["command"] } } # General - more overhead
```

## Debugging

### Enable Hook Logging

ゲートウェイは起動時にフックのロードをログします。

```
Registered hook: session-memory -> command:new
Registered hook: command-logger -> command
Registered hook: boot-md -> gateway:startup
```

### Check Discovery

検出されたすべてのフックを一覧表示します。

```bash
openclaw hooks list --verbose
```

### Check Registration

ハンドラー内で、呼び出されたときにログを出力します。

```typescript
const handler: HookHandler = async (event) => {
  console.log("[my-handler] Triggered:", event.type, event.action);
  // Your logic
};
```

### Verify Eligibility

フックが有効でない理由を確認してください:

```bash
openclaw hooks info my-hook
```

出力内の不足している要件を確認してください。

## Testing

### Gateway Logs

フックの実行を確認するために、ゲートウェイログを監視します。

```bash
# macOS
./scripts/clawlog.sh -f

# Other platforms
tail -f ~/.openclaw/gateway.log
```

### Test Hooks Directly

ハンドラーを単体でテストします。

```typescript
import { test } from "vitest";
import { createHookEvent } from "./src/hooks/hooks.js";
import myHandler from "./hooks/my-hook/handler.js";

test("my handler works", async () => {
  const event = createHookEvent("command", "new", "test-session", {
    foo: "bar",
  });

  await myHandler(event);

  // Assert side effects
});
```

## Architecture

### Core Components

- **`src/hooks/types.ts`**: 型定義
- **`src/hooks/workspace.ts`**: ディレクトリスキャンとロード
- **`src/hooks/frontmatter.ts`**: HOOK.md メタデータのパース
- **`src/hooks/config.ts`**: 適格性チェック
- **`src/hooks/hooks-status.ts`**: ステータスレポート
- **`src/hooks/loader.ts`**: 動的モジュールローダー
- **`src/cli/hooks-cli.ts`**: CLI コマンド
- **`src/gateway/server-startup.ts`**: ゲートウェイ起動時にフックをロード
- **`src/auto-reply/reply/commands-core.ts`**: コマンドイベントをトリガー

### Discovery Flow

```
Gateway startup
    ↓
Scan directories (workspace → managed → bundled)
    ↓
Parse HOOK.md files
    ↓
Check eligibility (bins, env, config, os)
    ↓
Load handlers from eligible hooks
    ↓
Register handlers for events
```

### Event Flow

```
User sends /new
    ↓
Command validation
    ↓
Create hook event
    ↓
Trigger hook (all registered handlers)
    ↓
Command processing continues
    ↓
Session reset
```

## Troubleshooting

### Hook Not Discovered

1. ディレクトリ構造を確認します。

   ```bash
   ls -la ~/.openclaw/hooks/my-hook/
   # Should show: HOOK.md, handler.ts
   ```

2. HOOK.md の形式を確認します。

   ```bash
   cat ~/.openclaw/hooks/my-hook/HOOK.md
   # Should have YAML frontmatter with name and metadata
   ```

3. 検出されたすべてのフックを一覧表示します。

   ```bash
   openclaw hooks list
   ```

### Hook Not Eligible

要件を確認します。

```bash
openclaw hooks info my-hook
```

不足しているものを探してください:

- バイナリ（PATH を確認）
- 環境変数
- 設定値
- OS 互換性

### Hook Not Executing

1. フックが有効化されていることを確認します。

   ```bash
   openclaw hooks list
   # Should show ✓ next to enabled hooks
   ```

2. フックが再ロードされるよう、ゲートウェイプロセスを再起動します。

3. エラーがないか、ゲートウェイログを確認します。

   ```bash
   ./scripts/clawlog.sh | grep hook
   ```

### Handler Errors

TypeScript や import エラーがないか確認します。

```bash
# Test import directly
node -e "import('./path/to/handler.ts').then(console.log)"
```

## Migration Guide

### From Legacy Config to Discovery

**Before**:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "handlers": [
        {
          "event": "command:new",
          "module": "./hooks/handlers/my-handler.ts"
        }
      ]
    }
  }
}
```

**After**:

1. フックディレクトリを作成します。

   ```bash
   mkdir -p ~/.openclaw/hooks/my-hook
   mv ./hooks/handlers/my-handler.ts ~/.openclaw/hooks/my-hook/handler.ts
   ```

2. HOOK.md を作成します。

   ```markdown
   ---
   name: my-hook
   description: "My custom hook"
   metadata: { "openclaw": { "emoji": "🎯", "events": ["command:new"] } }
   ---

   # My Hook

   Does something useful.
   ```

3. 設定を更新します。

   ```json
   {
     "hooks": {
       "internal": {
         "enabled": true,
         "entries": {
           "my-hook": { "enabled": true }
         }
       }
     }
   }
   ```

4. 確認後、ゲートウェイプロセスを再起動します。

   ```bash
   openclaw hooks list
   # Should show: 🎯 my-hook ✓
   ```

**Benefits of migration**:

- 自動検出
- CLI 管理
- 適格性チェック
- より良いドキュメント
- 一貫した構造

## See Also

- [CLI Reference: hooks](/cli/hooks)
- [Bundled Hooks README](https://github.com/openclaw/openclaw/tree/main/src/hooks/bundled)
- [Webhook Hooks](/automation/webhook)
- [Configuration](/gateway/configuration#hooks)
