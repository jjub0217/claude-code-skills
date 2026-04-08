# 코드 리팩토링 스캔 자동화

Best Practice 규칙 기반으로 코드를 스캔하고, 위반 사항을 리팩토링한 뒤, PR 생성 및 Notion에 근거 문서를 작성하는 자동화 워크플로우.

**전제 조건**:
- Best Practice 규칙 파일이 설치되어 있어야 함
- GitHub CLI (gh) 필수. Notion MCP는 선택사항.

## 설정

### 필수
| 항목 | 설명 | 예시 |
|------|------|------|
| 대상 프레임워크 | 스캔 대상 프레임워크 | `React` |
| 규칙 파일 경로 | Best Practice 규칙 파일이 위치한 디렉토리 | `.agents/skills/vercel-react-best-practices/rules/` |
| 스캔 대상 경로 | 기본 스캔 대상 디렉토리 | `src/features/admin/` |
| GitHub 레포 | `{owner}/{repo}` 형식 | `your-org/your-repo` |
| base 브랜치 | PR의 base 브랜치 | `develop` |

### 선택 (Notion 연동)
| 항목 | 설명 | 예시 |
|------|------|------|
| DAILY SCRUM DB | Notion Database ID | `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` |
| 사용자 ID | Notion 사용자 ID | `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` |
| 사용자 이름 | Notion 테이블에 표시할 이름 | `홍길동` |
| Notion API Token 경로 | MCP 로그에서 토큰을 확인할 경로 | `~/Library/Caches/claude-cli-nodejs/{프로젝트경로}/mcp-logs-notion/*.txt` |
| 블로그 초안 경로 | 블로그 초안 파일 경로 | `docs/blog-drafts/` |

### 선택 (제외 경로)
| 항목 | 설명 | 예시 |
|------|------|------|
| 제외 경로 | 스캔에서 제외할 경로 목록 | `node_modules/`, `dist/`, `*.test.tsx` |

### 선택 (이슈/PR 라벨)
| 항목 | 설명 | 기본값 |
|------|------|--------|
| 리팩토링 라벨 | GitHub 이슈에 부여할 라벨 | `REFACTOR` |

> Notion 설정이 없으면 Step 7(Notion 근거 문서 생성)은 건너뜁니다.

## 📌 컨벤션 참고
- 커밋: `docs/conventions.md`
- PR 템플릿: `.github/PULL_REQUEST_TEMPLATE.md`
- 브랜치: `종류/이슈번호--브랜치이름` (소문자)

---

## 🔄 작업 순서

### Step 1: 스캔 대상 확인

사용자에게 질문:
- "코드 전체를 스캔할까요, 특정 파일/폴더만 스캔할까요?"

기본값: `{설정의 스캔 대상 경로}` 전체

### Step 2: 규칙 기반 코드 스캔

규칙 파일을 기준으로 대상 코드를 스캔한다. 아래 패턴을 **병렬로** 탐색한다.

#### 스캔 패턴 목록

**CRITICAL (Eliminating Waterfalls)**:
- `async-parallel`: 독립적 비동기 작업의 직렬 await 또는 개별 .then() 호출 탐지
  - 패턴: `useEffect` 내부에서 여러 개의 독립 fetch를 개별 호출하는 경우
  - 패턴: 연속된 `await` 문으로 독립 API를 직렬 호출하는 경우

**CRITICAL (Bundle Size)**:
- `bundle-barrel-imports`: barrel file(`index.ts`)에서 전체 import 탐지
- `bundle-dynamic-imports`: 모달, 차트 등 무거운 컴포넌트의 static import 탐지

**MEDIUM-HIGH (Client-Side Data Fetching)**:
- `client-swr-dedup`: useEffect+setState로 데이터 fetching하는 패턴 탐지 (useQuery/SWR 미사용)
- `client-event-listeners`: 글로벌 이벤트 리스너 중복 등록 탐지

