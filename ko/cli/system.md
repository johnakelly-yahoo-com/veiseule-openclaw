---
title: "system"
---

# `openclaw system`

Gateway(게이트웨이)를 위한 시스템 수준 도우미입니다. 시스템 이벤트를 큐에 넣고, 하트비트를 제어하며,
프레즌스를 확인합니다.

## 일반 명령어

```bash
openclaw system event --text "Check for urgent follow-ups" --mode now
openclaw system heartbeat enable
openclaw system heartbeat last
openclaw system presence
```

## `system event`

**main** 세션에 시스템 이벤트를 큐에 넣습니다. 다음 하트비트가 이를 프롬프트에 `System:` 줄로
주입합니다. 하트비트를 즉시 트리거하려면 `--mode now`를 사용하고,
다음 예약된 틱까지 기다리려면 `next-heartbeat`를 사용합니다.

플래그:

- `--text <text>`: 필수 시스템 이벤트 텍스트입니다.
- `--mode <mode>`: `now` 또는 `next-heartbeat` (기본값)입니다.
- `--json`: 기계 판독 가능한 출력입니다.

## `system heartbeat last|enable|disable`

하트비트 제어:

- `last`: 마지막 하트비트 이벤트를 표시합니다.
- `enable`: 하트비트를 다시 켭니다 (비활성화되어 있었던 경우 사용).
- `disable`: 하트비트를 일시 중지합니다.

플래그:

- `--json`: 기계 판독 가능한 출력입니다.

## `system presence`

Gateway(게이트웨이)가 인지하고 있는 현재 시스템 프레즌스 항목(노드,
인스턴스 및 유사한 상태 라인)을 나열합니다.

플래그:

- `--json`: 기계 판독 가능한 출력입니다.

## 참고 사항

- 현재 설정(로컬 또는 원격)으로 접근 가능한 실행 중인 Gateway(게이트웨이)가 필요합니다.
- 시스템 이벤트는 일시적이며 재시작 간에 유지되지 않습니다.
