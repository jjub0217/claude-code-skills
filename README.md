# Claude Code Skills

Claude Code에서 사용하는 개발 생산성 스킬 모음입니다.

## 사전 요구사항

### 필수

| 도구 | 설명 | 설치 방법 |
|------|------|-----------|
| [Claude Code](https://claude.ai/download) | Anthropic의 CLI 기반 AI 코딩 도구. 이 스킬들이 동작하는 환경 | `npm install -g @anthropic-ai/claude-code` |
| [GitHub CLI](https://cli.github.com/) | 이슈/PR 자동화에 사용 (bug-fix, create-issue, commit-push 등) | `brew install gh` (macOS) / [설치 가이드](https://github.com/cli/cli#installation) |
| Git | 버전 관리 | 대부분의 OS에 기본 설치됨 |

### 선택

| 도구 | 설명 | 필요한 스킬 |
|------|------|-------------|
| [Notion MCP](https://github.com/anthropics/claude-code-notion) | Notion 연동 (QA 티켓, Daily Scrum 등) | bug-fix, context-sync, daily-scrum, refactor-scan |
| Node.js 18+ | Claude Code 실행에 필요 | 전체 |

### 환경 확인

```bash
# Claude Code 설치 확인
claude --version

# GitHub CLI 설치 및 로그인 확인
gh --version
gh auth status

# Git 확인
git --version
```

### Claude Code 스킬 구조

Claude Code는 프로젝트의 `.claude/skills/` 폴더에서 스킬을 인식합니다.

```
your-project/
  .claude/
    skills/
      bug-fix/
        SKILL.md    ← Claude Code가 이 파일을 읽고 실행
      learn-log/
        SKILL.md
      ...
```

스킬은 `/스킬명` 형식으로 실행합니다:
```
> /bug-fix
> /learn-log
> /commit-push
```

## Quick Start

```bash
# 1. 레포 클론
git clone https://github.com/jjub0217/claude-code-skills.git

# 2. 프로젝트에 전체 스킬 심볼릭 링크
mkdir -p /your-project/.claude/skills
ln -s /path/to/claude-code-skills/skills/* /your-project/.claude/skills/

# 3. Claude Code 세션을 재시작 (스킬은 세션 시작 시 로드됨)
#    이미 실행 중인 세션에서는 새로 추가한 스킬이 인식되지 않습니다.

# 4. 스킬의 SKILL.md에서 설정 섹션을 프로젝트에 맞게 수정 후 사용
#    예: GitHub 레포, base 브랜치, Notion DB ID 등
```

## 스킬 목록

### 학습/성장

| 스킬 | 설명 | 설정 필요 | Notion |
|------|------|-----------|--------|
| [learn-log](skills/learn-log) | 세션 히스토리를 분석하여 Q&A, 인사이트, 의사결정을 주간 학습 로그에 자동 기록 | 선택 | 불필요 |
| [blog-write](skills/blog-write) | 학습 로그의 블로그 후보를 기반으로 기술 블로그 초안 작성 | 선택 | 불필요 |
| [study-roadmap](skills/study-roadmap) | 채용공고 + 이력서 → 갭 분석 → 맞춤 공부 로드맵 자동 생성 | 선택 | 불필요 |

### 개발 워크플로우

| 스킬 | 설명 | 설정 필요 | Notion |
|------|------|-----------|--------|
| [bug-fix](skills/bug-fix) | 버그 수정 자동화 — QA 티켓 생성 → GitHub 이슈 → worktree에서 수정 → PR 생성 | **필수** | 선택 |
| [create-issue](skills/create-issue) | GitHub 이슈 생성 + 브랜치 생성 자동화 | **필수** | 불필요 |
| [commit-push](skills/commit-push) | 커밋 → 푸시 → PR 생성 자동화 | **필수** | 불필요 |
| [refactor-scan](skills/refactor-scan) | Best Practice 규칙 기반 코드 리팩토링 자동화 | **필수** | 선택 |

### 프로젝트 관리

| 스킬 | 설명 | 설정 필요 | Notion |
|------|------|-----------|--------|
| [context-sync](skills/context-sync) | Notion + GitHub에서 정보 수집 → 싱크 문서 + Daily Scrum 생성 | **필수** | 선택 |
| [daily-scrum](skills/daily-scrum) | GitHub 커밋 기반 Daily Scrum 페이지 자동 생성 | **필수** | 선택 |

### 유틸리티

| 스킬 | 설명 | 설정 필요 | Notion |
|------|------|-----------|--------|
| [my-session-wrap](skills/my-session-wrap) | 세션 종료 시 작업 정리, 문서 업데이트, 학습 기록 | 없음 | 불필요 |
| [my-trouble-detect](skills/my-trouble-detect) | 에러/삽질 발생 시 컨텍스트 자동 수집 및 분석 | 선택 | 불필요 |

### 설정 가이드 요약

각 스킬의 `SKILL.md` 상단에 설정 테이블이 있습니다.

| 설정 유형 | 해당 스킬 | 설정 내용 |
|-----------|----------|-----------|
| GitHub 레포 (필수) | bug-fix, create-issue, commit-push, context-sync, daily-scrum, refactor-scan | `{owner}/{repo}` 형식 |
| base 브랜치 (필수/선택) | bug-fix, create-issue, commit-push, daily-scrum, refactor-scan | 기본값 `develop` |
| Notion DB ID (선택) | bug-fix, context-sync, daily-scrum, refactor-scan | Notion 연동 시 필요 |
| 파일 경로 (선택) | learn-log, blog-write, study-roadmap, my-trouble-detect | 출력 경로 커스터마이징 |

## 설치 방법

### 전체 설치

```bash
# 레포 클론
git clone https://github.com/jjub0217/claude-code-skills.git

# 프로젝트의 .claude/skills/ 에 전체 심볼릭 링크 생성
mkdir -p /your-project/.claude/skills
ln -s /path/to/claude-code-skills/skills/* /your-project/.claude/skills/
```

### 개별 설치

원하는 스킬만 골라서 심볼릭 링크하면 됩니다.

```bash
mkdir -p /your-project/.claude/skills
ln -s /path/to/claude-code-skills/skills/bug-fix /your-project/.claude/skills/bug-fix
```

## 커스터마이징

스킬 설치 후 `SKILL.md`의 설정 섹션을 프로젝트에 맞게 수정하세요.

- **필수 설정**: GitHub 레포, base 브랜치 등 스킬 동작에 필수인 항목
- **선택 설정 (Notion)**: Notion DB ID, 사용자 ID 등. 미설정 시 로컬 마크다운으로 대체
- **선택 설정 (경로)**: 파일 출력 경로 등. 기본값이 제공됨

각 스킬 폴더의 `README.md`에서 설정 요약을 확인할 수 있습니다.

## 기술 스택

- [Claude Code](https://claude.com/claude-code) — Anthropic의 CLI 기반 AI 코딩 도구
- GitHub CLI (`gh`) — 이슈/PR 자동화
- Notion MCP (선택) — 프로젝트 관리 연동

## 라이선스

MIT
