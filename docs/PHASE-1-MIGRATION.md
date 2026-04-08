# Phase 1: `declaw` — Migration CLI Spec

**버전**: Draft v0.1
**작성일**: 2026-04-09
**상태**: 스펙 확정, Day 0 구현 대기

---

## 🎯 Mission (한 줄)

> **"OpenClaw에서 Claude Code로 30초 만에 이동한다."**

## 📦 What It Is

`declaw`는 **단방향 마이그레이션 CLI**입니다. 기존 OpenClaw 유저가 자기 환경의 메모리·MCP·skills를 Claude Code로 이식할 수 있게 해줍니다.

- 🚀 **한 줄 설치**: `npx declaw`
- 🔍 **자동 감지**: `~/.openclaw/` 구조를 스캔해서 뭐가 있는지 파악
- 🔒 **Dry-run 기본**: 실제 파일 변경 전 항상 변경사항 미리보기
- ✅ **충돌 안전**: 기존 Claude Code 설정 보존, 백업 생성
- 🌐 **크로스 플랫폼**: macOS · Linux · WSL

## 🎭 Who It's For

### 1차 타겟: 기존 OpenClaw 유저
- OpenClaw를 사용 중이지만 Claude Code 공식 기능(Auto Memory, Channels 등)을 활용하고 싶은 사람
- OpenClaw의 개인 메모리·IDENTITY·skills를 버리지 않고 이식하고 싶은 사람
- CLI에 익숙한 개발자

### 2차 타겟: OpenClaw-like 시스템 유저
- 자체 `.openclaw` 또는 유사 구조를 직접 만든 사람
- Claude Code로 갈아타거나 병행하려는 사람

### Non-goals
- ❌ Claude Code를 대체하려는 사람
- ❌ OpenClaw를 써본 적 없는 사람 (Phase 2 이후)
- ❌ 양방향 실시간 동기화가 필요한 사람 (Phase 2+)
- ❌ SaaS 기능을 원하는 사람 (Phase 3+)

## 📐 Scope — 무엇을 마이그레이션하는가

### 1. **Memory Migration**

**Source**: `~/.openclaw/workspace/*.md` + `~/.openclaw/workspace/memory/`
- `AGENTS.md`, `IDENTITY.md`, `TOOLS.md`, `MEMORY.md`, `HEARTBEAT.md`, `SOUL.md`, `USER.md`
- `memory/YYYY-MM-DD-*.md` (일일 로그)
- `memory/*.md` (주제별 파일)

**Target**: `~/.claude/projects/<project>/memory/`
- Claude Code 공식 auto memory 경로
- `MEMORY.md` 인덱스 자동 갱신
- 각 파일에 frontmatter 자동 추가 (type, description)

**변환 규칙**:
| OpenClaw 파일 | → | Claude Code 경로 | 메모리 타입 |
|---|---|---|---|
| `IDENTITY.md` | → | `user_identity.md` | `user` |
| `USER.md` | → | `user_profile.md` | `user` |
| `AGENTS.md` | → | `reference_agents.md` | `reference` |
| `TOOLS.md` | → | `reference_tools.md` | `reference` |
| `HEARTBEAT.md` | → | `reference_heartbeat.md` | `reference` |
| `SOUL.md` | → | `reference_soul.md` | `reference` |
| `MEMORY.md` (지표/상태) | → | `project_active_state.md` | `project` |
| `memory/YYYY-MM-DD-*.md` | → | `memory/daily/YYYY-MM-DD-*.md` | 그대로 |
| `memory/<topic>.md` | → | `memory/<topic>.md` | 그대로 |

**특이 처리**:
- TTL 코멘트 (`[stable]`, `[active]`, `[volatile]`)는 **그대로 보존** (Claude Code가 무시해도 복원 가능)
- 에이전트 이름 (`Black` 등)은 `IDENTITY.md` 내용에서 자동 추출 → frontmatter에 기록
- 중복 파일명은 suffix 자동 추가 (`_openclaw`)

### 2. **MCP Migration**

**Source**: `~/.openclaw/workspace/config/mcporter.json` (또는 자동 감지)

```json
{
  "mcpServers": {
    "my-tool": { "command": "...", "args": [...] },
    "exa": { ... }
  }
}
```

**Target**: `~/.claude.json` (Claude Code 공식 MCP 저장 위치)

**마이그레이션 방법**:
- 각 MCP 서버마다 `claude mcp add --scope user` CLI 명령 실행
- 또는 `~/.claude.json`의 `mcpServers`에 직접 병합 (CLI 실패 시 fallback)

**충돌 처리**:
- 동일 이름 MCP가 Claude Code에 이미 있으면 → 스킵 + 경고
- `--force` 플래그로 덮어쓰기
- `--rename` 플래그로 자동 이름 변경 (e.g. `my-tool` → `my-tool_openclaw`)

