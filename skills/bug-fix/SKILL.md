# 버그 수정 자동화

버그 발견 시 Test Scenario 생성 → QA 티켓 생성 → GitHub 이슈/브랜치 생성 → 버그 수정 → PR 생성(QA 티켓 링크 포함) → QA 티켓에 PR 링크 기입

**전제 조건**: GitHub CLI (gh) 필수. Notion MCP는 선택사항.

## 설정

### 필수
| 항목 | 설명 | 예시 |
|------|------|------|
| GitHub 레포 | `{owner}/{repo}` 형식 | `jjub0217/cuddle-market` |
| base 브랜치 | PR의 base 브랜치 | `develop` |

### 선택 (Notion 연동)
| 항목 | 설명 | 예시 |
|------|------|------|
| Test Scenario DB | Notion collection ID | `collection://30ff2b30-7961-811b-96df-000b78510e15` |
| QA DB | Notion collection ID | `collection://30ff2b30-7961-8123-9218-000be0dabe38` |
| 개발자 ID | Notion 사용자 ID | `7c32774b-0096-4545-a9fe-7cfec90faa15` |
| 카테고리 → 접두사 매핑 | 프로젝트별 카테고리 | 아래 표 참조 |
| 기능 목록 | QA 티켓의 기능 분류 | 프로젝트에 맞게 설정 |

**카테고리 → 접두사 매핑 예시** (커들마켓 기준):

| 카테고리 | 접두사 |
|---|---|
| 로그인 | SI (Sign In) |
| 아이디 찾기 | ID |
| 아이디 설정 | IS (ID Setting) |
| 비밀번호 설정 | PW (Password) |
| 상품 등록 | PD (Product) |
| 어드민 | AD (Admin) |

**기능 목록 예시** (커들마켓 기준):
`사용자 인증` | `주문리스트` | `배송지리스트` | `결제` | `통계` | `상품리스트` | `신고 관리` | `프로필`

> Notion 설정이 없으면 QA 티켓은 프로젝트 내 `qa/` 폴더에 마크다운으로 생성됩니다.

## 📌 컨벤션 참고
- 커밋: `docs/conventions.md`
- PR 템플릿: `.github/PULL_REQUEST_TEMPLATE.md`
- 브랜치: `종류/이슈번호--브랜치이름` (소문자)

---

## 🔄 작업 순서

### Step 1: 버그 정보 수집

사용자에게 질문:
- "어떤 버그가 발생했나요? (에러 메시지, 재현 방법, 발생 위치 등)"

사용자의 답변을 바탕으로 다음을 판단:

**Test Scenario 정보**:
- **카테고리**: 설정에 정의된 카테고리 중 해당하는 것 (없으면 가장 가까운 것)
- **시나리오 ID**: 카테고리별 접두사 + 순번 (예: `SI-006`)
  - 기존 시나리오 ID를 조회하여 해당 카테고리의 마지막 순번 + 1로 생성
  - 접두사는 설정의 **카테고리 → 접두사 매핑**을 참조
- **시나리오명**: 무엇을 검증하는 시나리오인지 간단히 설명
- **테스트 절차**: 버그를 재현하는 구체적 단계
- **기대결과**: 정상 동작 시 시스템이 보여야 하는 결과

**QA 티켓 정보**:
- **보고유형**: `🚨 결함/버그` | `🎨 UI` | `🔌 장애` | `🐢 성능 이슈` 중 하나
- **환경**: `🌐 웹` | `🖥️ 데스크탑` | `📱 모바일` 중 해당하는 것
- **기능**: 설정에 정의된 기능 목록 중 해당하는 것
- **작업항목 제목**: 버그를 한 줄로 요약

**사용자에게 확인**:
- "다음 내용으로 Test Scenario + QA 티켓을 생성하시겠습니까?"
- 판단한 분류 정보 전체 표시

### Step 2: Test Scenario 생성

#### Notion 설정 시
`notion-create-pages` 도구를 사용하여 Test Scenario DB에 페이지 생성:
- **DB**: `{설정의 Test Scenario DB}`
- **시나리오 ID**: 판단한 시나리오 ID
- **카테고리**: 판단한 카테고리
- **시나리오명**: 판단한 시나리오명
- **테스트 절차**: 판단한 재현 단계
- **기대결과**: 판단한 기대 동작