**MEDIUM (Re-render)**:
- `rerender-derived-state-no-effect`: useEffect+setState로 파생 상태를 동기화하는 패턴 탐지
- `rerender-memo-with-default-value`: 컴포넌트 props에 인라인 기본값 (`= []`, `= {}`) 탐지
- `rerender-functional-setstate`: setState에 함수형 업데이터 미사용 탐지

**MEDIUM (Rendering)**:
- `rendering-conditional-render`: `&&` 조건부 렌더링 탐지 (삼항 연산자로 변경 필요)

#### 스캔 방법

```
Grep 도구로 각 패턴을 병렬 검색:
1. useEffect 내부 setState: Grep(pattern="useEffect\(.*\{[\s\S]*?set[A-Z]", multiline=true)
2. && 조건부 렌더링: Grep(pattern="&& \(", glob="*.tsx")
3. 직렬 await: Grep(pattern="await.*;\s*\n\s*(?:const|let).*=\s*await", multiline=true)
4. barrel import: Grep(pattern="from '\./index'|from '\.'")
5. 인라인 기본값: Grep(pattern="= \[\]|= \{\}")

각 탐지 결과에 대해 파일을 직접 읽어서 실제 위반인지 확인한다.
예: useEffect 내부 setState가 파생 상태 동기화인지, DOM API 제어인지 구분
```

#### 스캔 결과 정리

위반 사항을 다음 형식으로 정리:

```
| 순위 | 규칙 ID | 우선순위 | 파일 | 설명 |
|------|---------|----------|------|------|
| 1 | async-parallel | CRITICAL | MembersDashboard.tsx | 3개 독립 fetch 병렬화 미적용 |
| 2 | rendering-conditional-render | MEDIUM | Modal 6개 | && → 삼항 연산자 |
```

**사용자에게 스캔 결과 보여주고 확인** (AskUserQuestion 도구 사용):

위반 목록 전체를 텍스트로 표시한 뒤, AskUserQuestion 도구로 다음 두 가지 질문을 동시에 보여준다:

```
질문 1: "리팩토링을 진행할까요?" (header: "진행 여부")
- options:
  - "전체 진행 (권장)" — 발견된 모든 위반 사항을 리팩토링합니다
  - "선택 항목만 진행" — 제외할 항목을 선택한 뒤 나머지만 리팩토링합니다
  - "취소" — 리팩토링을 진행하지 않습니다

질문 2 (multiSelect: true): "제외할 위반 항목을 선택하세요" (header: "제외 항목")
- options: 스캔에서 발견된 위반 항목들을 각각 옵션으로 나열
  - label: "{순위}. {규칙ID}" / description: "{파일} — {설명}"
  - 예: label: "1. async-parallel" / description: "RecommendContent.jsx — 3개 독립 fetch 병렬화 미적용"
```

- "전체 진행"을 선택하면 제외 항목 무시하고 전체 리팩토링
- "선택 항목만 진행"을 선택하면 질문 2에서 체크된 항목을 제외하고 리팩토링
- "취소"를 선택하면 워크플로우 종료

### Step 3: GitHub 이슈 생성

이슈 템플릿: **Refactor**

```bash
gh issue create --title "refactor: Best Practice 규칙 기반 코드 리팩토링" --body "$(cat <<'EOF'
## ♻️ 리팩토링 대상

Best Practice 규칙으로 코드를 스캔하여 발견된 위반 사항

## 📌 개선 이유

공개된 성능 최적화 규칙 기준으로 코드의 성능과 일관성을 개선

### 발견된 위반 목록

{스캔 결과 표}

## 🛠 기대 결과

{위반 사항별 체크리스트}
EOF
)" --label "{설정의 리팩토링 라벨}"
```

이슈 번호 저장 → `{ISSUE_NUMBER}`

### Step 4: 브랜치 생성

```bash
git checkout {설정의 base 브랜치}
git pull origin {설정의 base 브랜치}
git checkout -b refactor/{ISSUE_NUMBER}--best-practice
```

### Step 5: 리팩토링 수행

스캔에서 발견된 위반 사항을 **우선순위 순서대로** 수정한다.

