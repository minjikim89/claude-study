# 설계 패턴 12선 — 프로덕션 코드에서 추출한 에이전트 패턴

## 왜 설계 패턴인가?

Claude Code 소스 코드에서 **8개 핵심 패턴 + 4개 인프라 패턴**을 추출했다. 이 패턴들은 단순한 코드 기법이 아니라, 에이전트 시스템이 안정적으로 동작하기 위한 **필수 설계 전략**이다.

일반 애플리케이션과 에이전트 시스템의 결정적 차이: 에이전트는 **다음에 무엇을 할지 예측 불가능**하다. 사용자 입력에 따라 파일을 읽을 수도, 명령을 실행할 수도, 아무것도 안 할 수도 있다. 이 불확실성 속에서 안정성을 유지하려면 견고한 패턴이 필요하다.

## 노코드 도구에서의 대응 개념

12개 패턴 각각을 익숙한 도구로 비유하면:

### 핵심 패턴 8개

| # | 패턴 | 노코드 비유 |
|---|---|---|
| 1 | **Generator Streaming** | Google Docs에서 상대방이 타이핑하는 것을 실시간으로 보는 것 |
| 2 | **Feature Gate** | Notion의 Feature Preview 토글 — 기능을 켜고 끌 수 있음 |
| 3 | **Memoized Context** | 브라우저가 자주 방문하는 페이지를 캐싱하는 것 |
| 4 | **Withhold & Recover** | 자동 교정(Autocorrect)이 오타를 눈치채기 전에 고치는 것 |
| 5 | **Lazy Import** | 앱이 현재 화면만 로드하고 나머지는 이동할 때 로드하는 것 |
| 6 | **Immutable State** | Google Sheets의 버전 히스토리 — 실수로 덮어쓸 수 없음 |
| 7 | **Interruption Resilience** | Google Docs의 자동 저장 — 브라우저가 죽어도 내용이 보존됨 |
| 8 | **Dependency Injection** | USB 포트 — 같은 포트에 다른 장치를 연결할 수 있음 |

### 인프라 패턴 4개

| # | 패턴 | 노코드 비유 |
|---|---|---|
| 9 | **Tool Concurrency** | 요리할 때 병렬 vs 순차 작업 — 야채 썰기는 동시에, 볶기는 순서대로 |
| 10 | **Auto-Compact** | 회의록 노트가 가득 찼을 때 핵심만 요약하는 것 |
| 11 | **Permission Pipeline** | 빌딩 보안 — 로비, 엘리베이터, 사무실 문 각각에서 체크 |
| 12 | **API Retry** | 전화가 끊겼을 때 자동 재다이얼 |

## 실제 동작 원리

### 패턴 1: Generator Streaming

**무엇**: API 응답을 한 번에 받지 않고, 토큰 단위로 실시간 스트리밍하여 표시.

```
일반적 방식:
  [====대기====] → [전체 응답 한 번에 표시]

Generator Streaming:
  [토][큰][하][나][씩] → [실시간으로 화면에 표시]
```

**Claude Code 구현**: `query()` 함수가 async generator로 구현되어 `yield`로 각 토큰을 반환. React Ink가 이를 실시간으로 렌더링.

### 패턴 2: Feature Gate

**무엇**: 런타임에 기능을 켜고 끌 수 있는 토글 시스템.

```
Feature Gate 체크:
  기능 X 사용 요청
  │
  ├─ Gate ON  → 기능 실행
  └─ Gate OFF → 대체 경로 또는 비활성
```

**Claude Code 구현**: 7가지 실행 모드, Beta 기능 토글, MCP 서버별 도구 활성화/비활성화가 모두 Feature Gate 패턴.

### 패턴 3: Memoized Context

**무엇**: 한 번 계산한 결과를 캐시하여 반복 계산을 방지.

```
첫 번째 호출:
  CLAUDE.md 파싱 → 결과 생성 → 캐시에 저장 → 반환

두 번째 호출:
  캐시 확인 → 결과 존재 → 바로 반환 (파싱 생략)
```

**Claude Code 구현**: 시스템 프롬프트, CLAUDE.md, 환경 정보가 세션 내에서 캐싱. 14개 캐시 벡터가 존재.

### 패턴 4: Withhold & Recover

**무엇**: 문제를 사전에 감지하여 사용자가 알아차리기 전에 복구.

