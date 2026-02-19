---
summary: "在 rootless Podman 容器中執行 OpenClaw"
read_when:
  - 你希望使用 Podman 而非 Docker 來建立容器化的 gateway
title: "Podman"
---

# Podman

在 **rootless** Podman 容器中執行 OpenClaw gateway。 使用與 Docker 相同的映像檔（從儲存庫中的 [Dockerfile](https://github.com/openclaw/openclaw/blob/main/Dockerfile) 建置）。

## 需求

- Podman（rootless）
- 一次性設定需要 Sudo（建立使用者、建置映像檔）

## 快速開始

**1. 一次性設定**（從儲存庫根目錄執行；會建立使用者、建置映像檔並安裝啟動腳本）：

```bash
./setup-podman.sh
```

這也會建立最小化的 `~openclaw/.openclaw/openclaw.json`（設定 `gateway.mode="local"`），使 gateway 無需執行精靈即可啟動。

預設情況下，容器**不會**安裝為 systemd 服務，你需要手動啟動（見下文）。 若要採用類似正式環境的設定（自動啟動與自動重新啟動），請改為將其安裝為 systemd Quadlet 使用者服務：

```bash
./setup-podman.sh --quadlet
```

（或設定 `OPENCLAW_PODMAN_QUADLET=1`；使用 `--container` 僅安裝容器與啟動腳本。）

**2. 啟動 gateway**（手動，用於快速冒煙測試）：

```bash
./scripts/run-openclaw-podman.sh launch
```

**3. Onboarding 精靈**（例如新增通道或 providers）：

```bash
./scripts/run-openclaw-podman.sh launch setup
```

接著開啟 `http://127.0.0.1:18789/`，並使用 `~openclaw/.openclaw/.env` 中的權杖（或 setup 輸出的值）。

## Systemd（Quadlet，選用）

如果你執行了 `./setup-podman.sh --quadlet`（或設定 `OPENCLAW_PODMAN_QUADLET=1`），將會安裝一個 [Podman Quadlet](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html) 單元，使 gateway 以 openclaw 使用者的 systemd 使用者服務形式執行。 該服務會在設定結束時啟用並啟動。

- **啟動：** `sudo systemctl --machine openclaw@ --user start openclaw.service`
- **停止：** `sudo systemctl --machine openclaw@ --user stop openclaw.service`
- **狀態：** `sudo systemctl --machine openclaw@ --user status openclaw.service`
- **日誌：** `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`

quadlet 檔案位於 `~openclaw/.config/containers/systemd/openclaw.container`。 若要變更連接埠或環境變數，請編輯該檔案（或其引用的 `.env`），然後執行 `sudo systemctl --machine openclaw@ --user daemon-reload` 並重新啟動服務。 開機時，若已為 openclaw 啟用 lingering（在可使用 loginctl 時，安裝程序會進行此設定），服務會自動啟動。

若初始設定時未使用 quadlet，之後要新增 quadlet，請重新執行：`./setup-podman.sh --quadlet`。

## openclaw 使用者（不可登入）

`setup-podman.sh` 會建立一個專用的系統使用者 `openclaw`：

- **Shell：** `nologin` — 不可互動式登入；可降低攻擊面。

- **Home：** 例如 `/home/openclaw` — 存放 `~/.openclaw`（設定、工作區）以及啟動腳本 `run-openclaw-podman.sh`。

- **Rootless Podman：** 該使用者必須擁有 **subuid** 與 **subgid** 範圍。 許多發行版在建立使用者時會自動指派這些範圍。 若安裝過程顯示警告，請在 `/etc/subuid` 與 `/etc/subgid` 中新增以下內容：

  ```text
  openclaw:100000:65536
  ```

  然後以該使用者身分啟動 gateway（例如透過 cron 或 systemd）：

  ```bash
  sudo -u openclaw /home/openclaw/run-openclaw-podman.sh
  sudo -u openclaw /home/openclaw/run-openclaw-podman.sh setup
  ```

- **設定：** 只有 `openclaw` 與 root 可以存取 `/home/openclaw/.openclaw`。 若要編輯設定：請在 gateway 執行後使用 Control UI，或執行 `sudo -u openclaw $EDITOR /home/openclaw/.openclaw/openclaw.json`。

## 環境變數與設定

- **Token：** 儲存在 `~openclaw/.openclaw/.env` 中，變數名稱為 `OPENCLAW_GATEWAY_TOKEN`。 若缺少該值，`setup-podman.sh` 與 `run-openclaw-podman.sh` 會自動產生（使用 `openssl`、`python3` 或 `od`）。
- **選用：** 在該 `.env` 中可設定 provider 金鑰（例如 `GROQ_API_KEY`、`OLLAMA_API_KEY`）及其他 OpenClaw 環境變數。
- **主機連接埠：** 預設腳本會對應 `18789`（gateway）與 `18790`（bridge）。 啟動時可透過 `OPENCLAW_PODMAN_GATEWAY_HOST_PORT` 與 `OPENCLAW_PODMAN_BRIDGE_HOST_PORT` 覆寫**主機端**的連接埠對應。
- **路徑：** 主機上的設定與工作區預設為 `~openclaw/.openclaw` 與 `~openclaw/.openclaw/workspace`。 可透過 `OPENCLAW_CONFIG_DIR` 與 `OPENCLAW_WORKSPACE_DIR` 覆寫啟動腳本所使用的主機路徑。

## 常用指令

- **日誌：** 使用 quadlet：`sudo journalctl --machine openclaw@ --user -u openclaw.service -f`。 使用腳本：`sudo -u openclaw podman logs -f openclaw`
- **停止：** 使用 quadlet：`sudo systemctl --machine openclaw@ --user stop openclaw.service`。 使用腳本：`sudo -u openclaw podman stop openclaw`
- **再次啟動：** 使用 quadlet：`sudo systemctl --machine openclaw@ --user start openclaw.service`。 使用腳本：重新執行啟動腳本或 `podman start openclaw`
- **移除容器：** `sudo -u openclaw podman rm -f openclaw` — 主機上的設定與工作區會保留

## 疑難排解

- **在 config 或 auth-profiles 上出現 Permission denied（EACCES）：** 容器預設使用 `--userns=keep-id`，並以執行腳本的主機使用者相同的 uid/gid 執行。 請確保主機上的 `OPENCLAW_CONFIG_DIR` 與 `OPENCLAW_WORKSPACE_DIR` 目錄屬於該使用者。
- **Gateway 無法啟動（缺少 `gateway.mode=local`）：** 請確認 `~openclaw/.openclaw/openclaw.json` 存在，且設定 `gateway.mode="local"`。 若檔案不存在，`setup-podman.sh` 會自動建立。
- **Rootless Podman fails for user openclaw:** 請檢查 `/etc/subuid` 和 `/etc/subgid` 是否包含 `openclaw` 的一行（例如：`openclaw:100000:65536`）。 如果缺少，請新增該行並重新啟動。
- **Container name in use:** 啟動腳本使用 `podman run --replace`，因此在重新啟動時會取代現有的容器。 若要手動清理：`podman rm -f openclaw`。
- **Script not found when running as openclaw:** 請確認已執行 `setup-podman.sh`，以便將 `run-openclaw-podman.sh` 複製到 openclaw 的家目錄（例如：`/home/openclaw/run-openclaw-podman.sh`）。
- **Quadlet service not found or fails to start:** 編輯 `.container` 檔案後，請執行 `sudo systemctl --machine openclaw@ --user daemon-reload`。 Quadlet 需要 cgroups v2：`podman info --format '{{.Host.CgroupsVersion}}'` 應顯示 `2`。

## 可選：以你自己的使用者身分執行

若要以一般使用者身分執行 gateway（不建立專用的 openclaw 使用者）：請建置映像檔，在 `~/.openclaw/.env` 中建立包含 `OPENCLAW_GATEWAY_TOKEN` 的檔案，並使用 `--userns=keep-id` 以及掛載至你的 `~/.openclaw` 來執行容器。 啟動腳本是為 openclaw 使用者流程所設計；若為單一使用者設定，你也可以手動執行腳本中的 `podman run` 指令，並將設定與工作目錄指向你的家目錄。 大多數使用者建議：使用 `setup-podman.sh` 並以 openclaw 使用者身分執行，以便將設定與程序隔離。
