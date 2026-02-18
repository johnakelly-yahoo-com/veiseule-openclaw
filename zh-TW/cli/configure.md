---
title: "設定"
---

# `openclaw configure`

用於設定憑證、裝置與代理預設值的互動式提示。

注意：**Model** 區段現在包含 `agents.defaults.models` 允許清單的多選項（顯示於 `/model` 與模型選擇器中）。

提示：不加子命令執行 `openclaw config` 會開啟相同的精靈。若需非互動式編輯，請使用 `openclaw config get|set|unset`。 Use
`openclaw config get|set|unset` for non-interactive edits.

相關內容：

- Gateway 設定參考：[設定](/gateway/configuration)
- 設定 CLI：[設定](/cli/config)

注意事項：

- 選擇 Gateway 執行位置時，總是會更新 `gateway.mode`。若你只需要這一步，可在不設定其他區段的情況下選擇「Continue」。 You can select "Continue" without other sections if that is all you need.
- 以頻道為導向的服務（Slack/Discord/Matrix/Microsoft Teams）在設定期間會提示設定頻道／聊天室的允許清單。您可以輸入名稱或 ID；精靈會在可能的情況下將名稱解析為 ID。

## 範例

```bash
openclaw configure
openclaw configure --section models --section channels
```