생성된 Test Scenario 페이지 URL을 저장해둔다 → `{TEST_SCENARIO_URL}`

#### Notion 미설정 시 (로컬 fallback)
프로젝트 내 `qa/` 폴더에 Test Scenario 마크다운 생성:
- 파일: `qa/test-scenarios/{시나리오ID}.md`
- 내용: 시나리오명, 카테고리, 테스트 절차, 기대결과

### Step 3: QA 티켓 생성

#### Notion 설정 시
`notion-create-pages` 도구를 사용하여 QA DB에 페이지 생성:
- **DB**: `{설정의 QA DB}`
- **작업항목**: 버그 요약 제목
- **보고유형**: 판단한 유형
- **진행현황**: `접수`
- **환경**: 판단한 환경
- **기능**: 판단한 기능
- **DEV**: `{설정의 개발자 ID}`
- **시나리오 ID**: `{TEST_SCENARIO_URL}` (relation으로 연결)

생성된 Notion 페이지 URL을 저장해둔다 → `{NOTION_QA_URL}`

**페이지 본문에 템플릿 채우기** (`replace_content`):
```markdown
## 테스트 결과
[버그 현상 설명]

---
## 시나리오
1. [재현 단계]

---
## 예상 원인
- [원인 1]
- [원인 2 (있으면)]

---
## 첨부자료
- 파일: [관련 파일 경로:라인번호]
```

#### Notion 미설정 시 (로컬 fallback)
프로젝트 내 `qa/` 폴더에 QA 티켓 마크다운 생성:
- 파일: `qa/tickets/{YYYY-MM-DD}-{버그요약}.md`
- 내용:

```markdown
# {작업항목 제목}

## 상태: 접수
## 보고유형: {유형}
## 환경: {환경}
## 기능: {기능}

## 테스트 결과
{버그 현상 설명}

---
## 시나리오
1. {재현 단계}

---
## 예상 원인
- {원인}

---
## 첨부자료
- 파일: {경로}
```

### Step 4: GitHub 이슈 생성

**이슈 생성**:
```bash
gh issue create --title "fix: 버그 요약 제목" --body "$(cat <<'EOF'
## 🐛 버그 설명
[버그 상세 설명]

## 🔍 재현 방법
1. [재현 단계]

## ✅ 기대 동작
[정상 동작 설명]

## 📋 QA 티켓
{NOTION_QA_URL 또는 qa/tickets/ 파일 경로}
EOF
)" --label "FIX"
```

이슈 번호 저장 → `{ISSUE_NUMBER}`

**Notion 설정 시**: Notion QA 티켓 상태 업데이트: `접수` → `진행중`

### Step 5: Worktree에서 버그 수정 + 커밋 + 푸시 + PR 생성

**Agent 도구를 `isolation: "worktree"`로 실행**하여, 별도의 작업 디렉토리에서 수정 작업을 수행한다.

이렇게 하면 메인 작업 디렉토리에 영향을 주지 않고, 여러 버그를 병렬로 수정할 수 있다.

**Agent에게 전달할 정보**:
- 이슈 번호: `{ISSUE_NUMBER}`
- 브랜치 이름: `fix/{ISSUE_NUMBER}--브랜치이름`
- QA 티켓 URL 또는 파일 경로: `{NOTION_QA_URL}` 또는 `qa/tickets/` 경로
- 버그 설명 및 재현 방법
- 관련 파일 경로

**Agent가 worktree 내에서 수행할 작업**:

#### 5-1: 브랜치 생성
```bash
git checkout develop
git pull origin develop
git checkout -b fix/{ISSUE_NUMBER}--브랜치이름
```

#### 5-2: 버그 수정
- 관련 코드 분석
- 버그 원인 파악
- 수정 작업 수행

#### 5-3: 변경사항 분석
```bash
git status
git diff
git log -5 --oneline
```

#### 5-4: 커밋
커밋 메시지 형식: `fix: 한글 내용(#{ISSUE_NUMBER})`
```bash
git add [파일들]
git commit -m "fix: 한글 내용(#{ISSUE_NUMBER})"
```

#### 5-5: 푸시
```bash
git push -u origin fix/{ISSUE_NUMBER}--브랜치이름
```

