# Claude Code 내부 아키텍처 — 소스 코드가 말해주는 것들

## Claude Code는 어떻게 만들어졌는가?

2026년 3월 31일, Claude Code의 소스 코드가 공개되었다. **512,000줄의 TypeScript**, 1,884개 파일. 이 코드를 분석하면 우리가 매일 쓰는 도구가 내부에서 어떻게 작동하는지 정확히 알 수 있다.

Claude Code는 세 가지 설계 원칙 위에 만들어졌다:

| 원칙 | 의미 | 구현 예시 |
|---|---|---|
| **Safety** | 사용자 시스템을 보호 | Permission Pipeline, 도구 실행 전 권한 확인 |
| **Performance** | 빠른 응답과 효율적 리소스 사용 | Lazy Import, Memoized Context, 병렬 도구 실행 |
| **Extensibility** | 새 기능을 쉽게 추가 | MCP 프로토콜, Hook 시스템, Plugin 구조 |

기술 스택은 **TypeScript + React Ink**(터미널 UI 렌더링) + **Zustand**(상태 관리) + **Bun**(런타임)으로 구성된다.

## 노코드 도구에서의 대응 개념

### 4단계 실행 모델 = Zapier의 Trigger-Filter-Action-Output

Claude Code는 내부적으로 4단계를 거쳐 동작한다. 이것은 Zapier의 워크플로우 구조와 정확히 대응된다:

```
Zapier:
  Trigger → Filter → Action → Output

Claude Code:
  Startup → Query Loop → Tool Execution → Display
```

| Zapier | Claude Code | 설명 |
|---|---|---|
| Trigger (이벤트 감지) | Startup (초기화) | 환경 로딩, CLAUDE.md 파싱, 권한 확인 |
| Filter (조건 검사) | Query Loop (쿼리 분석) | 사용자 입력 해석, 어떤 도구를 쓸지 결정 |
| Action (실행) | Tool Execution (도구 실행) | 파일 읽기, 코드 작성, 명령 실행 |
| Output (결과) | Display (표시) | React Ink로 터미널에 결과 렌더링 |

### 7가지 실행 모드 = Notion AI의 다양한 뷰

Notion에서 같은 데이터베이스를 테이블, 보드, 갤러리 등 다른 뷰로 볼 수 있는 것처럼, Claude Code도 같은 코어 엔진을 **7가지 모드**로 실행할 수 있다:

```
같은 엔진, 다른 인터페이스:
  Notion: Table View / Board View / Gallery View / Calendar View
  Claude Code: Interactive / Non-interactive / Pipe / Print / API / MCP / Tool
```

### Context Injection = Cursor의 .cursorrules 로딩

Cursor가 프로젝트를 열 때 `.cursorrules` 파일을 자동으로 읽어서 AI의 동작을 커스터마이징하는 것처럼, Claude Code도 시작 시 **CLAUDE.md, Memory, 환경 정보**를 자동으로 컨텍스트에 주입한다.

```
Cursor 시작:
  .cursorrules 읽기 → AI에 규칙 적용

Claude Code 시작:
  CLAUDE.md 로드 → Memory 로드 → 환경 감지 → 컨텍스트 구성
```

## 실제 동작 원리

### Startup: 6단계 초기화

Claude Code를 실행하면 화면에 프롬프트가 뜨기 전에 6단계가 순서대로 실행된다:

```
1. 환경 감지 (OS, Shell, Git 상태)
2. 설정 파일 로드 (settings.json)
3. CLAUDE.md 파싱 (프로젝트 + 글로벌)
4. Auto Memory 로드
5. MCP 서버 연결
6. Permission 설정 확인
```

### 7가지 실행 모드

| 모드 | 설명 | 사용 예 |
|---|---|---|
| **Interactive** | 터미널에서 대화형 사용 | `claude` 실행 후 직접 대화 |
| **Non-interactive** | 명령 하나를 실행하고 종료 | `claude -p "이 파일 분석해줘"` |
| **Pipe** | stdin/stdout으로 데이터 전달 | `cat file.ts \| claude -p "리뷰해줘"` |
| **Print** | 시스템 프롬프트만 출력 | 디버깅/검증 용도 |
| **API** | 외부에서 HTTP로 호출 | 다른 프로그램에서 Claude Code 사용 |
| **MCP** | MCP 서버로 동작 | 다른 AI 에이전트가 Claude Code를 도구로 사용 |
| **Tool** | SDK 도구로 동작 | 프로그래밍 방식으로 통합 |

### 4단계 실행 모델 상세

