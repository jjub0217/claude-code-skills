# bug-fix

버그 수정 자동화 -- QA 티켓 생성 → GitHub 이슈 → worktree에서 수정 → PR 생성까지 한번에 처리하는 스킬.

## 기능
- Test Scenario + QA 티켓 자동 생성 (Notion 또는 로컬 마크다운)
- GitHub 이슈 자동 생성
- Worktree에서 격리된 버그 수정 + 커밋 + 푸시 + PR 생성
- QA 티켓에 PR 링크 자동 기입

## 설정
| 구분 | 항목 | 예시 |
|------|------|------|
| 필수 | GitHub 레포 | `your-org/your-repo` |
| 필수 | base 브랜치 | `develop` |
| 선택 | Test Scenario DB (Notion) | Notion collection ID |
| 선택 | QA DB (Notion) | Notion collection ID |
| 선택 | 개발자 ID (Notion) | Notion 사용자 ID |
| 선택 | 카테고리 → 접두사 매핑 | 프로젝트에 맞게 설정 |

Notion 미설정 시 `qa/` 폴더에 마크다운으로 대체 생성됩니다.

## 사용법
```
/bug-fix
```

## 예시
"버그 발견했어"라고 요청하면, 버그 정보를 수집하고 QA 티켓 → 이슈 → worktree 수정 → PR 생성까지 자동으로 처리합니다.
