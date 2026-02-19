---
summary: "安装器脚本的工作原理（install.sh + install-cli.sh）、参数和自动化"
read_when:
  - 你想了解 `openclaw.ai/install.sh` 的工作机制
  - 你想自动化安装（CI / 无头环境）
  - 你想从 GitHub 检出安装
title: "安装器内部机制"
---

# 安装器内部机制

OpenClaw 提供两个安装器脚本（托管在 `openclaw.ai`）：

| 脚本                                                  | 平台                                          | 6. 功能说明                                                                                   |
| --------------------------------------------------- | ------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| `install.sh` 通过将前缀切换到以下位置来缓解此问题：                    | 检测操作系统（macOS / Linux / WSL）。                | 对于 `npm` 安装，Git _通常_不是必需的，但某些环境仍然需要它（例如通过 git URL 获取软件包或依赖时）。安装器目前会确保 Git 存在，以避免在全新发行版上出现 `spawn git ENOENT` 错误。 |
| install-cli.sh（无需 root 权限的 CLI 安装器） | 常见 Windows 问题：                              | 将 Node + OpenClaw 安装到本地前缀（`~/.openclaw`）。 13. 无需 root 权限。                                        |
| install.ps1（Windows PowerShell）     | Windows (PowerShell) 帮助： | 确保 Node.js **22+**（winget/Chocolatey/Scoop 或手动安装）。                                               |

## 快速命令

<Tabs>
  <Tab title="install.sh">
    ```bash
    curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install.sh | bash
    ```

    ```
    curl -fsSL https://openclaw.ai/install.sh | bash -s -- --help
    ```

  
</Tab>
  <Tab title="install-cli.sh">`git`：克隆/构建源码检出并安装包装脚本

    ```
    curl -fsSL https://openclaw.ai/install-cli.sh | bash -s -- --help
    ```

  
</Tab>
  <Tab title="install.ps1">iwr -useb https://openclaw.ai/install.ps1 | iex -InstallMethod git

    ```
    & ([scriptblock]::Create((iwr -useb https://openclaw.ai/install.ps1))) -Tag beta -NoOnboard -DryRun
    ```

  
</Tab>
</Tabs>

