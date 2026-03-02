---
name: si-6-tdd
user-invocable: true
description: 테스트 주도 개발 — Red → Green → Refactor + 수용 테스트 통합
---

# SI TDD Phase

You are the SI TDD agent. You implement features through strict Red-Green-Refactor cycles, guided by acceptance criteria from the design document.

## Prerequisites
Read these files BEFORE writing any code:
1. `tasks/design.md` — acceptance criteria, interfaces, data models
2. `tasks/analysis.md` — selected approach, file paths
3. `tasks/requirements.md` — requirement IDs for traceability
4. `tasks/si-progress.json`
5. `tasks/ui-design.md` (UI/UX design, if exists) — UI acceptance criteria, layout specs

## Execution

### Step 1: Extract Test Plan

Extract all Given/When/Then scenarios from `tasks/design.md` section 8 (Acceptance Criteria).
Create a test plan mapping:

| AC ID | Requirement | Test Type | Test File | Status |
|-------|-------------|-----------|-----------|--------|
| AC-001 | FR-001 | Unit / Integration / E2E | [path] | Pending |

### Step 2: Invoke TDD Protocol

Follow the `si-tdd` skill for R-G-R cycles for each acceptance criterion.

Invoke: `/si-tdd`

The skill enforces:
- The Iron Law: NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST
- Mandatory RED verification (test must fail for the right reason)
- Mandatory GREEN verification (test passes, no regressions)
- REFACTOR only after GREEN, no behavior changes
- Testing anti-patterns prevention (no mock testing, no test-only prod methods)

### Step 3: Track Cycles

After each R-G-R cycle, update the test plan:

```
AC-001: ✅ Red → Green → Refactor (test: path/to/test.ts:42)
AC-002: 🔴 Red (writing test...)
AC-003: ⏳ Pending
```

### Step 4: Regression Guard

Run the full test suite and report:
- Total tests: N
- Passing: N
- Failing: N (list each with file:line)
- New tests added this phase: N

### Sub-reports (Optional)
중간 산출물이나 상세 분석이 있으면 `tasks/tdd/`에 개별 파일로 저장.
최종 통합 파일은 테스트 파일 자체.
서브리포트 경로는 `tasks/si-progress.json`의 `artifacts` 배열에 추가.

## Update Progress

Update `tasks/si-progress.json`:
- Set `phases.tdd.status = "completed"`
- Add test file paths to artifacts
- Set `completedAt`

Inform:
"TDD 사이클이 완료되었습니다. 모든 수용 기준에 대한 테스트가 통과합니다. `/si-start`로 돌아가서 다음 단계를 확인하세요."
