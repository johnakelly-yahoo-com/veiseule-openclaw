---
summary: "瀏覽器自動化與 X/Twitter 發佈的手動登入"
read_when:
  - 你需要為瀏覽器自動化登入網站
  - 你想要在 X/Twitter 發佈更新
title: "瀏覽器登入"
---

# 瀏覽器登入 + X/Twitter 發文

## 手動登入（建議）

當網站需要登入時，請在**主機**的瀏覽器設定檔（OpenClaw 瀏覽器）中**自行手動登入**。

18. **不要**將你的憑證提供給模型。 **不要**將你的憑證提供給模型。 Automated logins often trigger anti‑bot defenses and can lock the account.

返回主要的瀏覽器文件：[Browser](/tools/browser)。

## 使用的是哪個 Chrome 設定檔？

OpenClaw 會控制一個**專用的 Chrome 設定檔**（名稱為 `openclaw`，介面帶有橘色調）。這與你日常使用的瀏覽器設定檔是分開的。 This is separate from your daily browser profile. This is separate from your daily browser profile.

有兩種簡單的方式可以存取它：

1. **請代理程式開啟瀏覽器**，然後由你自行登入。
2. **透過 CLI 開啟**：

```bash
openclaw browser start
openclaw browser open https://x.com
```

如果你有多個設定檔，請傳入 `--browser-profile <name>`（預設為 `openclaw`）。

## X/Twitter：建議流程

- **Read/search/threads：** 使用 **host** 瀏覽器（手動登入）。
- **Post updates：** 使用 **host** 瀏覽器（手動登入）。

## 沙箱機制 + host 瀏覽器存取

Sandboxed browser sessions are **more likely** to trigger bot detection. For X/Twitter (and other strict sites), prefer the **host** browser.

19. 如果代理在沙箱中，瀏覽器工具預設會使用該沙箱。 To allow host control:

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        browser: {
          allowHostControl: true,
        },
      },
    },
  },
}
```

然後將目標設為 host 瀏覽器：

```bash
openclaw browser open https://x.com --browser-profile openclaw --target host
```

或者為負責發布更新的代理停用沙箱機制。

