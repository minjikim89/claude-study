# Plugins — 기능을 패키징하고 공유하는 앱스토어

## Plugin이란?

지금까지 배운 Skill, Hook, MCP 서버, Agent는 모두 `.claude/` 디렉토리 안에 개별 파일로 존재했다. 잘 작동하지만, **다른 프로젝트나 팀에 공유하려면 파일을 일일이 복사**해야 했다.

Plugin은 이 문제를 해결한다. **여러 기능(Skill + Hook + MCP + Agent)을 하나의 폴더로 묶어 패키지로 만들고, Marketplace를 통해 설치/배포**할 수 있게 하는 구조다.

> 요약: Plugin = 기능 묶음 패키지 + 앱스토어 배포

## 노코드 도구와의 비교

| 노코드 도구 | Plugin 대응 |
|---|---|
| **Zapier의 Template** | 다른 사람이 만든 Plugin 설치 (`/plugin install`) |
| **Notion의 Template Gallery** | Plugin Marketplace (공식/커뮤니티) |
| **Chrome 웹스토어의 확장 프로그램** | Plugin 패키지 (Skill + Hook + MCP 통합) |
| **WordPress 플러그인** | Claude Code Plugin (설치 → 기능 추가 → 활성화/비활성화) |

핵심 차이: 노코드 도구는 **앱 안에서 마켓플레이스를 클릭**하지만, Claude Code는 **`/plugin` 명령어로 설치하고 관리**한다.

## Standalone vs Plugin — 언제 어떤 걸 쓰나?

| | **Standalone** (`.claude/` 직접 설정) | **Plugin** (패키지로 묶기) |
|---|---|---|
| **Skill 이름** | `/hello` | `/plugin-name:hello` |
| **적합한 상황** | 개인용, 프로젝트 전용, 빠른 실험 | 팀 공유, 커뮤니티 배포, 여러 프로젝트 재사용 |
| **관리** | 수동 복사 | 버전 관리 + 자동 업데이트 |
| **비유** | 내 컴퓨터에 저장한 매크로 | 회사 전체가 쓰는 공식 도구 |

> **팁**: `.claude/`에서 먼저 만들어 테스트하고, 공유 필요 시 Plugin으로 전환하는 게 가장 자연스러운 흐름이다.

## Plugin 구조

Plugin은 **하나의 폴더** 안에 모든 것을 담는다.

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json          ← 메타데이터 (이름, 설명, 버전)
├── commands/                ← Slash 커맨드 (마크다운 파일)
├── skills/                  ← Agent Skill (SKILL.md)
├── agents/                  ← 커스텀 에이전트 정의
├── hooks/
│   └── hooks.json           ← 이벤트 훅 설정
├── .mcp.json                ← MCP 서버 설정
├── .lsp.json                ← LSP 서버 설정 (코드 인텔리전스)
└── settings.json            ← 기본 설정값
```

### plugin.json — Plugin의 신분증

```json
{
  "name": "my-plugin",
  "description": "코드 리뷰 자동화 플러그인",
  "version": "1.0.0",
  "author": {
    "name": "Your Name"
  }
}
```

| 필드 | 역할 |
|---|---|
| `name` | Plugin 고유 식별자 (= Skill 이름 앞에 붙는 네임스페이스) |
| `description` | Marketplace에서 보이는 설명 |
| `version` | 버전 관리 (업데이트 감지에 사용) |

> **주의**: `commands/`, `skills/`, `agents/` 등은 `.claude-plugin/` 디렉토리 **바깥**(Plugin 루트)에 놓아야 한다. `.claude-plugin/` 안에는 `plugin.json`만 위치.

## Plugin 만들기 — 최소 예제

### 1. 디렉토리 생성

```bash
mkdir -p my-plugin/.claude-plugin
mkdir -p my-plugin/skills/review
```

### 2. plugin.json 작성

```json
// my-plugin/.claude-plugin/plugin.json
{
  "name": "review-plugin",
  "description": "코드 리뷰 자동화",
  "version": "1.0.0"
}
```

### 3. Skill 작성

```markdown
<!-- my-plugin/skills/review/SKILL.md -->
---
description: Review code for bugs, security, and performance
disable-model-invocation: true
---