**특이 처리**:
- OpenClaw 내부 전용 MCP(OpenClaw 플러그인 시스템에 의존하는 것들)는 **기본 스킵**
- Claude Code와 호환되는 표준 stdio/http MCP만 마이그레이션
- 환경변수/토큰은 그대로 복사 (사용자 경고 표시)

### 3. **Skills Migration**

**Source**: `~/.openclaw/workspace/skills/*` (단일 `.md` 또는 디렉토리)

**Target**: `~/.claude/skills/*`

**변환 규칙**:
- 단일 `.md` 파일 → 그대로 복사
- 디렉토리 스킬 (`SKILL.md` 포함) → 통째로 복사
- Frontmatter 검증:
  - `name`, `description` 필수
  - `allowed-tools`, `argument-hint` 선택
- Claude Code 스킬 스키마와 호환 안 되면 **경고 + 스킵** (사용자가 수동 검토)

**특이 처리**:
- 워크스페이스 전용 스킬(`/my-workspace-*`처럼 특정 환경에만 의미있는 것들)은 스킵 옵션 (`--skip-workspace-specific`)
- 개인 스킬은 본인 환경에서만 의미 있으므로, 다른 사람과 공유 시 리뷰 필요

## 🔍 Detection Logic

사용자 환경이 다양하므로 **하드코딩 금지**. 자동 감지:

```
1. ~/.openclaw/ 존재 확인
2. ~/.openclaw/workspace/ 심볼릭/실제 여부 확인
   - 심볼릭이면 → target path 따라감
3. 다음 순서로 스캔:
   a. workspace/*.md (최상위 메모리 파일)
   b. workspace/memory/**/*.md (재귀)
   c. workspace/config/mcporter.json 또는 workspace/config/*.json에서 mcpServers 키 탐색
   d. workspace/skills/*
4. 감지 결과를 dry-run 리포트로 출력
```

## 💻 CLI Interface

### 기본 흐름 (5분 UX)

```bash
# 1. 실행 (dry-run 기본)
npx declaw

# 출력:
# 🔍 Detecting OpenClaw at ~/.openclaw/ ... ✓ Found
# 📋 Scanning...
#   Memory:  12 files (3 identity, 9 topics)
#   MCP:     5 servers
#   Skills:  34 items (12 workspace-specific, skipped)
#
# 📝 Migration Plan (dry-run):
#   → ~/.claude/projects/.../memory/user_identity.md (new)
#   → ~/.claude/projects/.../memory/user_profile.md (merge with existing)
#   → MCP add: 3 servers (2 skipped: overlap)
#   → Skills: 22 new files
#
# 🚀 Run with --apply to execute. Or --help for options.

# 2. 실제 실행
npx declaw --apply
```

### 전체 옵션

```bash
# 영역별
declaw memory [--only FILE]
declaw mcp    [--only NAME]
declaw skills [--only NAME]

# 실행 제어
declaw --apply          # 실제 적용 (기본은 dry-run)
declaw --dry-run        # 명시적 dry-run
declaw --diff           # 변경 내용 diff 형식 출력

# 충돌 처리
declaw --on-conflict skip      # 기본
declaw --on-conflict overwrite
declaw --on-conflict backup
declaw --on-conflict rename

# 진단
declaw doctor           # 환경 점검
declaw scan             # 상세 스캔 리포트

# 백업/롤백
declaw backup           # 수동 백업 생성
declaw rollback         # 마지막 마이그레이션 취소

# 설정
declaw config get <key>
declaw config set <key> <value>

# 도움말
declaw --help
declaw --version
```

### Dry-run 출력 포맷 (예시)

```
$ npx declaw

╭───────────────────────────────────────────╮
│  declaw — OpenClaw → Claude Code          │
│  v0.1.0 · Phase 1 Migration               │
╰───────────────────────────────────────────╯

🔍 Source:  ~/.openclaw/ (detected)
🎯 Target:  ~/.claude/

📋 Scan Result
   Memory   : 12 files (35 KB total)
   MCP      : 5 servers
   Skills   : 34 items (12 will be skipped)

📝 Plan (dry-run — nothing changed yet)
   
   Memory (→ ~/.claude/projects/<your-home>/memory/)
     + NEW      user_identity.md        (from IDENTITY.md)
     + NEW      reference_agents.md     (from AGENTS.md)
     ⚠ MERGE    user_profile.md         (existing → add OpenClaw content)
     ⚠ CONFLICT project_active_state.md (backup + new)
     ...

   MCP (→ ~/.claude.json)
     + ADD      github (stdio)
     + ADD      context7 (stdio)
     ⏭ SKIP     exa (already registered)
     ⏭ SKIP     openclaw-internal-plugin (not portable)
     ...

   Skills (→ ~/.claude/skills/)
     + NEW      memory-ttl/ (directory)
     ⏭ SKIP     workspace-specific-* (12 items)
     ...

💾 Backup will be created at: ~/.declaw/backups/2026-04-09-0945/

✨ To apply:    declaw --apply
❓ To inspect:  declaw diff
```

