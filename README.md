# SI Workflow Plugin

SI 스타일 소프트웨어 개발 워크플로우 — 멀티 플랫폼 지원 (Claude Code, Codex, Cursor, Copilot).

## Pipeline

```
Research → PRD → Analysis → Architect → UI Design → TDD → Develop → E2E → Acceptance
```

## 설치

### Claude Code (플러그인)

```bash
# 마켓플레이스 추가 (최초 1회)
/plugin marketplace add JongHyunLee84/si

# 설치
/plugin install si@si
```

### Codex / Cursor / Copilot (Agent Skills)

프로젝트 루트에 이 저장소를 클론하거나 서브모듈로 추가:

```bash
git clone https://github.com/JongHyunLee84/si.git
```

`AGENTS.md`와 `skills/` 디렉토리가 에이전트에 자동 인식됩니다.

## 프로젝트 구조

```
si/
├── AGENTS.md                    # Codex/Cursor/Copilot 진입점
├── CLAUDE.md                    # Claude Code 프로젝트 지침
├── .claude-plugin/              # Claude Code 플러그인 매니페스트
│
├── skills/
│   │ # ── Layer 1: 워크플로우 진입점 (번호 = 순서) ──
│   ├── si-start/SKILL.md        # 오케스트레이터 (플랫폼별 Router)
│   ├── si-1-research/SKILL.md   # → si-deep-research 위임
│   ├── si-2-prd/SKILL.md        # → si-prd 위임
│   ├── si-3-analysis/SKILL.md   # 독립 (gap 분석)
│   ├── si-4-architect/SKILL.md  # 독립 (설계 + ADR)
│   ├── si-5-ui-design/SKILL.md  # → si-ui-design 위임
│   ├── si-6-tdd/SKILL.md        # → si-tdd 위임
│   ├── si-7-develop/SKILL.md    # 독립 (구현 가이드)
│   ├── si-8-e2e/SKILL.md        # 독립 (E2E)
│   ├── si-9-acceptance/SKILL.md # → si-code-review 위임
│   ├── si-save/SKILL.md         # 상태 저장
│   │
│   │ # ── Layer 2: 능력 엔진 (워크플로우 스킬이 호출) ──
│   ├── si-deep-research/        # 리서치 프로토콜 (+ references/, evals/)
│   ├── si-prd/                  # 인터뷰 기반 PRD
│   ├── si-ui-design/            # UI/UX 디자인 (+ references/)
│   ├── si-tdd/                  # R-G-R 사이클
│   └── si-code-review/          # 정량 채점
│
├── settings/                    # 범용 (변경 없음)
│   ├── rules/
│   └── templates/
└── hooks/                       # Claude Code 전용 (변경 없음)
```

## 워크플로우 스킬

| Phase | Skill | 호출 |
|-------|-------|------|
| - | si-start | `/si-start` |
| 1 | si-1-research | `/si-1-research` |
| 2 | si-2-prd | `/si-2-prd` |
| 3 | si-3-analysis | `/si-3-analysis` |
| 4 | si-4-architect | `/si-4-architect` |
| 5 | si-5-ui-design | `/si-5-ui-design` |
| 6 | si-6-tdd | `/si-6-tdd` |
| 7 | si-7-develop | `/si-7-develop` |
| 8 | si-8-e2e | `/si-8-e2e` |
| 9 | si-9-acceptance | `/si-9-acceptance` |
| - | si-save | `/si-save` |

## 능력 스킬

| Skill | 호출원 | 역할 |
|-------|--------|------|
| si-deep-research | si-1-research | 체계적 리서치 (폴백 체인, 병렬 수집) |
| si-prd | si-2-prd | 인터뷰 기반 PRD (AskUserQuestion 강제) |
| si-ui-design | si-5-ui-design | Pencil MCP 활용 UI 디자인 + HTML 프로토타입 폴백 |
| si-tdd | si-6-tdd | R-G-R + AC 트래킹 |
| si-code-review | si-9-acceptance | 정량 채점 + severity 분류 |

## 게이트 (사용자 승인 필요)

1. Research 완료 후 — 진행 가치 판단
2. PRD 완료 후 — 요구사항 확인
3. Architect 완료 후 — GO/NO-GO 설계 리뷰
4. UI Design 완료 후 — UI 디자인 승인

## 산출물

각 Phase가 `tasks/` 디렉토리에 산출물을 생성:

**최종 통합 파일:**
- `tasks/si-progress.json` — Phase 추적
- `tasks/research-report.md` — 리서치 결과
- `tasks/requirements.md` — 요구사항 정의서
- `tasks/analysis.md` — 분석 보고서
- `tasks/design.md` — 설계 문서
- `tasks/ui-design.md` — UI 디자인 명세
- `tasks/acceptance-report.md` — 최종 검증 리포트

**서브리포트 (중간 산출물):**
- `tasks/research/` — 토픽별 리서치
- `tasks/prd/` — 인터뷰 노트, 페르소나 상세
- `tasks/analysis/` — 옵션별 상세 분석
- `tasks/architect/` — 기술 리서치 버퍼, ADR 상세
- `tasks/ui-design/` — 스크린별 상세
- `tasks/tdd/` — 테스트 계획 상세
- `tasks/develop/` — 구현 노트
- `tasks/e2e/` — 시나리오별 테스트 결과
- `tasks/acceptance/` — 리뷰 상세

서브디렉토리는 필요할 때만 생성됩니다.

## 차용 패턴

- **cc-sdd**: EARS 요구사항, Gap 분석, GO/NO-GO 리뷰
- **shinpr**: 에스컬레이션 체크, 정량 채점, 금지 어휘
- **ATDD**: Given/When/Then 수용 기준

## License

MIT
