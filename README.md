# Claude Code 공부 기록

> 노코드 도구(Zapier, Make, Cursor 등)만 쓰다가 터미널을 처음 열어본 사람이,
> Claude Code를 배워가며 기록하는 학습 저장소.

## 이 저장소는 왜 만들었나

다양한 노코드/로우코드 도구를 써보다가 Claude Code를 접하였는데 **터미널에서 돌아가는 AI**라는 점이 처음엔 낯설었지만, 써보니 기존 노코드 도구들과 비교되는 지점이 많았다. 오히려 노코드 도구의 한계를 넘는 부분도 있음.

이 저장소는 그 학습 과정을 기록한다:
- 노코드 도구에서 익숙한 개념이 Claude Code에서는 어떻게 대응되는지
- 터미널/CLI 환경에서 처음 부딪히는 것들과 해결 과정
- Claude Code만의 고유한 강점과 실무 활용 전략

## 기존 노코드 도구 vs Claude Code — 왜 다른가

| 익숙한 도구 | 하는 일 | Claude Code에서는 |
|---|---|---|
| **Zapier/Make** | 앱 간 자동화 워크플로우 | Skill + MCP로 자동화 레시피 구성 |
| **Notion AI** | 문서 내 AI 보조 | CLAUDE.md로 프로젝트 규칙 설정, 파일 직접 조작 |
| **Cursor** | 코드 에디터 내 AI 코딩 | 터미널에서 파일 시스템 전체에 접근하며 코딩 |
| **ChatGPT Memory** | 대화 기억 | Auto Memory + CLAUDE.md (투명하게 파일로 관리) |

## 학습 목차

### 기초: Claude Code는 어떻게 작동하는가

| # | 주제 | 핵심 질문 | 파일 |
|---|------|----------|------|
| 01 | Claude Code 체계 | 전체 구조와 동작 원리는? | [docs/01-claude-code-체계.md](docs/01-claude-code-체계.md) |
| 02 | 기억 체계 | AI가 대화를 기억하는 구조는? | [docs/02-기억체계.md](docs/02-기억체계.md) |
| 03 | Skills | Zapier 워크플로우 같은 자동화를 마크다운으로? | [docs/03-skills.md](docs/03-skills.md) |

### 확장: 작업을 나누고 연결하기

| # | 주제 | 핵심 질문 | 파일 |
|---|------|----------|------|
| 04 | Subagent | 작업을 위임하는 부하 직원 구조 | [docs/04-subagent.md](docs/04-subagent.md) |
| 05 | Agent Teams | 팀원끼리 직접 소통하는 협업 구조 | [docs/05-agent-teams.md](docs/05-agent-teams.md) |
| 06 | Agentic Framework vs Claude Code | CrewAI 같은 프레임워크와 뭐가 다른가? | [docs/06-agentic-vs-claude-code.md](docs/06-agentic-vs-claude-code.md) |

### 실전: 외부 연동과 자동화

| # | 주제 | 핵심 질문 | 파일 |
|---|------|----------|------|
| 07 | MCP (Model Context Protocol) | 외부 서비스(Slack, DB 등)를 어떻게 연결하나? | [docs/07-mcp.md](docs/07-mcp.md) |
| 08 | Hooks | 이벤트 기반 자동 실행은 어떻게 설정하나? | [docs/08-hooks.md](docs/08-hooks.md) |
| 09 | 컨텍스트 윈도우 관리 | 200k 토큰의 현실적 한계와 대처법은? | [docs/09-컨텍스트-윈도우-관리.md](docs/09-컨텍스트-윈도우-관리.md) |
| 10 | 실전 워크플로우 | 실무에서 이 기능들을 어떻게 조합하나? | [docs/10-실전-워크플로우.md](docs/10-실전-워크플로우.md) |

### 심화: 실전에서 부딪히는 문제와 해결

| # | 주제 | 핵심 질문 | 파일 |
|---|------|----------|------|
| 11 | MCP + API 하이브리드 | MCP가 지원 안 하는 기능은 어떻게 처리하나? | [docs/11-mcp-api-하이브리드.md](docs/11-mcp-api-하이브리드.md) |

## 학습 방식

각 문서는 다음 구조로 작성:
1. **개념 설명** — 이게 뭔지, 왜 필요한지
2. **노코드 도구 비교** — 익숙한 도구에서의 대응 개념
3. **실제 동작 원리** — 내부에서 어떻게 작동하는지
4. **실전 시나리오** — 언제, 어떻게 쓰는지

## 환경

- **OS**: macOS
- **터미널**: 기본 터미널 (zsh)
- **Claude Code**: Anthropic CLI
- **배경**: 노코드/로우코드 도구 사용 경험, 코딩은 초보