## ⚙️ Tech Stack

| 영역 | 선택 | 이유 |
|---|---|---|
| **언어** | TypeScript (Node 20+) | npm 배포, 크로스 플랫폼, Claude Code 사용자는 이미 Node |
| **CLI 프레임워크** | commander.js + prompts | 단순, 의존성 적음 |
| **TUI** | ink (선택) | dry-run 리포트 가독성 |
| **테스트** | vitest | 빠름, ESM |
| **번들** | tsup | 간단 |
| **배포** | npm (`declaw` bare) + `@declaw/cli` | bare 패키지 확보됨 |
| **백업** | `~/.declaw/backups/` (tar.gz) | 롤백 가능 |
| **로깅** | `~/.declaw/logs/` | 디버깅 |

## 🗂 패키지 구조

```
declaw/
├── README.md
├── LICENSE (MIT)
├── package.json
├── tsconfig.json
├── tsup.config.ts
├── src/
│   ├── index.ts           # CLI entry point
│   ├── commands/
│   │   ├── migrate.ts
│   │   ├── memory.ts
│   │   ├── mcp.ts
│   │   ├── skills.ts
│   │   ├── doctor.ts
│   │   ├── scan.ts
│   │   ├── backup.ts
│   │   └── rollback.ts
│   ├── detect/
│   │   ├── openclaw.ts    # ~/.openclaw/ 감지
│   │   └── claude-code.ts # ~/.claude/ 감지
│   ├── migrate/
│   │   ├── memory.ts      # 메모리 변환 로직
│   │   ├── mcp.ts         # MCP 등록 로직
│   │   └── skills.ts      # 스킬 복사 로직
│   ├── utils/
│   │   ├── backup.ts
│   │   ├── diff.ts
│   │   └── frontmatter.ts
│   └── types.ts
├── test/
│   ├── fixtures/          # 가짜 ~/.openclaw/ 구조
│   └── *.test.ts
└── docs/
    ├── USAGE.md
    └── MIGRATION-MAP.md
```

## 📋 14-Day Implementation Plan

### Week 1 — Core Migration

**Day 1 (월): 레포 셋업**
- GitHub 레포 `declaw` 생성
- `declaw.sh` 도메인 구입 (또는 `claw.sh` 대안)
- package.json, tsconfig, tsup 설정
- CI (GitHub Actions: lint + test)
- `declaw --version` 동작

**Day 2 (화): Detection**
- `~/.openclaw/` 감지 (존재 + 구조)
- `workspace/` 심볼릭 resolve
- Scan 결과 JSON으로 출력
- `declaw scan` 명령어

**Day 3 (수): Memory Migration — Read**
- OpenClaw 메모리 파일 파서
- 매핑 규칙 적용 (IDENTITY → user_identity 등)
- Frontmatter 자동 생성
- dry-run 출력 포맷

**Day 4 (목): Memory Migration — Write**
- 실제 파일 쓰기 (--apply)
- 충돌 처리 (skip/overwrite/backup/rename)
- 백업 생성 로직
- MEMORY.md 인덱스 자동 갱신

**Day 5 (금): MCP Migration**
- `mcporter.json` 파싱
- `claude mcp add` 래핑 (subprocess)
- `~/.claude.json` 직접 병합 (fallback)
- 중복 감지, skip/rename

### Week 2 — Skills + UX + Release

**Day 6 (월): Skills Migration**
- 단일 `.md` + 디렉토리 스킬 복사
- Frontmatter 검증
- `--skip-workspace-specific` 옵션
- 호환 검증

**Day 7 (화): CLI Polish**
- commander 명령어 구조 완성
- 에러 메시지
- 진행 표시 (progress bar)
- `--help` 출력

**Day 8 (수): `declaw doctor`**
- 환경 진단 (Node 버전, Claude Code 존재, OpenClaw 존재)
- 권한 체크
- 의존성 체크
- 문제 있을 시 수정 제안

**Day 9 (목): Backup + Rollback**
- 자동 백업 (tar.gz)
- `declaw rollback` — 마지막 마이그레이션 되돌리기
- 백업 보존 정책 (최근 5개)

