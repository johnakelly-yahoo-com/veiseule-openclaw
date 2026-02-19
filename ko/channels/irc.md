---
title: IRC
description: OpenClaw을 IRC 채널 및 다이렉트 메시지에 연결합니다.
---

OpenClaw을 클래식 채널(`#room`)과 다이렉트 메시지에서 사용하려면 IRC를 사용하세요.
IRC는 확장 플러그인으로 제공되지만, 설정은 메인 구성의 `channels.irc` 아래에서 진행합니다.

## 빠른 시작

1. `~/.openclaw/openclaw.json`에서 IRC 설정을 활성화하세요.
2. 최소한 다음을 설정하세요:

```json
{
  "channels": {
    "irc": {
      "enabled": true,
      "host": "irc.libera.chat",
      "port": 6697,
      "tls": true,
      "nick": "openclaw-bot",
      "channels": ["#openclaw"]
    }
  }
}
```

3. gateway를 시작/재시작하세요:

```bash
openclaw gateway run
```

## 기본 보안 설정

- `channels.irc.dmPolicy`의 기본값은 `"pairing"`입니다.
- `channels.irc.groupPolicy`의 기본값은 `"allowlist"`입니다.
- `groupPolicy="allowlist"`를 사용하는 경우, 허용할 채널을 정의하기 위해 `channels.irc.groups`를 설정하세요.
- 의도적으로 평문 전송을 허용하는 경우가 아니라면 TLS(`channels.irc.tls=true`)를 사용하세요.

## 접근 제어

IRC 채널에는 두 개의 별도 “게이트”가 있습니다:

1. **채널 접근** (`groupPolicy` + `groups`): 봇이 해당 채널의 메시지를 아예 수신할지 여부를 결정합니다.
2. **발신자 접근** (`groupAllowFrom` / 채널별 `groups["#channel"].allowFrom`): 해당 채널 내에서 누가 봇을 트리거할 수 있는지 결정합니다.

설정 키:

- DM 허용 목록(DM 발신자 접근): `channels.irc.allowFrom`
- 그룹 발신자 허용 목록(채널 발신자 접근): `channels.irc.groupAllowFrom`
- 채널별 제어(채널 + 발신자 + 멘션 규칙): `channels.irc.groups["#channel"]`
- `channels.irc.groupPolicy="open"`은 구성되지 않은 채널도 허용합니다 (**기본적으로 여전히 멘션 게이트 적용**)

허용 목록 항목에는 nick 또는 `nick!user@host` 형식을 사용할 수 있습니다.

### 자주 하는 실수: `allowFrom`은 채널이 아닌 DM에 대한 설정입니다.

다음과 같은 로그가 보인다면:

- `irc: drop group sender alice!ident@host (policy=allowlist)`

…이는 해당 발신자가 **그룹/채널** 메시지에 대해 허용되지 않았음을 의미합니다. 다음 중 하나로 해결할 수 있습니다:

- `channels.irc.groupAllowFrom`을 설정하거나(모든 채널에 전역 적용),
- 채널별 발신자 허용 목록을 설정하세요: `channels.irc.groups["#channel"].allowFrom`

예시 (`#tuirc-dev`에서 누구나 봇과 대화할 수 있도록 허용):

```json5
{
  channels: {
    irc: {
      groupPolicy: "allowlist",
      groups: {
        "#tuirc-dev": { allowFrom: ["*"] },
      },
    },
  },
}
```

## 응답 트리거(멘션)

채널이 허용되어 있고(`groupPolicy` + `groups`를 통해) 발신자도 허용된 경우에도, OpenClaw은 그룹 환경에서 기본적으로 **멘션 게이트**를 적용합니다.

즉, 다음과 같은 로그가 표시될 수 있습니다: `drop channel … (missing-mention)` — 메시지에 봇과 일치하는 멘션 패턴이 포함되지 않은 경우입니다.

IRC 채널에서 **멘션 없이도** 봇이 응답하도록 하려면, 해당 채널의 멘션 게이트를 비활성화하세요:

