# MCP + API 하이브리드 패턴 — MCP가 안 되는 일은 어떻게?

## 문제 상황

MCP 서버는 외부 서비스의 **모든 API를 감싸지 않는다**. 예를 들어 Notion MCP는 페이지 생성·수정·검색을 지원하지만, **삭제(archive)는 지원하지 않는다**. 공식 REST API에서는 가능한 기능인데 MCP에 빠져 있는 것이다.

이런 갭이 발생하는 이유:
- MCP 서버 개발자가 모든 API endpoint를 구현하지 않음
- 위험한 작업(삭제, 권한 변경 등)을 의도적으로 제외
- 서비스 API 업데이트가 MCP 서버에 반영되기까지 시간차 존재

## 노코드 도구에서의 대응 개념

| 노코드 도구 | MCP + API 하이브리드 대응 |
|---|---|
| **Zapier**: 기본 액션 + Webhooks by Zapier | MCP 도구 + Bash/Python으로 API 직접 호출 |
| **Make**: 내장 모듈 + HTTP 모듈 | MCP 도구 + curl/requests로 보완 |
| **n8n**: 전용 노드 + HTTP Request 노드 | MCP + Skill로 API 호출 래핑 |

핵심은 동일하다: **기본 제공 기능으로 80%를 처리하고, 나머지 20%는 범용 HTTP 호출로 보완**하는 구조.

## 실무 패턴 3가지

### Pattern 1: MCP 우선 + API 보완 (가장 일반적)

```
일상 작업 (80%)              특수 작업 (20%)
  ├─ 페이지 생성               ├─ 페이지 삭제 (archive)
  ├─ DB 쿼리                  ├─ 권한 변경
  ├─ 페이지 수정               ├─ 벌크 작업
  ├─ 검색                     └─ 고급 필터링
  │
  └→ MCP 도구로 처리            └→ API 직접 호출로 처리
```

**장점**: MCP의 편의성(Claude가 자연어로 도구 호출)을 최대한 활용하면서, 부족한 부분만 보완
**단점**: 두 가지 방식이 섞여서 사용법이 일관적이지 않음

### Pattern 2: 전부 API로 통일

MCP를 쓰지 않고 모든 작업을 API 호출 스크립트로 처리.

**선택하는 경우**:
- MCP가 지원하는 기능이 너무 제한적일 때
- 세밀한 에러 핸들링이 필요할 때
- 대규모 배치 자동화 파이프라인을 구축할 때

**단점**: Claude의 도구 통합(tool calling) 이점을 잃음. "Notion에서 검색해줘"라고 말하면 알아서 MCP 도구를 고르는 편의성이 사라짐.

### Pattern 3: 커스텀 MCP 서버로 갭 메우기

부족한 API를 직접 MCP 서버로 감싸서, Claude가 모든 작업을 도구로 호출 가능하게 만드는 방식.

```
Notion 공식 MCP              직접 만든 보완 MCP
  ├─ create-page               ├─ archive-page (삭제)
  ├─ update-page               ├─ bulk-update
  ├─ search                    └─ set-permissions
  └─ query-database
```

**장점**: 가장 깔끔한 사용 경험 (모든 기능이 도구로 통합)
**단점**: MCP 서버 개발 필요 (Python/TypeScript), 유지보수 부담

### 어떤 패턴을 선택할까?

```
MCP가 지원 안 하는 기능이 있다
  │
  ├─ 가끔 필요 (월 1~2회)
  │   └→ Pattern 1: 그때그때 API 호출하면 됨
  │
  ├─ 자주 필요 (주 여러 회)
  │   └→ Pattern 1 + Skill: API 호출을 Skill로 감싸서 재사용
  │
  └─ 핵심 워크플로우의 일부
      └→ Pattern 3: 커스텀 MCP 서버로 통합
```

## 구현 예시: Notion MCP + API 하이브리드

### Notion MCP가 지원하는 것 / 안 하는 것

| 작업 | MCP 지원 | API 지원 | 비고 |
|------|:--------:|:--------:|------|
| 페이지 생성 | O | O | |
| 페이지 수정 | O | O | |
| 페이지 검색 | O | O | |
| DB 쿼리 | O | O | |
| 댓글 조회/작성 | O | O | |
| 페이지 삭제 (archive) | X | O | `archived: true` PATCH |
| 페이지 영구 삭제 | X | X | API도 미지원 |
| DB 속성(property) 삭제 | X | O | property를 `null`로 PATCH |
| 블록 삭제 | X | O | block DELETE endpoint |

### 시나리오: "완료된 태스크를 아카이브해줘"

이 워크플로우는 MCP만으로는 완성할 수 없다:
1. **DB에서 완료 태스크 조회** → MCP로 가능 (query-database)
2. **각 태스크를 아카이브** → MCP 미지원 → API 직접 호출 필요

### 구현 방법 A: Claude에게 직접 API 호출 요청

별도 설정 없이 Claude에게 자연어로 요청:

```
Notion에서 Tasks DB의 Done이 true인 항목을 조회하고,
각 페이지를 Notion API로 archive 처리해줘.
API 키는 환경변수 NOTION_API_KEY를 사용해.
```

