# Hooks — 이벤트 기반 자동 실행

## Hooks란?

Skill이 **사용자가 직접 호출하는 레시피**라면, Hooks는 **특정 이벤트가 발생했을 때 자동으로 실행되는 스크립트**다.

"파일을 저장할 때마다 포맷팅 실행", "커밋 전에 린트 체크" 같은 자동화가 Hooks로 가능하다.

## 노코드 도구에서의 대응 개념

| 노코드 도구 | Hooks 대응 |
|---|---|
| **Zapier의 Trigger** | Hook 이벤트 (무엇이 발생했을 때) |
| **Make의 Watch Module** | 특정 이벤트를 감지하여 자동 실행 |
| **GitHub Actions의 on: push** | 이벤트 기반 자동 워크플로우 |
| **Notion의 자동화 규칙** | "페이지 생성 시 → 템플릿 적용" 같은 자동 규칙 |

**핵심 차이**: 노코드 도구는 GUI에서 트리거를 설정하지만, Hooks는 **설정 파일(settings.json)에 셸 명령을 등록**하는 방식이다.

## 동작 원리

```
Claude Code 작업 흐름:

  사용자 요청 → Claude 도구 실행 → [Hook 체크] → 결과 반환
                                       │
                                       ▼
                              이벤트 매칭되면?
                              ├─ Yes → 셸 명령 실행 → 결과를 Claude에 전달
                              └─ No  → 패스
```

Hooks는 Claude Code의 도구 실행 **전후**에 개입할 수 있다:

| 시점 | 설명 | 용도 |
|---|---|---|
| **PreToolUse** | 도구 실행 **전** | 실행 차단, 조건부 허용, 경고 |
| **PostToolUse** | 도구 실행 **후** | 포맷팅, 린트, 로깅, 알림 |
| **Notification** | Claude가 알림을 보낼 때 | 데스크톱 알림, Slack 알림 연동 |
| **Stop** | Claude가 응답을 마칠 때 | 최종 검증, 자동 후처리 |

## 설정 방법

`~/.claude/settings.json` 또는 프로젝트의 `.claude/settings.json`에 추가:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Write",
        "hooks": [
          {
            "type": "command",
            "command": "echo 'File write detected: $CLAUDE_FILE_PATH'"
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Write",
        "hooks": [
          {
            "type": "command",
            "command": "npx prettier --write \"$CLAUDE_FILE_PATH\" 2>/dev/null || true"
          }
        ]
      }
    ]
  }
}
```

### 설정 구조

| 필드 | 설명 | 예시 |
|---|---|---|
| `matcher` | 어떤 도구에 반응할지 | `"Write"`, `"Bash"`, `"Read"` |
| `type` | Hook 유형 | `"command"` (셸 명령 실행) |
| `command` | 실행할 셸 명령 | `"npx prettier --write ..."` |

### 사용 가능한 환경 변수

Hook 명령 안에서 Claude Code가 제공하는 환경 변수를 사용할 수 있다:

| 변수 | 설명 |
|---|---|
| `$CLAUDE_FILE_PATH` | 대상 파일 경로 |
| `$CLAUDE_TOOL_NAME` | 실행된 도구 이름 |
| `$CLAUDE_TOOL_INPUT` | 도구에 전달된 입력 (JSON) |

## 실전 시나리오

### 1. 파일 저장 시 자동 포맷팅

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write",
        "hooks": [
          {
            "type": "command",
            "command": "npx prettier --write \"$CLAUDE_FILE_PATH\" 2>/dev/null || true"
          }
        ]
      }
    ]
  }
}
```

**효과**: Claude가 파일을 작성할 때마다 Prettier가 자동으로 코드 포맷을 정리한다. 노코드 도구 비유로는 "문서 저장 시 자동 정렬"과 같다.

### 2. 위험한 명령 차단

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "if echo \"$CLAUDE_TOOL_INPUT\" | grep -q 'rm -rf'; then echo 'BLOCKED: rm -rf is not allowed' >&2; exit 1; fi"
          }
        ]
      }
    ]
  }
}
```

**효과**: `rm -rf` 같은 위험한 명령을 Claude가 실행하려 할 때 자동으로 차단한다.

### 3. 작업 완료 알림

```json
{
  "hooks": {
    "Notification": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "osascript -e 'display notification \"$CLAUDE_NOTIFICATION_MESSAGE\" with title \"Claude Code\"'"
          }
        ]
      }
    ]
  }
}
```

**효과**: Claude가 긴 작업을 마치면 macOS 알림을 띄워준다. 다른 일을 하다가도 완료를 놓치지 않는다.

### 4. 파일 변경 시 린트 체크

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write",
        "hooks": [
          {
            "type": "command",
            "command": "npx eslint \"$CLAUDE_FILE_PATH\" --fix 2>/dev/null || true"
          }
        ]
      }
    ]
  }
}
```

## Skill vs Hooks — 자동화 방식의 차이

| | **Skill** | **Hooks** |
|---|---|---|
| **트리거** | 사용자가 `/명령어`로 호출 | 이벤트 발생 시 자동 |
| **정의 방식** | 마크다운 (SKILL.md) | JSON 설정 + 셸 명령 |
| **실행 주체** | Claude가 지시를 읽고 실행 | 셸이 직접 명령 실행 |
| **용도** | 복잡한 멀티스텝 작업 | 단순하고 반복적인 자동 처리 |
| **비유** | 요리 레시피 (직접 시작) | 센서 + 자동 스프링클러 (조건 충족 시 자동) |
| **예시** | "/weekly-report" | "파일 저장 → 자동 포맷팅" |

## 노코드 도구 사용자를 위한 정리

Zapier/Make를 써봤다면 이렇게 이해하면 된다:

```
노코드 도구 구조:
  Trigger (이벤트 감지) → Action (실행) → Result (결과)

Claude Code Hooks 구조:
  이벤트 (도구 실행 전/후) → Shell Command (실행) → 결과를 Claude에 전달
```

차이점은 **액션이 셸 명령**이라는 것. GUI에서 클릭하는 대신 터미널 명령을 적는다. 하지만 원리는 동일하다: "X가 일어나면 자동으로 Y를 해라."

## 핵심 정리

- Hooks = **이벤트 기반 자동 실행** (도구 사용 전/후에 셸 명령 실행)
- PreToolUse(실행 전), PostToolUse(실행 후), Notification, Stop 4가지 시점
- `settings.json`에 JSON으로 설정
- Skill(수동 호출) + Hooks(자동 실행) 조합으로 완전한 자동화 구현
- 노코드 도구의 Trigger → Action 패턴과 동일한 구조