Review the code for:
1. Potential bugs or edge cases
2. Security concerns
3. Performance issues
```

### 4. 로컬 테스트

```bash
claude --plugin-dir ./my-plugin
```

Claude Code 안에서 `/review-plugin:review` 로 실행 가능.

## Marketplace — Plugin 앱스토어

Marketplace는 Plugin 카탈로그다. **"스토어를 추가 → 원하는 Plugin 설치"** 2단계 구조.

### 공식 Marketplace

Claude Code를 시작하면 **공식 Anthropic Marketplace**가 자동으로 등록되어 있다. `/plugin`을 실행하고 **Discover** 탭에서 바로 탐색 가능.

### 공식 Marketplace의 주요 Plugin 카테고리

**1. Code Intelligence (코드 인텔리전스)**

언어 서버(LSP)를 연결해서 Claude가 타입 에러, 정의 점프, 참조 검색을 즉시 수행할 수 있게 한다.

| 언어 | Plugin 이름 | 필요 바이너리 |
|---|---|---|
| TypeScript | `typescript-lsp` | `typescript-language-server` |
| Python | `pyright-lsp` | `pyright-langserver` |
| Go | `gopls-lsp` | `gopls` |
| Rust | `rust-analyzer-lsp` | `rust-analyzer` |

**2. External Integrations (외부 서비스 연동)**

MCP 서버를 미리 설정해둔 Plugin. 수동 설정 없이 바로 사용 가능.

- GitHub, GitLab (소스 관리)
- Slack (커뮤니케이션)
- Notion, Linear, Jira (프로젝트 관리)
- Figma (디자인)
- Vercel, Supabase (인프라)

**3. Development Workflows (개발 워크플로우)**

- `commit-commands` — Git 커밋/푸시/PR 워크플로우
- `pr-review-toolkit` — PR 리뷰 에이전트
- `plugin-dev` — Plugin 개발 도구

## Plugin 설치/관리 흐름

```
/plugin                              ← Plugin 관리자 UI 열기
├── Discover 탭: 설치 가능한 Plugin 탐색
├── Installed 탭: 설치된 Plugin 관리
├── Marketplaces 탭: Marketplace 추가/제거
└── Errors 탭: 오류 확인
```

### 기본 명령어

```bash
# Marketplace 추가 (GitHub 저장소)
/plugin marketplace add owner/repo

# Plugin 설치
/plugin install plugin-name@marketplace-name

# Plugin 비활성화 (삭제 X, 끄기만)
/plugin disable plugin-name@marketplace-name

# Plugin 다시 활성화
/plugin enable plugin-name@marketplace-name

# Plugin 완전 제거
/plugin uninstall plugin-name@marketplace-name
```

### 설치 범위 (Scope)

| Scope | 누가 사용? | 저장 위치 |
|---|---|---|
| **User** | 나만, 모든 프로젝트에서 | 사용자 설정 |
| **Project** | 이 저장소의 모든 협업자 | `.claude/settings.json` |
| **Local** | 나만, 이 저장소에서만 | 로컬 설정 |

## 기존 설정을 Plugin으로 전환하기

이미 `.claude/` 안에 Skills나 Hooks를 만들어뒀다면, 간단히 Plugin으로 전환 가능하다.

```bash
# 1. Plugin 디렉토리 생성
mkdir -p my-plugin/.claude-plugin

# 2. plugin.json 작성
# { "name": "my-plugin", "version": "1.0.0", ... }

# 3. 기존 파일 복사
cp -r .claude/commands my-plugin/
cp -r .claude/skills my-plugin/

# 4. 테스트
claude --plugin-dir ./my-plugin
```

| 전환 전 (Standalone) | 전환 후 (Plugin) |
|---|---|
| 이 프로젝트에서만 사용 | Marketplace로 공유 가능 |
| `.claude/commands/`에 파일 | `plugin-name/commands/`에 파일 |
| 수동 복사로 공유 | `/plugin install`로 설치 |

## 팀 Marketplace 운영

팀 전체가 같은 Plugin을 사용하게 하려면, 프로젝트 `.claude/settings.json`에 Marketplace를 등록한다.

```json
{
  "extraKnownMarketplaces": {
    "team-tools": {
      "source": {
        "source": "github",
        "repo": "our-org/claude-plugins"
      }
    }
  },
  "enabledPlugins": {
    "formatter@team-tools": true,
    "reviewer@team-tools": true
  }
}
```

팀원이 프로젝트를 열면 Marketplace 설치 안내가 자동으로 뜬다.

## 기존 학습 내용과의 관계

```
Plugin = 패키징 + 배포 레이어
  ├── Skills (03)     ← Plugin 안에 skills/ 디렉토리로 포함
  ├── Hooks (08)      ← Plugin 안에 hooks/hooks.json으로 포함
  ├── MCP (07)        ← Plugin 안에 .mcp.json으로 포함
  ├── Agents (04, 05) ← Plugin 안에 agents/ 디렉토리로 포함
  └── LSP (신규)      ← Plugin만의 기능: 코드 인텔리전스 서버 연동
```

이전에 배운 Skills, Hooks, MCP 서버가 모두 Plugin의 **구성 요소**다. Plugin은 이것들을 하나로 묶어 **패키지로 배포하는 계층**을 추가한 것이다.

## 핵심 정리

- Plugin = 기존 기능(Skill + Hook + MCP + Agent)을 **하나의 폴더로 패키징**한 것
- Marketplace = Plugin을 **검색/설치/업데이트**하는 앱스토어
- `/plugin` 명령어로 설치, 비활성화, 제거 등 관리
- `.claude/`에서 먼저 실험 → 공유 필요 시 Plugin으로 전환이 자연스러운 흐름
- 팀 Marketplace를 설정하면 팀원 전체에 표준 도구 자동 배포 가능
- Plugin 요구 버전: **Claude Code v1.0.33+**
