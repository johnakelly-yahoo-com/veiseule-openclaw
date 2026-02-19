---
summary: "Plan: Playwright キューから CDP を使用して browser act:evaluate を分離し、エンドツーエンドのデッドラインとより安全な ref 解決を実現する"
owner: "openclaw"
status: "ドラフト"
last_updated: "2026-02-10"
title: "Browser Evaluate CDP リファクタリング"
---

# Browser Evaluate CDP リファクタリング計画

## コンテキスト

`act:evaluate` はページ内でユーザー提供の JavaScript を実行します。 現在は Playwright（`page.evaluate` または `locator.evaluate`）を介して実行されています。 Playwright はページごとに CDP コマンドを直列化するため、evaluate がハングまたは長時間実行されるとページのコマンドキューがブロックされ、そのタブ上の後続のすべての操作が「ハング」しているように見えることがあります。

PR #13498 では、実用的なセーフティネット（制限付き evaluate、abort 伝播、およびベストエフォートのリカバリ）が追加されています。 本ドキュメントでは、`act:evaluate` を本質的に Playwright から分離し、evaluate がハングしても通常の Playwright 操作を妨げないようにする、より大規模なリファクタリングについて説明します。

## 目標

- `act:evaluate` が同じタブ上の後続のブラウザ操作を恒久的にブロックしないこと。
- タイムアウトはエンドツーエンドで単一の信頼できる情報源とし、呼び出し元が予算を信頼できるようにすること。
- Abort とタイムアウトを、HTTP とプロセス内ディスパッチの両方で同じ方法で扱うこと。
- evaluate のための要素ターゲティングを、すべてを Playwright から切り離すことなくサポートすること。
- 既存の呼び出し元およびペイロードとの後方互換性を維持すること。

## 非目標

- すべてのブラウザ操作（click、type、wait など）を置き換えます。 CDP 実装に置き換えます。
- PR #13498 で導入された既存のセーフティネットを削除します（有用なフォールバックとしては残ります）。
- 既存の `browser.evaluateEnabled` ゲートを超える新しい unsafe 機能を導入します。
- evaluate のためにプロセス分離（worker プロセス／スレッド）を追加します。 このリファクタリング後も復旧が困難な
  スタック状態が発生する場合は、それはフォローアップの検討事項とします。

## 現在のアーキテクチャ（なぜスタックするのか）

高レベルでは：

- 呼び出し元が `act:evaluate` をブラウザ制御サービスに送信します。
- ルートハンドラーが Playwright を呼び出して JavaScript を実行します。
- Playwright はページコマンドを直列化するため、完了しない evaluate がキューをブロックします。
- キューがスタックすると、そのタブでの後続の click／type／wait 操作がハングしているように見えることがあります。

## 提案アーキテクチャ

### 1. デッドライン伝播

単一の予算（budget）という概念を導入し、すべてをそこから導出します：

- 呼び出し元が `timeoutMs`（または将来のデッドライン）を設定します。
- 外側のリクエストタイムアウト、ルートハンドラーロジック、ページ内の実行予算はすべて同じ予算を使用し、必要に応じてシリアライズのオーバーヘッド分のわずかな余裕を持たせます。
- Abort は `AbortSignal` としてあらゆる箇所に伝播され、キャンセルの一貫性を保ちます。

実装方針：

- 小さなヘルパー（例：`createBudget({ timeoutMs, signal })`）を追加し、以下を返します：
  - `signal`: 連動した AbortSignal
  - `deadlineAtMs`: 絶対デッドライン
  - `remainingMs()`: 子操作のための残り予算
- このヘルパーを以下で使用します：
  - `src/browser/client-fetch.ts`（HTTP およびインプロセスディスパッチ）
  - `src/node-host/runner.ts`（プロキシパス）
  - ブラウザ操作の実装（Playwright および CDP）

### 2. Evaluate エンジンの分離（CDP パス）

Playwright のページ単位のコマンドキューを共有しない、CDP ベースの evaluate 実装を追加します。 重要な特性は、evaluate のトランスポートが独立した WebSocket 接続であり、ターゲットにアタッチされた別個の CDP セッションであることです。

実装方針：