```
일반적 에러 처리:
  실행 → 에러 발생 → 사용자에게 에러 표시 → 수동 복구

Withhold & Recover:
  실행 → 에러 감지 → 자동 재시도/복구 → 사용자는 모름
```

**Claude Code 구현**: 도구 실행 실패 시 자동 재시도, 컨텍스트 손상 시 자동 복구, API 오류 시 백오프 재시도.

### 패턴 5: Lazy Import

**무엇**: 모든 모듈을 시작 시 로드하지 않고, 필요할 때만 로드.

```
Eager Import (시작이 느림):
  시작 → [모듈A 로드][모듈B 로드][모듈C 로드] → 준비 완료

Lazy Import (시작이 빠름):
  시작 → [코어만 로드] → 준비 완료
  ... 나중에 필요할 때 → [모듈B 로드]
```

**Claude Code 구현**: MCP 서버 연결, Plugin 로드, 무거운 유틸리티가 모두 Lazy Import. 888KB의 초기 번들 크기를 최소화.

### 패턴 6: Immutable State

**무엇**: 상태를 직접 변경하지 않고, 새로운 상태를 생성하여 교체.

```
Mutable (위험):
  state.messages.push(newMsg)  → 이전 상태가 사라짐

Immutable (안전):
  newState = {...state, messages: [...state.messages, newMsg]}
  → 이전 상태가 보존됨
```

**Claude Code 구현**: Zustand 스토어에서 대화 히스토리를 Immutable하게 관리. 이전 상태로 롤백 가능.

### 패턴 7: Interruption Resilience

**무엇**: 작업 도중 중단되어도 데이터가 손실되지 않는 설계.

```
중단 시나리오:
  파일 쓰기 중 → Ctrl+C → ?

취약한 구현: 파일이 반만 쓰여서 손상
Resilient 구현: 임시 파일에 쓰기 → 완료 후 교체 (atomic write)
```

**Claude Code 구현**: Memory 자동 저장, 세션 상태 복구, 도구 실행 중 인터럽트 처리.

### 패턴 8: Dependency Injection (DI)

**무엇**: 구체적 구현 대신 인터페이스에 의존하여 교체 가능하게 만드는 것.

```
DI 없이:
  Agent Loop → 직접 Anthropic API 호출

DI 있으면:
  Agent Loop → API Interface → [Anthropic API / Mock API / Local Model]
  같은 코드가 다른 환경에서 동작 가능
```

**Claude Code 구현**: API 클라이언트, 파일 시스템 접근, 권한 체커가 모두 DI 패턴. 테스트 시 Mock으로 교체 가능.

### 패턴 9: Tool Concurrency

**무엇**: 안전한 도구는 병렬로, 위험한 도구는 순차로 실행하는 전략.

```
Safe 도구 (Read, Glob): 최대 10개 병렬 실행
Unsafe 도구 (Write, Bash): 1개씩 순차 실행
```

### 패턴 10: Auto-Compact

**무엇**: 컨텍스트 한계에 도달하면 Sub-agent가 자동으로 요약/압축.

```
임계값: context_window - 13,000 토큰
실패 시: Circuit Breaker 3회 후 중단
복원: 파일 50K + Skill 25K 재주입
```

### 패턴 11: Permission Pipeline

**무엇**: 도구 실행 전 여러 단계의 권한 확인을 통과해야 하는 구조.

```
도구 실행 요청
  │
  ▼
  1단계: 도구 유형 확인 (Safe/Unsafe)
  │
  ▼
  2단계: 프로젝트 설정 확인 (allowlist/denylist)
  │
  ▼
  3단계: 사용자 설정 확인 (자동 허용 여부)
  │
  ▼
  4단계: 사용자에게 확인 요청 (필요시)
  │
  ▼
  실행 허가 또는 차단
```

**Claude Code 구현**: 23단계의 권한 체크 파이프라인이 모든 도구 실행 전에 동작.

### 패턴 12: API Retry

**무엇**: API 호출 실패 시 지수 백오프(exponential backoff)로 재시도.

```
시도 1: 실패 → 1초 대기
시도 2: 실패 → 2초 대기
시도 3: 실패 → 4초 대기
시도 4: 성공 → 결과 반환

최대 재시도: 보통 3-5회
```

### 전체 패턴 요약 테이블

