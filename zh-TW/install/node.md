---
title: "Node.js"
---

# Node.js

OpenClaw 需要 **Node 22 或更新版本**。[安裝程式腳本](/install#install-methods) 會自動偵測並安裝 Node — 此頁面適用於你想自行設定 Node，並確保所有內容都正確連線（版本、PATH、全域安裝）的情況。

## 檢查你的版本

```bash
node -v
```

如果顯示 `v22.x.x` 或更高版本，就表示沒問題。如果尚未安裝 Node 或版本過舊，請從下方選擇一種安裝方式。

## 安裝 Node

<Tabs>
  <Tab title="macOS">
    **Homebrew**（建議）：

    ```bash
    brew install node
    ```

    或從 [nodejs.org](https://nodejs.org/) 下載 macOS 安裝程式。

  </Tab>
  <Tab title="Linux">
    **Ubuntu / Debian：**

    ```bash
    curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
    sudo apt-get install -y nodejs
    ```

    **Fedora / RHEL：**

    ```bash
    sudo dnf install nodejs
    ```

    或使用版本管理工具（見下方）。

  </Tab>
  <Tab title="Windows">
    **winget**（建議）：

    ```powershell
    winget install OpenJS.NodeJS.LTS
    ```

    **Chocolatey：**

    ```powershell
    choco install nodejs-lts
    ```

    或從 [nodejs.org](https://nodejs.org/) 下載 Windows 安裝程式。

  </Tab>
</Tabs>

<Accordion title="Using a version manager (nvm, fnm, mise, asdf)">
  版本管理工具可讓你輕鬆切換不同的 Node 版本。常見選項：

- [**fnm**](https://github.com/Schniz/fnm) — 快速、跨平台
- [**nvm**](https://github.com/nvm-sh/nvm) — 在 macOS/Linux 上廣泛使用
- [**mise**](https://mise.jdx.dev/) — 多語言（Node、Python、Ruby 等）

以 fnm 為例：

```bash
fnm install 22
fnm use 22
```

  <Warning>
  請確保你的版本管理工具已在 Shell 啟動檔中初始化（`~/.zshrc` 或 `~/.bashrc`）。若未初始化，新的終端機工作階段中可能找不到 `openclaw`，因為 PATH 不會包含 Node 的 bin 目錄。
  </Warning>
</Accordion>

## 疑難排解

### `openclaw: command not found`

這幾乎總是表示 npm 的全域 bin 目錄未加入你的 PATH。

<Steps>
  <Step title="Find your global npm prefix">
    ```bash
    npm prefix -g
    ```
  </Step>
  <Step title="Check if it's on your PATH">
    ```bash
    echo "$PATH"
    ```

    請在輸出中尋找 `<npm-prefix>/bin`（macOS/Linux）或 `<npm-prefix>`（Windows）。

  </Step>
  <Step title="Add it to your shell startup file">
    <Tabs>
      <Tab title="macOS / Linux">
        加入至 `~/.zshrc` 或 `~/.bashrc`：

        ```bash
        export PATH="$(npm prefix -g)/bin:$PATH"
        ```

        接著開啟新的終端機（或在 zsh 中執行 `rehash`／在 bash 中執行 `hash -r`）。
      </Tab>
      <Tab title="Windows">
        透過「設定 → 系統 → 環境變數」，將 `npm prefix -g` 的輸出加入系統 PATH。
      </Tab>
    </Tabs>

  </Step>
</Steps>

### 在 `npm install -g` 上的權限錯誤（Linux）

若你看到 `EACCES` 錯誤，請將 npm 的全域 prefix 切換至使用者可寫入的目錄：

```bash
mkdir -p "$HOME/.npm-global"
npm config set prefix "$HOME/.npm-global"
export PATH="$HOME/.npm-global/bin:$PATH"
```

將 `export PATH=...` 這一行加入你的 `~/.bashrc` 或 `~/.zshrc`，以使設定永久生效。