수정 시 각 규칙의 상세 가이드를 참조:
```
{설정의 규칙 파일 경로}/{규칙ID}.md
```

**수정 원칙**:
- 기능 변경 없이 성능/패턴만 개선
- 한 규칙씩 수정 후 동작 확인
- 수정 전/후 코드를 기록 (블로그 및 Notion 문서용)

### Step 6: 커밋 → 푸시 → PR 생성

**커밋**:
```bash
git add [수정된 파일들]
git commit -m "refactor: Best Practice 규칙 기반 코드 리팩토링(#{ISSUE_NUMBER})"
```

**푸시**:
```bash
git push -u origin refactor/{ISSUE_NUMBER}--best-practice
```

**PR 생성**:
```bash
gh pr create --base {설정의 base 브랜치} --title "refactor: Best Practice 규칙 기반 코드 리팩토링(#{ISSUE_NUMBER})" --body "$(cat <<'EOF'
## 📌 개요

- Best Practice 규칙으로 코드를 스캔하여 발견된 위반 사항을 리팩토링

## 🔧 작업 내용

{위반 사항별 수정 내용 체크리스트}

## 📎 관련 이슈

Closes #{ISSUE_NUMBER}

## 💬 리뷰어 참고 사항

- 적용된 규칙별 before/after 코드는 Notion 문서 참조
- {NOTION_PAGE_URL}
EOF
)"
```

PR URL 저장 → `{PR_URL}`

### Step 7: Notion DAILY SCRUM 페이지 생성 (Notion 설정 시에만)

> Notion 설정이 없으면 이 단계를 건너뜁니다.

DAILY SCRUM DB에 리팩토링 근거 문서를 생성한다.

**페이지 제목**: `Best Practice 리팩토링 — {위반 규칙 요약}`

**Notion API Token 확인**:
```bash
cat {설정의 Notion API Token 경로} | grep "Authorization: Bearer" | head -1
```

**페이지 생성** (curl):
```bash
curl -X POST 'https://api.notion.com/v1/pages' \
  -H "Authorization: Bearer {NOTION_API_TOKEN}" \
  -H "Content-Type: application/json" \
  -H "Notion-Version: 2022-06-28" \
  -d '{
    "parent": {
      "database_id": "{설정의 DAILY SCRUM DB}"
    },
    "properties": {
      "이름": {
        "title": [{"text": {"content": "Best Practice 리팩토링 — 위반 규칙 요약"}}]
      },
      "작성일시": {
        "date": {"start": "YYYY-MM-DD"}
      },
      "참여자": {
        "people": [{"id": "{설정의 사용자 ID}"}]
      },
      "구분": {
        "select": {"name": "개발"}
      }
    }
  }'
```

생성된 페이지 ID 저장 → `{NOTION_PAGE_ID}`

**페이지 본문 작성** (curl):
```bash
curl -X PATCH 'https://api.notion.com/v1/blocks/{NOTION_PAGE_ID}/children' \
  -H "Authorization: Bearer {NOTION_API_TOKEN}" \
  -H "Content-Type: application/json" \
  -H "Notion-Version: 2022-06-28" \
  -d '{
    "children": [
      // 각 위반 규칙별로 다음 블록 구조를 반복:
      // heading_2: 규칙 ID + 규칙명
      // divider
      // paragraph: 왜 이 규칙을 지켜야 하는지 (규칙 파일에서 발췌)
      // heading_3: Before
      // code block: 수정 전 코드
      // heading_3: After
      // code block: 수정 후 코드
      // paragraph: 개선 효과 설명
      // divider
      //
      // 마지막에 PR 링크 추가:
      // heading_2: PR
      // paragraph: {PR_URL}
    ]
  }'
```

**본문 구조** (위반 규칙 1개당):
```
## {규칙 ID}: {규칙 설명}
---
왜 이렇게 리팩토링하면 좋은지에 대한 근거 설명
({설정의 규칙 파일 경로}/{규칙ID}.md 내용 참조)

### Before
(수정 전 코드 블록)

### After
(수정 후 코드 블록)

개선 효과: (구체적 설명)
---
```

