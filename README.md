# declaw

> **OpenClaw에서 Claude Code로 30초 만에 이동한다.**

`declaw`는 OpenClaw의 메모리·MCP 서버·Skills를 Claude Code로 이식해주는 **단방향 마이그레이션 CLI**입니다. 한 번 실행하면 끝. `oh-my-zsh` 스타일로 `npx` 한 줄 설치.

```bash
npx declaw
```

---

**상태**: Phase 1 스펙 확정 (2026-04-09)
**도메인**: `declaw.sh` (가용성 확인됨)
**npm**: `declaw` (bare) + `@declaw/cli` (scope)

## 🎯 핵심 가치 한 줄

> **"OpenClaw 유저가 Claude Code로 갈아타는 순간의 모든 마찰을 제거한다."**

## 💡 해결하는 문제

OpenClaw에서 쌓은 메모리·IDENTITY·skills·MCP 설정을 Claude Code로 이식하는 **표준 방법이 없음**. 수동 복사는:
- ❌ 파일 매핑 규칙 모름 (IDENTITY.md를 어디에 둬야 하나?)
- ❌ Claude Code의 frontmatter 스키마 맞춰야 함
- ❌ MCP 설정은 `~/.claude.json`에 `claude mcp add`로 등록 필요
- ❌ Skills 디렉토리 구조 다름
- ❌ 실수하면 기존 Claude Code 설정 깨짐

→ `declaw` 한 줄이면 **자동 감지 + 매핑 + 백업 + 적용**.

## 🚀 5분 설치 (예상 UX)

```bash
# 1. 실행 (dry-run 기본)
$ npx declaw

╭───────────────────────────────────────────╮
│  declaw — OpenClaw → Claude Code          │
│  v0.1.0 · Phase 1 Migration               │
╰───────────────────────────────────────────╯

🔍 Source:  ~/.openclaw/ (detected)
🎯 Target:  ~/.claude/

📋 Scan Result
   Memory   : 12 files
   MCP      : 5 servers
   Skills   : 34 items (12 will be skipped — workspace-specific)

📝 Plan (dry-run — nothing changed yet)
   + 10 new memory files
   + 3 new MCP servers (2 skipped — already registered)
   + 22 new skills

💾 Backup will be created at: ~/.declaw/backups/2026-04-09-0945/

✨ To apply:    declaw --apply
❓ To inspect:  declaw diff

# 2. 실제 적용
$ npx declaw --apply
✅ Migration complete in 2.3s
   10 memories · 3 MCPs · 22 skills
   Backup: ~/.declaw/backups/2026-04-09-0945/
```

## 📦 What It Does (공식 Claude Code 계층 기준)

`declaw`는 OpenClaw 파일을 **Claude Code 공식 문서에 맞는 올바른 위치**로 배치합니다. Auto memory는 Claude가 자동 작성하는 공간이므로, 사용자가 편집한 파일들은 `CLAUDE.md` / `rules` / `hooks`로 분산됩니다.

### 5가지 Destination 카테고리

| 카테고리 | Claude Code 위치 | 예시 |
|---|---|---|
| **user-claudemd** | `~/.claude/CLAUDE.md` | `IDENTITY.md`, `USER.md`, `SOUL.md` 통합 |
| **project-claudemd** | `./CLAUDE.md` (cwd 기준) | `AGENTS.md` |
| **user-rules** | `~/.claude/rules/*.md` | `TOOLS.md` (path-scoped 가능) |
| **user-hook** | `~/.claude/hooks/*.sh` + `settings.json` | `HEARTBEAT.md` → hook 변환 |
| **auto-memory** | `~/.claude/projects/<proj>/memory/` | `MEMORY.md` (병합) + 일일/주제 메모리 |

### 3영역 스코프

| 영역 | Source | Target |
|---|---|---|
| **Memory & Instructions** | `~/.openclaw/workspace/*.md` + `memory/` | `CLAUDE.md` / `rules` / `hooks` / `auto-memory` (5 destinations) |
| **MCP** | `~/.openclaw/workspace/config/mcporter.json` | `claude mcp add --scope user` (공식 CLI) |
| **Skills** | `~/.openclaw/workspace/skills/*` | `~/.claude/skills/*` |

각 영역마다:
- ✅ 자동 감지 (하드코딩 없음)
- ✅ 공식 frontmatter 스펙 검증 (Skills)
- ✅ 충돌 감지 + skip/append/merge/convert/rename 선택
- ✅ 백업 + rollback 지원
- ✅ Dry-run 기본 (실제 적용은 `--apply`)

## 🎭 Who It's For

