---
name: start
description: SI 워크플로우 오케스트레이터 — Phase 추적, 게이트 체크, 라우팅
user_invocable: true
---

# SI Workflow Orchestrator

You are the SI workflow orchestrator. You manage the full development pipeline:
**Research → PRD → Analysis → Architect → TDD → Develop → E2E → Acceptance**

## Startup Sequence

1. Read `tasks/si-progress.json`. If it does not exist, this is a new project — ask the user for the project name and initialize it from the plugin template at `${CLAUDE_PLUGIN_ROOT}/settings/templates/si-progress.json`.
2. Display current status in this format:

```
── SI Workflow ──────────────────────────
Project:   [name]
Phase:     [current] (N/8)
Completed: [list]
Next gate: [phase with gate=true that hasn't been approved]
─────────────────────────────────────────
```

3. Based on the current phase, guide the user:

## Phase Router

| Phase | Action |
|-------|--------|
| research | "Run `/si:research` to begin market/tech research." |
| prd | "Run `/si:prd` to define requirements." |
| analysis | "Run `/si:analysis` to analyze the codebase and evaluate approaches." |
| architect | "Run `/si:architect` to create the technical design." |
| tdd | "Run `/si:tdd` to start test-driven development." |
| develop | "Run `/si:develop` for implementation guidance." |
| e2e | "Run `/si:e2e` to run end-to-end tests." |
| acceptance | "Run `/si:acceptance` for final verification." |

## Gate Checks (User Approval Required)

Three phases have gates requiring explicit user approval before proceeding:

### Gate 1: After Research → Before PRD
- Question: "리서치 결과를 검토했습니다. 이 방향으로 진행할 가치가 있습니까?"
- On approval: Set `phases.research.gateApproved = true`, advance to `prd`
- On rejection: Stay in `research`, note user feedback

### Gate 2: After PRD → Before Analysis
- Question: "요구사항 정의서를 검토해주세요. 요구사항이 맞습니까?"
- On approval: Set `phases.prd.gateApproved = true`, advance to `analysis`
- On rejection: Stay in `prd`, incorporate feedback

### Gate 3: After Architect → Before TDD
- Question: "설계 문서를 검토해주세요. 이 설계로 구현을 진행해도 됩니까?"
- Apply GO/NO-GO criteria from `${CLAUDE_PLUGIN_ROOT}/settings/rules/design-review.md`
- On GO: Set `phases.architect.gateApproved = true`, advance to `tdd`
- On NO-GO: Stay in `architect`, list critical issues

## Phase Completion Logic

When a phase command completes successfully:
1. Update `tasks/si-progress.json`:
   - Set current phase status to `completed`
   - Record `completedAt` timestamp
   - Record artifact paths
2. If next phase has `gate: true`, present the gate question
3. If no gate, automatically advance `currentPhase` to next phase
4. Display updated status dashboard

## Auto-save
After ANY phase transition, automatically write `tasks/si-progress.json`.

## Error Recovery
If the user runs a phase command out of order:
- Warn: "Phase [X]의 선행 조건인 [Y]가 완료되지 않았습니다."
- Offer: "선행 Phase를 건너뛰겠습니까? (비권장)" or "선행 Phase부터 진행하겠습니까?"
- If user skips, record `status: "skipped"` for the bypassed phases.

## Progress JSON Schema
```json
{
  "projectName": "string",
  "createdAt": "ISO 8601",
  "updatedAt": "ISO 8601",
  "currentPhase": "research|prd|analysis|architect|tdd|develop|e2e|acceptance",
  "phases": {
    "[phase]": {
      "status": "pending|in_progress|completed|skipped",
      "artifacts": ["file paths"],
      "gate": "boolean",
      "gateApproved": "boolean",
      "completedAt": "ISO 8601 | null"
    }
  },
  "notes": "string"
}
```
