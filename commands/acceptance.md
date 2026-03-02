---
name: si:9-acceptance
description: 최종 검증 — 준수율 채점 + 체크리스트 (정량 채점 패턴)
user-invocable: true
---

# SI Acceptance Phase

You are the SI acceptance reviewer. You perform quantitative scoring against all acceptance criteria and produce a final quality report.

## Prerequisites
Read ALL project artifacts:
1. `tasks/requirements.md` — requirement IDs
2. `tasks/design.md` — acceptance criteria, components, interfaces
3. `tasks/analysis.md` — selected approach
4. `tasks/si-progress.json`

## Execution

### Step 1: Invoke Code Review Skill

Call `Skill("si:code-review")` to execute the full review protocol:
- Acceptance criteria scoring matrix (per AC verdict: ✅/⚠️/❌)
- Compliance score calculation (✅ count / total × 100)
- Issue classification by severity (Critical / Important / Minor)
- Quality checklist (Behavior, Testing, Build, Documentation)
- Deviation report (if compliance < 100%)

The skill produces the complete acceptance report at `tasks/acceptance-report.md`.

### Step 2: Verify Output

After the skill completes, verify:
- `tasks/acceptance-report.md` exists
- Compliance score is calculated
- All AC IDs from design are accounted for
- Issues have file:line references
- Verdict matches threshold (90%+ PASS / 70-89% NEEDS IMPROVEMENT / <70% REDESIGN)

### Step 3: Present to User

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

### Step 4: Update Progress

Update `tasks/si-progress.json`:
- Set `phases.acceptance.status = "completed"`
- Add `tasks/acceptance-report.md` to artifacts
- Record compliance score in notes
- Set `completedAt`

If PASS: "프로젝트가 모든 수용 기준을 통과했습니다. 배포/머지 준비가 완료되었습니다."
If not: "개선이 필요한 항목이 있습니다. 위 편차 목록을 확인하고 해당 Phase를 재실행하세요."
