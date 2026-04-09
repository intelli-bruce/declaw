# declaw Mapping Rules

**Version**: v0.2
**Last verified**: 2026-04-09 (Claude Code docs + OpenClaw docs)
**Source**: `REFERENCES.md`

OpenClaw 워크스페이스 파일을 Claude Code의 **공식 위치**로 배치하는 규칙입니다. 창작 금지. 공식 문서 기반으로만 결정.

---

## 🎯 핵심 원칙

1. **Auto memory는 Claude가 자동 작성하는 공간**입니다. 사용자가 직접 편집한 지시사항은 auto memory가 아닌 **CLAUDE.md / rules / hooks**로 가야 합니다.
2. **MEMORY.md는 공식 auto memory 인덱스**입니다 — 파일명 보존, 기존 내용 병합.
3. **파일명 접두사를 임의로 바꾸지 마세요** (`user_*`, `reference_*` 같은 것은 비공식).
4. **서브디렉토리 금지** — `memory/daily/`, `memory/topics/`는 공식 명시 없음. 플랫 구조로.
5. **Frontmatter 자동 추가 금지** — auto memory는 순수 Markdown. `.claude/rules/*.md`만 `paths:` frontmatter 지원.

---

## 📂 Destination 카테고리 (5개)

| 카테고리 | Claude Code 위치 | 공식 출처 |
|---|---|---|
| `user-claudemd` | `~/.claude/CLAUDE.md` | [memory.md](https://code.claude.com/docs/en/memory.md) — "User instructions" |
| `project-claudemd` | `./CLAUDE.md` 또는 `./.claude/CLAUDE.md` | [memory.md](https://code.claude.com/docs/en/memory.md) — "Project instructions" |
| `user-rules` | `~/.claude/rules/*.md` | [memory.md](https://code.claude.com/docs/en/memory.md) — "User rules" (재귀 발견) |
| `user-hook` | `~/.claude/hooks/*.sh` + `~/.claude/settings.json` | [hooks.md](https://code.claude.com/docs/en/hooks.md) |
| `auto-memory` | `~/.claude/projects/<project>/memory/*.md` | [memory.md](https://code.claude.com/docs/en/memory.md) — "Auto memory" |

---

## 🗂 File Mapping Table

### Memory & Instructions (OpenClaw 최상위 `.md`)

| OpenClaw 파일 | Destination Category | Target | 메커니즘 | Rationale |
|---|---|---|---|---|
| `IDENTITY.md` | `user-claudemd` | `~/.claude/CLAUDE.md` (append section `## Identity`) | Idempotent append | 정체성/페르소나는 모든 프로젝트에 주입되어야 하므로 user scope instructions |
| `USER.md` | `user-claudemd` | `~/.claude/CLAUDE.md` (append section `## User Profile`) | Idempotent append | 사용자 프로필도 user scope |
| `SOUL.md` | `user-claudemd` | `~/.claude/CLAUDE.md` (append section `## Mission & Values`) | Idempotent append | 미션/가치 = user instructions |
| `AGENTS.md` | `project-claudemd` | `./CLAUDE.md` (cwd 기준) | New file or append | 프로젝트별 에이전트 역할 분담. 전역이면 `~/.claude/CLAUDE.md`로 이동 옵션 |
| `TOOLS.md` | `user-rules` | `~/.claude/rules/tools.md` | New file | 도구 사용 규칙. `paths:` frontmatter로 조건부 로드 활용 가능 |
| `HEARTBEAT.md` | `user-hook` | `~/.claude/hooks/heartbeat.sh` + `~/.claude/settings.json` 수정 | **Convert** (파일 → 자동화) | 정기 체크는 파일이 아닌 SessionStart hook 또는 cron. 원본 Markdown은 shell 스크립트로 변환 필요 |
| `MEMORY.md` | `auto-memory` | `~/.claude/projects/<project>/memory/MEMORY.md` | **Merge** (이름 보존) | 공식 auto memory 인덱스. 기존 MEMORY.md 있으면 백업 후 `<!-- from openclaw -->` 구분자로 append |

### Memory subdir (`~/.openclaw/workspace/memory/`)

| OpenClaw | Destination Category | Target | 메커니즘 |
|---|---|---|---|
| `memory/YYYY-MM-DD*.md` (일일 로그) | `auto-memory` | `~/.claude/projects/<project>/memory/<filename>` | Copy flat (subdirectory 금지) |
| `memory/<topic>.md` (주제별) | `auto-memory` | `~/.claude/projects/<project>/memory/<topic>.md` | Copy |
| `memory/<subdir>/*.md` (재귀) | `auto-memory` | 기본 **skip** (공식 구조와 충돌). `--include-subdirs` 옵션 시 플랫화해서 복사 (prefix 자동) |

### MCP Servers (`~/.openclaw/workspace/config/mcporter.json`)

| Source | 메커니즘 |
|---|---|
| `.mcpServers.<name>` | `claude mcp add <name> --scope user -e KEY=VAL ... -- <command> <args>` (공식 CLI) |

**Skip 조건**:
- 이미 `claude mcp list`에 있으면 skip (이름 일치)
- OpenClaw 내부 전용 플러그인 (tavily, acpx 등 OpenClaw 플러그인 시스템에 의존) → skip with warning

**충돌 해결**:
- 기본: skip + 경고
- `--force`: 덮어쓰기
- `--rename`: `<name>_openclaw` 자동 이름 변경

### Skills (`~/.openclaw/workspace/skills/*`)

| Source | Destination | 메커니즘 |
|---|---|---|
| `skills/<name>/SKILL.md` (디렉토리 스킬) | `~/.claude/skills/<name>/` | 디렉토리 전체 복사 + SKILL.md frontmatter 검증 |
| `skills/<name>.md` (단일 파일 스킬) | `~/.claude/skills/<name>.md` | 파일 복사 + frontmatter 검증 |

**공식 frontmatter 필드** (검증 대상):
- 필수 추천: `name`, `description`
- 선택: `allowed-tools`, `paths`, `disable-model-invocation`, `user-invocable`, `agent`, `context`, `argument-hint`, `model`, `effort`, `hooks`, `shell`

비공식 필드 있으면 warning 출력 (제거는 하지 않음 — 사용자 판단).

**Skip 조건 (기본)**:
- 워크스페이스 전용 prefix: 사용자가 특정 프로젝트 전용으로 만든 스킬 (이름이 특정 프로젝트/조직명으로 시작하는 것들)
- 예: `my-company-*`, `project-x-*` 등 — 사용자 환경마다 다르므로, 스캔 시 **패턴 감지 + 사용자 확인**으로 결정
- `--include-workspace` 옵션으로 override 가능
- `--include-workspace` 옵션으로 override 가능
- 충돌: `~/.claude/skills/<name>`이 이미 있으면 skip + 경고

---

## 🚫 명시적으로 하지 말아야 할 것

| ❌ Don't | 이유 |
|---|---|
| `IDENTITY.md` → `user_identity.md` (auto memory) | auto memory는 Claude 자동 작성용 |
| `USER.md` → `user_profile.md` (auto memory) | 동일 |
| `memory/daily/`, `memory/topics/` 서브디렉토리 생성 | 공식 명시 없음 |
| Auto memory 파일에 frontmatter 추가 | 순수 Markdown이 공식 |
| `MEMORY.md` 이름 변경 (`project_active_state.md` 등) | 공식 인덱스 파일 |
| `~/.claude/settings.json`에 `mcpServers` 직접 편집 | `claude mcp add` CLI가 공식 |
| Managed policy CLAUDE.md 편집 (`/Library/Application Support/ClaudeCode/CLAUDE.md`) | 조직 정책 경로, 개인 사용자 영역 아님 |
| Enterprise policy memory 건드리기 | 동일 |
| 원본 OpenClaw 파일 삭제 | 단방향 복사, 원본 보존 |

---

## 🛡 Status 타입

실행 중 각 파일에 부여되는 상태:

| Status | 의미 | 색상 (리포트) |
|---|---|---|
| `NEW` | 대상에 파일 없음 → 새로 생성 | 초록 |
| `APPEND` | CLAUDE.md에 섹션 추가 (idempotent — 같은 섹션 있으면 skip) | 시안 |
| `MERGE` | 기존 파일과 병합 (MEMORY.md 전용) | 노랑 |
| `CONVERT` | 파일 → 다른 형식 변환 (HEARTBEAT → hook) | 주황 |
| `NEEDS-CWD` | 프로젝트 경로 필요 (AGENTS.md) — 사용자 확인 | 파랑 |
| `CONFLICT` | 같은 이름 존재 — skip/overwrite/rename 선택 | 노랑 |
| `SKIP` | 워크스페이스 전용 또는 이미 등록됨 | 회색 |

---

## 📐 프로젝트 경로 감지 (`<project>` 결정)

Auto memory 경로의 `<project>` 부분은 Claude Code가 자동 결정:
- **Git 레포 안**: 레포 루트 경로 기반
- **여러 worktree**: 동일 git repo면 같은 memory 디렉토리 공유
- **Git 밖**: 프로젝트 루트 디렉토리 경로 기반

**declaw**는 이 계산을 직접 하지 않습니다. 대신:
1. `find -L ~/.claude/projects -type d -name "memory" | head -1`로 기존 auto memory 디렉토리 탐지
2. 여러 개 있으면 사용자에게 선택 요청
3. 없으면 cwd 기반으로 생성 제안

---

## 🔄 변경 이력

| 버전 | 날짜 | 변경 |
|---|---|---|
| v0.2 | 2026-04-09 | 공식 문서 기반 전면 재작성. Auto memory 오분류 수정. 5 카테고리로 분산. |
| v0.1 | 2026-04-09 | 초안 (전부 auto memory로 잘못 매핑). Deprecated. |