- 新しいモジュール（例：`src/browser/cdp-evaluate.ts`）を追加し、以下を行います：
  - 設定された CDP エンドポイント（ブラウザレベルのソケット）に接続します。
  - `Target.attachToTarget({ targetId, flatten: true })` を使用して `sessionId` を取得します。
  - 以下のいずれかで実行します:
    - ページレベルの evaluate には `Runtime.evaluate`、
    - 要素の evaluate には `DOM.resolveNode` と `Runtime.callFunctionOn`。
  - タイムアウトまたは abort 時：
    - セッションに対して `Runtime.terminateExecution` をベストエフォートで送信します。
    - WebSocket を閉じ、明確なエラーを返します。

注意：

- これはページ内で引き続き JavaScript を実行するため、終了処理が副作用を引き起こす可能性があります。 利点
  は、Playwright のキューを詰まらせないこと、そして CDP セッションを終了することでトランスポート
  レイヤーでキャンセル可能であることです。

### 3. Ref ストーリー（全面的な書き換えなしでの要素ターゲティング）

難しいのは要素ターゲティングです。 CDP には DOM ハンドルまたは `backendDOMNodeId` が必要ですが、
現在ほとんどのブラウザ操作はスナップショットからの ref に基づく Playwright ロケーターを使用しています。

推奨アプローチ：既存の ref を維持しつつ、オプションで CDP で解決可能な id を付与します。

#### 3.1 保存される Ref 情報の拡張

保存されている role ref メタデータを拡張し、オプションで CDP id を含められるようにします：

- 現在： `{ role, name, nth }`
- 提案： `{ role, name, nth, backendDOMNodeId?: number }`

これにより既存の Playwright ベースのアクションはすべてそのまま動作し、`backendDOMNodeId` が利用可能な場合は CDP evaluate でも
同じ `ref` 値を受け取れるようになります。

#### 3.2 スナップショット時に backendDOMNodeId を設定する

role スナップショットを生成する際：

1. 現在と同様に既存の role ref マップ（role, name, nth）を生成します。
2. CDP（`Accessibility.getFullAXTree`）経由で AX ツリーを取得し、
   同じ重複処理ルールを使用して `(role, name, nth) -> backendDOMNodeId` の並行マップを計算します。
3. id を現在のタブの保存済み ref 情報にマージします。

ある ref のマッピングに失敗した場合は、`backendDOMNodeId` を undefined のままにします。 これによりこの機能は
ベストエフォートとなり、安全にロールアウトできます。

#### 3.3 Ref を用いた Evaluate の動作

`act:evaluate` において：

- `ref` が存在し、かつ `backendDOMNodeId` を持つ場合は、CDP 経由で要素 evaluate を実行します。
- `ref` が存在するが `backendDOMNodeId` を持たない場合は、Playwright パスにフォールバックします（
  セーフティネット付き）。

オプションのエスケープハッチ：

- リクエスト形式を拡張し、上級ユーザー（および
  デバッグ用）のために `backendDOMNodeId` を直接受け取れるようにします。ただし、主要インターフェースは引き続き `ref` とします。

### 4. 最終手段のリカバリーパスを維持する

CDP evaluate を使用していても、タブや接続が詰まる原因は他にもあります。 以下のための最終手段として、
既存のリカバリーメカニズム（terminate execution + Playwright の切断）を維持します：

- レガシー呼び出し元
- CDP のアタッチがブロックされている環境
- Playwright の予期しないエッジケース

## 実装計画（単一イテレーション）

### 成果物

- Playwright のページ単位コマンドキューの外側で実行される、CDP ベースの evaluate エンジン。
- 呼び出し元とハンドラーで一貫して使用される、単一のエンドツーエンドのタイムアウト／中断予算。
- 要素 evaluate のためにオプションで `backendDOMNodeId` を保持できる Ref メタデータ。
- `act:evaluate` は、可能な場合は CDP エンジンを優先し、使用できない場合は Playwright にフォールバックします。
- スタックした evaluate が後続のアクションを妨げないことを証明するテスト。
- 失敗やフォールバックを可視化するログ／メトリクス。

### 実装チェックリスト

1. `timeoutMs` と上流の `AbortSignal` を以下に結び付ける共通の「budget」ヘルパーを追加する:
   - 単一の `AbortSignal`
   - 絶対的なデッドライン
   - 下流の処理向けの `remainingMs()` ヘルパー
2. `timeoutMs` がどこでも同じ意味になるよう、すべての呼び出し元パスをそのヘルパーを使うように更新する:
   - `src/browser/client-fetch.ts`（HTTP およびプロセス内ディスパッチ）
   - `src/node-host/runner.ts`（node プロキシパス）
   - `/act` を呼び出す CLI ラッパー（`browser evaluate` に `--timeout-ms` を追加）
