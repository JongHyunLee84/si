# GO / NO-GO Design Review Criteria

## When to Apply
- After Architect phase completes, before TDD/Develop begins
- After any significant design change during development

## Review Structure

### Mandatory Checks (ALL must pass for GO)

1. **Requirements Coverage**
   - Every requirement in PRD has a traceable design element
   - No orphan design elements (everything maps to a requirement)

2. **Interface Contracts**
   - All public APIs have typed signatures
   - Error semantics are consistent (throw vs return vs empty)
   - No implicit dependencies between modules

3. **Data Model Integrity**
   - Schema changes are backward-compatible OR migration path defined
   - No circular dependencies in data flow

4. **Acceptance Criteria Defined**
   - Every requirement has Given/When/Then scenarios
   - No implementation leakage in scenarios (behavior only, no code references)

### Critical Issues Format
Maximum 3 critical issues. Each issue:
```
**[CRIT-N] Title** (5-7 lines max)
- What: [specific problem]
- Impact: [what breaks or degrades]
- Resolution: [concrete action to fix]
```

## Decision Matrix

| Condition | Decision |
|-----------|----------|
| 0 critical issues, all mandatory checks pass | **GO** |
| 1-3 critical issues, all resolvable in <1 day | **CONDITIONAL GO** (fix before TDD) |
| Any mandatory check fails | **NO-GO** (return to Architect) |
| >3 critical issues | **NO-GO** (return to Analysis) |

## No-Effect Zone Documentation
Explicitly list what is NOT affected by this design:
- Modules that remain unchanged
- APIs that keep their current contracts
- Data that is not migrated or modified

This prevents scope creep during implementation.