### ✅ 타겟 유저
- OpenClaw 또는 유사 `.md` 기반 메모리 시스템 사용자
- Claude Code 공식 기능(Auto Memory, Channels)을 활용하고 싶은 사람
- CLI에 익숙한 개발자
- macOS / Linux / WSL

### ❌ Non-targets (지금은)
- OpenClaw 써본 적 없는 사람 (Phase 2)
- Windows native (Phase 2)
- 양방향 실시간 동기화 필요 (Phase 2+)
- SaaS 기능 (Phase 3+)

## 📐 설계 원칙

1. **단방향 단발성** — 한 번 돌리면 끝. 실시간 동기화는 나중.
2. **Dry-run 기본** — `--apply` 없으면 절대 파일 변경 안 함.
3. **자동 백업** — 적용 전 항상 백업. `declaw rollback`로 복원 가능.
4. **감지 우선** — 경로·파일명 하드코딩 금지. 자동 탐지.
5. **Claude Code 공식 우선** — 공식 스키마/경로 준수.
6. **최소 의존성** — TypeScript + commander + prompts만. 작게.
7. **0 네트워크** — 로컬만. 텔레메트리는 옵트인.

## 🛠 Tech Stack

| 영역 | 선택 |
|---|---|
| 언어 | TypeScript (Node 20+) |
| CLI | commander.js + prompts |
| TUI | ink (선택) |
| 테스트 | vitest |
| 번들 | tsup |
| 배포 | npm (`declaw` + `@declaw/cli`) |
| 라이선스 | MIT |

## 📂 문서

| 문서 | 내용 |
|---|---|
| [docs/PHASE-1-MIGRATION.md](docs/PHASE-1-MIGRATION.md) | **⭐ 주 문서** — Phase 1 마이그레이션 CLI 상세 스펙, 14일 플랜 |

## 🗺 Roadmap (간략)

```
Phase 1 (W1-W2)   ← 지금 여기
  declaw — OpenClaw → Claude Code 단방향 마이그레이션
   ↓
Phase 1.5 (W3-W4)
  declaw doctor 확장, 베타 피드백 반영
   ↓
Phase 2 (W5-W12)
  declaw sync — 주기적 재마이그레이션
  일반화 (다른 memory 시스템 지원)
  Windows 지원
   ↓
Phase 3 (W13+)
  Memory & Harness Service — 장기 비전
```

## 📊 왜 `declaw`라는 이름?

- **declaw** = 고양이 발톱 빼기 → "OpenClaw에서 발톱을 뺀다" 메타포
- 바이럴 잠재력 (한 번 들으면 "잠깐 뭐?" 반응)
- 짧음 (6자), 발음 쉬움, 기억 쉬움
- 상표 안전 (일반명사)
- `.sh` TLD와 완벽한 조합 (`declaw.sh` = CLI 느낌)

## 🎯 다음 액션 (Day 0)

1. [ ] `declaw.sh` 도메인 구매 (~$60/년)
2. [ ] `github.com/declaw` org 생성 (또는 개인 계정)
3. [ ] npm 계정 publisher 설정
4. [ ] GitHub 레포 `declaw` 생성
5. [ ] Day 1 시작: 스켈레톤 + CI

## 🙋 FAQ (예상)

**Q. 기존 Claude Code 설정이 있는데 덮어쓰지 않나요?**
A. 기본은 dry-run. `--apply` 해도 충돌 시 기본 동작은 **skip**. 덮어쓰려면 명시적 `--on-conflict overwrite`.

**Q. 실수로 잘못 실행했어요.**
A. `declaw rollback`으로 마지막 마이그레이션 되돌리기.

**Q. 제 OpenClaw 구조가 표준과 다를 수 있어요.**
A. 자동 감지합니다. 만약 감지 못 하면 `declaw scan`으로 리포트 확인 후 이슈 등록해주세요. 다양한 환경에서 동작하도록 설계됐습니다.

**Q. OpenClaw를 계속 쓰고 싶은데 Claude Code도 쓰고 싶어요.**
A. 단방향 마이그레이션이지만, 파괴적이지 않습니다. OpenClaw 원본은 그대로 남고, 복사본만 Claude Code에 생성됩니다. 양방향 동기화는 Phase 2.

**Q. Telegram/Discord 연동은?**
A. OpenClaw Telegram bridge는 마이그레이션 범위 밖. Claude Code는 공식 **Channels**(v2.1.80+)로 같은 기능 제공.

## 🧑‍💻 Contributing

Day 14 공개 후 이슈/PR 환영. 그전에는 비공개 베타 (5명).

## 📄 License

MIT

---

_"Memory that ages. Identity that persists. Migrations that don't suck."_
