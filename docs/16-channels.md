# Channels — 외부 시스템이 Claude에게 말을 거는 구조

## Channels란?

지금까지의 Claude Code는 항상 **내가 먼저 말을 걸어야** 반응했다. 터미널에 앉아서 메시지를 보내고, Claude가 답하는 구조.

Channels는 이 방향을 뒤집는다. **외부 시스템(Telegram, Discord, CI 파이프라인 등)이 Claude에게 먼저 이벤트를 push하고, Claude가 그것을 처리해서 같은 채널로 답장**하는 구조다.

터미널 앞에 없어도 된다. Telegram 메시지 하나가 도착하면, 내 컴퓨터에서 실행 중인 Claude가 그걸 읽고, 로컬 파일을 열고, 작업하고, 다시 Telegram으로 결과를 보낸다.

> 요약: Channels = **외부 세계가 Claude에게 말을 거는 창구**

## 노코드 도구와의 비교

| 노코드 도구 | Channels 대응 |
|---|---|
| **Zapier의 Webhook Trigger** | 외부 이벤트가 도착하면 워크플로우 시작 |
| **Make의 Webhook 모듈** | CI 이벤트, 모니터링 알림을 받아서 처리 |
| **Slack Bot** | 채팅에 메시지 보내면 봇이 응답 |
| **Discord Bot** | 서버/DM에서 봇과 양방향 대화 |
| **n8n의 Webhook Node** | HTTP 요청 받아 자동화 시작 |

핵심 차이:
- 노코드 봇은 **클라우드에서 실행** → 로컬 파일 접근 불가
- Channels는 **내 컴퓨터의 Claude가 처리** → 로컬 파일, Git, MCP 서버 모두 사용 가능
- 노코드는 **설정된 자동화만** 실행 → Channels는 **Claude가 상황 판단**해서 처리

## 동작 원리

```
┌─ 외부 ───────────────────────────────────────────────┐
│                                                      │
│  Telegram                Discord               CI    │
│   폰 메시지               DM / 서버              알림  │
│      │                      │                   │    │
└──────┼──────────────────────┼───────────────────┼────┘
       │                      │                   │
       ▼                      ▼                   ▼
┌─ Channel Plugin (MCP 서버) ────────────────────────────┐
│                                                        │
│  메시지/이벤트를 수신 → <channel> 이벤트으로 변환         │
│                                                        │
└────────────────────────┬───────────────────────────────┘
                         │ push
                         ▼
┌─ 내 컴퓨터 — Claude Code 세션 (실행 중) ────────────────┐
│                                                        │
│  <channel source="telegram"> 이벤트 수신               │
│  → Claude가 읽고 판단                                   │
│  → 로컬 파일/Git/MCP 서버로 작업                         │
│  → reply 도구로 같은 채널에 답장                          │
│                                                        │
└────────────────────────────────────────────────────────┘
```

**핵심 구조**:
- Channel Plugin은 **MCP 서버의 특수한 형태** — 단, 일반 MCP와 달리 Claude가 pull하지 않고 이벤트가 **push**됨
- 이벤트는 오직 **세션이 열려 있는 동안에만** 도착 (백그라운드 유지 필요)
- 터미널 출력: 수신 메시지는 보이지만, **답장 내용은 터미널에 표시되지 않고 채널로 직접 전달**

## Channels vs Remote Control vs Scheduled Tasks

세 기능 모두 "터미널 밖에서 Claude를 다룬다"는 인상이 있지만, **누가 주체인지**가 완전히 다르다.

| | **Remote Control** | **Channels** | **Scheduled Tasks** |
|---|---|---|---|
| **주체** | 내가 세션을 모바일에서 직접 조종 | 외부 시스템이 Claude에게 이벤트 push | 시간이 되면 Claude가 자동 실행 |
| **트리거** | 내가 직접 메시지 입력 | 외부 이벤트 도착 (메시지, 알림, webhook) | 설정된 시간/주기 |
| **흐름 방향** | 나 → Claude | 외부 → Claude → 외부 | Claude 자동 실행 |
| **적합한 상황** | 진행 중 작업을 이어서 조종 | CI 알림 처리, 봇 응답, 원격 지시 | 정기 보고, 주기적 모니터링 |
| **비유** | TV 리모컨 | 초인종 | 알람 시계 |

