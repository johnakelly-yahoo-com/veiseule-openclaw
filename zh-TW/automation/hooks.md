---
summary: "Hooks：用於指令與生命週期事件的事件驅動自動化"
read_when:
  - 當你需要針對 /new、/reset、/stop 以及代理程式生命週期事件進行事件驅動自動化時
  - 你想要建置、安裝或除錯 hooks
title: "鉤子"
---

# 鉤子

Hooks 提供可擴充的事件驅動系統，用於回應代理指令與事件來自動化動作。 Hooks 會從目錄中自動探索，並可透過 CLI 指令管理，方式類似 OpenClaw 中的 skills。

## 入門引導

Hooks 是在事件發生時執行的小型腳本。 有兩種類型：

- **Hooks**（本頁）：在 Gateway 閘道器 內執行，當代理程式事件觸發時運作，例如 `/new`、`/reset`、`/stop`，或生命週期事件。
- **Webhooks**：外部的 HTTP webhooks，讓其他系統觸發 OpenClaw 中的工作。 **Webhooks**：外部 HTTP webhook，讓其他系統在 OpenClaw 中觸發工作。請參閱 [Webhook Hooks](/automation/webhook)，或使用 `openclaw webhooks` 來進行 Gmail 輔助指令。

Hooks 也可以打包在外掛中；請參閱 [Plugins](/tools/plugin#plugin-hooks)。

常見用途：

- 在重置工作階段時儲存記憶快照
- 為疑難排解或合規需求保留指令稽核紀錄
- 在工作階段開始或結束時觸發後續自動化
- 在事件發生時，將檔案寫入代理程式工作區或呼叫外部 API

如果你能寫一個小型的 TypeScript 函式，你就能寫一個 hook。 Hooks 會自動被探索，你可以透過 CLI 啟用或停用它們。

## 概覽

Hooks 系統可讓你：

- 在發出 `/new` 時將工作階段上下文儲存到記憶中
- 記錄所有指令以供稽核
- 在代理程式生命週期事件上觸發自訂自動化
- 不需修改核心程式碼即可擴充 OpenClaw 的行為

## 入門指南

### 內建 Hooks

OpenClaw 隨附四個會自動被探索的內建 hooks：

- **💾 session-memory**：在你發出 `/new` 時，將工作階段內容儲存至你的代理程式工作區（預設為 `~/.openclaw/workspace/memory/`）
- **📝 command-logger**：將所有指令事件記錄到 `~/.openclaw/logs/commands.log`
- **🚀 boot-md**：在 Gateway 閘道器 啟動時執行 `BOOT.md`（需要啟用內部 hooks）
- **😈 soul-evil**：在清除期間或隨機情況下，將注入的 `SOUL.md` 內容替換為 `SOUL_EVIL.md`

列出可用 hooks：

```bash
openclaw hooks list
```

啟用 hook：

```bash
openclaw hooks enable session-memory
```

檢查 hook 狀態：

```bash
openclaw hooks check
```

取得詳細資訊：

```bash
openclaw hooks info session-memory
```

### 新手導覽

在新手導覽（`openclaw onboard`）期間，系統會提示你啟用建議的 hooks。 The wizard automatically discovers eligible hooks and presents them for selection.

## Hook 探索機制

Hooks 會從三個目錄中自動探索（依優先順序）：

1. **工作區 hooks**：`<workspace>/hooks/`（每個代理程式一組，最高優先順序）
2. **受管理 hooks**：`~/.openclaw/hooks/`（使用者安裝，跨工作區共用）
3. **內建 hooks**：`<openclaw>/dist/hooks/bundled/`（隨 OpenClaw 一同提供）

受管理的 hook 目錄可以是 **單一 hook** 或 **hook 套件**（套件目錄）。

每個 hook 都是一個目錄，內含：

```
my-hook/
├── HOOK.md          # Metadata + documentation
└── handler.ts       # Handler implementation
```

## Hook 套件（npm／封存檔）

Hook 套件是標準的 npm 套件，透過 `package.json` 中的 `openclaw.hooks` 匯出一個或多個 hooks。使用以下指令安裝： Install them with:

```bash
openclaw hooks install <path-or-spec>
```

`package.json` 範例：

```json
{
  "name": "@acme/my-hooks",
  "version": "0.1.0",
  "openclaw": {
    "hooks": ["./hooks/my-hook", "./hooks/other-hook"]
  }
}
```

每個項目都指向一個 hook 目錄，該目錄包含 `HOOK.md` 與 `handler.ts`（或 `index.ts`）。
Hook 套件可以攜帶相依套件；它們會被安裝在 `~/.openclaw/hooks/<id>` 下。
Hook packs can ship dependencies; they will be installed under `~/.openclaw/hooks/<id>`.

## 架構

### HOOK.md 格式

`HOOK.md` 檔案包含 YAML frontmatter 的中繼資料，以及 Markdown 文件說明：

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

`metadata.openclaw` 物件支援：

- **`emoji`**：CLI 顯示用的表情符號（例如 `"💾"`）
- **`events`**：要監聽的事件陣列（例如 `["command:new", "command:reset"]`）
- **`export`**：要使用的具名匯出（預設為 `"default"`）
- **`homepage`**：文件 URL
- **`requires`**：選用需求
  - **`bins`**：PATH 中需要的二進位檔（例如 `["git", "node"]`）
  - **`anyBins`**：至少必須存在其中一個二進位檔
  - **`env`**：必要的環境變數
  - **`config`**：必要的設定路徑（例如 `["workspace.dir"]`）
  - **`os`**：必要的平台（例如 `["darwin", "linux"]`）
- **`always`**：略過資格檢查（布林值）
- **`install`**：安裝方式（對於內建 hooks：`[{"id":"bundled","kind":"bundled"}]`）

### Handler Implementation

`handler.ts` 檔案會匯出一個 `HookHandler` 函式：

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

每個事件包含：

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

## 事件類型

### 指令事件

當代理程式指令被發出時觸發：

- **`command`**：所有指令事件（通用監聽器）
- **`command:new`**：當發出 `/new` 指令時
- **`command:reset`**：當發出 `/reset` 指令時
- **`command:stop`**：當發出 `/stop` 指令時

### 代理程式事件

- **`agent:bootstrap`**：在工作區啟動檔案被注入之前（hooks 可能會修改 `context.bootstrapFiles`）

### Gateway 事件

在 Gateway 閘道器 啟動時觸發：

- **`gateway:startup`**：在頻道啟動且 hooks 載入完成之後

### 工具結果 Hooks（外掛 API）

這些 hooks 並非事件串流監聽器；它們允許外掛在 OpenClaw 儲存結果之前，同步調整工具結果。

- **`tool_result_persist`**: transform tool results before they are written to the session transcript. Must be synchronous; return the updated tool result payload or `undefined` to keep it as-is. See [Agent Loop](/concepts/agent-loop).

### 未來事件

規劃中的事件類型：

- **`session:start`**：當新的工作階段開始時
- **`session:end`**：當工作階段結束時
- **`agent:error`**：當代理程式遇到錯誤時
- **`message:sent`**：當訊息被送出時
- **`message:received`**：當訊息被接收時

## 建立自訂 Hooks

### 1. 選擇位置

- **工作區 hooks**（`<workspace>/hooks/`）：每個代理程式一組，最高優先順序
- **受管理 hooks**（`~/.openclaw/hooks/`）：跨工作區共用

### 2. 建立目錄結構

```bash
mkdir -p ~/.openclaw/hooks/my-hook
cd ~/.openclaw/hooks/my-hook
```

### 3. 建立 HOOK.md

```markdown
---
name: my-hook
description: "Does something useful"
metadata: { "openclaw": { "emoji": "🎯", "events": ["command:new"] } }
---

# My Custom Hook

This hook does something useful when you issue `/new`.
```

### 4. 建立 handler.ts

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

### 5. 啟用並測試

```bash
# Verify hook is discovered
openclaw hooks list

# Enable it
openclaw hooks enable my-hook

# Restart your gateway process (menu bar app restart on macOS, or restart your dev process)

# Trigger the event
# Send /new via your messaging channel
```

## 設定

### 新的設定格式（建議）

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

### 每個 Hook 的設定

Hooks 可以有自訂設定：

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

Load hooks from additional directories:

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

### 舊版設定格式（仍支援）

舊的設定格式仍可使用，以維持向後相容：

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

**Migration**: Use the new discovery-based system for new hooks. Legacy handlers are loaded after directory-based hooks.

## CLI 指令

### 列出 Hooks

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

### Hook 資訊

```bash
# Show detailed info about a hook
openclaw hooks info session-memory

# JSON output
openclaw hooks info session-memory --json
```

### 檢查資格

```bash
# Show eligibility summary
openclaw hooks check

# JSON output
openclaw hooks check --json
```

### 啟用／停用

```bash
# Enable a hook
openclaw hooks enable session-memory

# Disable a hook
openclaw hooks disable command-logger
```

## Bundled hook reference

### session-memory

Saves session context to memory when you issue `/new`.

**事件**：`command:new`

**需求**：必須設定 `workspace.dir`

**輸出**：`<workspace>/memory/YYYY-MM-DD-slug.md`（預設為 `~/.openclaw/workspace`）

**功能說明**：

1. Uses the pre-reset session entry to locate the correct transcript
2. 擷取最後 15 行對話
3. 使用 LLM 產生具描述性的檔名 slug
4. Saves session metadata to a dated memory file

**輸出範例**：

```markdown
# Session: 2026-01-16 14:30:00 UTC

- **Session Key**: agent:main:main
- **Session ID**: abc123def456
- **Source**: telegram
```

**檔名範例**：

- `2026-01-16-vendor-pitch.md`
- `2026-01-16-api-design.md`
- `2026-01-16-1430.md`（若 slug 產生失敗時的後備時間戳）

**啟用**：

```bash
openclaw hooks enable session-memory
```

### command-logger

將所有指令事件記錄到集中式稽核檔案。

**事件**：`command`

**需求**：無

**輸出**：`~/.openclaw/logs/commands.log`

**功能說明**：

1. Captures event details (command action, timestamp, session key, sender ID, source)
2. 以 JSONL 格式附加至記錄檔
3. 在背景中靜默執行

**記錄範例**：

```jsonl
{"timestamp":"2026-01-16T14:30:00.000Z","action":"new","sessionKey":"agent:main:main","senderId":"+1234567890","source":"telegram"}
{"timestamp":"2026-01-16T15:45:22.000Z","action":"stop","sessionKey":"agent:main:main","senderId":"user@example.com","source":"whatsapp"}
```

**檢視記錄**：

```bash
# View recent commands
tail -n 20 ~/.openclaw/logs/commands.log

# Pretty-print with jq
cat ~/.openclaw/logs/commands.log | jq .

# Filter by action
grep '"action":"new"' ~/.openclaw/logs/commands.log | jq .
```

**啟用**：

```bash
openclaw hooks enable command-logger
```

### soul-evil

在清除期間或隨機情況下，將注入的 `SOUL.md` 內容替換為 `SOUL_EVIL.md`。

**事件**：`agent:bootstrap`

**文件**：[SOUL Evil Hook](/hooks/soul-evil)

**輸出**：不寫入任何檔案；交換僅在記憶體中進行。

**啟用**：

```bash
openclaw hooks enable soul-evil
```

**設定**：

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

在 Gateway 閘道器 啟動時（頻道啟動後）執行 `BOOT.md`。
必須啟用內部 hooks 才會執行。
Internal hooks must be enabled for this to run.

**事件**：`gateway:startup`

**需求**：必須設定 `workspace.dir`

**功能說明**：

1. 從你的工作區讀取 `BOOT.md`
2. 透過代理程式執行器執行指示
3. 透過訊息工具送出任何要求的對外訊息

**啟用**：

```bash
openclaw hooks enable boot-md
```

## 最佳實務

### 保持處理器快速

Hooks run during command processing. Keep them lightweight:

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

### 妥善處理錯誤

Always wrap risky operations:

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

### 儘早過濾事件

如果事件不相關，請提早返回：

```typescript
const handler: HookHandler = async (event) => {
  // Only handle 'new' commands
  if (event.type !== "command" || event.action !== "new") {
    return;
  }

  // Your logic here
};
```

### 使用明確的事件鍵

盡可能在中繼資料中指定精確的事件：

```yaml
metadata: { "openclaw": { "events": ["command:new"] } } # Specific
```

而不是：

```yaml
metadata: { "openclaw": { "events": ["command"] } } # General - more overhead
```

## Debugging

### 啟用 Hook 記錄

Gateway 閘道器 會在啟動時記錄 hook 載入情況：

```
Registered hook: session-memory -> command:new
Registered hook: command-logger -> command
Registered hook: boot-md -> gateway:startup
```

### 檢查探索結果

列出所有已探索到的 hooks：

```bash
openclaw hooks list --verbose
```

### 檢查註冊狀態

In your handler, log when it's called:

```typescript
const handler: HookHandler = async (event) => {
  console.log("[my-handler] Triggered:", event.type, event.action);
  // Your logic
};
```

### Verify Eligibility

檢查為何 hook 不符合資格：

```bash
openclaw hooks info my-hook
```

Look for missing requirements in the output.

## 測試

### Gateway 記錄

監控 Gateway 閘道器 記錄以查看 hook 的執行情況：

```bash
# macOS
./scripts/clawlog.sh -f

# Other platforms
tail -f ~/.openclaw/gateway.log
```

### 直接測試 Hooks

以隔離方式測試你的處理器：

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

### 核心元件

- **`src/hooks/types.ts`**：型別定義
- **`src/hooks/workspace.ts`**：目錄掃描與載入
- **`src/hooks/frontmatter.ts`**：HOOK.md 中繼資料解析
- **`src/hooks/config.ts`**：資格檢查
- **`src/hooks/hooks-status.ts`**：狀態回報
- **`src/hooks/loader.ts`**：動態模組載入器
- **`src/cli/hooks-cli.ts`**：CLI 指令
- **`src/gateway/server-startup.ts`**：在 Gateway 閘道器 啟動時載入 hooks
- **`src/auto-reply/reply/commands-core.ts`**：觸發指令事件

### 探索流程

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

### 事件流程

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

### Hook 未被探索

1. Check directory structure:

   ```bash
   ls -la ~/.openclaw/hooks/my-hook/
   # Should show: HOOK.md, handler.ts
   ```

2. 驗證 HOOK.md 格式：

   ```bash
   cat ~/.openclaw/hooks/my-hook/HOOK.md
   # Should have YAML frontmatter with name and metadata
   ```

3. 列出所有已探索的 hooks：

   ```bash
   openclaw hooks list
   ```

### Hook 不符合資格

檢查需求：

```bash
openclaw hooks info my-hook
```

Look for missing:

- Binaries (check PATH)
- 環境變數
- 設定值
- 作業系統相容性

### Hook 未執行

1. 確認 hook 已啟用：

   ```bash
   openclaw hooks list
   # Should show ✓ next to enabled hooks
   ```

2. Restart your gateway process so hooks reload.

3. 檢查 Gateway 記錄中的錯誤：

   ```bash
   ./scripts/clawlog.sh | grep hook
   ```

### Handler Errors

檢查是否有 TypeScript／import 錯誤：

```bash
# Test import directly
node -e "import('./path/to/handler.ts').then(console.log)"
```

## 遷移指南

### From Legacy Config to Discovery

**之前**：

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

**之後**：

1. 建立 hook 目錄：

   ```bash
   mkdir -p ~/.openclaw/hooks/my-hook
   mv ./hooks/handlers/my-handler.ts ~/.openclaw/hooks/my-hook/handler.ts
   ```

2. 建立 HOOK.md：

   ```markdown
   ---
   name: my-hook
   description: "My custom hook"
   metadata: { "openclaw": { "emoji": "🎯", "events": ["command:new"] } }
   ---

   # My Hook

   Does something useful.
   ```

3. 更新設定：

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

4. 驗證並重新啟動你的 Gateway 閘道器 行程：

   ```bash
   openclaw hooks list
   # Should show: 🎯 my-hook ✓
   ```

**遷移的好處**：

- 自動探索
- CLI 管理
- Eligibility checking
- 更完善的文件
- Consistent structure

## See Also

- [CLI Reference: hooks](/cli/hooks)
- [Bundled Hooks README](https://github.com/openclaw/openclaw/tree/main/src/hooks/bundled)
- [Webhook Hooks](/automation/webhook)
- [Configuration](/gateway/configuration#hooks)