3. `src/browser/cdp-evaluate.ts` を実装する:
   - ブラウザレベルの CDP ソケットに接続する
   - `Target.attachToTarget` で `sessionId` を取得する
   - ページ evaluate のために `Runtime.evaluate` を実行する
   - 要素 evaluate のために `DOM.resolveNode` + `Runtime.callFunctionOn` を実行する
   - タイムアウト／abort 時: ベストエフォートで `Runtime.terminateExecution` を実行し、その後ソケットを閉じる
4. 保存されるロール参照メタデータを拡張し、オプションで `backendDOMNodeId` を含められるようにする:
   - Playwright アクション向けの既存の `{ role, name, nth }` の動作を維持する
   - CDP の要素ターゲティング用に `backendDOMNodeId?: number` を追加する
5. スナップショット作成時に `backendDOMNodeId` を（ベストエフォートで）設定する:
   - CDP 経由で AX ツリーを取得する（`Accessibility.getFullAXTree`）
   - `(role, name, nth) -> backendDOMNodeId` を計算し、保存済みの参照マップにマージする
   - マッピングが曖昧または欠落している場合は、その id を undefined のままにする
6. `act:evaluate` のルーティングを更新する:
   - `ref` がない場合: 常に CDP evaluate を使用する
   - `ref` が `backendDOMNodeId` に解決される場合: CDP 要素 evaluate を使用する
   - それ以外の場合: Playwright evaluate にフォールバックする（引き続き制限付きかつ abort 可能）
7. 既存の「最終手段」リカバリーパスは、デフォルトではなくフォールバックとして維持する。
8. テストを追加する:
   - スタックした evaluate が予算内でタイムアウトし、次の click/type が成功すること
   - abort が evaluate（クライアント切断またはタイムアウト）をキャンセルし、後続のアクションをブロックしないこと
   - マッピング失敗時に適切に Playwright へフォールバックすること
9. 可観測性を追加する:
   - evaluate の実行時間およびタイムアウトのカウンター
   - terminateExecution の使用状況
   - フォールバック率（CDP -> Playwright）とその理由

### 受け入れ基準

- 意図的にハングさせた `act:evaluate` が呼び出し元の予算内で応答を返し、その後のアクションのために
  タブをブロックしないこと。
- `timeoutMs` は、CLI、agent tool、node proxy、およびプロセス内呼び出し全体で一貫して動作します。
- `ref` を `backendDOMNodeId` にマッピングできる場合、element evaluate は CDP を使用します。それ以外の場合でも、フォールバックパスは制限付きで回復可能です。

## テスト計画

- ユニットテスト:
  - role ref と AX ツリーノード間の `(role, name, nth)` マッチングロジック。
  - 予算ヘルパーの動作（ヘッドルーム、残り時間の計算）。
- 統合テスト:
  - CDP evaluate のタイムアウトが予算内で返り、次のアクションをブロックしないこと。
  - Abort が evaluate をキャンセルし、ベストエフォートで終了処理をトリガーすること。
- 契約テスト:
  - `BrowserActRequest` と `BrowserActResponse` の互換性が維持されていることを確認する。

## リスクと対策

- マッピングは完全ではない:
  - 対策: ベストエフォートでマッピングし、Playwright evaluate へのフォールバックを行い、デバッグツールを追加する。
- `Runtime.terminateExecution` には副作用がある:
  - 対策: タイムアウト／Abort 時のみ使用し、その動作をエラー内で文書化する。
- 追加のオーバーヘッド:
  - 対策: スナップショットが要求された場合のみ AX ツリーを取得し、ターゲットごとにキャッシュし、CDP セッションを短命に保つ。
- 拡張機能リレーの制限:
  - 対策: ページ単位のソケットが利用できない場合はブラウザレベルのアタッチ API を使用し、現在の Playwright パスをフォールバックとして維持する。

## 未解決の質問

- 新しいエンジンは `playwright`、`cdp`、または `auto` として設定可能にすべきでしょうか？
- 上級ユーザー向けに新しい "nodeRef" 形式を公開すべきでしょうか、それとも `ref` のみに留めるべきでしょうか？
- フレームスナップショットやセレクタスコープ付きスナップショットは、AX マッピングにどのように関与すべきでしょうか？
