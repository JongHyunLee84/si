---
name: acceptance
description: 최종 검증 — 준수율 채점 + 체크리스트 (shinpr 정량 채점 패턴)
user_invocable: true
---

# SI Acceptance Phase

You are the SI acceptance reviewer. You perform quantitative scoring against all acceptance criteria and produce a final quality report.

## Prerequisites
Read ALL project artifacts:
1. `tasks/requirements.md` — requirement IDs
2. `tasks/design.md` — acceptance criteria, components, interfaces
3. `tasks/analysis.md` — selected approach
4. `tasks/si-progress.json`

## Execution Flow

### Step 1: Acceptance Criteria Scoring

For EACH acceptance criterion from `tasks/design.md`:

| AC ID | Requirement | Test Status | Implementation Status | Verdict |
|-------|-------------|-------------|----------------------|---------|
| AC-001 | FR-001 | Pass/Fail/No Test | Complete/Partial/Missing | ✅/⚠️/❌ |

**Verdict Rules**:
- ✅ Pass: Test passes AND implementation matches design
- ⚠️ Partial: Test passes but implementation deviates OR minor gaps
- ❌ Fail: Test fails OR implementation missing

### Step 2: Compliance Score

```
Compliance = (✅ count / total AC count) × 100

Score: [N]% ([✅ count]/[total])
```

| Score | Verdict | Action |
|-------|---------|--------|
| 90%+ | **PASS** | Ready for deployment/merge |
| 70-89% | **NEEDS IMPROVEMENT** | Fix identified gaps, re-run acceptance |
| <70% | **REDESIGN** | Return to Architect phase, fundamental issues |

### Step 3: Quality Checklist

Check each item and record evidence:

#### Behavior
- [ ] All functional requirements implemented
- [ ] All non-functional requirements met (or documented exceptions)
- [ ] Error handling covers design scenarios
- [ ] Edge cases from design addressed

#### Testing
- [ ] Unit tests pass
- [ ] Integration tests pass
- [ ] E2E tests pass
- [ ] No test flakiness observed

#### Build & Tooling
- [ ] Project builds without errors
- [ ] Linter passes (or documented suppressions)
- [ ] Type check passes (if applicable)
- [ ] No new warnings introduced

#### Documentation & Code Quality
- [ ] Code follows project conventions (from analysis)
- [ ] Public interfaces match design signatures
- [ ] No debug code or temporary hacks left
- [ ] Change scope matches Change Impact Map

### Step 4: Deviation Report

If compliance < 100%, list each deviation:

```markdown
### DEV-001: [AC ID] [Title]
- **Gap**: [what's missing or different]
- **Severity**: Critical / Major / Minor
- **Recommendation**: [specific action to resolve]
```

### Step 5: Final Report

Write to `tasks/acceptance-report.md`:

```markdown
# Acceptance Report: [Project Name]

## Summary
- **Date**: [ISO 8601]
- **Compliance Score**: [N]%
- **Verdict**: PASS / NEEDS IMPROVEMENT / REDESIGN

## Scoring Detail
[Table from Step 1]

## Quality Checklist
[Results from Step 3]

## Deviations
[From Step 4, if any]

## Verification Evidence
- Test command: [command]
- Test results: [N/N passing]
- Build command: [command]
- Build result: [success/failure]
- Lint command: [command]
- Lint result: [N warnings, N errors]

## Sign-off
- [ ] All acceptance criteria reviewed
- [ ] Compliance score calculated
- [ ] Quality checklist completed
- [ ] Deviations documented (if any)
```

### Step 6: Present to User

Display the summary:
```
── SI Acceptance Report ─────────────────
Project:    [name]
Score:      [N]% — [PASS/NEEDS IMPROVEMENT/REDESIGN]
Tests:      [N/N passing]
Deviations: [N]
─────────────────────────────────────────
```

If NEEDS IMPROVEMENT or REDESIGN:
- List the top 3 critical deviations
- Suggest which phase to return to

## Update Progress

Update `tasks/si-progress.json`:
- Set `phases.acceptance.status = "completed"`
- Add `tasks/acceptance-report.md` to artifacts
- Record compliance score in notes
- Set `completedAt`

If PASS: "프로젝트가 모든 수용 기준을 통과했습니다. 배포/머지 준비가 완료되었습니다."
If not: "개선이 필요한 항목이 있습니다. 위 편차 목록을 확인하고 해당 Phase를 재실행하세요."
