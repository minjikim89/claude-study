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

| # | 주제 | 핵심 질문 | 파일 |
|---|------|----------|------|
| 01 | Skills | Zapier 워크플로우 같은 자동화를 마크다운으로? | [docs/01-skills.md](docs/01-skills.md) |
| 02 | Agentic Framework vs Claude Code | CrewAI 같은 프레임워크와 뭐가 다른가? | [docs/02-agentic-vs-claude-code.md](docs/02-agentic-vs-claude-code.md) |
| 03 | Claude Code 체계 | 설정 파일 구조와 동작 원리는? | [docs/03-claude-code-체계.md](docs/03-claude-code-체계.md) |
| 04 | 기억 체계 | AI가 대화를 기억하는 구조는? | [docs/04-기억체계.md](docs/04-기억체계.md) |
| 05 | Subagent | 작업을 위임하는 부하 직원 구조 | [docs/05-subagent.md](docs/05-subagent.md) |
| 06 | Agent Teams | 팀원끼리 직접 소통하는 협업 구조 | [docs/06-agent-teams.md](docs/06-agent-teams.md) |

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