**마지막 섹션**:
```
## PR
{PR_URL}

## GitHub 이슈
https://github.com/{설정의 GitHub 레포}/issues/{ISSUE_NUMBER}
```

Notion 페이지 URL 저장 → `{NOTION_PAGE_URL}`

### Step 8: 블로그 초안 업데이트 (설정 시에만)

> 블로그 초안 경로가 설정되어 있으면 실행합니다.

`{설정의 블로그 초안 경로}` 하위의 관련 파일 TODO 섹션들을 실제 작업 결과로 업데이트한다:

- **리팩토링 과정**: 각 규칙별 수정 상세 기술
- **전/후 비교**: before/after 코드 예시, 변경 파일 목록

### Step 9: 완료 안내

```
✅ Best Practice 리팩토링 완료!

📊 스캔 결과: {위반 N건} 발견 → 전체 수정 완료
🐛 GitHub 이슈: #{ISSUE_NUMBER}
🔀 PR: {PR_URL}
🌿 브랜치: refactor/{ISSUE_NUMBER}--best-practice
📝 Notion 근거 문서: {NOTION_PAGE_URL} (Notion 설정 시)
📄 블로그 초안: {설정의 블로그 초안 경로} (설정 시)

📌 다음 단계:
- 코드 리뷰 및 머지는 직접 진행해주세요
```

---

## ⚠️ 주의사항

### 금지 사항
- `--no-verify` 플래그 사용 금지
- force push 절대 금지
- main/master에 직접 커밋 금지
- PR base 브랜치는 반드시 `{설정의 base 브랜치}`

### 스캔 시 주의
- useEffect 내 setState가 모두 위반은 아님 (DOM API 제어, 이벤트 리스너 등록은 정상)
- `&&` 조건부 렌더링에서 좌항이 항상 boolean인 경우 실제 버그 위험은 낮으나, 일관성 차원에서 수정

---

## 📊 전체 워크플로우 요약

### 🤖 자동화 범위 (이 명령어가 수행)
1. **스캔 대상 확인** (사용자 질문) — 메인 세션
2. **규칙 기반 코드 스캔** (병렬 탐색) — 메인 세션
3. **스캔 결과 확인** (사용자 확인) — 메인 세션
4. **GitHub 이슈 생성** — 메인 세션
5. **브랜치 생성** ({설정의 base 브랜치}에서) — 메인 세션
6. **리팩토링 수행** (우선순위 순) — 메인 세션
7. **커밋 → 푸시 → PR 생성** — 메인 세션
8. **Notion 근거 문서 생성** (설정 시에만) — 메인 세션
9. **블로그 초안 업데이트** (설정 시에만) — 메인 세션
10. **완료 안내** — 메인 세션

### 👤 사용자가 직접 수행
11. **코드 리뷰 및 머지**

---

## 💡 사용자 인터랙션

각 단계마다 사용자 확인:
1. "코드 전체를 스캔할까요, 특정 파일/폴더만 스캔할까요?"
2. "다음 위반 사항이 발견되었습니다. 리팩토링을 진행할까요?" + 위반 목록
3. (PR 생성 완료 후) 전체 링크 안내

## 설정 예시 (커들마켓 기준)

```yaml
대상 프레임워크: React
규칙 파일 경로: .agents/skills/vercel-react-best-practices/rules/
스캔 대상 경로: src/features/admin/
GitHub 레포: jjub0217/cuddle-market
base 브랜치: develop
리팩토링 라벨: REFECTOR
DAILY SCRUM DB: 30ff2b30-7961-817e-9601-d836505c5b04
사용자 ID: 7c32774b-0096-4545-a9fe-7cfec90faa15
사용자 이름: 강주현
Notion API Token 경로: ~/Library/Caches/claude-cli-nodejs/{프로젝트경로}/mcp-logs-notion/*.txt
블로그 초안 경로: docs/blog-drafts/
```
