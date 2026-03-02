# SI Workflow

9-stage software development workflow:
Research → PRD → Analysis → Architect → UI Design → TDD → Develop → E2E → Acceptance

## Quick Start
Run `/si-start` to begin or resume.

## Workflow Skills (Phase 순서)

| # | Skill | Description |
|---|-------|-------------|
| - | si-start | 오케스트레이터: Phase 추적, 게이트, 라우팅 |
| 1 | si-1-research | 시장/기술 리서치 |
| 2 | si-2-prd | 인터뷰 기반 요구사항 정의 |
| 3 | si-3-analysis | 코드/시스템 분석, 갭 분석 |
| 4 | si-4-architect | 기술 설계, ADR, 수용 기준 |
| 5 | si-5-ui-design | UI/UX 디자인 |
| 6 | si-6-tdd | TDD (Red-Green-Refactor) |
| 7 | si-7-develop | 구현 가이드 |
| 8 | si-8-e2e | E2E 테스트 |
| 9 | si-9-acceptance | 준수율 채점 + 최종 검증 |
| - | si-save | 진행 상태 저장 |

## Capability Skills (워크플로우 스킬이 호출)

| Skill | Called By | Description |
|-------|-----------|-------------|
| si-deep-research | si-1-research | 체계적 리서치 (폴백 체인, 병렬 수집) |
| si-prd | si-2-prd | 인터뷰 기반 PRD (AskUserQuestion 강제) |
| si-ui-design | si-5-ui-design | Pencil MCP 활용 UI 디자인 + HTML 프로토타입 폴백 |
| si-tdd | si-6-tdd | R-G-R + AC 트래킹 |
| si-code-review | si-9-acceptance | 정량 채점 + severity 분류 |

## Gates (사용자 승인 필요)

1. Research 완료 후 — 진행 가치 판단
2. PRD 완료 후 — 요구사항 확인
3. Architect 완료 후 — GO/NO-GO 설계 리뷰
4. UI Design 완료 후 — UI 디자인 승인

## Artifacts

All outputs → `tasks/` directory. State → `tasks/si-progress.json`.

### Sub-reports
각 Phase의 중간 산출물은 `tasks/<phase>/`에 개별 파일로 저장:

```
tasks/
  research/       # 토픽별 리서치
  prd/            # 인터뷰 노트
  analysis/       # 상세 분석
  architect/      # 기술 리서치 버퍼, spike
  ui-design/      # 스크린별 상세
  tdd/            # 테스트 계획 상세
  develop/        # 구현 노트
  e2e/            # 시나리오별 결과
  acceptance/     # 리뷰 상세
```

최종 통합 파일은 `tasks/<name>.md`에 유지 (하위 호환).

## Settings
- `settings/rules/` — EARS format, gap analysis framework, design review checklist
- `settings/templates/` — analysis, design, progress JSON templates