| # | 패턴 | 분류 | Claude Code 근거 | 핵심 수치 |
|---|---|---|---|---|
| 1 | Generator Streaming | 핵심 | query() async generator | 토큰 단위 실시간 전달 |
| 2 | Feature Gate | 핵심 | 7 실행 모드, Beta 토글 | 7개 모드 전환 |
| 3 | Memoized Context | 핵심 | 시스템 프롬프트 캐싱 | 14개 캐시 벡터 |
| 4 | Withhold & Recover | 핵심 | 자동 재시도, 에러 복구 | 사용자 무감지 복구 |
| 5 | Lazy Import | 핵심 | MCP/Plugin 지연 로드 | 888KB 번들 최소화 |
| 6 | Immutable State | 핵심 | Zustand 상태 관리 | 롤백 가능 |
| 7 | Interruption Resilience | 핵심 | Atomic write, 세션 복구 | 중단 시 무손실 |
| 8 | Dependency Injection | 핵심 | API/FS/Permission DI | Mock 테스트 가능 |
| 9 | Tool Concurrency | 인프라 | Safe=병렬, Unsafe=순차 | 최대 10 병렬 |
| 10 | Auto-Compact | 인프라 | Sub-agent 요약 | 250K/일 절감 |
| 11 | Permission Pipeline | 인프라 | 다단계 권한 체크 | 23단계 파이프라인 |
| 12 | API Retry | 인프라 | 지수 백오프 재시도 | 3-5회 재시도 |

## 실전 시나리오

### "내 프로젝트에는 어떤 패턴이 필요한가?"

프로젝트 유형별로 우선 적용할 패턴이 다르다:

**웹 앱 (Next.js, React 등)**

| 우선순위 | 패턴 | 이유 |
|---|---|---|
| 높음 | Lazy Import | 번들 크기가 직접적으로 로딩 속도에 영향 |
| 높음 | Immutable State | React 상태 관리의 기본 원칙 |
| 중간 | Feature Gate | A/B 테스트, 점진적 기능 배포에 필수 |
| 중간 | API Retry | 네트워크 불안정 환경 대비 |

**자동화 스크립트 (Python, Node.js)**

| 우선순위 | 패턴 | 이유 |
|---|---|---|
| 높음 | API Retry | 외부 API 의존도가 높은 스크립트의 안정성 |
| 높음 | Interruption Resilience | 긴 작업 중 중단 시 데이터 보호 |
| 중간 | Withhold & Recover | 에러가 나도 자동으로 복구하여 무인 실행 가능 |

**AI 에이전트 시스템**

| 우선순위 | 패턴 | 이유 |
|---|---|---|
| 높음 | Generator Streaming | 사용자 체감 속도 향상 |
| 높음 | Tool Concurrency | 에이전트 실행 효율 극대화 |
| 높음 | Auto-Compact | 긴 대화의 안정적 유지 |
| 높음 | Permission Pipeline | 에이전트의 행동 범위 제어 |
| 중간 | Memoized Context | 반복 연산 제거로 비용 절감 |
| 중간 | DI | 다양한 모델/도구 교체 지원 |

### 패턴 조합의 시너지

12개 패턴은 독립적으로도 유용하지만, 조합하면 시너지가 발생한다:

```
예: "사용자가 대규모 코드 분석을 요청했을 때"

1. Generator Streaming → 응답을 실시간 표시
2. Tool Concurrency → 10개 파일을 병렬로 읽기
3. Memoized Context → 이미 파싱한 CLAUDE.md를 캐시에서 재사용
4. Permission Pipeline → 각 파일 읽기 전 권한 확인
5. Auto-Compact → 컨텍스트가 차면 자동 요약
6. API Retry → 중간에 API 오류 나면 자동 재시도
```

6개 패턴이 하나의 요청을 처리하는 데 동시에 작동한다. 이것이 에이전트 시스템이 단순한 API 래퍼와 다른 점이다.

## 핵심 정리

- Claude Code 소스에서 **8개 핵심 + 4개 인프라 = 12개 설계 패턴** 추출
- 에이전트 시스템의 핵심 과제: 불확실한 실행 흐름에서의 안정성 확보
- 핵심 수치: 250K/일 절감, 23단계 권한, 14개 캐시 벡터, 888KB 번들, 최대 10 병렬
- 프로젝트 유형(웹앱 / 자동화 / AI 에이전트)에 따라 우선 적용 패턴이 다름
- 12개 패턴은 조합되어 시너지를 발휘 — 하나의 요청에 여러 패턴이 동시에 작동
