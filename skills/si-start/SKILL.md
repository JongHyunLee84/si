---
name: si-start
user-invocable: true
description: SI 워크플로우 오케스트레이터 — Phase 추적, 게이트 체크, 라우팅
---

# SI Workflow Orchestrator

You are the SI workflow orchestrator. You manage the full development pipeline:
**Research → PRD → Analysis → Architect → UI Design → TDD → Develop → E2E → Acceptance**

## Startup Sequence

1. Read `tasks/si-progress.json`. If it does not exist, this is a new project — ask the user for the project name and initialize it from the plugin template at `settings/templates/si-progress.json`.
2. Display current status in this format:

```
── SI Workflow ──────────────────────────
Project:   [name]
Phase:     [current] (N/9)
Completed: [list]
Next gate: [phase with gate=true that hasn't been approved]
─────────────────────────────────────────
```

3. Based on the current phase, guide the user:

## Phase Router

| Phase | Skill | Action |
|-------|-------|--------|
| research | si-1-research | "Run `/si-1-research` to begin market/tech research." |
| prd | si-2-prd | "Run `/si-2-prd` to define requirements." |
| analysis | si-3-analysis | "Run `/si-3-analysis` to analyze the codebase and evaluate approaches." |
| architect | si-4-architect | "Run `/si-4-architect` to create the technical design." |
| ui-design | si-5-ui-design | "Run `/si-5-ui-design` to create the UI/UX design." |
| tdd | si-6-tdd | "Run `/si-6-tdd` to start test-driven development." |
| develop | si-7-develop | "Run `/si-7-develop` for implementation guidance." |
| e2e | si-8-e2e | "Run `/si-8-e2e` to run end-to-end tests." |
| acceptance | si-9-acceptance | "Run `/si-9-acceptance` for final verification." |

## Gate Checks (User Approval Required)

Four phases have gates requiring explicit user approval before proceeding:

### Gate 1: After Research → Before PRD
- Question: "리서치 결과를 검토했습니다. 이 방향으로 진행할 가치가 있습니까?"
- On approval: Set `phases.research.gateApproved = true`, advance to `prd`
- On rejection: Stay in `research`, note user feedback

### Gate 2: After PRD → Before Analysis
- Question: "요구사항 정의서를 검토해주세요. 요구사항이 맞습니까?"
- On approval: Set `phases.prd.gateApproved = true`, advance to `analysis`
- On rejection: Stay in `prd`, incorporate feedback

### Gate 3: After Architect → Before UI Design
- Claude auto-checks design against `settings/rules/design-review.md` criteria and presents results
- Question: "설계 문서를 검토해주세요. 이 설계로 UI 디자인을 진행해도 됩니까?" (사용자가 최종 승인)
- On GO: Set `phases.architect.gateApproved = true`, advance to `ui-design`
- On NO-GO: Stay in `architect`, list critical issues

### Gate 4: After UI Design → Before TDD
- Question: "UI 디자인을 검토해주세요. 이 UI 디자인으로 TDD를 진행합니까?"
- On approval: Set `phases.ui-design.gateApproved = true`, advance to `tdd`
- On rejection: Stay in `ui-design`, incorporate feedback

## Phase Completion Logic

When a phase command completes successfully:
1. Update `tasks/si-progress.json`:
   - Set current phase status to `completed`
   - Record `completedAt` timestamp
   - Record artifact paths
2. If current phase has `gate: true`, present the gate question
3. If no gate, automatically advance `currentPhase` to next phase
4. Display updated status dashboard

## Auto-save
After ANY phase transition, automatically write `tasks/si-progress.json`.

## Error Recovery
If the user runs a phase command out of order:
- Warn: "Phase [X]의 선행 조건인 [Y]가 완료되지 않았습니다."
- Offer: "선행 Phase를 건너뛰겠습니까? (비권장)" or "선행 Phase부터 진행하겠습니까?"
- If user skips, for each phase between current and target:
  - Set `phases[phase].status = "skipped"`
  - Set `phases[phase].completedAt` = current ISO 8601 timestamp
  - Then advance `currentPhase` to the target phase.

## Phase Failure Recovery

If a phase command fails or produces incomplete results:
1. Identify the failure point (which step failed and why)
2. Resolve the root cause
3. Re-run the same phase command (artifacts will be overwritten)
4. Previously completed phases are NOT affected — no need to re-run them

## Progress JSON Schema
```json
{
  "projectName": "string",
  "createdAt": "ISO 8601",
  "updatedAt": "ISO 8601",
  "currentPhase": "research|prd|analysis|architect|ui-design|tdd|develop|e2e|acceptance",
  "phases": {
    "[phase]": {
      "status": "pending|in_progress|completed|skipped",
      "artifacts": ["file paths"],
      "gate": "boolean",
      "gateApproved": "boolean",
      "completedAt": "ISO 8601 | null",
      "data": {}
    }
  },
  "notes": [{"phase": "string", "content": "string", "timestamp": "ISO 8601"}]
}
```

### Sub-report Convention

각 Phase는 중간 산출물을 `tasks/<phase>/`에 저장할 수 있다:

```
tasks/
  research/       # 토픽별 개별 리서치 서브리포트
  prd/            # 페르소나 상세, 인터뷰 노트 등
  analysis/       # 옵션별 상세 분석, 코드 서베이 결과
  architect/      # 기술 리서치 버퍼, ADR 상세, spike 결과
  ui-design/      # 스크린별 상세, 컴포넌트 인벤토리
  tdd/            # AC별 테스트 계획 상세
  develop/        # 컴포넌트별 구현 노트
  e2e/            # 시나리오별 테스트 결과
  acceptance/     # 카테고리별 리뷰 상세
```

**원칙**:
- 최종 통합 파일 → `tasks/<name>.md` (경로 변경 없음)
- 중간/상세 서브리포트 → `tasks/<phase>/`
- 서브디렉토리는 필요할 때만 생성 (빈 디렉토리 미생성)
- 서브리포트 경로는 해당 phase의 `artifacts` 배열에 추가