Claude가 실행하는 흐름:

```
Step 1: MCP 도구로 DB 쿼리
  → notion.query-database (filter: Done = true)
  → 결과: page_id 목록

Step 2: Bash에서 API 직접 호출 (MCP 미지원이므로)
  → curl -X PATCH https://api.notion.com/v1/pages/{page_id} \
      -H "Authorization: Bearer $NOTION_API_KEY" \
      -H "Notion-Version: 2022-06-28" \
      -d '{"archived": true}'
```

**장점**: 바로 사용 가능, 설정 불필요
**단점**: 매번 API 키 위치와 호출 방식을 설명해야 함

### 구현 방법 B: Skill로 래핑하여 재사용 (추천)

자주 쓰는 하이브리드 작업을 Skill로 만들어두면, 매번 설명할 필요 없이 슬래시 명령으로 호출 가능.

```
.claude/skills/
  └─ notion-archive/
      └─ SKILL.md
```

**SKILL.md 예시:**

```markdown
---
name: notion-archive
description: Notion에서 완료된 태스크를 아카이브. "아카이브", "완료 정리" 요청에 사용.
---

# Notion 완료 태스크 아카이브

## 환경
- NOTION_API_KEY: 환경변수에서 읽기
- Tasks DB ID: 6d1161325e944014b3c369983796ef15

## 실행 순서

### 1단계: 완료 태스크 조회 (MCP 사용)
Notion MCP의 query-database 도구로 Tasks DB에서 Done = true인 항목 조회.

### 2단계: 아카이브 확인
조회된 태스크 목록을 사용자에게 보여주고, 아카이브 진행 여부를 확인받는다.
아카이브는 되돌릴 수 없으므로 반드시 사전 확인.

### 3단계: API로 아카이브 실행
각 페이지에 대해 Notion REST API를 호출하여 archived: true로 변경.

curl -X PATCH "https://api.notion.com/v1/pages/{page_id}" \
  -H "Authorization: Bearer $NOTION_API_KEY" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{"archived": true}'

### 4단계: 결과 보고
성공/실패 건수를 테이블로 출력.
```

**사용법:**

```
/notion-archive
또는
"완료된 태스크 정리해줘"
```

### 구현 방법 C: Python 스크립트로 분리

반복 실행이 잦거나 로직이 복잡하면, 별도 스크립트로 분리:

```python
# scripts/notion_archive.py
import os
import requests

NOTION_API_KEY = os.environ["NOTION_API_KEY"]
TASKS_DB_ID = "6d1161325e944014b3c369983796ef15"
HEADERS = {
    "Authorization": f"Bearer {NOTION_API_KEY}",
    "Notion-Version": "2022-06-28",
    "Content-Type": "application/json",
}

def query_done_tasks():
    """Done = true인 태스크 조회"""
    url = f"https://api.notion.com/v1/databases/{TASKS_DB_ID}/query"
    payload = {
        "filter": {
            "property": "Done",
            "checkbox": {"equals": True}
        }
    }
    resp = requests.post(url, headers=HEADERS, json=payload)
    resp.raise_for_status()
    return resp.json()["results"]

def archive_page(page_id: str):
    """페이지를 아카이브 처리"""
    url = f"https://api.notion.com/v1/pages/{page_id}"
    resp = requests.patch(url, headers=HEADERS, json={"archived": True})
    resp.raise_for_status()
    return resp.json()

def main():
    tasks = query_done_tasks()
    print(f"Found {len(tasks)} completed tasks")

    for task in tasks:
        title = task["properties"]["Name"]["title"][0]["plain_text"]
        result = archive_page(task["id"])
        status = "archived" if result.get("archived") else "failed"
        print(f"  {status}: {title}")

if __name__ == "__main__":
    main()
```

Skill에서 이 스크립트를 호출:

```markdown
## 실행
python scripts/notion_archive.py 실행
```

## 패턴별 비교 정리

| 기준 | A: 직접 요청 | B: Skill 래핑 | C: 스크립트 분리 |
|------|:-----------:|:------------:|:---------------:|
| 초기 설정 | 없음 | Skill 파일 1개 | Skill + Python 파일 |
| 재사용성 | 낮음 (매번 설명) | 높음 (슬래시 명령) | 높음 (스크립트 실행) |
| 에러 처리 | Claude 임의 판단 | Skill에 명시 가능 | 코드로 정밀 제어 |
| 추천 상황 | 1회성 작업 | 주기적 반복 작업 | 복잡한 로직, 배치 처리 |

## 핵심 정리

- MCP는 외부 서비스 API의 **부분집합**만 지원 — 모든 기능을 커버하지 않음
- 부족한 부분은 **API 직접 호출(curl, Python requests)**로 보완하는 **하이브리드 패턴**이 가장 일반적
- 자주 쓰는 하이브리드 작업은 **Skill로 래핑**하면 매번 설명할 필요 없이 재사용 가능
- 복잡한 로직이나 에러 처리가 필요하면 **별도 스크립트로 분리**
- MCP의 한계 = **직접 Skill을 만들어야 하는 이유**. 범용 도구의 갭을 나만의 워크플로우로 채우는 것이 진짜 자동화