#### 5-6: PR 생성
**중요**: PR body에 `📋 QA 티켓` 섹션을 추가하여 QA 티켓 링크를 포함한다.

```bash
gh pr create --base develop --title "fix: 제목(#{ISSUE_NUMBER})" --body "$(cat <<'EOF'
## 📌 개요

- [버그 설명 및 수정 내용]

## 🔧 작업 내용

- [ ] [수정한 내용 항목별 정리]

## 📎 관련 이슈

Closes #{ISSUE_NUMBER}

## 📋 QA 티켓

{NOTION_QA_URL 또는 qa/tickets/ 파일 경로}

## 📸 스크린샷 (선택)

(필요시 추가)

## 💬 리뷰어 참고 사항

- [리뷰 포인트]
EOF
)"
```

**Agent 실행이 끝나면** PR URL을 반환받는다 → `{PR_URL}`

**사용자 확인**: Agent 실행 전에 "worktree에서 버그 수정을 진행하시겠습니까?" 확인

### Step 6: QA 티켓 업데이트

#### Notion 설정 시
`notion-update-page` 도구를 사용:

1. **페이지 본문에 PR 링크 추가** (`update_content`):
```markdown
## PR
{PR_URL}

## GitHub 이슈
https://github.com/{설정의 GitHub 레포}/issues/{ISSUE_NUMBER}
```

2. **진행현황 업데이트**: `진행중` → `테스트중`

#### Notion 미설정 시 (로컬 fallback)
QA 티켓 마크다운 파일에 PR 링크와 GitHub 이슈 링크를 추가하고, 상태를 `테스트중`으로 변경.

### Step 7: Worktree 정리

Agent 실행이 완료되면 사용된 worktree를 즉시 정리한다:
```bash
git worktree list
# .claude/worktrees/ 하위의 agent worktree를 모두 제거
git worktree remove .claude/worktrees/{agent-id}
```

**중요**: PR 생성 완료 후 반드시 worktree를 정리해야 한다. 정리하지 않으면 VSCode 검색에서 중복 파일이 나타난다.

### Step 8: 완료 안내

```
✅ 버그 수정 완료!

📋 QA 티켓: {NOTION_QA_URL 또는 qa/tickets/ 파일 경로}
🐛 GitHub 이슈: #{ISSUE_NUMBER}
🔀 PR: {PR_URL}
🌿 브랜치: fix/{ISSUE_NUMBER}--브랜치이름

📌 다음 단계:
- 코드 리뷰 및 머지는 직접 진행해주세요
- 머지 후 QA 티켓 상태를 "배포 완료"로 변경해주세요
```

---

## ⚠️ 주의사항

### 금지 사항
- `--no-verify` 플래그 사용 금지
- force push 절대 금지
- main/master에 직접 커밋 금지
- PR base 브랜치는 반드시 `{설정의 base 브랜치}`

---

## 📊 전체 워크플로우 요약

### 🤖 자동화 범위 (이 명령어가 수행)
1. **버그 정보 수집** (사용자 질문) — 메인 세션
2. **Test Scenario 생성** (Notion 또는 로컬 fallback) — 메인 세션
3. **QA 티켓 생성** (접수 상태 + 시나리오 ID 연결) — 메인 세션
4. **GitHub 이슈 생성** (fix/ 타입) — 메인 세션
5. **QA 티켓 상태 업데이트** (접수 → 진행중) — 메인 세션
6. **버그 수정 + 커밋 + 푸시 + PR 생성** — Worktree Agent
7. **Worktree 정리** (agent worktree 즉시 제거) — 메인 세션
8. **QA 티켓 업데이트** (PR 링크 기입 + 테스트중) — 메인 세션
9. **완료 안내** — 메인 세션

### 👤 사용자가 직접 수행
10. **코드 리뷰 및 머지**
11. **QA 티켓 "배포 완료" 변경**

---

## 💡 사용자 인터랙션

각 단계마다 사용자 확인:
1. "어떤 버그가 발생했나요?"
2. "다음 내용으로 Test Scenario + QA 티켓을 생성하시겠습니까?" + 분류 정보 표시
3. "브랜치 이름: `fix/이슈번호--이름` - worktree에서 버그 수정을 진행하시겠습니까?"
4. (Agent 완료 후) 수정 내용 + PR URL 안내
5. (완료 후) 전체 링크 안내