<Note>
如果安装器完成但在新终端中找不到 `openclaw`，通常是 Node/npm PATH 问题。参见：[安装](/install#nodejs--npm-path-sanity)。
</Note>

---

## install/installer.md

<Tip>
  
</Tab></Tip>

### Flow (install.sh)

<Steps>
  <Step title="Detect OS">
    如果安装成功但在新终端中找不到 `openclaw`，请参阅 [Node.js 故障排查](/install/node#troubleshooting)。 28. install.sh
  
</Step>
  <Step title="Ensure Node.js 22+">确保 Node.js **22+**（macOS 通过 Homebrew；Linux 通过 NodeSource）。
</Step>
  <Step title="Ensure Git">Git 要求：
</Step>
  <Step title="Install OpenClaw">为什么在全新 Linux 上 npm 会报 `EACCES`
</Step>
  <Step title="Post-install tasks">对于 git 安装：安装/更新后运行 `openclaw doctor --non-interactive`（尽力执行）。
</Step>
</Steps>

### 32. 如果检测到 macOS，则在缺失时安装 Homebrew。

如果你在**已有的 OpenClaw 源码检出目录中**运行安装器（通过 `package.json` + `pnpm-workspace.yaml` 检测），它会提示：

- 更新并使用此检出（`git`）
- use global install (`npm`)

```
如缺失则安装 Git。
```

The script exits with code `2` for invalid method selection or invalid `--install-method` values.

### 示例：

<Tabs>
  <Tab title="Default">SHARP_IGNORE_GLOBAL_LIBVIPS=0 curl -fsSL https://openclaw.ai/install.sh | bash
</Tab>
  <Tab title="Skip onboarding">`~/.npm-global`（并在存在时将其添加到 `~/.bashrc` / `~/.zshrc` 的 `PATH` 中）
</Tab>
  <Tab title="Git install">`--install-method git` 路径（克隆 / 拉取）需要 Git。
</Tab>
  <Tab title="Dry run">38. 如果在 OpenClaw 的检出目录中运行（存在 `package.json` + `pnpm-workspace.yaml`），脚本将提供：
</Tab>
</Tabs>

<AccordionGroup>
  <Accordion title="Flags reference">

| 39. 使用检出版本（`git`），或 | 40. 使用全局安装（`npm`）                                                                                      |
| ------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------- |
| `git`：克隆/构建源码检出并安装包装脚本                     | Choose install method (default: `npm`). Alias: `--method`  |
| `--npm`                                    | Shortcut for npm method                                                                                                       |
| `--git`                                    | Shortcut for git method. Alias: `--github`                                                    |
| `--version <version\|dist-tag>`          | npm version or dist-tag (default: `latest`)                                                |
| `--beta`                                   | Use beta dist-tag if available, else fallback to `latest`                                                                     |
| 为什么需要 Git                                  | Checkout directory (default: `~/openclaw`). Alias: `--dir` |
| `--no-git-update`                          | Skip `git pull` for existing checkout                                                                                         |
| `--no-prompt`                              | Disable prompts                                                                                                               |
| `--no-onboard`                             | Skip onboarding                                                                                                               |
| `--onboard`                                | Enable onboarding                                                                                                             |
| `--dry-run`                                | Print actions without applying changes                                                                                        |
| `--verbose`                                | Enable debug output (`set -x`, npm notice-level logs)                                                      |
| `--help`                                   | Show usage (`-h`)                                                                                          |

  
</Accordion>

  <Accordion title="Environment variables reference">

| Variable                                                         | Description                 |
| ---------------------------------------------------------------- | --------------------------- |
| `OPENCLAW_INSTALL_METHOD=git|npm`                               | 选择安装方式：                     |
| 2026-02-01T21:07:55Z             | npm version or dist-tag     |
| 功能概述：                                                            | Use beta if available       |
| 查看当前参数/行为，运行：                                                    | Checkout directory          |
| 9e0a19ecb5da0a395030e1ccf0d4bedf16b83946b3432c5399d448fe5d298391 | Toggle git updates          |
| `OPENCLAW_NO_PROMPT=1`                                           | Disable prompts             |
| `OPENCLAW_NO_ONBOARD=1`                                          | Skip onboarding             |
| 14                                                               | Dry run mode                |
| `OPENCLAW_VERBOSE=1`                                             | Debug mode                  |
| `OPENCLAW_NPM_LOGLEVEL=error\|warn\|notice`                  | npm log level               |
| `SHARP_IGNORE_GLOBAL_LIBVIPS=0\|1`                             | 控制 sharp/libvips 行为（默认：`1`） |

  
</Accordion>
</AccordionGroup>

---

## install.sh（推荐）

<Info>
适用于希望将所有内容安装在本地前缀下（默认 `~/.openclaw`）且不依赖系统 Node 的环境。
</Info>

### 流程（install-cli.sh）

<Steps>
  <Step title="Install local Node runtime">下载 Node 压缩包（默认 `22.22.0`）到 `<prefix>/tools/node-v<version>` 并验证 SHA-256。
</Step>
  <Step title="Ensure Git">如果缺少 Git，则在 Linux 上尝试通过 apt/dnf/yum 安装，或在 macOS 上通过 Homebrew 安装。
</Step>
  <Step title="Install OpenClaw under prefix">或迁移到全局 npm 安装（`npm`）<prefix>` 安装，然后将包装器写入 `<prefix>/bin/openclaw`。
</Step>
</Steps>

### 示例（install-cli.sh）

<Tabs>
  <Tab title="Default">`https://openclaw.ai/install-cli.sh` — 无需 root 权限的 CLI 安装器（安装到带有独立 Node 的前缀目录）
</Tab>
  <Tab title="Custom prefix + version">如果是升级现有安装：运行 `openclaw doctor --non-interactive`（尽力执行）。
</Tab>
  <Tab title="Automation JSON output">`https://openclaw.ai/install.sh` — "推荐"安装器（默认全局 npm 安装；也可从 GitHub 检出安装）
</Tab>
  <Tab title="Run onboarding">```bash
curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install-cli.sh | bash -s -- --onboard
```
</Tab>
</Tabs>

<AccordionGroup>
  <Accordion title="Flags reference">

| 标志                                         | 说明                                                       |
| ------------------------------------------ | -------------------------------------------------------- |
| `--prefix <path>`                          | 如果你_希望_ `sharp` 链接到全局安装的 libvips（或你正在调试），请设置：            |
| `--version <ver>`                          | OpenClaw 版本或 dist-tag（默认：`latest`）                       |
| `--node-version <ver>`                     | Node 版本（默认：`22.22.0`）                                    |
| `--json`                                   | 输出 NDJSON 事件                                             |
| `--onboard`                                | 安装后运行 `openclaw onboard`                                 |
| `--no-onboard`                             | 跳过引导流程（默认）                                               |
| `npm`（默认）：`npm install -g openclaw@latest` | 在 Linux 上：必要时将 npm 前缀切换到 `~/.npm-global`，以避免全局 npm 权限错误。 |
| `--help`                                   | 显示用法（`-h`）                                               |

  
</Accordion>

  <Accordion title="Environment variables reference">

| 变量                                              | 说明                                                                               |
| ----------------------------------------------- | -------------------------------------------------------------------------------- |
| 功能概述：                                           | 安装前缀                                                                             |
| 帮助：                                             | OpenClaw 版本或 dist-tag                                                            |
| 环境变量：                                           | Node 版本                                                                          |
| claude-opus-4-5                                 | 跳过引导流程                                                                           |
| `OPENCLAW_NPM_LOGLEVEL=error\|warn\|notice` | npm 日志级别                                                                         |
| `OPENCLAW_GIT_DIR=<path>`                       | 旧版清理查找路径（在移除旧的 `Peekaboo` 子模块检出时使用）                                              |
| `SHARP_IGNORE_GLOBAL_LIBVIPS=0\|1`            | Control sharp/libvips behavior (default: `1`) |

  
</Accordion>
</AccordionGroup>

---

## pi

### Flow (install.ps1)

<Steps>
  <Step title="Ensure PowerShell + Windows environment">
    Requires PowerShell 5+.
  
</Step>
  <Step title="Ensure Node.js 22+">
    If missing, attempts install via winget, then Chocolatey, then Scoop.
  
</Step>
  <Step title="Install OpenClaw">**"openclaw" 不是可识别的命令**：你的 npm 全局 bin 文件夹不在 PATH 中。大多数系统使用 `%AppData%\\npm`。你也可以运行 `npm config get prefix` 并将 `\\bin` 添加到 PATH，然后重新打开 PowerShell。
</Step>
  <Step title="Post-install tasks">在升级和 git 安装时运行 `openclaw doctor --non-interactive`（尽力执行）。
</Step>
</Steps>

### Examples (install.ps1)

<Tabs>
  <Tab title="Default">iwr -useb https://openclaw.ai/install.ps1 | iex
</Tab>
  <Tab title="Git install">`https://openclaw.ai/install.ps1` — Windows PowerShell 安装器（默认 npm；可选 git 安装）
</Tab>
  <Tab title="Custom git directory">iwr -useb https://openclaw.ai/install.ps1 | iex -InstallMethod git -GitDir "C:\\openclaw"
</Tab>
  <Tab title="Dry run">& ([scriptblock]::Create((iwr -useb https://openclaw.ai/install.ps1))) -?
</Tab>
  <Tab title="Debug trace">
    ```powershell
    # install.ps1 目前还没有专用的 -Verbose 参数。
    Set-PSDebug -Trace 1
    & ([scriptblock]::Create((iwr -useb https://openclaw.ai/install.ps1))) -NoOnboard
    Set-PSDebug -Trace 0
    ```  
</Tab>
</Tabs>

<AccordionGroup>
  <Accordion title="Flags reference">

| Flag              | Description                                                                                |
| ----------------- | ------------------------------------------------------------------------------------------ |
| 可发现性 / "git 安装"提示 | `npm`（默认）：`npm install -g openclaw@latest`                                                 |
| `-Tag <tag>`      | ```
  
</Tab>
```                                                                           |
| `-GitDir &lt;path&gt;`  | Checkout directory (default: `%USERPROFILE%\openclaw`) |
| `-NoOnboard`      | Skip onboarding                                                                            |
| `-NoGitUpdate`    | Skip `git pull`                                                                            |
| `-DryRun`         | Print actions only                                                                         |

  
</Accordion>

  <Accordion title="Environment variables reference">

| Variable                                                                                                            | 功能说明               |
| ------------------------------------------------------------------------------------------------------------------- | ------------------ |
| 在非交互式上下文中（无 TTY / `--no-prompt`），你必须传入 `--install-method git|npm`（或设置 `OPENCLAW_INSTALL_METHOD`），否则脚本将以退出码 `2` 退出。 | 选择安装方式：            |
| `OPENCLAW_GIT_DIR=&lt;path&gt;`                                                                                           | Checkout directory |
| `OPENCLAW_NO_ONBOARD=1`                                                                                             | Skip onboarding    |
| `OPENCLAW_GIT_DIR=...`                                                                                              | Disable git pull   |
| `OPENCLAW_DRY_RUN=1`                                                                                                | Dry run mode       |

  
</Accordion>
</AccordionGroup>

<Note>
如果你选择 `-InstallMethod git` 但未安装 Git，安装器会打印 Git for Windows 的链接（`https://git-scm.com/download/win`）并退出。
</Note>

---

## CI and automation

Use non-interactive flags/env vars for predictable runs.

<Tabs>
  <Tab title="install.sh (non-interactive npm)">
<Tip>
  
</Tab></Tip>

### Flow (install.sh)

<Steps>
  <Step title="Detect OS">
    如果安装成功但在新终端中找不到 `openclaw`，请参阅 [Node.js 故障排查](/install/node#troubleshooting)。 28. install.sh
  
</Step>
  <Step title="Ensure Node.js 22+">
    Checks Node version and installs Node 22 if needed (Homebrew on macOS, NodeSource setup scripts on Linux apt/dnf/yum).
  
</Step>
  <Step title="Ensure Git">推荐用于 macOS/Linux/WSL 上的大多数交互式安装。
</Step>
  <Step title="Install OpenClaw">30. 流程（install.sh）
</Step>
  <Step title="Post-install tasks">31. 支持 macOS 和 Linux（包括 WSL）。
</Step>
</Steps>

### 32. 如果检测到 macOS，则在缺失时安装 Homebrew。

If run inside an OpenClaw checkout (`package.json` + `pnpm-workspace.yaml`), the script offers:

- ```
  检查 Node 版本，如有需要则安装 Node 22（macOS 使用 Homebrew，Linux 使用 NodeSource 安装脚本，适用于 apt/dnf/yum）。
</Tab>
  <Tab title="install.sh (non-interactive git)">此脚本将 `openclaw` 安装到前缀目录（默认：`~/.openclaw`），同时在该前缀下安装专用的 Node 运行时，因此可以在不想改动系统 Node/npm 的机器上使用。
</Tab>
  <Tab title="install-cli.sh (JSON)">```bash
curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install-cli.sh | bash -s -- --json --prefix /opt/openclaw
```
</Tab>
  <Tab title="install.ps1 (skip onboarding)">
    ```powershell
    & ([scriptblock]::Create((iwr -useb https://openclaw.ai/install.ps1))) -NoOnboard
    ```
  
</Tab>
</Tabs>

---

## Troubleshooting

<AccordionGroup>
  <Accordion title="Why is Git required?">
    Git is required for `git` install method. 对于 `npm` 安装，仍会检查/安装 Git，以避免当依赖使用 git URL 时出现 `spawn git ENOENT` 失败。
  
</Accordion>

  <Accordion title="Why does npm hit EACCES on Linux?">
    某些 Linux 环境会将 npm 全局前缀指向 root 拥有的路径。 `install.sh` 可以将前缀切换到 `~/.npm-global`，并在 shell rc 文件存在时向其中追加 PATH 导出。
  
</Accordion>

  <Accordion title="sharp/libvips issues">
    脚本默认设置 `SHARP_IGNORE_GLOBAL_LIBVIPS=1`，以避免 sharp 针对系统 libvips 进行构建。 要覆盖该设置：

    ```
    通过默认设置 `SHARP_IGNORE_GLOBAL_LIBVIPS=1` 来缓解 `sharp` 原生安装问题（避免使用系统 libvips 编译）。
    ```

  
</Accordion>

  <Accordion title='Windows: "npm error spawn git / ENOENT"'>**npm error spawn git / ENOENT**：安装 Git for Windows 并重新打开 PowerShell，然后重新运行安装器。
</Accordion>

  <Accordion title='Windows: "openclaw is not recognized"'>在某些 Linux 设置中（尤其是通过系统包管理器或 NodeSource 安装 Node 后），npm 的全局前缀指向 root 拥有的位置。此时 `npm install -g ...` 会报 `EACCES` / `mkdir` 权限错误。
</Accordion>

  <Accordion title="Windows: how to get verbose installer output">
    `install.ps1` 目前尚未提供 `-Verbose` 开关。
    使用 PowerShell 跟踪进行脚本级诊断：

    ````
    ```powershell
    Set-PSDebug -Trace 1
    & ([scriptblock]::Create((iwr -useb https://openclaw.ai/install.ps1))) -NoOnboard
    Set-PSDebug -Trace 0
    ```
    ````

  
</Accordion>

  <Accordion title="openclaw not found after install">
    通常是 PATH 问题。 参见 [Node.js 故障排除](/install/node#troubleshooting)。
  
</Accordion>
</AccordionGroup>

