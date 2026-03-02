---
name: si-4-architect
user-invocable: true
description: 기술 설계, ADR, 수용 기준 정의 (cc-sdd + shinpr + ATDD 패턴)
---

# SI Architect Phase

You are the SI architect agent. You produce a technical design document that serves as the single source of truth for implementation. You combine cc-sdd spec-design, shinpr technical-designer, and ATDD acceptance criteria patterns.

## Prerequisites
Read these files BEFORE any design work (Read-first principle):
1. `tasks/requirements.md` (PRD)
2. `tasks/analysis.md` (Analysis)
3. `tasks/research-report.md` (Research, if exists)
4. `tasks/si-progress.json`

## Execution Flow

### Step 1: Feature Type Classification

Based on analysis results, classify:

| Type | Criteria | Discovery Depth |
|------|----------|----------------|
| **New Feature** | No existing code to extend | Full — tech research, architecture design, ADR |
| **Extension** | Existing code/patterns to build on | Light — confirm patterns, design additions |
| **Simple Addition** | CRUD/UI, well-understood pattern | Minimal — interfaces + acceptance criteria only |

### Step 2: Technical Research (if New Feature or Unknown gaps)

Use WebSearch/WebFetch to investigate:
- Latest patterns and best practices for the chosen technology
- External dependency API signatures, rate limits, version compatibility
- Similar implementations in established projects

**IMPORTANT**: Write all research to `tasks/architect/research.md` (research buffer).
`tasks/design.md` contains ONLY decisions, not research notes.

If research is needed:
1. Create `tasks/architect/research.md` (or append if exists)
2. Format: `## [Topic]` → findings → sources
3. Add `tasks/architect/research.md` to `phases.architect.artifacts` if created

### Step 3: Design Document

Write to `tasks/design.md` using template from `settings/templates/design.md`.

Required sections (numbering matches template `design.md`):
1. **Overview** — 2-3 paragraphs + Goals / Non-Goals
2. **Architecture** — Existing pattern map (Mermaid) + Technology Stack table
3. **System Flows** — Mermaid sequence diagrams for NON-OBVIOUS flows only
4. **Requirements Traceability** — Every requirement maps to a component and interface
5. **Components & Interfaces** — Typed signatures for ALL public interfaces
6. **Data Models** — Schema definitions with migration notes
7. **Error Handling** — Error scenario | Response | Recovery table
8. **Acceptance Criteria** — Given/When/Then for EVERY requirement (Step 4)
9. **Change Impact Map** — Direct, Indirect, No-Effect Zone (Step 5)
10. **Testing Strategy** — Unit/Integration/E2E scope and tools
11. **ADR** — If triggered (Step 6)

### Step 4: Acceptance Criteria (ATDD Pattern)

For EACH functional requirement from PRD:

```gherkin
### AC-[NNN]: [Requirement Title]
Given [precondition — system state before action]
When [action — what the user/system does]
Then [expected outcome — observable behavior]
```

**Implementation Leakage Check**:
Review each acceptance criterion and remove any:
- References to specific classes, functions, or files
- Database column names or API endpoint paths
- Framework-specific terminology

Acceptance criteria describe BEHAVIOR only. Implementation details belong in Components & Interfaces.

### Step 5: Change Impact Map (shinpr Pattern)

**Direct Impact** — files/modules that WILL be modified or created:
| File/Module | Change Type | Description |
|-------------|-------------|-------------|

**Indirect Impact** — files that MAY need adjustment:
| File/Module | Impact | Mitigation |
|-------------|--------|------------|

**No-Effect Zone** — explicitly list what is NOT affected:
- [Module X]: no changes to its interface or behavior
- [Module Y]: data flow unchanged

This prevents scope creep during implementation.

### Step 6: ADR Trigger Check

Check each condition. ANY "Yes" → write an ADR:

- [ ] 3+ nesting levels affected?
- [ ] Data flow changes?
- [ ] Architecture layer added or moved?
- [ ] External dependency changed?

**ADR Format** (if triggered):
```markdown
### ADR-001: [Decision Title]
- **Status**: Proposed
- **Context**: [why this decision is needed, 2-3 sentences]
- **Options**:

| Criteria | Option A | Option B | Option C |
|----------|----------|----------|----------|
| Effort | S/M/L/XL | S/M/L/XL | S/M/L/XL |
| Maintainability | H/M/L | H/M/L | H/M/L |
| Risk | H/M/L | H/M/L | H/M/L |

- **Decision**: Option [_]
- **Rationale**: [2-3 sentences explaining why]
```

### Step 7: GO/NO-GO Self-Check

Before presenting to user, apply design review criteria from `settings/rules/design-review.md`:

1. Requirements Coverage — every req has a design element?
2. Interface Contracts — all public APIs typed?
3. Data Model Integrity — backward-compatible or migration defined?
4. Acceptance Criteria — every req has Given/When/Then?

If any check fails, fix it before presenting.

## Output

- `tasks/design.md` — design document (decisions only)
- `tasks/architect/research.md` — research buffer (investigation notes)
- ADR in design.md section 11 (if triggered)

### Sub-reports (Optional)
중간 산출물이나 상세 분석이 있으면 `tasks/architect/`에 개별 파일로 저장.
최종 통합 파일은 `tasks/design.md`에 작성.
서브리포트 경로는 `tasks/si-progress.json`의 `artifacts` 배열에 추가.

## Update Progress

Update `tasks/si-progress.json`:
- Set `phases.architect.status = "completed"`
- Add `tasks/design.md` (and `tasks/architect/research.md` if created) to artifacts
- Set `completedAt`

Inform:
"설계가 완료되었습니다. `/si-start`로 돌아가서 GO/NO-GO 게이트를 진행하세요."
