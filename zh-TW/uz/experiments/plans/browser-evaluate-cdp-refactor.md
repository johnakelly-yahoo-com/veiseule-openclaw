---
summary: "Plan: 使用 CDP 將 browser act:evaluate 從 Playwright 佇列中隔離，並加入端對端期限控制與更安全的 ref 解析"
owner: "openclaw"
status: "draft"
last_updated: "2026-02-10"
title: "Browser Evaluate CDP Refactor"
---

# Browser Evaluate CDP Refactor Plan

## 背景

`act:evaluate` 會在頁面中執行使用者提供的 JavaScript。 目前它是透過 Playwright
（`page.evaluate` 或 `locator.evaluate`）執行。 Playwright 會對每個頁面的 CDP 指令進行序列化，因此
若 evaluate 卡住或執行時間過長，可能會阻塞頁面指令佇列，導致該分頁後續的所有操作
看起來都像是「卡住」。

PR #13498 新增了一個務實的安全機制（限制 evaluate 執行時間、傳遞 abort，以及盡力復原）。 本文件說明一項較大型的重構，使 `act:evaluate` 在架構上
與 Playwright 隔離，因此即使 evaluate 卡住，也不會影響一般的 Playwright 操作。

## 目標

- `act:evaluate` 不得永久阻塞同一分頁後續的瀏覽器操作。
- 逾時設定應作為端對端的單一事實來源，使呼叫端可以依賴該時間預算。
- Abort 與 timeout 在 HTTP 與行程內派送中應以相同方式處理。
- 支援在 evaluate 中進行元素定位，而無需將所有功能都移出 Playwright。
- 維持對現有呼叫端與 payload 的向後相容性。

## 非目標

- 取代所有瀏覽器操作（click、type、wait 等） 改為使用 CDP 實作。
- 移除 PR #13498 引入的現有安全機制（它仍然是有用的備援）。
- 在現有 `browser.evaluateEnabled` 控制之外引入新的不安全能力。
- 為 evaluate 加入行程隔離（worker process/thread）。 如果在此重構之後仍然出現難以復原的
  卡住狀態，這將作為後續的改進方向。

## 目前架構（為何會卡住）

在高層次上：

- 呼叫端將 `act:evaluate` 傳送至瀏覽器控制服務。
- 路由處理器呼叫 Playwright 來執行該 JavaScript。
- Playwright 會將頁面指令序列化，因此若 evaluate 永遠無法完成，就會阻塞佇列。
- 佇列被卡住意味著該分頁後續的 click/type/wait 操作可能看起來像是掛起。

## 建議架構

### 1. 期限傳遞

引入單一的時間預算概念，並由此衍生所有設定：

- 呼叫端設定 `timeoutMs`（或未來某個截止時間）。
- 外層請求逾時、路由處理邏輯，以及頁面內的執行預算
  全都使用相同的時間預算，並在必要時為序列化開銷預留少量緩衝。
- Abort 會以 `AbortSignal` 的形式在各處傳遞，使取消機制保持一致。

實作方向：

- 新增一個小型輔助工具（例如 `createBudget({ timeoutMs, signal })`），回傳：
  - `signal`：連動的 AbortSignal
  - `deadlineAtMs`：絕對截止時間
  - `remainingMs()`：供子操作使用的剩餘時間預算
- 將此輔助工具用於：
  - `src/browser/client-fetch.ts`（HTTP 與行程內派送）
  - `src/node-host/runner.ts`（proxy 路徑）
  - 瀏覽器動作實作（Playwright 與 CDP）

### 2. 分離 Evaluate 引擎（CDP 路徑）

新增一個基於 CDP 的 evaluate 實作，不與 Playwright 每頁的指令
佇列共用。 其關鍵特性是 evaluate 傳輸使用獨立的 WebSocket 連線，
並附加到目標的獨立 CDP session。

實作方向：

