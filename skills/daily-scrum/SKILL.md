# DAILY SCRUM 페이지 생성

오늘 날짜로 DAILY SCRUM 페이지를 생성하고 내용을 채워주세요.

**전제 조건**: GitHub CLI (gh) 필수. Notion MCP는 선택사항.

## 설정

### 필수
| 항목 | 설명 | 예시 |
|------|------|------|
| GitHub 레포 | `{owner}/{repo}` 형식 | `jjub0217/cuddle-market` |
| base 브랜치 | git log 조회 대상 브랜치 | `develop` |

### 선택 (Notion 연동)
| 항목 | 설명 | 예시 |
|------|------|------|
| DAILY SCRUM DB | Notion Database ID | `30ff2b30-7961-817e-9601-d836505c5b04` |
| 사용자 ID | Notion 사용자 ID | `7c32774b-0096-4545-a9fe-7cfec90faa15` |
| 사용자 이름 | 테이블에 표시할 이름 | `강주현` |

> Notion 설정이 없으면 `scrum/YYYY-MM-DD-daily-scrum.md` 로컬 파일로 생성됩니다.

## 작업 순서

1. **오늘 날짜 확인**: 오늘이 몇 월 몇 일인지 확인

2. **Git 로그 분석**
   - 어제 날짜의 커밋 내역 조회: `git log --since="어제 00:00" --until="어제 23:59" --oneline`
   - 오늘 날짜의 커밋 내역 조회: `git log --since="오늘 00:00" --oneline`
   - 커밋 메시지에서 작업 내용 추출

3. **Daily Scrum 생성** — Notion 설정 여부에 따라 분기

---

### 분기 A: Notion 설정 있음 → Notion 페이지 생성

#### 3A-1. DAILY SCRUM 데이터베이스에 페이지 생성 (curl 사용 필수)

   > **중요**: MCP Notion 도구의 `API-post-page`는 `parent.page_id`만 지원하여 데이터베이스에 직접 페이지를 생성할 수 없습니다. 반드시 아래 curl 명령어를 사용하세요.

   > **API Token 확인 방법**: MCP 로그 파일에서 토큰 확인
   > `cat ~/Library/Caches/claude-cli-nodejs/-Users-osejin-Desktop-cuddle-market/mcp-logs-notion/*.txt | grep "Authorization: Bearer" | head -1`

   ```bash
   curl -X POST 'https://api.notion.com/v1/pages' \
     -H "Authorization: Bearer {NOTION_API_TOKEN}" \
     -H "Content-Type: application/json" \
     -H "Notion-Version: 2022-06-28" \
     -d '{
       "parent": {
         "database_id": "{DAILY_SCRUM_DB_ID}"
       },
       "properties": {
         "이름": {
           "title": [{"text": {"content": "YYYY년 MM월 DD일"}}]
         },
         "작성일시": {
           "date": {"start": "YYYY-MM-DD"}
         },
         "참여자": {
           "people": [{"id": "{USER_ID}"}]
         },
         "구분": {
           "select": {"name": "개발"}
         }
       }
     }'
   ```

   - Database ID: 설정의 `DAILY SCRUM DB` 값 사용
   - 페이지 제목: `YYYY년 MM월 DD일` 형식 (예: `2025년 12월 3일`)
   - 작성일시: 오늘 날짜 (YYYY-MM-DD 형식)
   - 참여자: 설정의 `사용자 ID` 값 사용
   - 구분: "개발" 선택

