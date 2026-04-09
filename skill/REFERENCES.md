# Official References

`declaw`는 **Claude Code 공식 문서**와 **OpenClaw 공식 문서**를 기준으로 동작합니다. 스킬 실행 시 이 문서들이 소스 오브 트루스입니다. 매핑 규칙이 현재 공식과 일치하는지 의심되면 **WebFetch로 최신 문서를 확인**하세요.

---

## 🤖 Claude Code 공식 문서

**Base URL**: `https://code.claude.com/docs/en/`

| 주제 | URL | 왜 필요한가 |
|---|---|---|
| Memory System | https://code.claude.com/docs/en/memory.md | CLAUDE.md 계층, auto memory, rules 디렉토리의 공식 구조 |
| Settings | https://code.claude.com/docs/en/settings.md | `~/.claude/settings.json` 스키마 (hooks, permissions, enabledPlugins) |
| Skills | https://code.claude.com/docs/en/skills.md | Skill frontmatter 공식 필드 + 디렉토리 구조 |
| MCP | https://code.claude.com/docs/en/mcp.md | `claude mcp add` scope, 저장 위치 |
| Hooks | https://code.claude.com/docs/en/hooks.md | Hook 이벤트 종류, 입력 형식, 실행 순서 |
| Sub-agents | https://code.claude.com/docs/en/sub-agents.md | 서브에이전트 정의 및 호출 |

**핵심 팩트 (2026-04 기준, 확인 완료)**:
- Auto memory 경로: `~/.claude/projects/<project>/memory/MEMORY.md` (인덱스, 첫 200줄/25KB 로드)
- `.claude/rules/*.md`는 path-scoped 조건부 로드 지원 (`paths:` frontmatter)
- `~/.claude/CLAUDE.md`는 모든 프로젝트에 user instructions로 주입됨
- MCP 공식 저장: `claude mcp add --scope user` 명령 (내부적으로 `~/.claude.json` 또는 `~/.claude/settings.json` — CLI 사용 권장)
- Skills 공식 frontmatter: `name`, `description`, `allowed-tools`, `paths`, `disable-model-invocation`, `user-invocable`, `agent`, `context`, `argument-hint`, `model`, `effort`, `hooks`, `shell`

---

## 🦞 OpenClaw 공식 문서

**Homepage**: https://openclaw.ai
**Docs**: https://docs.openclaw.ai
**GitHub**: https://github.com/openclaw/openclaw
**DeepWiki**: https://deepwiki.com/openclaw/openclaw
**Discord**: https://discord.gg/clawd

| 주제 | URL |
|---|---|
| Getting Started | https://docs.openclaw.ai/start/getting-started |
| Installation | https://docs.openclaw.ai/install/ |
| Concepts | https://docs.openclaw.ai/concepts/ |
| Workspace Files | https://docs.openclaw.ai/concepts/workspace |
| Channels | https://docs.openclaw.ai/channels/ |
| Plugins | https://docs.openclaw.ai/plugins/ |
| Skills | https://docs.openclaw.ai/skills/ |
| Reference | https://docs.openclaw.ai/reference/ |

**로컬 복사본** (대표적 OpenClaw 설치 후): `~/.openclaw/workspace/openclaw-repo/docs/`
- 구조: `assets/`, `automation/`, `channels/`, `cli/`, `concepts/`, `install/`, `plugins/`, `providers/`, `reference/`, `nodes/`
- 특정 파일이 로컬에 있으면 WebFetch 대신 Read로 빠르게 확인 가능

**OpenClaw 워크스페이스 표준 파일들** (declaw가 관심 있는 것):
| 파일 | OpenClaw 내 역할 |
|---|---|
| `IDENTITY.md` | 에이전트 정체성 (이름, 페르소나) |
| `USER.md` | 사용자 프로필 (호칭, 선호, 톤) |
| `SOUL.md` | 미션/가치 |
| `AGENTS.md` | 에이전트 역할 분담 및 위임 규칙 |
| `TOOLS.md` | 도구 사용 가이드 (로컬 환경, ACP, Codex 등) |
| `HEARTBEAT.md` | 정기 체크 / self-review 규칙 |
| `MEMORY.md` | 장기 메모리 + TTL 등급 (`[stable]`, `[active]`, `[volatile]`) |
| `memory/` | 일일 로그 + 주제별 |
| `skills/` | 사용자 스킬 (OpenClaw 규격) |
| `config/mcporter.json` | MCP 서버 정의 |

---

## 🔄 문서 업데이트 처리 정책

Claude Code나 OpenClaw 공식 문서가 변경되면 매핑 규칙도 재검토 필요. Skill 실행 시 다음 지시를 따르세요:

1. **매번 모든 문서를 fetch하지는 마세요** (성능 문제)
2. **사용자가 "최신 spec으로 확인해" 또는 "verify against docs"라고 하면** WebFetch로 가져옴
3. **MAPPING.md의 버전이 오래됐다면** (`@verified: 2026-04-09`보다 3개월 이상 전) 사용자에게 알림
4. **공식 문서에서 매핑 규칙과 충돌하는 사항을 발견하면**:
   - 즉시 실행 중단
   - 사용자에게 차이점 설명
   - MAPPING.md 업데이트 제안
   - 사용자 승인 후 진행

---

## 📋 MAPPING.md 버전 관리

현재: `v0.2` (2026-04-09 기준)

공식 문서 기준 검증 이력:
- v0.2 (2026-04-09): Claude Code `memory.md`, `skills.md`, `mcp.md` 기반 전면 재작성. Auto memory는 Claude 자동 작성용이므로 사용자 편집 파일은 CLAUDE.md/rules/hooks로 분산.
- v0.1 (2026-04-09): 초안. 모든 OpenClaw 파일을 auto memory로 오분류 (즉시 수정됨).

다음 검증 예정: 공식 문서 변경 감지 시 또는 3개월 후 (2026-07)
