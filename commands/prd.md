---
name: si:2-prd
description: 인터뷰 기반 요구사항 정의서(PRD) 작성
user-invocable: true
---

# SI PRD Phase

You are creating a Product Requirements Document through structured interview for the SI workflow.

## Prerequisites
- Read `tasks/research-report.md` if it exists (from Research phase)
- Read `tasks/si-progress.json` for project context

## Execution

### Step 1: Invoke PRD Skill

Call `Skill("si:prd")` to execute the full PRD protocol:
- Phase 0: Initial read and ambiguity classification
- Phase 1: Mandatory interview (AskUserQuestion, batch questions, no assumptions)
- Phase 2: Brief & confirmation
- Phase 3: PRD synthesis (Value Hypothesis + Scoped PRD + EARS conversion)
- Phase 4: User review and iteration

The skill handles the entire interview-to-PRD workflow including:
- interview-then-plan interview discipline (no assumptions, confirm before proceed)
- product-manager PRD structure (Persona, JTBD, Value Hypothesis, Success Metrics)
- EARS format enforcement per `${CLAUDE_PLUGIN_ROOT}/settings/rules/ears-format.md`

### Step 2: Verify Output

After the skill completes, verify:
- `tasks/requirements.md` exists
- All requirements use EARS patterns (no free-form text)
- Value Hypothesis is present and falsifiable
- Non-Goals section exists and is non-empty
- Success Metrics have measurable criteria

### Step 3: User Review Gate

Present the requirements summary to the user:
- List all FR/NFR with IDs and priorities
- Highlight any assumptions made
- Ask: "요구사항을 검토해주세요. 추가, 수정, 삭제할 항목이 있습니까?"

Iterate until the user approves.

### Step 4: Update Progress

Update `tasks/si-progress.json`:
- Set `phases.prd.status = "completed"`
- Add `tasks/requirements.md` to artifacts
- Set `completedAt` to current timestamp

Inform: "요구사항 정의가 완료되었습니다. `/si:start`로 돌아가서 게이트 승인을 진행하세요."
