---
title: "環境變數"
---

# 環境變數

OpenClaw 會從多個來源讀取環境變數。規則是 **絕不覆寫既有值**。

## 優先順序（最高 → 最低）

1. **程序環境**（Gateway 閘道器 程序已從父層 shell／daemon 取得的值）。
2. **目前工作目錄中的 `.env`**（dotenv 預設；不會覆寫）。
3. **位於 `~/.openclaw/.env` 的全域 `.env`**（亦稱為 `$OPENCLAW_STATE_DIR/.env`；不會覆寫）。
4. **`~/.openclaw/openclaw.json` 中的設定 `env` 區塊**（僅在缺少時套用）。
5. **選用的登入 shell 匯入**（`env.shellEnv.enabled` 或 `OPENCLAW_LOAD_SHELL_ENV=1`），僅針對缺少的預期金鑰套用。

如果設定檔完全缺失，則會跳過步驟 4；若啟用，Shell 匯入仍會執行。

## 設定檔 `env` 區塊

設定內嵌環境變數的兩種等效方式（兩者皆不會覆寫）：

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: {
      GROQ_API_KEY: "gsk-...",
    },
  },
}
```

## Shell 環境匯入

`env.shellEnv` 會執行你的登入 shell，並僅匯入 **缺少** 的預期金鑰：

```json5
{
  env: {
    shellEnv: {
      enabled: true,
      timeoutMs: 15000,
    },
  },
}
```

環境變數對應：

- `OPENCLAW_LOAD_SHELL_ENV=1`
- `OPENCLAW_SHELL_ENV_TIMEOUT_MS=15000`

## 設定檔中的環境變數替換

你可以在設定的字串值中，使用 `${VAR_NAME}` 語法直接參考環境變數：

```json5
{
  models: {
    providers: {
      "vercel-gateway": {
        apiKey: "${VERCEL_GATEWAY_API_KEY}",
      },
    },
  },
}
```

完整細節請參閱［Configuration: Env var substitution］(/gateway/configuration#env-var-substitution-in-config)。

## 與路徑相關的環境變數

| 變數                     | 目的                                                                                                                                                                                                                                  |
| ---------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `OPENCLAW_HOME`        | 覆寫用於所有內部路徑解析的主目錄（`~/.openclaw/`、agent 目錄、sessions、credentials）。Useful when running OpenClaw as a dedicated service user. |
| `OPENCLAW_STATE_DIR`   | 覆寫狀態目錄（預設為 `~/.openclaw`）。                                                                                                                                            |
| `OPENCLAW_CONFIG_PATH` | 1. 覆寫設定檔路徑（預設為 `~/.openclaw/openclaw.json`）。                                                                                                                                                                 |

### 2. `OPENCLAW_HOME`

3. 設定後，`OPENCLAW_HOME` 會取代系統家目錄（`$HOME` / `os.homedir()`）以進行所有內部路徑解析。 4. 這可為無頭服務帳號啟用完整的檔案系統隔離。

5. **優先順序：** `OPENCLAW_HOME` > `$HOME` > `USERPROFILE` > `os.homedir()`

6. **範例**（macOS LaunchDaemon）：

```xml
7. <key>EnvironmentVariables</key>
<dict>
  <key>OPENCLAW_HOME</key>
  <string>/Users/kira</string>
</dict>
```

8. `OPENCLAW_HOME` 也可以設定為波浪號路徑（例如 `~/svc`），在使用前會先以 `$HOME` 展開。

## 12. 相關

- [Gateway 設定](/gateway/configuration)
- [FAQ：環境變數與 .env 載入](/help/faq#env-vars-and-env-loading)
- [模型總覽](/concepts/models)