```
Phase 1: Startup
  ┌─────────────────────────────────────┐
  │ 환경 감지 → 설정 로드 → CLAUDE.md   │
  │ → Memory → MCP → Permission        │
  └─────────────┬───────────────────────┘
                │
Phase 2: Query Loop
  ┌─────────────▼───────────────────────┐
  │ 사용자 입력 수신                      │
  │ → 시스템 프롬프트 + 컨텍스트 조합     │
  │ → API 호출 (SSE 스트리밍)            │
  └─────────────┬───────────────────────┘
                │
Phase 3: Tool Execution
  ┌─────────────▼───────────────────────┐
  │ 응답에 도구 호출이 있으면?            │
  │ ├─ Yes → 권한 확인 → 도구 실행       │
  │ │        → 결과를 컨텍스트에 추가     │
  │ │        → Phase 2로 돌아감          │
  │ └─ No  → Phase 4로                  │
  └─────────────┬───────────────────────┘
                │
Phase 4: Display
  ┌─────────────▼───────────────────────┐
  │ React Ink으로 터미널 UI 렌더링       │
  │ → 마크다운 포맷팅                    │
  │ → 다음 입력 대기                     │
  └─────────────────────────────────────┘
```

### Agent Loop: 11단계

Phase 2-3이 반복되는 것이 **Agent Loop**다. 세부적으로는 11단계로 나뉜다:

| 단계 | 처리 내용 |
|---|---|
| 1 | 사용자 메시지 수신 |
| 2 | 시스템 프롬프트 조합 (CLAUDE.md + Memory + 환경) |
| 3 | API 요청 전송 (SSE 스트리밍) |
| 4 | 응답 토큰 수신 시작 |
| 5 | 도구 호출 감지 |
| 6 | 권한 확인 (Permission Pipeline) |
| 7 | Hook 실행 (PreToolUse) |
| 8 | 도구 실행 |
| 9 | Hook 실행 (PostToolUse) |
| 10 | 결과를 컨텍스트에 추가 |
| 11 | 추가 도구 호출 필요 여부 판단 → 있으면 3번으로, 없으면 응답 완료 |

### 소스 디렉토리 구조

```
claude-code/src/
├── utils/          564 파일  — 유틸리티 함수 (가장 큰 디렉토리)
├── components/     389 파일  — React Ink UI 컴포넌트
├── tools/          287 파일  — 도구 정의 (Read, Write, Bash 등)
├── hooks/          156 파일  — Hook 시스템
├── mcp/            134 파일  — MCP 클라이언트/서버
├── permissions/     98 파일  — 권한 관리
├── memory/          87 파일  — Memory 시스템
└── agents/          69 파일  — Subagent, Agent Loop
```

`utils/`가 564 파일로 가장 큰 이유는 컨텍스트 관리, 토큰 계산, 파일 시스템 작업, 캐시 등 핵심 인프라가 여기에 집중되어 있기 때문이다.

## 실전 시나리오

### "메시지를 입력하면 내부에서 무슨 일이 일어나는가"

`src/auth/ 폴더의 로그인 흐름을 분석해줘`라고 입력했을 때의 전체 과정:

```
1. [Query Loop] 메시지 수신
   → "src/auth/ 폴더의 로그인 흐름을 분석해줘"

2. [Query Loop] 시스템 프롬프트 조합
   → CLAUDE.md 규칙 + Auto Memory + 현재 Git 상태 + OS 정보

3. [Query Loop] API 요청 (SSE 스트리밍)
   → Claude에게 컨텍스트 전체를 전송

4. [Tool Execution] Claude가 도구 호출 결정
   → "먼저 src/auth/ 디렉토리를 확인해야 한다" → Glob 도구 호출

5. [Permission] 권한 확인
   → Glob은 읽기 전용 → 자동 허용

6. [Hook] PreToolUse 체크
   → 등록된 Hook 없음 → 패스

7. [Tool Execution] Glob 실행
   → src/auth/login.ts, src/auth/session.ts, src/auth/middleware.ts 발견

8. [Query Loop] 결과를 컨텍스트에 추가 → 다시 API 호출
   → "이제 이 파일들을 읽어야 한다" → Read 도구 3회 호출

9. [Tool Execution] Read 3회 병렬 실행 (안전한 도구이므로)
   → 3개 파일 내용을 동시에 읽어옴

10. [Query Loop] 파일 내용을 컨텍스트에 추가 → 다시 API 호출
    → Claude가 분석 결과를 생성

11. [Display] React Ink로 마크다운 포맷팅하여 터미널에 렌더링
    → 사용자에게 분석 결과 표시
```

이 과정에서 Agent Loop는 **3회 반복**되었다 (Glob 1회 + Read 3회 병렬 + 최종 응답). 이것이 Claude Code가 단순한 챗봇이 아닌 **에이전트**인 이유다 — 스스로 판단하고, 도구를 선택하고, 결과를 바탕으로 다음 행동을 결정한다.

## 핵심 정리

- Claude Code = **512K줄 TypeScript**, 1,884 파일로 구성된 에이전트 시스템
- 기술 스택: TypeScript + React Ink(TUI) + Zustand(상태) + Bun(런타임)
- **3대 설계 원칙**: Safety, Performance, Extensibility
- **4단계 실행 모델**: Startup → Query Loop → Tool Execution → Display
- **7가지 모드**: Interactive, Non-interactive, Pipe, Print, API, MCP, Tool
- **Agent Loop 11단계**가 핵심 — 도구 호출이 필요 없을 때까지 반복
- 소스 구조: utils(564) > components(389) > tools(287) 순으로 큰 디렉토리
