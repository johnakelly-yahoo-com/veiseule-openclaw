---
title: "Node.js"
---

# Node.js

OpenClawは**Node 22以降**を必要とします。[インストーラースクリプト](/install#install-methods) は Node を自動的に検出・インストールします。このページは、Node を自分でセットアップし、すべてが正しく接続されている（バージョン、PATH、グローバルインストールなど）ことを確認したい場合のためのものです。

## バージョンの確認

```bash
node -v
```

`v22.x.x`またはそれ以上が表示されれば問題ありません。Node がインストールされていない、またはバージョンが古すぎる場合は、以下のインストール方法を選択してください。

## Node のインストール

<Tabs>
  <Tab title="macOS">
    **Homebrew**（推奨）:

    ```bash
    brew install node
    ```

    または [nodejs.org](https://nodejs.org/) から macOS インストーラーをダウンロードしてください。

  </Tab>
  <Tab title="Linux">
    **Ubuntu / Debian:**

    ```bash
    curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
    sudo apt-get install -y nodejs
    ```

    **Fedora / RHEL:**

    ```bash
    sudo dnf install nodejs
    ```

    もしくは、バージョンマネージャーを使用してください（下記参照）。

  </Tab>
  <Tab title="Windows">
    **winget**（推奨）:

    ```powershell
    winget install OpenJS.NodeJS.LTS
    ```

    **Chocolatey:**

    ```powershell
    choco install nodejs-lts
    ```

    または [nodejs.org](https://nodejs.org/) から Windows インストーラーをダウンロードしてください。

  </Tab>
</Tabs>

<Accordion title="Using a version manager (nvm, fnm, mise, asdf)">
  バージョンマネージャーを使うと、Node のバージョンを簡単に切り替えられます。代表的な選択肢は次のとおりです。

- [**fnm**](https://github.com/Schniz/fnm) — 高速でクロスプラットフォーム
- [**nvm**](https://github.com/nvm-sh/nvm) — macOS / Linux で広く利用されています
- [**mise**](https://mise.jdx.dev/) — ポリグロット（Node、Python、Ruby など）

fnm の例:

```bash
fnm install 22
fnm use 22
```

  <Warning>
  バージョンマネージャーがシェルの起動ファイル（`~/.zshrc` または `~/.bashrc`）で初期化されていることを確認してください。初期化されていない場合、PATH に Node の bin ディレクトリが含まれないため、新しいターミナルセッションで `openclaw` が見つからないことがあります。
  </Warning>
</Accordion>

## トラブルシューティング

### `openclaw: command not found`

これはほぼ常に、npm のグローバル bin ディレクトリが PATH に含まれていないことを意味します。

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

    出力内に `<npm-prefix>/bin`（macOS / Linux）または `<npm-prefix>`（Windows）が含まれているか確認してください。

  </Step>
  <Step title="Add it to your shell startup file">
    <Tabs>
      <Tab title="macOS / Linux">
        `~/.zshrc` または `~/.bashrc` に追加します:

        ```bash
        export PATH="$(npm prefix -g)/bin:$PATH"
        ```

        その後、新しいターミナルを開く（または zsh では `rehash`、bash では `hash -r` を実行）してください。
      </Tab>
      <Tab title="Windows">
        `npm prefix -g` の出力を、設定 → システム → 環境変数 からシステム PATH に追加してください。
      </Tab>
    </Tabs>

  </Step>
</Steps>

### `npm install -g` に関する権限エラー（Linux）

`EACCES` エラーが表示される場合は、npm のグローバルプレフィックスをユーザーが書き込み可能なディレクトリに変更してください。

```bash
mkdir -p "$HOME/.npm-global"
npm config set prefix "$HOME/.npm-global"
export PATH="$HOME/.npm-global/bin:$PATH"
```

これを永続化するには、`export PATH=...` の行を `~/.bashrc` または `~/.zshrc` に追加してください。
