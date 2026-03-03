# Factory — Software Development Toolbox

독립적으로 호출 가능한 소프트웨어 개발 도구 상자 — 멀티 플랫폼 지원 (Claude Code, Codex, Cursor, Copilot).

## 컨셉

파이프라인이 아닌 **도구 상자**. 9개 스킬을 순서 없이 필요할 때 호출. 각 스킬은 `factory/<skill>/`에 산출물을 남기고, 다른 스킬은 이를 참고할 수 있지만 강제하지 않는다.

## 설치

### Claude Code (플러그인)

```bash
# 마켓플레이스 추가 (최초 1회)
/plugin marketplace add JongHyunLee84/si

# 설치
/plugin install factory@factory
```

### Codex / Cursor / Copilot (Agent Skills)

프로젝트 루트에 이 저장소를 클론하거나 서브모듈로 추가:

```bash
git clone https://github.com/JongHyunLee84/si.git
```

`AGENTS.md`와 `skills/` 디렉토리가 에이전트에 자동 인식됩니다.

## 프로젝트 구조

```
factory-plugin/
├── AGENTS.md                       # Codex/Cursor/Copilot 진입점
├── CLAUDE.md                       # Claude Code 프로젝트 지침
├── .claude-plugin/                 # Claude Code 플러그인 매니페스트
│
├── skills/
│   ├── factory-research/SKILL.md   # 시장/기술 리서치 (+ references/, evals/)
│   ├── factory-prd/SKILL.md        # 인터뷰 기반 요구사항 정의
│   ├── factory-analysis/SKILL.md   # 코드/시스템 분석
│   ├── factory-architect/SKILL.md  # 기술 설계 + GO/NO-GO
│   ├── factory-ui-design/SKILL.md  # UI/UX 디자인 (+ references/)
│   ├── factory-tdd/SKILL.md        # TDD (R-G-R)
│   ├── factory-develop/SKILL.md    # 구현 가이드
│   ├── factory-e2e/SKILL.md        # E2E 테스트
│   └── factory-code-review/SKILL.md # 코드 리뷰 + 수용 검증
│
├── settings/
│   ├── rules/                      # EARS format, gap analysis, design review
│   └── templates/                  # factory-analysis, factory-architect 템플릿
└── hooks/                          # Claude Code 전용 (세션 시작 시 아티팩트 스캔)
```

## 스킬

| Skill | 호출 | Description |
|-------|------|-------------|
| factory-research | `/factory-research` | 시장/기술 리서치 (단일/멀티 토픽 자동 선택) |
| factory-prd | `/factory-prd` | 인터뷰 기반 요구사항 정의 (EARS 강제) |
| factory-analysis | `/factory-analysis` | 코드/시스템 분석, 갭 분석, 3옵션 평가 |
| factory-architect | `/factory-architect` | 기술 설계, ADR, 수용 기준, GO/NO-GO 자체 검증 |
| factory-ui-design | `/factory-ui-design` | UI/UX 디자인 (Pencil MCP / HTML 프로토타입 폴백) |
| factory-tdd | `/factory-tdd` | TDD — Red → Green → Refactor + AC 트래킹 |
| factory-develop | `/factory-develop` | 구현 가이드 — 에스컬레이션, 중복 판정, 품질 기준 |
| factory-e2e | `/factory-e2e` | E2E 테스트 — 수용 기준 기반 통합 검증 |
| factory-code-review | `/factory-code-review` | 정량 채점 + 90/70/fail 판정 + severity 분류 |

## 추천 흐름 (강제 아님)

```
research → prd → analysis → architect → ui-design → tdd → develop → e2e → code-review
```

각 스킬은 독립적이지만, 위 순서로 실행하면 이전 스킬의 산출물을 자연스럽게 참고합니다.

## 산출물

모든 산출물은 `factory/` 디렉토리에 저장:

| 스킬 | 산출물 |
|------|--------|
| factory-research | `factory/research/research.md`, `factory/research/topic-*.md` |
| factory-prd | `factory/prd/prd.md` |
| factory-analysis | `factory/analysis/analysis.md` |
| factory-architect | `factory/architect/architect.md`, `factory/architect/research.md` |
| factory-ui-design | `factory/ui-design/ui-design.md` |
| factory-tdd | `factory/tdd/` (테스트 플랜, 사이클 기록) |
| factory-develop | `factory/develop/` (구현 노트) |
| factory-e2e | `factory/e2e/` (E2E 리포트) |
| factory-code-review | `factory/code-review/code-review.md` |

## 차용 패턴

- **cc-sdd**: EARS 요구사항, Gap 분석, GO/NO-GO 리뷰
- **shinpr**: 에스컬레이션 체크, 정량 채점, 금지 어휘
- **ATDD**: Given/When/Then 수용 기준

## License

MIT