#### 3A-2. 페이지 내용 채우기 (curl 사용 필수)

   > 생성된 페이지 ID를 사용하여 아래 curl 명령어로 블록을 추가하세요.

   ```bash
   curl -X PATCH 'https://api.notion.com/v1/blocks/{PAGE_ID}/children' \
     -H "Authorization: Bearer {NOTION_API_TOKEN}" \
     -H "Content-Type: application/json" \
     -H "Notion-Version: 2022-06-28" \
     -d '{
       "children": [
         {"type": "heading_2", "heading_2": {"rich_text": [{"type": "text", "text": {"content": "전일 업무 진행상황 보고"}}], "color": "purple_background"}},
         {"type": "divider", "divider": {}},
         {"type": "table", "table": {"table_width": 2, "has_column_header": false, "has_row_header": false, "children": [{"type": "table_row", "table_row": {"cells": [[{"type": "text", "text": {"content": "{USER_NAME}"}}], [{"type": "text", "text": {"content": "전일 작업 내용"}}]]}}]}},
         {"type": "heading_2", "heading_2": {"rich_text": [{"type": "text", "text": {"content": "금일 예정 업무 보고"}}], "color": "purple_background"}},
         {"type": "divider", "divider": {}},
         {"type": "table", "table": {"table_width": 2, "has_column_header": false, "has_row_header": false, "children": [{"type": "table_row", "table_row": {"cells": [[{"type": "text", "text": {"content": "{USER_NAME}"}}], [{"type": "text", "text": {"content": "금일 예정 작업 내용"}}]]}}]}},
         {"type": "heading_2", "heading_2": {"rich_text": [{"type": "text", "text": {"content": "회의 안건"}}], "color": "purple_background"}},
         {"type": "divider", "divider": {}},
         {"type": "bulleted_list_item", "bulleted_list_item": {"rich_text": [{"type": "text", "text": {"content": "-"}}]}},
         {"type": "heading_2", "heading_2": {"rich_text": [{"type": "text", "text": {"content": "추가 논의사항 (ex. 이슈, 휴가 사용 예정 공유)"}}], "color": "purple_background"}},
         {"type": "divider", "divider": {}},
         {"type": "bulleted_list_item", "bulleted_list_item": {"rich_text": [{"type": "text", "text": {"content": "-"}}]}}
       ]
     }'
   ```

   페이지에 다음 블록들을 추가:

   a. **전일 업무 진행상황 보고** 섹션
      - Heading 2: "전일 업무 진행상황 보고" (purple_background)
      - Divider
      - Table 블록 생성:
        - `table_width: 2`
        - `has_column_header: false`
        - `has_row_header: false`
        - 첫 번째 행: ["{USER_NAME}", "어제의 git 커밋 내용을 간략하게 요약"]

   b. **금일 예정 업무 보고** 섹션
      - Heading 2: "금일 예정 업무 보고" (purple_background)
      - Divider
      - Table 블록 생성:
        - `table_width: 2`
        - `has_column_header: false`
        - `has_row_header: false`
        - 첫 번째 행: ["{USER_NAME}", "오늘의 git 커밋 내용 또는 작업 예정 사항"]

   c. **회의 안건** 섹션
      - Heading 2: "회의 안건" (purple_background)
      - Divider
      - Bulleted list item: "-"

   d. **추가 논의사항** 섹션
      - Heading 2: "추가 논의사항 (ex. 이슈, 휴가 사용 예정 공유)" (purple_background)
      - Divider
      - Bulleted list item: "-"

#### Notion 마크다운 스펙 확인

Notion 설정이 있는 경우, 페이지 생성 전에 반드시 `ReadMcpResourceTool(server="claude.ai Notion", uri="notion://docs/enhanced-markdown-spec")`으로 스펙을 확인할 것.

---

### 분기 B: Notion 설정 없음 → 로컬 마크다운 생성

`scrum/YYYY-MM-DD-daily-scrum.md` 파일을 생성합니다.

```markdown
# Daily Scrum - YYYY년 MM월 DD일

## 전일 업무 진행상황 보고

| 담당자 | 내용 |
|--------|------|
| {USER_NAME 또는 GitHub 사용자명} | (어제의 git 커밋 내용 요약. 없으면 "개발 커밋 없음") |

## 금일 예정 업무 보고

| 담당자 | 내용 |
|--------|------|
| {USER_NAME 또는 GitHub 사용자명} | (오늘의 git 커밋 내용 또는 작업 예정 사항. 없으면 "개발 커밋 없음") |

## 회의 안건

- \-

## 추가 논의사항 (ex. 이슈, 휴가 사용 예정 공유)

- \-
```

---

## 참고사항

- 테이블 헤더는 사용하지 않음 (`has_column_header: false`) — Notion 모드에서만 해당
- 커밋 메시지가 없는 경우, 사용자에게 수동으로 입력할 내용을 물어보기

## MCP 도구 제한사항 (Notion 설정 시에만 해당)

> **MCP Notion 도구 사용 금지**: `mcp__notion__API-post-page` 도구는 `parent.page_id`만 지원하여 데이터베이스에 페이지를 생성할 수 없습니다.
>
> **해결 방법**: 페이지 생성 및 블록 추가 시 반드시 **curl 명령어**를 사용하세요.
>
> MCP 도구는 다음 용도로만 사용:
> - `API-post-database-query`: 데이터베이스 조회
> - `API-get-block-children`: 블록 조회
> - `API-retrieve-a-page`: 페이지 조회

## 설정 예시 (커들마켓 기준)

```yaml
GitHub 레포: jjub0217/cuddle-market
base 브랜치: develop
DAILY SCRUM DB: 30ff2b30-7961-817e-9601-d836505c5b04
사용자 ID: 7c32774b-0096-4545-a9fe-7cfec90faa15
사용자 이름: 강주현
```