> **쉬운 판단 기준**:
> "내가 폰으로 명령하고 싶다" → Remote Control
> "외부 시스템이 Claude에게 알림 보내고 처리시키고 싶다" → Channels
> "시간 되면 자동으로 돌려놓고 싶다" → Scheduled Tasks

## Channels vs 일반 MCP

같은 Plugin/MCP 구조를 쓰지만 동작 방식이 다르다.

| | **일반 MCP 서버** | **Channel** |
|---|---|---|
| **방향** | Claude가 필요할 때 pull | 외부가 Claude에게 push |
| **트리거** | Claude의 작업 중 "DB에서 데이터 가져와" | 외부 메시지/이벤트 도착 |
| **예시** | Notion MCP, Slack MCP (조회용) | Telegram Channel, Discord Channel |
| **비유** | 도서관 사서 (물어보면 꺼내줌) | 우체부 (편지가 도착하면 초인종) |

## 지원 채널 (리서치 프리뷰)

현재 공식 지원 채널:

| 채널 | 플러그인 이름 | 용도 |
|---|---|---|
| **Telegram** | `telegram@claude-plugins-official` | 봇 DM, 그룹 메시지 |
| **Discord** | `discord@claude-plugins-official` | DM, 서버 채널 |
| **fakechat** | `fakechat@claude-plugins-official` | localhost 데모 (테스트용) |

> `fakechat`은 브라우저(`http://localhost:8787`)에서 바로 테스트할 수 있는 공식 데모 채널이다. 실제 앱 계정 없이 Channel 동작을 체험할 때 유용하다.

## 설치 및 설정 — Telegram 기준

### 사전 준비

```bash
# Bun이 필요 (Channel Plugin은 Bun 스크립트)
bun --version        # 없으면 설치 필요
```

Bun 설치: https://bun.sh/docs/installation

### 1단계: 플러그인 설치

```
/plugin install telegram@claude-plugins-official
```

처음이라면 Marketplace 등록 먼저:
```
/plugin marketplace add anthropics/claude-plugins-official
```

### 2단계: 봇 토큰 설정