- 新增模組，例如 `src/browser/cdp-evaluate.ts`，其功能為：
  - 連線至已設定的 CDP 端點（瀏覽器層級 socket）。
  - 使用 `Target.attachToTarget({ targetId, flatten: true })` 取得 `sessionId`。
  - 執行以下其中之一：
    - 頁面層級 evaluate 使用 `Runtime.evaluate`，或
    - 元素 evaluate 使用 `DOM.resolveNode` 加上 `Runtime.callFunctionOn`。
  - 在逾時或中止時：
    - 盡力為該 session 發送 `Runtime.terminateExecution`。
    - 關閉 WebSocket 並回傳明確的錯誤。

注意：

- 這仍然會在頁面中執行 JavaScript，因此終止可能會產生副作用。 優點
  是它不會卡住 Playwright 佇列，並且可以透過終止 CDP session 在傳輸層
  進行取消。

### 3. Ref 設計（無需全面重寫的元素定位方式）

困難之處在於元素定位。 CDP 需要 DOM handle 或 `backendDOMNodeId`，而
目前大多數瀏覽器動作使用基於快照中的 refs 的 Playwright locator。

建議做法：保留現有 refs，但附加一個可選的 CDP 可解析 id。

#### 3.1 擴充已儲存的 Ref 資訊

擴充已儲存的角色 ref 中繼資料，選擇性地包含 CDP id：

- 目前：`{ role, name, nth }`
- 建議：`{ role, name, nth, backendDOMNodeId?: number }`

這可保持所有現有基於 Playwright 的動作正常運作，並在 `backendDOMNodeId` 可用時，允許 CDP evaluate 接受相同的 `ref` 值。

#### 3.2 在快照產生時填入 backendDOMNodeId

在產生角色快照時：

1. 如同目前做法，產生既有的角色 ref 對應表（role, name, nth）。
2. 透過 CDP 取得 AX tree（`Accessibility.getFullAXTree`），並使用相同的重複處理規則，計算對應的
   `(role, name, nth) -> backendDOMNodeId` 映射。
3. 將該 id 合併回目前分頁所儲存的 ref 資訊中。

如果某個 ref 的對應失敗，請將 `backendDOMNodeId` 保持為 undefined。 這使得該功能
以 best-effort 方式運作，並可安全地推出。

#### 3.3 使用 Ref 評估行為

在 `act:evaluate` 中：

- 如果存在 `ref` 且包含 `backendDOMNodeId`，則透過 CDP 執行元素 evaluate。
- 如果存在 `ref` 但沒有 `backendDOMNodeId`，則回退至 Playwright 路徑（並
  使用安全機制）。

可選的逃生機制：

- 擴充請求結構以允許進階呼叫者（以及
  用於除錯）直接接受 `backendDOMNodeId`，同時仍以 `ref` 作為主要介面。

### 4. 保留最後的備援復原路徑

即使使用 CDP evaluate，仍有其他方式可能導致分頁或連線卡死。 保留
現有的復原機制（終止執行 + 中斷 Playwright 連線）作為以下情況的最後手段：

- 舊版呼叫者
- CDP 附加被阻擋的環境
- 未預期的 Playwright 邊緣情況

## 實作計畫（單次迭代）

### 交付項目

- 一個基於 CDP 的 evaluate 引擎，在 Playwright 每頁命令佇列之外執行。
- 由呼叫者與處理程序一致使用的單一端到端 timeout/abort 預算。
- 可選擇攜帶 `backendDOMNodeId` 以用於元素 evaluate 的 Ref 中繼資料。
- 在可能情況下，`act:evaluate` 優先使用 CDP 引擎，否則回退至 Playwright。
- 測試以證明卡住的 evaluate 不會阻塞後續操作。
- 讓失敗與回退情況可見的日誌／指標。

### 實作檢查清單

1. 新增共用的「budget」輔助工具，將 `timeoutMs` 與上游 `AbortSignal` 串接為：
   - 單一 `AbortSignal`
   - 絕對截止時間
   - 供下游操作使用的 `remainingMs()` 輔助函式
