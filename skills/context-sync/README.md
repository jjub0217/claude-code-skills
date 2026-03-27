# context-sync

Notion + GitHub에서 정보를 수집하여 싱크 문서 + Daily Scrum을 자동 생성하는 스킬.

## 기능
- GitHub PR/이슈/커밋 정보 자동 수집
- Notion 태스크/스크럼/일정 수집 (설정 시)
- 하이라이트 및 액션 아이템 자동 정리
- Daily Scrum 페이지 생성 (Notion 또는 로컬 마크다운)

## 설정
| 구분 | 항목 | Notion 필요 |
|------|------|-------------|
| 필수 | GitHub 레포 | 불필요 |
| 선택 | Notion 메인 페이지 ID | 필요 |
| 선택 | TASK DISTRIBUTION DB | 필요 |
| 선택 | DAILY SCRUM DB | 필요 |
| 선택 | Schedule List DB | 필요 |
| 선택 | 사용자 ID / 이름 | 필요 |

Notion 미설정 시 GitHub만 수집하고, Daily Scrum은 `scrum/` 폴더에 마크다운으로 생성됩니다.

## 사용법
```
/context-sync
```

## 예시
"싱크해줘"라고 요청하면, GitHub + Notion에서 최근 정보를 병렬 수집하여 싱크 문서와 Daily Scrum 페이지를 자동 생성합니다.
