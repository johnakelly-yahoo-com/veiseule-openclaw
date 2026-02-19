---
summary: "在无 root 权限的 Podman 容器中运行 OpenClaw"
read_when:
  - 你希望使用 Podman 而不是 Docker 来运行容器化的 Gateway
title: "Podman"
---

# Podman

在 **rootless** Podman 容器中运行 OpenClaw gateway。 使用与 Docker 相同的镜像（从仓库中的 [Dockerfile](https://github.com/openclaw/openclaw/blob/main/Dockerfile) 构建）。

## 要求

- Podman（rootless）
- 一次性设置需要 sudo（创建用户、构建镜像）

## 快速开始

**1. 一次性设置**（在仓库根目录运行；创建用户、构建镜像、安装启动脚本）：

```bash
./setup-podman.sh
```

这还会创建一个最小化的 `~openclaw/.openclaw/openclaw.json`（设置 `gateway.mode="local"`），以便无需运行向导即可启动 gateway。

默认情况下，容器**不会**作为 systemd 服务安装，需要手动启动（见下文）。 对于生产环境风格的部署（支持自动启动和自动重启），可以将其安装为 systemd Quadlet 用户服务：

```bash
./setup-podman.sh --quadlet
```

（或设置 `OPENCLAW_PODMAN_QUADLET=1`；使用 `--container` 仅安装容器和启动脚本。）

**2. 启动 gateway**（手动方式，用于快速冒烟测试）：

```bash
./scripts/run-openclaw-podman.sh launch
```

**3. 引导向导**（例如添加渠道或 providers）：

```bash
./scripts/run-openclaw-podman.sh launch setup
```

然后打开 `http://127.0.0.1:18789/`，并使用 `~openclaw/.openclaw/.env` 中的 token（或 setup 输出的值）。

## Systemd（Quadlet，可选）

如果你运行了 `./setup-podman.sh --quadlet`（或设置了 `OPENCLAW_PODMAN_QUADLET=1`），将会安装一个 [Podman Quadlet](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html) 单元，使 gateway 作为 openclaw 用户的 systemd 用户服务运行。 该服务会在设置结束时启用并启动。

- **启动：** `sudo systemctl --machine openclaw@ --user start openclaw.service`
- **停止：** `sudo systemctl --machine openclaw@ --user stop openclaw.service`
- **状态：** `sudo systemctl --machine openclaw@ --user status openclaw.service`
- **日志：** `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`

quadlet 文件位于 `~openclaw/.config/containers/systemd/openclaw.container`。 如需更改端口或环境变量，请编辑该文件（或其引用的 `.env` 文件），然后执行 `sudo systemctl --machine openclaw@ --user daemon-reload` 并重启服务。 如果为 openclaw 启用了 lingering（当 loginctl 可用时，setup 会执行此操作），服务将在系统启动时自动启动。

如果在最初未使用 quadlet 的情况下完成了设置，之后想添加 quadlet，请重新运行：`./setup-podman.sh --quadlet`。

## openclaw 用户（不可登录）

`setup-podman.sh` 会创建一个专用的系统用户 `openclaw`：

- **Shell:** `nologin` — 不允许交互式登录；降低攻击面。

- **Home:** 例如 `/home/openclaw` — 包含 `~/.openclaw`（配置、工作区）以及启动脚本 `run-openclaw-podman.sh`。

- **Rootless Podman:** 该用户必须拥有 **subuid** 和 **subgid** 范围。 许多发行版在创建用户时会自动分配这些。 如果设置过程中出现警告，请在 `/etc/subuid` 和 `/etc/subgid` 中添加以下行：

  ```text
  openclaw:100000:65536
  ```

  然后以该用户启动 gateway（例如通过 cron 或 systemd）：

  ```bash
  sudo -u openclaw /home/openclaw/run-openclaw-podman.sh
  sudo -u openclaw /home/openclaw/run-openclaw-podman.sh setup
  ```

- **Config:** 只有 `openclaw` 和 root 可以访问 `/home/openclaw/.openclaw`。 要编辑配置：在 gateway 运行后使用 Control UI，或执行 `sudo -u openclaw $EDITOR /home/openclaw/.openclaw/openclaw.json`。

## 环境与配置

- **Token:** 存储在 `~openclaw/.openclaw/.env` 中，变量名为 `OPENCLAW_GATEWAY_TOKEN`。 如果缺失，`setup-podman.sh` 和 `run-openclaw-podman.sh` 会自动生成（使用 `openssl`、`python3` 或 `od`）。
- **可选：** 在该 `.env` 中可以设置 provider 密钥（例如 `GROQ_API_KEY`、`OLLAMA_API_KEY`）以及其他 OpenClaw 环境变量。
- **主机端口：** 默认情况下，脚本映射 `18789`（gateway）和 `18790`（bridge）。 在启动时可通过 `OPENCLAW_PODMAN_GATEWAY_HOST_PORT` 和 `OPENCLAW_PODMAN_BRIDGE_HOST_PORT` 覆盖 **主机** 端口映射。
- **路径：** 主机上的配置和工作区默认位于 `~openclaw/.openclaw` 和 `~openclaw/.openclaw/workspace`。 可通过 `OPENCLAW_CONFIG_DIR` 和 `OPENCLAW_WORKSPACE_DIR` 覆盖启动脚本使用的主机路径。

## 常用命令

- **日志：** 使用 quadlet：`sudo journalctl --machine openclaw@ --user -u openclaw.service -f`。 使用脚本：`sudo -u openclaw podman logs -f openclaw`
- **停止：** 使用 quadlet：`sudo systemctl --machine openclaw@ --user stop openclaw.service`。 使用脚本：`sudo -u openclaw podman stop openclaw`
- **再次启动：** 使用 quadlet：`sudo systemctl --machine openclaw@ --user start openclaw.service`。 使用脚本：重新运行启动脚本或执行 `podman start openclaw`
- **删除容器：** `sudo -u openclaw podman rm -f openclaw` — 主机上的配置和工作区将被保留

## 故障排除

- **配置或 auth-profiles 出现 Permission denied (EACCES)：** 容器默认使用 `--userns=keep-id`，并以运行脚本的主机用户相同的 uid/gid 运行。 请确保主机上的 `OPENCLAW_CONFIG_DIR` 和 `OPENCLAW_WORKSPACE_DIR` 归该用户所有。
- **Gateway 启动被阻止（缺少 `gateway.mode=local`）：** 确保 `~openclaw/.openclaw/openclaw.json` 存在，并设置 `gateway.mode="local"`。 如果文件缺失，`setup-podman.sh` 会创建该文件。
- **Rootless Podman 在 openclaw 用户下失败：** 检查 `/etc/subuid` 和 `/etc/subgid` 是否包含 `openclaw` 的条目（例如 `openclaw:100000:65536`）。 如果缺失，请添加后重启。
- **容器名称已被占用：** 启动脚本使用 `podman run --replace`，因此再次启动时会替换现有容器。 如需手动清理：`podman rm -f openclaw`。
- **以 openclaw 用户运行时找不到脚本：** 请确保已运行 `setup-podman.sh`，以便将 `run-openclaw-podman.sh` 复制到 openclaw 的主目录（例如 `/home/openclaw/run-openclaw-podman.sh`）。
- **找不到 Quadlet 服务或启动失败：** 编辑 `.container` 文件后，运行 `sudo systemctl --machine openclaw@ --user daemon-reload`。 Quadlet 需要 cgroups v2：`podman info --format '{{.Host.CgroupsVersion}}'` 应显示 `2`。

## 可选：以你自己的用户运行

若要以普通用户运行 gateway（不使用专用的 openclaw 用户）：构建镜像，在 `~/.openclaw/.env` 中创建 `OPENCLAW_GATEWAY_TOKEN`，并使用 `--userns=keep-id` 及挂载到你的 `~/.openclaw` 来运行容器。 该启动脚本专为 openclaw-user 流程设计；对于单用户设置，你也可以手动运行脚本中的 `podman run` 命令，并将 config 和 workspace 指向你的 home 目录。 推荐大多数用户使用：运行 `setup-podman.sh` 并以 openclaw 用户身份执行，以便隔离配置和进程。