2. 更新所有呼叫路徑以使用該輔助工具，使 `timeoutMs` 在各處具有一致意義：
   - `src/browser/client-fetch.ts`（HTTP 與行程內派送）
   - `src/node-host/runner.ts`（node 代理路徑）
   - 呼叫 `/act` 的 CLI 包裝器（為 `browser evaluate` 新增 `--timeout-ms`）
3. 實作 `src/browser/cdp-evaluate.ts`：
   - 連線至瀏覽器層級的 CDP socket
   - 使用 `Target.attachToTarget` 取得 `sessionId`
   - 執行 `Runtime.evaluate` 以進行頁面 evaluate
   - 執行 `DOM.resolveNode` + `Runtime.callFunctionOn` 以進行元素 evaluate
   - 在 timeout/abort 時：盡力執行 `Runtime.terminateExecution`，然後關閉 socket
4. 擴充已儲存的角色 ref 中繼資料，以可選方式包含 `backendDOMNodeId`：
   - 對於 Playwright 操作，保留既有的 `{ role, name, nth }` 行為
   - 為 CDP 元素定位新增 `backendDOMNodeId?: number`
5. 在建立快照時填入 `backendDOMNodeId`（盡力而為）：
   - 透過 CDP 取得 AX tree（`Accessibility.getFullAXTree`）
   - 計算 `(role, name, nth) -> backendDOMNodeId` 並合併至已儲存的 ref 對應表中
   - 若對應關係有歧義或缺失，則將 id 保持為 undefined
6. 更新 `act:evaluate` 路由邏輯：
   - 若沒有 `ref`：一律使用 CDP evaluate
   - 若 `ref` 可解析為 `backendDOMNodeId`：使用 CDP 元素 evaluate
   - 否則：回退至 Playwright evaluate（仍受限且可中止）
7. 保留現有的「最後手段」復原路徑作為備援，而非預設路徑。
8. 新增測試：
   - 卡住的 evaluate 會在預算時間內逾時，且下一次 click/type 可成功執行
   - abort 會取消 evaluate（客戶端中斷或逾時）並解除對後續操作的阻塞
   - 對應失敗時可乾淨地回退至 Playwright
9. 新增可觀測性：
   - evaluate 持續時間與逾時計數器
   - terminateExecution 使用情況
   - 回退率（CDP -> Playwright）及其原因

### 驗收標準

- 刻意卡住的 `act:evaluate` 會在呼叫者預算時間內返回，且不會導致
  分頁在後續操作中卡死。
- `timeoutMs` 在 CLI、agent tool、node proxy 以及行程內呼叫中行為一致。
- 若 `ref` 可對應至 `backendDOMNodeId`，元素 evaluate 會使用 CDP；否則
  回退路徑仍受限且可復原。

## 測試計畫

- 單元測試：
  - `(role, name, nth)` 在 role refs 與 AX tree 節點之間的比對邏輯。
  - 預算輔助工具行為（緩衝空間、剩餘時間計算）。
- 整合測試：
  - CDP evaluate 逾時會在預算內返回，且不會阻塞下一個操作。
  - Abort 會取消 evaluate 並盡力觸發終止。
- 契約測試：
  - 確保 `BrowserActRequest` 與 `BrowserActResponse` 保持相容。

## 風險與緩解措施

- 對應並不完美：
  - 緩解措施：盡力對應、回退至 Playwright evaluate，並新增除錯工具。
- `Runtime.terminateExecution` 具有副作用：
  - 緩解措施：僅在逾時／中止時使用，並在錯誤中記錄其行為。
- 額外負擔：
  - 緩解措施：僅在請求快照時取得 AX tree、依 target 快取，並保持
    CDP session 為短生命週期。
- 擴充功能轉發限制：
  - 緩解措施：當無法使用每頁 socket 時改用瀏覽器層級的 attach API，並
    保留目前的 Playwright 路徑作為備援。

## 開放問題

- 新的引擎是否應該可設定為 `playwright`、`cdp` 或 `auto`？
- 我們是否要為進階使用者公開新的「nodeRef」格式，還是僅保留 `ref`？
- 框架快照（frame snapshots）與選擇器範圍快照（selector scoped snapshots）應如何參與 AX 對應？
