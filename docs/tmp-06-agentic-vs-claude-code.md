# Agentic Framework vs. Claude Code — 개념 대응 비교

## 왜 비교하는가?

AI 에이전트 생태계에는 CrewAI, LangGraph, AutoGen 같은 "Agentic Framework"들이 있다. 이들은 코드로 에이전트를 정의하고 오케스트레이션하는 방식이다.

Claude Code는 이런 프레임워크와 완전히 다른 접근을 취한다. **코드 대신 마크다운**, **런타임 대신 컨텍스트 주입**, **설정 대신 자연어**를 사용한다.

하지만 해결하려는 문제는 동일하기 때문에, 개념 대 개념으로 대응시키면 Claude Code의 구조를 빠르게 이해할 수 있다.

## 근본적 철학의 차이

| | **Agentic Framework** | **Claude Code** |
|---|---|---|
| **에이전트 정의** | 코드(Python 등)로 클래스/객체 생성 | 마크다운 파일로 지시사항 작성 |
| **실행 방식** | 프레임워크 런타임이 에이전트 코드를 실행 | Claude가 마크다운을 읽고 지시에 따라 행동 |
| **도구 연결** | 에이전트별로 tool 함수를 바인딩 | Claude가 기본 보유한 도구 (Bash, Read, Write 등) 사용 |
| **진입 장벽** | Python 코딩 + 프레임워크 API 학습 필요 | 마크다운 작성 능력만 있으면 됨 |
| **비유** | 전문가를 **고용**해서 도구를 쥐어주고 일을 시킴 | 이미 있는 비서에게 **레시피 카드를 건네줌** |

## 개념별 상세 대응

### 1. 시스템 프롬프트 / 전역 설정

```
Agentic Framework              Claude Code
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
System Prompt / Global       →  CLAUDE.md
  Role                            "한국어로 응답한다"
  Backstory                       "AI Native Camp 프로젝트"
  Constraints                     "STOP PROTOCOL을 따른다"
```

Agentic 프레임워크에서 에이전트의 성격, 역할, 제약조건을 코드로 설정하는 부분이 Claude Code에서는 **CLAUDE.md 파일** 하나로 대체된다. 세 단계 레벨(전역/프로젝트/로컬)로 범위를 조절할 수 있다.

### 2. 태스크별 프롬프트

```
Agentic Framework              Claude Code
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Task-specific Prompt         →  Skill (SKILL.md)
  Task Instructions               레시피 본문
  Reference Data                  references/ 폴더
  Scripts                         scripts/ 폴더
```

프레임워크에서 각 에이전트에게 할당하는 Task 객체가 Claude Code에서는 **Skill**이다. Skill은 마크다운 파일이므로 비개발자도 작성할 수 있다.

### 3. 메모리 체계

```
Agentic Framework              Claude Code
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Long-term Memory             →  CLAUDE.md + Auto Memory
Working Memory               →  대화 컨텍스트
Shared Memory                →  Agent Teams 태스크 리스트
```

프레임워크에서 벡터DB나 외부 저장소로 구현하는 장기 기억이, Claude Code에서는 **파일 기반 기억 체계**로 대체된다. 별도의 인프라 없이 마크다운 파일이 곧 메모리다.

### 4. 도구 사용 (Tool Calling)

```
Agentic Framework              Claude Code
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Custom Tools (함수 정의)     →  내장 도구 (Bash, Read, Write, Glob, Grep 등)
External API 연동            →  MCP (Model Context Protocol)
```

프레임워크에서는 tool 함수를 직접 코딩해서 에이전트에 바인딩하지만, Claude Code는 **파일 시스템 조작, 터미널 실행 등의 도구를 기본 내장**하고 있다. 외부 서비스 연동이 필요하면 **MCP**로 확장한다.

### 5. 작업 위임 / 멀티에이전트

```
Agentic Framework              Claude Code
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Agent Delegation             →  Subagent (독립 공간, 결과만 보고)
Multi-agent Orchestration    →  Agent Teams (독립 세션, 직접 소통)
Sequential Workflow          →  Subagent 순차 호출
Parallel Workflow            →  Agent Teams 병렬 실행
```

## 전체 아키텍처 한눈에 보기

```
Claude Code 아키텍처
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
영구 기억      →  CLAUDE.md + Auto Memory     (항상 로드)
레시피         →  Skill                        (호출 시 로드)
외부 연결      →  MCP                          (외부 서비스 플러그)
작업 위임      →  Subagent                     (독립 공간, 결과만 보고)
팀 협업        →  Agent Teams                  (독립 세션, 직접 소통)
```

## 노코드 도구 사용자 관점에서의 매핑

기존에 Zapier, Make, n8n 같은 노코드 자동화 도구를 사용했다면:

| 노코드 도구 개념 | Claude Code 대응 | 설명 |
|---|---|---|
| **워크플로우 템플릿** | Skill | 반복 작업을 템플릿화 |
| **글로벌 설정 / Preferences** | CLAUDE.md | 모든 작업에 적용되는 기본값 |
| **변수 / 데이터 저장** | Auto Memory | 학습한 내용을 자동 저장 |
| **외부 앱 연동 (Integration)** | MCP | 외부 서비스에 접근 |
| **병렬 실행 (Parallel Path)** | Agent Teams | 여러 작업을 동시 실행 |
| **서브 워크플로우** | Subagent | 작업 일부를 위임 |

## 핵심 정리

- Agentic Framework는 **코드 중심**, Claude Code는 **텍스트(마크다운) 중심**
- 같은 문제를 해결하지만 진입 장벽이 완전히 다름
- Claude Code의 모든 설정은 결국 **"컨텍스트 윈도우에 텍스트를 주입하는 것"**으로 귀결됨
- 프레임워크의 복잡한 설정이 Claude Code에서는 마크다운 파일 몇 개로 단순화됨
