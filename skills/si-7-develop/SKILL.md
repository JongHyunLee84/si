---
name: si-7-develop
user-invocable: true
description: 구현 가이드 — 에스컬레이션 체크, 중복 판정, 품질 기준 적용
---

# SI Develop Phase

You are the SI development guide. You oversee implementation work, ensuring it follows the design, maintains quality standards, and catches issues early.

## Prerequisites
Read these files:
1. `tasks/design.md` — the source of truth for what to build
2. `tasks/analysis.md` — selected approach, conventions
3. `tasks/si-progress.json`

## Execution Flow

### Step 1: Implementation Checklist

From `tasks/design.md`, extract all components and interfaces into an ordered implementation checklist:

```markdown
## Implementation Checklist
- [ ] [Component 1]: [interface/responsibility]
  - Dependencies: [what must exist first]
  - Files: [target paths]
- [ ] [Component 2]: ...
```

Order by dependency — implement leaf dependencies first.

### Step 2: Pre-Implementation Checks (Per Component)

Before writing each component, verify:

#### Escalation Check (shinpr pattern)
ANY "Yes" → STOP and confirm with user:
- [ ] Does this change an architecture layer boundary?
- [ ] Does this add an external dependency not in the design?
- [ ] Does this modify a data model beyond what's designed?
- [ ] Does this affect security or authentication flow?

#### Duplication Check (shinpr 5-criteria)
Score the new component against existing code:

| Criterion | Score (0-2) | Evidence |
|-----------|------------|----------|
| Same input/output signature | | [file:line] |
| Same business rule | | [file:line] |
| Same data access pattern | | [file:line] |
| Same error handling | | [file:line] |
| Same UI pattern | | [file:line] |
| **Total** | **/10** | |

- 0-3: Proceed — genuinely new code
- 4-6: Extract shared abstraction first, then implement
- 7-10: Reuse existing code — extend, don't duplicate

### Step 3: Implementation Guidance

For each component:

1. **Confirm interface** — matches `tasks/design.md` typed signatures
2. **Follow conventions** — from `tasks/analysis.md` conventions section
3. **Error handling** — matches `tasks/design.md` error table
4. **Tests exist** — from TDD phase (if running after si-6-tdd)

### Step 4: Quality Checks (Continuous)

After implementing each component:

1. **Lint/Format**: Run project linter
2. **Type Check**: Run type checker (if applicable)
3. **Tests**: Run related tests
4. **Build**: Verify compilation (if applicable)

If any check fails → fix before moving to next component.

### Step 5: Integration Verification

After all components are implemented:

1. Run full test suite
2. Check for any new warnings or deprecation notices
3. Verify no unintended changes (git diff review)
4. Confirm Change Impact Map from design — did changes stay within expected scope?

If changes exceeded expected scope → document why in `tasks/si-progress.json` notes.

## Scope Creep Prevention

If during implementation you discover:
- A needed refactoring → log as TODO, don't fix now (unless blocking)
- A missing requirement → log in `tasks/requirements.md` Open Questions
- A design flaw → STOP, note the issue, suggest returning to si-4-architect
- An unrelated bug → log as separate issue, don't fix in this flow

## Output

Implementation is done directly in the project codebase.
Record what was built in `tasks/si-progress.json` notes.

### Sub-reports (Optional)
중간 산출물이나 상세 분석이 있으면 `tasks/develop/`에 개별 파일로 저장.
서브리포트 경로는 `tasks/si-progress.json`의 `artifacts` 배열에 추가.

## Update Progress

Update `tasks/si-progress.json`:
- Set `phases.develop.status = "completed"`
- Add implementation file paths to artifacts
- Set `completedAt`

Inform:
"구현이 완료되었습니다. `/si-start`로 돌아가서 E2E 테스트를 진행하세요."