**Day 10 (금): E2E Test**
- 가짜 `~/.openclaw/` fixture 생성
- 전체 흐름 자동 테스트
- 메인테이너 실제 환경에서 dry-run
- 버그 수정

**Day 11 (월): README + Demo GIF**
- 영문 README 작성
- 5분 설치 가이드
- 데모 GIF (asciinema)
- 스크린샷

**Day 12 (화): 퍼블리시 준비**
- `npm publish` dry-run
- GitHub 릴리즈 노트
- 한국어 README (`README.ko.md`)
- LICENSE (MIT)

**Day 13 (수): 비공개 베타 5명**
- OpenClaw 유저 네트워크에 공유
- 피드백 폼 (Google Form)
- 첫 24h 모니터링

**Day 14 (목): 공개 Launch**
- GitHub public
- npm publish
- 블로그 포스트 (한/영)
- X/Twitter, HN, Reddit r/ClaudeAI

## 🧪 Testing Strategy

### Fixtures
```
test/fixtures/
├── minimal/              # 최소 OpenClaw (IDENTITY + 1 memory)
├── full/                 # 완전 OpenClaw (실사용자 구조 복제)
├── partial/              # 일부만 있는 경우
├── conflicts/            # Claude Code에 기존 파일 있는 경우
└── broken/               # 손상된 OpenClaw (에러 핸들링 테스트)
```

### 커버리지 목표
- Detection: 95%+
- Migration logic: 90%+
- CLI commands: 80%+
- E2E: 핵심 시나리오 3개 (minimal/full/conflict)

### 실환경 검증
- Day 10에 메인테이너 실제 `~/.openclaw/` 대상으로 dry-run
- 파괴적 작업 없이 출력만 확인
- 이슈 발견 시 Day 11 전에 수정

## 🎯 Success Metrics

| 지표 | Day 14 | W4 | W8 |
|---|---|---|---|
| GitHub stars | 10 | 50 | 200 |
| npm weekly downloads | - | 100 | 500 |
| `--apply` 완주율 | - | 70% | 85% |
| 베타 피드백 건수 | 3 | 10 | 30 |
| HN/Reddit 포스트 | 1 | - | 1 (retrospective) |

## ⚠️ Risks & Mitigations

| 리스크 | 확률 | 완화 |
|---|---|---|
| **메인테이너 환경만 동작** (다른 OpenClaw 구성 대응 X) | 중간 | 자동 감지 + 유연한 파서, 베타 5명 다양한 환경 |
| **기존 Claude Code 설정 파괴** | 낮음 | Dry-run 기본, 자동 백업, rollback |
| **OpenClaw 자체 변경으로 호환 깨짐** | 낮음 | 버전 감지 + 경고 |
| **`claude mcp add` CLI 동작 안 함** | 낮음 | Direct JSON 병합 fallback |
| **시간 부족** (14일 못 지킴) | 높음 | MVP Day 1-10만 필수, 11-14는 단순 발행 |
| **OpenClaw 유저가 없음 (TAM 제로)** | 중간 | 메인테이너 네트워크 + 블로그 + Phase 2에서 일반화 |
| **Anthropic 정책 변경** (MCP API 등) | 낮음 | 버전 고정 + CI 호환성 테스트 |

## 🗺 Phase 2 Preview (참고)

Phase 1 완료 후 Phase 2에서 할 것:

- **`declaw sync`**: 주기적 재마이그레이션 (cron) — 양방향 아님, 단방향 리프레시
- **`declaw watch`**: 파일 변경 감지 + 자동 마이그레이션
- **일반화**: OpenClaw 외 다른 "memory + CLI" 시스템 지원 (예: memex, Obsidian 일부, cursor rules)
- **Windows 지원**: WSL이 아닌 native Windows
- **Interactive mode**: 대화형 선택 UI

Phase 3은 장기 Memory Service 비전입니다 (별도 로드맵).

## ❓ Open Questions

1. **도메인**: `declaw.sh` 구매 즉시 진행? (연 ~$60)
2. **GitHub org**: 개인 계정 or 신규 `github.com/declaw` org?
3. **npm publisher**: 개인 계정 or 조직?
4. **라이선스**: MIT (기본 추천) vs Apache 2.0?
5. **첫 공개 타이밍**: Day 14 공개 vs Day 10 조기 베타?
6. **OpenClaw 내부 전용 항목 처리**: 기본 스킵 vs 경고 후 옵션 제공?
7. **Telemetry**: 익명 사용 통계 수집 여부 (`--telemetry=off` 옵션은 필수)

---

## 📝 변경 이력

| 날짜 | 변경 |
|---|---|
| 2026-04-09 | 초안 작성 (v0.1) — 이름 `declaw` 확정, `declaw.sh` 도메인 가용 확인, 14일 플랜 정의 |
