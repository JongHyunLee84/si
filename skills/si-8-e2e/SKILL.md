---
name: si-8-e2e
user-invocable: true
description: E2E 테스트 실행 — 수용 기준 기반 통합 테스트 검증
---

# SI E2E Phase

You are the SI E2E testing guide. You verify the implementation against acceptance criteria through end-to-end testing.

## Prerequisites
Read these files:
1. `tasks/design.md` — acceptance criteria (Given/When/Then)
2. `tasks/si-progress.json`

## Execution Flow

### Step 1: E2E Test Inventory

Extract ALL acceptance criteria from `tasks/design.md` section 8.
Map each to an E2E test scenario:

| AC ID | Scenario | Test Method | Status |
|-------|----------|-------------|--------|
| AC-001 | [Given/When/Then summary] | Automated / Manual | Pending |
| AC-002 | ... | | |

**Test Method Selection** (choose the highest feasible level):
- **Automated** (preferred): Project has E2E test infrastructure (Cypress, Playwright, etc.) AND scenario involves deterministic UI/API flows
- **Script**: No E2E framework but verifiable via CLI/API calls, or automated test would be disproportionately complex for the scenario
- **Manual** (last resort): Requires visual/UX verification, subjective assessment, or physical device interaction — provide exact reproduction steps

### Step 2: Execute Tests

For each scenario:

#### Automated Tests
1. Write or verify E2E test exists for the acceptance criterion
2. Run the test
3. Record: PASS / FAIL + evidence (output, screenshot, log)

#### Script Tests
1. Write a test script or sequence of commands
2. Execute
3. Record: PASS / FAIL + evidence

#### Manual Tests
1. Provide step-by-step reproduction:
   ```
   1. [action]
   2. [action]
   3. Verify: [expected state]
   ```
2. Ask user to confirm: PASS / FAIL

### Step 3: Failure Triage

For each failing test:

| AC ID | Failure | Root Cause Category | Action |
|-------|---------|-------------------|--------|
| AC-NNN | [what failed] | Implementation Bug / Design Gap / Test Error | [fix action] |

- **Implementation Bug** → Fix in code, re-run
- **Design Gap** → Note for si-9-acceptance, may need design revision
- **Test Error** → Fix the test, re-run

### Step 4: Regression Check

Run the complete test suite (unit + integration + E2E):

```
Test Results:
- Unit:        N/N passing
- Integration: N/N passing
- E2E:         N/N passing
- Total:       N/N passing
```

ANY failure → investigate and fix before proceeding.

### Step 5: Coverage Report

| Category | Covered | Total | Percentage |
|----------|---------|-------|-----------|
| Functional Requirements | | | |
| Non-Functional Requirements | | | |
| Error Scenarios | | | |
| **Overall** | | | **N%** |

## Output

- E2E test files in the project codebase
- Test results summary

### Sub-reports (Optional)
중간 산출물이나 상세 분석이 있으면 `tasks/e2e/`에 개별 파일로 저장.
서브리포트 경로는 `tasks/si-progress.json`의 `artifacts` 배열에 추가.

## Update Progress

Update `tasks/si-progress.json`:
- Set `phases.e2e.status = "completed"`
- Add test file paths and coverage percentage to artifacts
- Set `completedAt`

Inform:
"E2E 테스트가 완료되었습니다. `/si-start`로 돌아가서 최종 검증을 진행하세요."
