---
summary: "「openclaw plugins」（清單、安裝、啟用／停用、doctor）的 CLI 參考"
read_when:
  - 2. 你想要安裝或管理進程內的 Gateway 外掛程式
  - 3. 你想要除錯外掛程式載入失敗
title: "plugins"
---

# `openclaw plugins`

4. 管理 Gateway 外掛程式／擴充（以進程內方式載入）。

5. 相關：

- 外掛系統：[Plugins](/tools/plugin)
- 外掛程式資訊清單 + 結構描述：[Plugin manifest](/plugins/manifest)
- 安全性強化：[Security](/gateway/security)

## 指令

```bash
openclaw plugins list
openclaw plugins info <id>
openclaw plugins enable <id>
openclaw plugins disable <id>
openclaw plugins doctor
openclaw plugins update <id>
openclaw plugins update --all
```

隨 OpenClaw 內建的外掛程式會出貨，但預設為停用。 8. 使用 `plugins enable` 來
啟用它們。

所有外掛都必須提供一個 `openclaw.plugin.json` 檔案，並包含內嵌的 JSON Schema
（`configSchema`，即使為空）。缺少或無效的資訊清單或結構描述，會導致
外掛無法載入，且設定驗證失敗。 9. 缺少／無效的資訊清單或結構描述會阻止
外掛程式載入並導致設定驗證失敗。

### Install

```bash
openclaw plugins install <path-or-spec>
```

安全性注意事項：將外掛程式安裝視同執行程式碼。 11. 優先使用固定（pinned）的版本。

Npm 規格僅限 **registry-only**（套件名稱 + 選填版本/標籤）。 Git/URL/file
specs 會被拒絕。 為了安全起見，安裝相依套件時會使用 `--ignore-scripts`。

支援的封存格式：`.zip`、`.tgz`、`.tar.gz`、`.tar`。

使用 `--link` 可避免複製本機目錄（會加入至 `plugins.load.paths`）：

```bash
openclaw plugins install -l ./my-plugin
```

### 解除安裝

```bash
管理 Gateway 外掛程式／擴充（以進程內方式載入）。
```

`uninstall` 會從 `plugins.entries`、`plugins.installs`、
外掛 allowlist，以及在適用時的 `plugins.load.paths` 連結項目中移除外掛紀錄。
對於啟用中的記憶體外掛，memory 插槽會重設為 `memory-core`。

預設情況下，解除安裝也會移除位於目前
state 目錄 extensions 根目錄下的外掛安裝目錄（`$OPENCLAW_STATE_DIR/extensions/<id>`）。 使用
`--keep-files` 以保留磁碟上的檔案。

支援 `--keep-config` 作為已棄用的 `--keep-files` 別名。

### Update

```bash
openclaw plugins update <id>
openclaw plugins update --all
openclaw plugins update <id> --dry-run
```

更新僅適用於從 npm 安裝的外掛（追蹤於 `plugins.installs`）。
