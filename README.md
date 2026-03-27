# Claude Code Skills

Claude Code에서 사용하는 개발 생산성 스킬 모음입니다.

## 스킬 목록

### 학습/성장

| 스킬 | 설명 |
|------|------|
| [learn-log](skills/learn-log) | 세션 히스토리를 분석하여 Q&A, 인사이트, 의사결정을 주간 학습 로그에 자동 기록 |
| [blog-write](skills/blog-write) | 학습 로그의 블로그 후보를 기반으로 기술 블로그 초안 작성 |
| [study-roadmap](skills/study-roadmap) | 채용공고 + 이력서 → 갭 분석 → 맞춤 공부 로드맵 자동 생성 |

### 개발 워크플로우

| 스킬 | 설명 |
|------|------|
| [bug-fix](skills/bug-fix) | 버그 수정 자동화 — QA 티켓 생성 → GitHub 이슈 → worktree에서 수정 → PR 생성 |
| [create-issue](skills/create-issue) | GitHub 이슈 생성 + 브랜치 생성 자동화 |
| [commit-push](skills/commit-push) | 커밋 → 푸시 → PR 생성 자동화 |
| [refactor-scan](skills/refactor-scan) | React Best Practice 기반 코드 리팩토링 자동화 |

### 프로젝트 관리

| 스킬 | 설명 |
|------|------|
| [context-sync](skills/context-sync) | Notion + GitHub에서 정보 수집 → 싱크 문서 + Daily Scrum 생성 |
| [daily-scrum](skills/daily-scrum) | GitHub 커밋 기반 Daily Scrum 페이지 자동 생성 |

### 유틸리티

| 스킬 | 설명 |
|------|------|
| [my-session-wrap](skills/my-session-wrap) | 세션 종료 시 작업 정리, 문서 업데이트, 학습 기록 |
| [my-trouble-detect](skills/my-trouble-detect) | 에러/삽질 발생 시 컨텍스트 자동 수집 및 분석 |

## 설치 방법

### 전체 설치

```bash
# 레포 클론
git clone https://github.com/jjub0217/claude-code-skills.git

# 프로젝트의 .claude/skills/ 에 심볼릭 링크 생성
ln -s /path/to/claude-code-skills/skills/learn-log /your-project/.claude/skills/learn-log
```

### 개별 설치

원하는 스킬 폴더만 프로젝트의 `.claude/skills/`에 복사하면 됩니다.

```bash
cp -r skills/bug-fix /your-project/.claude/skills/
```

## 커스터마이징

일부 스킬은 프로젝트별 설정이 필요합니다.

- **bug-fix**: Notion DB ID 또는 로컬 QA 문서 경로
- **context-sync**: Notion DB ID, GitHub 레포명
- **daily-scrum**: Notion DB ID 또는 로컬 scrum 문서 경로
- **create-issue**: 브랜치 네이밍 컨벤션, 이슈 템플릿
- **commit-push**: PR 템플릿, base 브랜치

각 스킬의 `SKILL.md` 내 주석을 참고하여 프로젝트에 맞게 수정하세요.

## 기술 스택

- [Claude Code](https://claude.com/claude-code) — Anthropic의 CLI 기반 AI 코딩 도구
- GitHub CLI (`gh`) — 이슈/PR 자동화
- Notion MCP (선택) — 프로젝트 관리 연동

## 라이선스

MIT