```json5
{
  channels: {
    irc: {
      groupPolicy: "allowlist",
      groups: {
        "#tuirc-dev": {
          requireMention: false,
          allowFrom: ["*"],
        },
      },
    },
  },
}
```

또는 **모든** IRC 채널을 허용하고(채널별 allowlist 없이) 멘션 없이도 응답하도록 하려면:

```json5
{
  channels: {
    irc: {
      groupPolicy: "open",
      groups: {
        "*": { requireMention: false, allowFrom: ["*"] },
      },
    },
  },
}
```

## 보안 참고 사항 (공개 채널에 권장)

공개 채널에서 `allowFrom: ["*"]`를 허용하면 누구나 봇에 프롬프트를 보낼 수 있습니다.
위험을 줄이려면 해당 채널에서 사용할 수 있는 도구를 제한하세요.

### 채널의 모든 사용자에게 동일한 도구 적용

```json5
{
  channels: {
    irc: {
      groups: {
        "#tuirc-dev": {
          allowFrom: ["*"],
          tools: {
            deny: ["group:runtime", "group:fs", "gateway", "nodes", "cron", "browser"],
          },
        },
      },
    },
  },
}
```

### 발신자별로 다른 도구 적용 (owner는 더 많은 권한 보유)

`toolsBySender`를 사용하여 `"*"`에는 더 엄격한 정책을, 자신의 닉에는 더 완화된 정책을 적용하세요:

```json5
{
  channels: {
    irc: {
      groups: {
        "#tuirc-dev": {
          allowFrom: ["*"],
          toolsBySender: {
            "*": {
              deny: ["group:runtime", "group:fs", "gateway", "nodes", "cron", "browser"],
            },
            eigen: {
              deny: ["gateway", "nodes", "cron"],
            },
          },
        },
      },
    },
  },
}
```

참고:

- `toolsBySender`의 키는 닉네임(예: `"eigen"`) 또는 보다 강력한 신원 매칭을 위한 전체 hostmask(`"eigen!~eigen@174.127.248.171"`)가 될 수 있습니다.
- 가장 먼저 일치하는 발신자 정책이 적용되며, `"*"`는 와일드카드 기본값입니다.

그룹 접근과 멘션 게이팅(그리고 이들이 어떻게 상호작용하는지)에 대한 자세한 내용은 다음을 참고하세요: [/channels/groups](/channels/groups).

## NickServ

연결 후 NickServ로 인증하려면:

```json
{
  "channels": {
    "irc": {
      "nickserv": {
        "enabled": true,
        "service": "NickServ",
        "password": "your-nickserv-password"
      }
    }
  }
}
```

연결 시 선택적 1회 등록:

```json
{
  "channels": {
    "irc": {
      "nickserv": {
        "register": true,
        "registerEmail": "bot@example.com"
      }
    }
  }
}
```

닉네임 등록이 완료된 후에는 반복적인 REGISTER 시도를 방지하기 위해 `register`를 비활성화하세요.

## 환경 변수

기본 계정은 다음을 지원합니다:

- `IRC_HOST`
- `IRC_PORT`
- `IRC_TLS`
- `IRC_NICK`
- `IRC_USERNAME`
- `IRC_REALNAME`
- `IRC_PASSWORD`
- `IRC_CHANNELS` (쉼표로 구분)
- `IRC_NICKSERV_PASSWORD`
- `IRC_NICKSERV_REGISTER_EMAIL`

## 문제 해결

- 봇이 연결되지만 채널에서 응답하지 않는 경우, `channels.irc.groups` **및** 멘션 게이팅으로 인해 메시지가 차단되고 있는지(`missing-mention`)를 확인하세요. 핑 없이 응답하도록 하려면 해당 채널에 `requireMention:false`를 설정하세요.
- 로그인에 실패하면 닉네임 사용 가능 여부와 서버 비밀번호를 확인하세요.
- 사용자 지정 네트워크에서 TLS에 실패하면 호스트/포트 및 인증서 설정을 확인하세요.
