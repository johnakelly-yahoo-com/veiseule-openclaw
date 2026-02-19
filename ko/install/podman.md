---
summary: "루트리스 Podman 컨테이너에서 OpenClaw 실행"
read_when:
  - Docker 대신 Podman으로 컨테이너화된 gateway를 사용하려는 경우
title: "Podman"
---

# Podman

**루트리스** Podman 컨테이너에서 OpenClaw gateway를 실행합니다. Docker와 동일한 이미지를 사용합니다 (저장소의 [Dockerfile](https://github.com/openclaw/openclaw/blob/main/Dockerfile)에서 빌드).

## 요구 사항

- Podman (루트리스)
- 1회성 설정을 위한 sudo (사용자 생성, 이미지 빌드)

## 빠른 시작

**1. 1회성 설정** (저장소 루트에서 실행; 사용자 생성, 이미지 빌드, 런치 스크립트 설치):

```bash
./setup-podman.sh
```

이 과정에서 최소한의 `~openclaw/.openclaw/openclaw.json`(`gateway.mode="local"` 설정)도 생성되므로, 마법사를 실행하지 않아도 gateway를 시작할 수 있습니다.

기본적으로 컨테이너는 systemd 서비스로 설치되지 않으며, 수동으로 시작해야 합니다(아래 참고). 자동 시작 및 재시작이 포함된 프로덕션 환경 구성을 원한다면, 대신 systemd Quadlet 사용자 서비스로 설치하세요:

```bash
./setup-podman.sh --quadlet
```

(또는 `OPENCLAW_PODMAN_QUADLET=1`을 설정하세요; 컨테이너와 런치 스크립트만 설치하려면 `--container`를 사용하세요.)

**2. gateway 시작** (빠른 스모크 테스트를 위한 수동 실행):

```bash
./scripts/run-openclaw-podman.sh launch
```

**3. 온보딩 마법사** (예: 채널 또는 제공자 추가):

```bash
./scripts/run-openclaw-podman.sh launch setup
```

그런 다음 `http://127.0.0.1:18789/`을 열고 `~openclaw/.openclaw/.env`의 토큰(또는 setup에서 출력된 값)을 사용하세요.

## Systemd (Quadlet, 선택 사항)

`./setup-podman.sh --quadlet`(또는 `OPENCLAW_PODMAN_QUADLET=1`)을 실행한 경우, openclaw 사용자에 대해 gateway가 systemd 사용자 서비스로 실행되도록 [Podman Quadlet](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html) 유닛이 설치됩니다. 서비스는 설정 마지막 단계에서 활성화되고 시작됩니다.

- **시작:** `sudo systemctl --machine openclaw@ --user start openclaw.service`
- **중지:** `sudo systemctl --machine openclaw@ --user stop openclaw.service`
- **상태:** `sudo systemctl --machine openclaw@ --user status openclaw.service`
- **로그:** `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`

quadlet 파일은 `~openclaw/.config/containers/systemd/openclaw.container`에 위치합니다. 포트나 환경 변수를 변경하려면 해당 파일(또는 그 파일이 참조하는 `.env`)을 수정한 뒤 `sudo systemctl --machine openclaw@ --user daemon-reload`를 실행하고 서비스를 재시작하세요. 부팅 시, openclaw에 대해 lingering이 활성화되어 있으면 서비스가 자동으로 시작됩니다(설정 과정에서 loginctl을 사용할 수 있으면 이를 구성합니다).

초기 설정 시 quadlet을 사용하지 않았다면, 이후에 추가하려면 다음을 다시 실행하세요: `./setup-podman.sh --quadlet`.

## openclaw 사용자(로그인 불가)

`setup-podman.sh`는 전용 시스템 사용자 `openclaw`를 생성합니다:

- **Shell:** `nologin` — 대화형 로그인을 허용하지 않아 공격 표면을 줄입니다.

- **Home:** 예: `/home/openclaw` — `~/.openclaw`(설정, 작업 공간) 및 실행 스크립트 `run-openclaw-podman.sh`를 포함합니다.

- **Rootless Podman:** 이 사용자는 **subuid** 및 **subgid** 범위를 가져야 합니다. 많은 배포판에서는 사용자를 생성할 때 이를 자동으로 할당합니다. 설정 중 경고가 출력되면 `/etc/subuid`와 `/etc/subgid`에 다음 줄을 추가하세요:

  ```text
  openclaw:100000:65536
  ```

  그런 다음 해당 사용자로 gateway를 시작하세요(예: cron 또는 systemd에서):

  ```bash
  sudo -u openclaw /home/openclaw/run-openclaw-podman.sh
  sudo -u openclaw /home/openclaw/run-openclaw-podman.sh setup
  ```

- **Config:** `/home/openclaw/.openclaw`에는 `openclaw`와 root만 접근할 수 있습니다. 설정을 수정하려면: gateway가 실행 중일 때 Control UI를 사용하거나, `sudo -u openclaw $EDITOR /home/openclaw/.openclaw/openclaw.json`을 실행하세요.

## 환경 변수 및 설정

- **Token:** `~openclaw/.openclaw/.env`에 `OPENCLAW_GATEWAY_TOKEN`으로 저장됩니다. `setup-podman.sh`와 `run-openclaw-podman.sh`는 토큰이 없으면 이를 생성합니다(`openssl`, `python3` 또는 `od` 사용).
- **선택 사항:** 해당 `.env`에서 provider 키(예: `GROQ_API_KEY`, `OLLAMA_API_KEY`) 및 기타 OpenClaw 환경 변수를 설정할 수 있습니다.
- **Host 포트:** 기본적으로 스크립트는 `18789`(gateway)와 `18790`(bridge)을 매핑합니다. 실행 시 `OPENCLAW_PODMAN_GATEWAY_HOST_PORT` 및 `OPENCLAW_PODMAN_BRIDGE_HOST_PORT`로 **host** 포트 매핑을 재정의할 수 있습니다.
- **경로:** 호스트 설정 및 작업 공간의 기본 경로는 `~openclaw/.openclaw` 및 `~openclaw/.openclaw/workspace`입니다. 실행 스크립트에서 사용하는 호스트 경로는 `OPENCLAW_CONFIG_DIR` 및 `OPENCLAW_WORKSPACE_DIR`로 재정의할 수 있습니다.

## 유용한 명령어

- **로그:** quadlet 사용 시: `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`. 스크립트 사용 시: `sudo -u openclaw podman logs -f openclaw`
- **중지:** quadlet 사용 시: `sudo systemctl --machine openclaw@ --user stop openclaw.service`. 스크립트 사용 시: `sudo -u openclaw podman stop openclaw`
- **다시 시작:** quadlet 사용 시: `sudo systemctl --machine openclaw@ --user start openclaw.service`. 스크립트 사용 시: 실행 스크립트를 다시 실행하거나 `podman start openclaw`를 사용하세요.
- **컨테이너 제거:** `sudo -u openclaw podman rm -f openclaw` — 호스트의 설정 및 작업 공간은 유지됩니다.

## 문제 해결

- **config 또는 auth-profiles에서 Permission denied (EACCES) 발생:** 컨테이너는 기본적으로 `--userns=keep-id`를 사용하며, 스크립트를 실행하는 호스트 사용자와 동일한 uid/gid로 실행됩니다. 호스트의 `OPENCLAW_CONFIG_DIR` 및 `OPENCLAW_WORKSPACE_DIR`가 해당 사용자 소유인지 확인하세요.
- **Gateway 시작 차단(`gateway.mode=local` 누락):** `~openclaw/.openclaw/openclaw.json` 파일이 존재하고 `gateway.mode="local"`로 설정되어 있는지 확인하세요. `setup-podman.sh`는 파일이 없으면 이를 생성합니다.
- **openclaw 사용자에서 Rootless Podman 실패:** `/etc/subuid` 및 `/etc/subgid`에 `openclaw` 항목(예: `openclaw:100000:65536`)이 있는지 확인하세요. 없다면 추가한 후 재시작하세요.
- **컨테이너 이름이 이미 사용 중:** 실행 스크립트는 `podman run --replace`를 사용하므로, 다시 시작하면 기존 컨테이너가 교체됩니다. 수동으로 정리하려면: `podman rm -f openclaw`.
- **openclaw 사용자로 실행할 때 스크립트를 찾을 수 없음:** `run-openclaw-podman.sh`가 openclaw의 홈 디렉터리(예: `/home/openclaw/run-openclaw-podman.sh`)에 복사되도록 `setup-podman.sh`를 실행했는지 확인하세요.
- **Quadlet 서비스를 찾을 수 없거나 시작에 실패함:** `.container` 파일을 수정한 후 `sudo systemctl --machine openclaw@ --user daemon-reload`를 실행하세요. Quadlet은 cgroups v2가 필요합니다: `podman info --format '{{.Host.CgroupsVersion}}'` 명령의 출력이 `2`여야 합니다.

## 선택 사항: 자신의 사용자로 실행

일반 사용자(전용 openclaw 사용자 없이)로 gateway를 실행하려면: 이미지를 빌드하고, `OPENCLAW_GATEWAY_TOKEN`이 포함된 `~/.openclaw/.env`를 생성한 뒤, `--userns=keep-id` 옵션과 `~/.openclaw`에 대한 마운트를 사용해 컨테이너를 실행하세요. 런치 스크립트는 openclaw 사용자 흐름에 맞게 설계되었습니다. 단일 사용자 설정의 경우, 스크립트에 있는 `podman run` 명령을 수동으로 실행하고, 설정 및 작업 디렉터리를 자신의 홈으로 지정하면 됩니다. 대부분의 사용자에게 권장: `setup-podman.sh`를 사용하고 openclaw 사용자로 실행하여 설정과 프로세스를 분리하세요.

