<div align="center">

# Claude Code Study

**노코드 도구만 쓰던 사람의 Claude Code 학습 기록**

Zapier, Make, Cursor에서 터미널로 — 낯선 세계의 실전 가이드

[![Docs](https://img.shields.io/badge/docs-16%20chapters-0ea5e9.svg)](./docs)
[![Level](https://img.shields.io/badge/level-beginner%20→%20advanced-f59e0b.svg)](#학습-로드맵)
[![Style](https://img.shields.io/badge/style-nocode%20비교-a855f7.svg)](#왜-노코드-도구-비교인가)
[![License: MIT](https://img.shields.io/badge/License-MIT-black.svg)](./LICENSE)

</div>

---

## 왜 이걸 만들었나

다양한 노코드/로우코드 도구를 써보다가 Claude Code를 접했다. **터미널에서 돌아가는 AI**라는 점이 처음엔 낯설었지만, 써보니 기존 노코드 도구들과 비교되는 지점이 많았고, 오히려 노코드 도구의 한계를 넘는 부분도 있었다.

이 저장소는 그 학습 과정을 기록한다:

- 노코드 도구에서 익숙한 개념이 Claude Code에서는 어떻게 대응되는지
- 터미널/CLI 환경에서 처음 부딪히는 것들과 해결 과정
- Claude Code만의 고유한 강점과 실무 활용 전략

## 왜 노코드 도구 비교인가

모든 문서는 **"이미 아는 도구"에서 시작해서 새로운 개념으로 연결**하는 구조다.

```
익숙한 도구                              Claude Code
─────────────────────────────────────────────────────────────
Zapier/Make     앱 간 자동화 워크플로우  →  Skill + MCP로 자동화 구성
Notion AI       문서 내 AI 보조          →  CLAUDE.md로 규칙 설정
Cursor          에디터 내 AI 코딩        →  터미널에서 파일 시스템 전체 접근
ChatGPT Memory  대화 기억                →  Auto Memory + CLAUDE.md
Chrome 확장     브라우저 기능 추가       →  Plugin Marketplace
```

## 학습 로드맵

### Stage 1 — 기초: Claude Code는 어떻게 작동하는가

| # | 주제 | 핵심 질문 |
|:---:|------|----------|
| [01](docs/01-claude-code-체계.md) | **Claude Code 체계** | 전체 구조와 동작 원리는? |
| [02](docs/02-기억체계.md) | **기억 체계** | AI가 대화를 기억하는 구조는? |
| [03](docs/03-skills.md) | **Skills** | Zapier 워크플로우 같은 자동화를 마크다운으로? |

### Stage 2 — 확장: 작업을 나누고 연결하기

| # | 주제 | 핵심 질문 |
|:---:|------|----------|
| [04](docs/04-subagent.md) | **Subagent** | 작업을 위임하는 부하 직원 구조 |
| [05](docs/05-agent-teams.md) | **Agent Teams** | 팀원끼리 직접 소통하는 협업 구조 |
| [06](docs/06-agentic-vs-claude-code.md) | **Agentic Framework vs Claude Code** | CrewAI 같은 프레임워크와 뭐가 다른가? |

### Stage 3 — 실전: 외부 연동과 자동화

| # | 주제 | 핵심 질문 |
|:---:|------|----------|
| [07](docs/07-mcp.md) | **MCP** | 외부 서비스(Slack, DB 등)를 어떻게 연결하나? |
| [08](docs/08-hooks.md) | **Hooks** | 이벤트 기반 자동 실행은 어떻게 설정하나? |
| [09](docs/09-컨텍스트-윈도우-관리.md) | **컨텍스트 윈도우 관리** | 200k 토큰의 현실적 한계와 대처법은? |
| [10](docs/10-실전-워크플로우.md) | **실전 워크플로우** | 실무에서 이 기능들을 어떻게 조합하나? |

### Stage 4 — 심화: 실전에서 부딪히는 문제와 해결

| # | 주제 | 핵심 질문 |
|:---:|------|----------|
| [11](docs/11-mcp-api-하이브리드.md) | **MCP + API 하이브리드** | MCP가 지원 안 하는 기능은 어떻게 처리하나? |
| [12](docs/12-스킬-vs-규칙-판단-가이드.md) | **스킬 vs 규칙 판단 가이드** | Skill로 만들까, CLAUDE.md 규칙으로 만들까? |
| [13](docs/13-plugins.md) | **Plugins** | 기능을 패키징하고 앱스토어처럼 공유하려면? |
| [14](docs/14-scheduled-tasks.md) | **Scheduled Tasks** | 반복 업무를 예약해서 자동으로 돌리려면? |
| [15](docs/15-remote-control.md) | **Remote Control** | 내 컴퓨터의 세션을 폰에서 이어하려면? |

### Reference

| # | 주제 |
|:---:|------|
| [99](docs/99-터미널-명령어-치트시트.md) | **터미널 명령어 치트시트** |

## 각 문서의 구조

모든 문서는 동일한 4단계로 구성된다:

```
┌─ 1. 개념 설명 ─────────────────────┐
│  이게 뭔지, 왜 필요한지             │
├─ 2. 노코드 도구 비교 ──────────────┤
│  익숙한 도구에서의 대응 개념         │
├─ 3. 실제 동작 원리 ────────────────┤
│  내부에서 어떻게 작동하는지          │
├─ 4. 실전 시나리오 ─────────────────┤
│  언제, 어떻게 쓰는지               │
└────────────────────────────────────┘
```

## 환경

| | |
|---|---|
| **OS** | macOS |
| **터미널** | zsh |
| **도구** | Claude Code (Anthropic CLI) |
| **배경** | 노코드/로우코드 도구 경험, 코딩은 초보 |
