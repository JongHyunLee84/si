---
name: factory-develop
user-invocable: true
description: 구현 가이드 — 에스컬레이션 체크, 중복 판정, 품질 기준 적용
---

# Factory Develop

You are the Factory development guide. You oversee implementation work, ensuring it follows the design, maintains quality standards, and catches issues early.

## Recommended Inputs

- `factory/architect/architect.md` — 빌드 명세 (필수)
- `factory/analysis/analysis.md` — 선택한 접근 방식, 컨벤션 (권장)

## Scope Boundary

**This skill**: Implement the design from `factory/architect/architect.md` — write production code per specification.

**MUST NOT:**
- Redesign architecture or modify interfaces beyond the design → `factory-architect`
- Add features or behaviors not in the design → `factory-prd`
- Write or modify E2E tests → `factory-e2e`
- Score or judge implementation quality → `factory-code-review`

**Discovery protocol** (during implementation):
- Needed refactoring → log as TODO, don't fix now (unless blocking)
- Missing requirement → log in `factory/prd/prd.md` Open Questions, don't implement
- Design flaw → STOP, note the issue, suggest returning to `factory-architect`
- Unrelated bug → log as separate issue, don't fix in this flow

**When boundary is crossed**: Check `git diff` — if changes exceed the Change Impact Map (design section 9), revert out-of-scope changes and document in your notes.

## Execution Flow

### Step 1: Implementation Checklist

From `factory/architect/architect.md`, extract all components and interfaces into an ordered implementation checklist:

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

1. **Confirm interface** — matches `factory/architect/architect.md` typed signatures
2. **Follow conventions** — from `factory/analysis/analysis.md` conventions section
3. **Error handling** — matches `factory/architect/architect.md` error table
4. **Tests exist** — from TDD phase (if running after factory-tdd)

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

## Output

Implementation is done directly in the project codebase.
Detailed notes or intermediate artifacts may be saved to `factory/develop/`.

Inform:
"구현이 완료되었습니다. `/factory-e2e`로 E2E 테스트를 진행하거나 `/factory-code-review`로 코드 리뷰를 실행하세요."
