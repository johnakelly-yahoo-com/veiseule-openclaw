---
title: "security"
---

# `openclaw security`

セキュリティツール（監査＋任意の修正）。

関連：

- セキュリティガイド： [Security](/gateway/security)

## 監査

```bash
openclaw security audit
openclaw security audit --deep
openclaw security audit --fix
```

監査では、複数の DM 送信者がメインセッションを共有している場合に警告を出し、共有受信箱に対して **セキュア DM モード**： `session.dmScope="per-channel-peer"`（またはマルチアカウントチャンネル向けの `per-account-channel-peer`）を推奨します。
また、サンドボックス化されていない状態で Web/ブラウザツールを有効にしたまま小規模モデル（`<=300B`）を使用している場合にも警告します。
また、小さなモデル (`<=300B`) がサンドボックス化されずに使用されている場合や、Web/ブラウザツールが有効になっている場合にも警告します。