[BotFather](https://t.me/BotFather)에서 `/newbot` → 토큰 복사 후:

```
/telegram:configure <토큰>
```

→ `~/.claude/channels/telegram/.env`에 저장됨

### 3단계: 채널 활성화 후 실행

```bash
claude --channels plugin:telegram@claude-plugins-official
```

### 4단계: 페어링 (첫 실행 시 1회)

1. Telegram에서 내 봇에게 아무 메시지나 보냄
2. 봇이 페어링 코드를 반환
3. Claude Code에서 입력:

```
/telegram:access pair <코드>
/telegram:access policy allowlist    ← 내 계정만 허용 (보안)
```

### Discord는?

구조는 동일하다. [Discord 개발자 포털](https://discord.com/developers/applications)에서 봇 생성 → Message Content Intent 활성화 → 서버 초대 → 토큰 설정.

```
/plugin install discord@claude-plugins-official
/discord:configure <토큰>
claude --channels plugin:discord@claude-plugins-official
```

## 보안 구조

모든 Channel Plugin은 **발신자 허용 목록(allowlist)** 을 유지한다.

```
허용 목록에 없는 계정이 메시지를 보냄
        → 조용히 무시됨 (오류 없음)

허용 목록에 있는 계정이 메시지를 보냄
        → Claude Code에 channel 이벤트로 전달
```

페어링 이후 `/telegram:access policy allowlist`를 설정하면, **등록된 계정만** Claude에게 접근 가능하다.

추가로:
- Channel이 활성화되려면 `--channels` 플래그가 반드시 필요 (`.mcp.json`에만 있으면 메시지 수신 불가)
- Team/Enterprise 플랜은 관리자가 `channelsEnabled` 설정을 켜야 사용 가능

## 실전 활용 시나리오

### 1. 스마트폰 원격 지시
```
[상황] 외출 중. 집 컴퓨터에서 Claude Code가 돌아가고 있음

Telegram 보냄: "오늘 변경한 파일 목록이랑 Git 상태 알려줘"

→ 내 컴퓨터의 Claude가 Git 명령어 실행
→ 결과를 Telegram으로 답장
→ 폰에서 바로 확인
```

### 2. CI 실패 알림 → 즉시 분석
```
[상황] Claude Code 세션을 열어두고 백그라운드로 CI 실행 중

Webhook → Channels: "빌드 실패: TypeError in auth.ts line 42"

→ Claude가 해당 파일을 열고 에러 분석
→ Discord로 "auth.ts 42번째 줄에서 undefined 참조. 수정안 커밋할까요?" 답장
→ 폰에서 "네" 보내면 Claude가 실제로 파일 수정 + 커밋
```

### 3. 장시간 작업 모니터링 + 개입
```
[상황] 큰 리팩토링 시작, 한두 시간 걸릴 예정

claude --channels plugin:telegram@claude-plugins-official 로 실행

중간에 Telegram: "지금 어디까지 됐어?"
→ Claude: "src/api/ 완료, src/db/ 작업 중 (43%)"

Telegram: "db/legacy.ts는 건드리지 마"
→ Claude가 즉시 방향 수정
```

## 사용 조건

| 항목 | 조건 |
|---|---|
| Claude Code 버전 | v2.1.80 이상 |
| 인증 방식 | claude.ai 로그인 (Console/API 키 미지원) |
| 의존성 | Bun 설치 필요 |
| 플랜 | Pro/Max (개인), Enterprise는 관리자 활성화 필요 |
| 상태 | 리서치 프리뷰 (--channels 플래그 opt-in 방식) |

## 현재 학습 내용과의 관계

```
Channels = "push 방향의 MCP" + "이벤트 기반 자동화"

연결 개념:
  ├── MCP (07)        ← Channel은 MCP의 특수 형태 (단, push 방향)
  ├── Plugins (13)    ← Channel은 Plugin으로 설치/관리
  ├── Hooks (08)      ← Hooks는 Claude Code 내부 이벤트 / Channels는 외부 이벤트
  └── Remote Control (15) ← 비슷해 보이지만 방향이 반대
```

## 핵심 정리

- Channels = **외부 이벤트가 실행 중인 세션으로 push**되는 구조 (MCP의 push 버전)
- Telegram, Discord 봇을 연결하면 **채팅 창이 Claude의 입출력 창구**가 됨
- 내 컴퓨터에서 실행 → **로컬 파일, Git, MCP 서버 전부 활용 가능**
- Remote Control(내가 조종)과 달리, Channels는 **외부 시스템이 Claude에게 말을 거는** 구조
- 보안: 허용 목록 기반 — 등록되지 않은 계정의 메시지는 무시
- `--channels` 플래그로 세션마다 opt-in, `Bun` 필수, **v2.1.80+** 필요
- 리서치 프리뷰 단계 — `--channels` 플래그 문법과 프로토콜은 변경될 수 있음

---

## Claude Code Catalog에서 실전 적용

이 챕터 개념을 바로 써보고 싶다면 → **[claude-code-catalog.vercel.app/ko/plugins](https://claude-code-catalog.vercel.app/ko/plugins)**

| 리소스 | 내용 |
|---|---|
| **Telegram Channel Plugin** | Telegram 봇 연결 전체 흐름 — install 명령어 + 토큰 설정 + 터미널 데모 |
| **Discord Channel Plugin** | Discord DM 연결 전체 흐름 |
| **News: Channels Research Preview** | Channels 기능 공식 발표 요약 (ko/en/ja) |
