---
summary: "`openclaw hooks` (에이전트 훅)용 CLI 참조"
read_when:
  - 에이전트 훅을 관리하려는 경우
  - 훅을 설치하거나 업데이트하려는 경우
title: "hooks"
---

# `openclaw hooks`

에이전트 훅을 관리합니다(`/new`, `/reset` 및 Gateway(게이트웨이) 시작과 같은 명령을 위한 이벤트 기반 자동화).

관련 항목:

- Hooks: [Hooks](/automation/hooks)
- 플러그인 훅: [Plugins](/tools/plugin#plugin-hooks)

## 모든 훅 나열

```bash
openclaw hooks list
```

워크스페이스, 관리됨, 번들 디렉토리에서 발견된 모든 훅을 나열합니다.

**옵션:**

- `--eligible`: 적격한 훅만 표시(요구 사항 충족)
- `--json`: JSON 으로 출력
- `-v, --verbose`: 누락된 요구 사항을 포함한 자세한 정보 표시

**출력 예시:**

```
Hooks (4/4 ready)

Ready:
  🚀 boot-md ✓ - Run BOOT.md on gateway startup
  📝 command-logger ✓ - Log all command events to a centralized audit file
  💾 session-memory ✓ - Save session context to memory when /new command is issued
  😈 soul-evil ✓ - Swap injected SOUL content during a purge window or by random chance
```

**예시(자세히):**

```bash
openclaw hooks list --verbose
```

부적격 훅에 대한 누락된 요구 사항을 표시합니다.

**예시(JSON):**

```bash
openclaw hooks list --json
```

프로그래밍 방식 사용을 위한 구조화된 JSON 을 반환합니다.

## 훅 정보 가져오기

```bash
openclaw hooks info <name>
```

특정 훅에 대한 자세한 정보를 표시합니다.

**인수:**

- `<name>`: 훅 이름(예: `session-memory`)

**옵션:**

- `--json`: JSON 으로 출력

**예시:**

```bash
openclaw hooks info session-memory
```

**출력:**

```
💾 session-memory ✓ Ready

Save session context to memory when /new command is issued

Details:
  Source: openclaw-bundled
  Path: /path/to/openclaw/hooks/bundled/session-memory/HOOK.md
  Handler: /path/to/openclaw/hooks/bundled/session-memory/handler.ts
  Homepage: https://docs.openclaw.ai/hooks#session-memory
  Events: command:new

Requirements:
  Config: ✓ workspace.dir
```

## 훅 적격성 확인

```bash
openclaw hooks check
```

훅 적격성 상태 요약을 표시합니다(준비됨 vs. 준비되지 않음 수).

**옵션:**

- `--json`: JSON 으로 출력

**출력 예시:**

```
Hooks Status

Total hooks: 4
Ready: 4
Not ready: 0
```

## 훅 활성화

```bash
openclaw hooks enable <name>
```

구성(`~/.openclaw/config.json`)에 추가하여 특정 훅을 활성화합니다.

**참고:** 플러그인에서 관리되는 훅은 `openclaw hooks list` 에서 `plugin:<id>` 로 표시되며,
여기서 활성화/비활성화할 수 없습니다. 대신 플러그인을 활성화/비활성화하십시오.

**인수:**

- `<name>`: 훅 이름(예: `session-memory`)

**예시:**

```bash
openclaw hooks enable session-memory
```

**출력:**

```
✓ Enabled hook: 💾 session-memory
```

**동작 내용:**

- 훅이 존재하고 적격한지 확인
- 구성의 `hooks.internal.entries.<name>.enabled = true` 업데이트
- 구성을 디스크에 저장

**활성화 후:**

- 훅이 다시 로드되도록 Gateway(게이트웨이)를 재시작하십시오(macOS 에서는 메뉴 막대 앱 재시작, 개발 환경에서는 게이트웨이 프로세스 재시작).

## 훅 비활성화

```bash
openclaw hooks disable <name>
```

구성을 업데이트하여 특정 훅을 비활성화합니다.

**인수:**

- `<name>`: 훅 이름(예: `command-logger`)

**예시:**

```bash
openclaw hooks disable command-logger
```

**출력:**

```
⏸ Disabled hook: 📝 command-logger
```

**비활성화 후:**

- 훅이 다시 로드되도록 Gateway(게이트웨이)를 재시작하십시오

## 훅 설치

```bash
openclaw hooks install <path-or-spec>
```

로컬 폴더/아카이브 또는 npm 에서 훅 팩을 설치합니다.

Npm 사양은 **registry-only** (패키지 이름 + 선택적 버전/태그)만 허용됩니다. Git/URL/file
사양은 허용되지 않습니다. 의존성 설치는 보안을 위해 `--ignore-scripts` 옵션으로 실행됩니다.

**동작 내용:**

- 훅 팩을 `~/.openclaw/hooks/<id>` 에 복사
- 설치된 훅을 `hooks.internal.entries.*` 에서 활성화
- 설치 기록을 `hooks.internal.installs` 에 저장

**옵션:**

- `-l, --link`: 복사 대신 로컬 디렉토리를 링크(`hooks.internal.load.extraDirs` 에 추가)

**지원되는 아카이브:** `.zip`, `.tgz`, `.tar.gz`, `.tar`

**예시:**

```bash
# Local directory
openclaw hooks install ./my-hook-pack

# Local archive
openclaw hooks install ./my-hook-pack.zip

# NPM package
openclaw hooks install @openclaw/my-hook-pack

# Link a local directory without copying
openclaw hooks install -l ./my-hook-pack
```

## 훅 업데이트

```bash
openclaw hooks update <id>
openclaw hooks update --all
```

설치된 훅 팩을 업데이트합니다(npm 설치만 해당).

**옵션:**

- `--all`: 추적 중인 모든 훅 팩 업데이트
- `--dry-run`: 쓰기 없이 변경 사항 미리보기

## 번들 훅

### session-memory

`/new` 를 실행할 때 세션 컨텍스트를 메모리에 저장합니다.

**활성화:**

```bash
openclaw hooks enable session-memory
```

**출력:** `~/.openclaw/workspace/memory/YYYY-MM-DD-slug.md`

**참고:** [session-memory 문서](/automation/hooks#session-memory)

### bootstrap-extra-files

`agent:bootstrap` 동안 추가 부트스트랩 파일(예: 모노레포 로컬 `AGENTS.md` / `TOOLS.md`)을 주입합니다.

**활성화:**

```bash
openclaw hooks enable bootstrap-extra-files
```

**참고:** [SOUL Evil Hook](/hooks/soul-evil)

### command-logger

모든 명령 이벤트를 중앙 감사 파일에 기록합니다.

**활성화:**

```bash
openclaw hooks enable command-logger
```

**출력:** `~/.openclaw/logs/commands.log`

**로그 보기:**

```bash
# Recent commands
tail -n 20 ~/.openclaw/logs/commands.log

# Pretty-print
cat ~/.openclaw/logs/commands.log | jq .

# Filter by action
grep '"action":"new"' ~/.openclaw/logs/commands.log | jq .
```

**참고:** [command-logger 문서](/automation/hooks#command-logger)

### boot-md

Gateway(게이트웨이) 시작 시(채널 시작 이후) `BOOT.md` 를 실행합니다.

**활성화**:

**이벤트**: `gateway:startup`

```bash
openclaw hooks enable boot-md
```

**참고:** [boot-md 문서](/automation/hooks#boot-md)
