---
summary: "OpenClawをルートレスPodmanコンテナで実行する"
read_when:
  - Dockerの代わりにPodmanでコンテナ化されたgatewayを使用したい場合
title: "Podman"
---

# Podman

OpenClaw gatewayを**ルートレス**Podmanコンテナで実行します。 Dockerと同じイメージを使用します（リポジトリの[Dockerfile](https://github.com/openclaw/openclaw/blob/main/Dockerfile)からビルド）。

## 要件

- Podman（ルートレス）
- 初回セットアップに sudo が必要です（ユーザー作成、イメージのビルド）

## クイックスタート

**1. 初回セットアップ**（リポジトリのルートから実行。ユーザー作成、イメージのビルド、起動スクリプトのインストールを行います）：

```bash
./setup-podman.sh
```

これにより、最小構成の `~openclaw/.openclaw/openclaw.json`（`gateway.mode="local"` を設定）も作成されるため、ウィザードを実行しなくても gateway を起動できます。

デフォルトでは、コンテナは systemd サービスとして**インストールされません**。手動で起動してください（以下参照）。 本番環境向けに自動起動や再起動を行う構成にする場合は、代わりに systemd Quadlet ユーザーサービスとしてインストールしてください：

```bash
./setup-podman.sh --quadlet
```

（または `OPENCLAW_PODMAN_QUADLET=1` を設定してください。コンテナと起動スクリプトのみをインストールする場合は `--container` を使用します。）

**2. gateway を起動**（簡易スモークテスト用の手動起動）：

```bash
./scripts/run-openclaw-podman.sh launch
```

**3. オンボーディングウィザード**（例：チャネルやプロバイダーを追加する場合）：

```bash
./scripts/run-openclaw-podman.sh launch setup
```

その後、`http://127.0.0.1:18789/` を開き、`~openclaw/.openclaw/.env` にあるトークン（またはセットアップ時に表示された値）を使用してください。

## Systemd（Quadlet、任意）

`./setup-podman.sh --quadlet`（または `OPENCLAW_PODMAN_QUADLET=1`）を実行すると、[Podman Quadlet](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html) ユニットがインストールされ、gateway が openclaw ユーザーの systemd ユーザーサービスとして実行されます。 セットアップの最後にサービスは有効化され、起動されます。

- **起動:** `sudo systemctl --machine openclaw@ --user start openclaw.service`
- **停止:** `sudo systemctl --machine openclaw@ --user stop openclaw.service`
- **状態確認:** `sudo systemctl --machine openclaw@ --user status openclaw.service`
- **ログ:** `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`

quadlet ファイルは `~openclaw/.config/containers/systemd/openclaw.container` にあります。 ポートや環境変数を変更する場合は、そのファイル（または参照している `.env`）を編集し、その後 `sudo systemctl --machine openclaw@ --user daemon-reload` を実行してサービスを再起動してください。 起動時には、openclaw に対して lingering が有効になっていればサービスは自動的に開始されます（loginctl が利用可能な場合、セットアップ時に設定されます）。

初回セットアップで quadlet を使用しなかった場合に**後から**追加するには、次を再実行してください：`./setup-podman.sh --quadlet`。

## openclaw ユーザー（ログイン不可）

`setup-podman.sh` は専用のシステムユーザー `openclaw` を作成します：

- **シェル:** `nologin` — 対話的ログイン不可。攻撃対象領域を縮小します。

- **ホーム:** 例 `/home/openclaw` — `~/.openclaw`（設定、ワークスペース）および起動スクリプト `run-openclaw-podman.sh` を格納します。

- **Rootless Podman:** このユーザーには **subuid** と **subgid** の範囲が必要です。 多くのディストリビューションでは、ユーザー作成時にこれらが自動的に割り当てられます。 セットアップで警告が表示された場合は、`/etc/subuid` と `/etc/subgid` に次の行を追加してください：

  ```text
  openclaw:100000:65536
  ```

  その後、そのユーザーとして gateway を起動します（例：cron や systemd から）：

  ```bash
  sudo -u openclaw /home/openclaw/run-openclaw-podman.sh
  sudo -u openclaw /home/openclaw/run-openclaw-podman.sh setup
  ```

- **設定:** `/home/openclaw/.openclaw` には `openclaw` と root のみがアクセスできます。 設定を編集するには、gateway 実行中に Control UI を使用するか、`sudo -u openclaw $EDITOR /home/openclaw/.openclaw/openclaw.json` を実行してください。

## 環境と設定

- **Token:** `~openclaw/.openclaw/.env` に `OPENCLAW_GATEWAY_TOKEN` として保存されます。 `setup-podman.sh` と `run-openclaw-podman.sh` は、存在しない場合に生成します（`openssl`、`python3`、または `od` を使用）。
- **任意:** その `.env` では、プロバイダーキー（例：`GROQ_API_KEY`、`OLLAMA_API_KEY`）やその他の OpenClaw 環境変数を設定できます。
- **ホストポート:** デフォルトでは、スクリプトは `18789`（gateway）と `18790`（bridge）をマッピングします。 起動時に `OPENCLAW_PODMAN_GATEWAY_HOST_PORT` と `OPENCLAW_PODMAN_BRIDGE_HOST_PORT` を使用して **ホスト** のポートマッピングを上書きできます。
- **パス:** ホストの設定およびワークスペースのデフォルトは `~openclaw/.openclaw` と `~openclaw/.openclaw/workspace` です。 起動スクリプトで使用するホストパスは、`OPENCLAW_CONFIG_DIR` と `OPENCLAW_WORKSPACE_DIR` で上書きできます。

## 便利なコマンド

- **ログ:** quadlet 使用時: `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`。 スクリプト使用時: `sudo -u openclaw podman logs -f openclaw`
- **停止:** quadlet 使用時: `sudo systemctl --machine openclaw@ --user stop openclaw.service`。 スクリプト使用時: `sudo -u openclaw podman stop openclaw`
- **再起動:** quadlet 使用時: `sudo systemctl --machine openclaw@ --user start openclaw.service`。 スクリプト使用時: 起動スクリプトを再実行するか、`podman start openclaw` を実行
- **コンテナの削除:** `sudo -u openclaw podman rm -f openclaw` — ホスト上の設定とワークスペースは保持されます

## トラブルシューティング

- **設定や auth-profiles で Permission denied（EACCES）:** コンテナはデフォルトで `--userns=keep-id` を使用し、スクリプトを実行したホストユーザーと同じ uid/gid で実行されます。 ホストの `OPENCLAW_CONFIG_DIR` と `OPENCLAW_WORKSPACE_DIR` がそのユーザーの所有であることを確認してください。
- **Gateway の起動がブロックされる（`gateway.mode=local` が未設定）:** `~openclaw/.openclaw/openclaw.json` が存在し、`gateway.mode="local"` が設定されていることを確認してください。 `setup-podman.sh` は、このファイルが存在しない場合に作成します。
- **ユーザー openclaw で rootless Podman が失敗する:** `/etc/subuid` と `/etc/subgid` に `openclaw` の行（例：`openclaw:100000:65536`）が含まれているか確認してください。 存在しない場合は追加し、再起動してください。
- **コンテナ名が使用中:** 起動スクリプトは `podman run --replace` を使用するため、再起動時に既存のコンテナは置き換えられます。 手動でクリーンアップする場合: `podman rm -f openclaw`。
- **openclaw として実行したときにスクリプトが見つからない:** `setup-podman.sh` を実行して、`run-openclaw-podman.sh` が openclaw のホーム（例：`/home/openclaw/run-openclaw-podman.sh`）にコピーされていることを確認してください。
- **Quadlet サービスが見つからない、または起動に失敗する:** `.container` ファイルを編集後、`sudo systemctl --machine openclaw@ --user daemon-reload` を実行してください。 Quadlet には cgroups v2 が必要です: `podman info --format '{{.Host.CgroupsVersion}}'` で `2` と表示されるはずです。

## 任意: 自分のユーザーで実行する

gateway を通常のユーザーで実行する場合（専用の openclaw ユーザーなし）: イメージをビルドし、`~/.openclaw/.env` に `OPENCLAW_GATEWAY_TOKEN` を作成し、`--userns=keep-id` と `~/.openclaw` へのマウントを指定してコンテナを実行します。 起動スクリプトは openclaw ユーザーでの利用を想定して設計されています。単一ユーザー構成の場合は、スクリプト内の `podman run` コマンドを手動で実行し、設定とワークスペースを自分のホームディレクトリに指定してください。 ほとんどのユーザーへの推奨: `setup-podman.sh` を使用し、openclaw ユーザーとして実行して、設定とプロセスを分離してください。
