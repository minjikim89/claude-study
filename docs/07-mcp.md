# MCP (Model Context Protocol) — 외부 세계와 연결하는 플러그

## MCP란?

Claude Code는 기본적으로 **로컬 파일 시스템 안에서** 동작한다. 파일을 읽고, 쓰고, 터미널 명령을 실행할 수 있지만, 외부 서비스(Slack, Google Drive, 데이터베이스 등)에는 직접 접근하지 못한다.

MCP(Model Context Protocol)는 이 한계를 해결하는 **표준화된 외부 연결 프로토콜**이다. Claude Code에 외부 서비스 접근 능력을 플러그 형태로 꽂아주는 구조다.

## 노코드 도구에서의 대응 개념

| 노코드 도구 | MCP 대응 |
|---|---|
| **Zapier의 App Connection** | MCP 서버 연결 (Slack, GitHub 등) |
| **Make의 Module** | MCP가 제공하는 개별 도구(tool) |
| **n8n의 Credential + Node** | MCP 서버 설정 + 도구 호출 |
| **Notion의 Integration** | MCP 서버로 Notion API 연결 |

핵심 차이: 노코드 도구는 **GUI에서 클릭으로 연결**하지만, MCP는 **설정 파일에 서버 주소를 등록**하는 방식이다.

## 동작 원리

```
┌─ Claude Code ─────────────────────────────────┐
│                                                │
│  "Slack에 메시지 보내줘"                        │
│       │                                        │
│       ▼                                        │
│  MCP 클라이언트 (내장)                          │
│       │                                        │
│       ▼                                        │
│  ┌─ MCP 서버: Slack ─────┐                     │
│  │  도구: send_message   │ ──▶ Slack API 호출  │
│  │  도구: read_channel   │                     │
│  │  도구: list_channels  │                     │
│  └───────────────────────┘                     │
│                                                │
│  ┌─ MCP 서버: GitHub ────┐                     │
│  │  도구: create_issue   │ ──▶ GitHub API 호출 │
│  │  도구: list_prs       │                     │
│  └───────────────────────┘                     │
│                                                │
│  ┌─ MCP 서버: PostgreSQL ┐                     │
│  │  도구: query          │ ──▶ DB 직접 접근    │
│  │  도구: list_tables    │                     │
│  └───────────────────────┘                     │
└────────────────────────────────────────────────┘
```

**구조 요약:**
- **MCP 클라이언트**: Claude Code에 내장되어 있음
- **MCP 서버**: 각 외부 서비스와 통신하는 중간 다리 역할
- **도구(Tool)**: MCP 서버가 제공하는 개별 기능 (메시지 보내기, 이슈 생성 등)

## MCP 서버 설정 방법

### 프로젝트 단위 설정

`.claude/mcp.json` 파일에 등록:

```json
{
  "mcpServers": {
    "slack": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-server-slack"],
      "env": {
        "SLACK_BOT_TOKEN": "xoxb-..."
      }
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-server-github"],
      "env": {
        "GITHUB_TOKEN": "ghp_..."
      }
    }
  }
}
```

### 전역 설정

`~/.claude/mcp.json`에 등록하면 모든 프로젝트에서 사용 가능.

### 설정 구조

| 필드 | 설명 | 예시 |
|---|---|---|
| `command` | MCP 서버 실행 명령 | `npx`, `python`, `docker` |
| `args` | 실행 인자 | 패키지명, 설정 파일 경로 등 |
| `env` | 환경 변수 (API 키 등) | `SLACK_BOT_TOKEN`, `GITHUB_TOKEN` |

## 주요 MCP 서버 예시

| 서비스 | MCP 서버 | 제공 도구 | 활용 시나리오 |
|---|---|---|---|
| **Slack** | @anthropic/mcp-server-slack | 메시지 전송, 채널 읽기 | 일일 보고 자동화 |
| **GitHub** | @anthropic/mcp-server-github | 이슈/PR 생성, 코드 검색 | PR 리뷰 자동화 |
| **PostgreSQL** | @anthropic/mcp-server-postgres | SQL 쿼리, 테이블 조회 | 데이터 분석 보고서 |
| **Google Drive** | 커뮤니티 제공 | 문서 읽기/쓰기 | 문서 자동 생성 |
| **Notion** | 커뮤니티 제공 | 페이지/DB 조작 | 프로젝트 관리 동기화 |
| **Filesystem** | @anthropic/mcp-server-filesystem | 파일 읽기/쓰기 | 특정 디렉토리 제한 접근 |

> Anthropic 공식 서버 외에도 커뮤니티에서 만든 MCP 서버가 다수 존재한다.

## MCP와 다른 Claude Code 기능의 관계

```
Claude Code 도구 체계
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
내장 도구      Read, Write, Bash, Glob, Grep 등
               → 로컬 파일 시스템 조작

MCP 도구       send_message, query, create_issue 등
               → 외부 서비스 조작

Skill          MCP 도구를 활용하는 레시피를 정의 가능
               → "매주 월요일 Slack에 주간 보고 전송" Skill

Subagent       MCP 도구를 사용할 수 있는 Subagent 생성 가능
               → DB 조회 전용 Subagent (query 도구만 허용)
```

## Skill + MCP 조합 예시

MCP로 외부 서비스에 접근 가능해지면, Skill과 조합하여 강력한 자동화를 만들 수 있다.

### 주간 보고 자동화 Skill

```markdown
# /weekly-report Skill

## 할 일
1. GitHub MCP로 이번 주 머지된 PR 목록 조회
2. 각 PR의 변경 요약 정리
3. Slack MCP로 #team-updates 채널에 포맷팅된 보고 전송

## 출력 형식
**[주간 개발 보고]**
- 완료: N건
- 진행 중: N건
- 주요 변경: ...
```

## 보안 고려사항

| 항목 | 주의사항 |
|---|---|
| **API 키 관리** | `env`에 직접 넣지 말고, 환경 변수 참조 권장 |
| **권한 최소화** | MCP 서버에 필요한 최소 권한만 부여 (read-only 등) |
| **프로젝트 vs 전역** | 민감한 연결은 프로젝트 단위로 제한 |
| **`.gitignore`** | API 키가 포함된 설정은 git에 올리지 않도록 주의 |

## 핵심 정리

- MCP = Claude Code를 **외부 서비스와 연결**하는 표준 프로토콜
- 노코드 도구의 "App Connection / Integration"에 해당
- 설정 파일(mcp.json)에 서버를 등록하면 Claude가 외부 도구를 사용 가능
- Skill + MCP 조합으로 노코드 도구 수준의 자동화 워크플로우 구현 가능
- API 키 관리와 권한 최소화에 주의
