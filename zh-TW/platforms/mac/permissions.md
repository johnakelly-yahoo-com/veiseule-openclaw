---
summary: "macOS 權限持久化（TCC）與簽署需求"
read_when:
  - 偵錯 macOS 權限提示遺失或卡住
  - 封裝或簽署 macOS 應用程式
  - 變更套件識別碼或應用程式安裝路徑
title: "macOS 權限"
---

# macOS 權限（TCC）

macOS 的權限授予機制相當脆弱。TCC 會將權限授予與以下項目關聯：
app's code signature, bundle identifier, and on-disk path. If any of those change,
macOS treats the app as new and may drop or hide prompts.

## 穩定權限的需求

- 相同路徑：從固定位置執行應用程式（對於 OpenClaw，`dist/OpenClaw.app`）。
- 相同套件識別碼：變更套件 ID 會建立新的權限身分。
- 已簽署的應用程式：未簽署或 ad-hoc 簽署的組建不會持久保存權限。
- 一致的簽章：使用真正的 Apple Development 或 Developer ID 憑證，
  以確保在重新組建之間簽章保持穩定。

Ad-hoc 簽章每次建置都會產生新的身分識別。macOS 會忘記先前的
grants, and prompts can disappear entirely until the stale entries are cleared.

## 當提示消失時的復原檢查清單

1. 退出應用程式。
2. 在「系統設定」->「隱私權與安全性」中移除應用程式項目。
3. 從相同路徑重新啟動應用程式並重新授與權限。
4. 若提示仍未出現，使用 `tccutil` 重設 TCC 項目後再試一次。
5. 有些權限只有在完整重新啟動 macOS 後才會再次出現。

重設範例（請視需要替換套件識別碼）：

```bash
sudo tccutil reset Accessibility bot.molt.mac
sudo tccutil reset ScreenCapture bot.molt.mac
sudo tccutil reset AppleEvents
```

## 檔案與資料夾權限（桌面／文件／下載）

macOS 也可能會對終端機／背景程序限制 Desktop、Documents 和 Downloads 的存取權限。若檔案讀取或目錄列表操作發生卡頓，請為執行檔案操作的相同程序情境授予存取權（例如 Terminal／iTerm、由 LaunchAgent 啟動的應用程式，或 SSH 程序）。

因應方式：若想避免逐資料夾授權，可將檔案移至 OpenClaw 工作區（`~/.openclaw/workspace`）。

如果您正在測試權限，請務必使用真實憑證進行簽章。Ad-hoc
builds are only acceptable for quick local runs where permissions do not matter.
