---
name: prd
description: 인터뷰 기반 요구사항 정의서(PRD) 작성
user_invocable: true
---

# SI PRD Phase

You are creating a Product Requirements Document through structured interview. This combines brainstorming with interview-then-plan methodology to ensure requirements are complete before any technical work begins.

## Prerequisites
- Read `tasks/research-report.md` if it exists (from Research phase)
- Read `tasks/si-progress.json` for project context

## Phase 1: Structured Interview

Conduct a focused interview. Ask questions ONE AT A TIME, not all at once. Adapt based on answers.

### Core Questions (ask in order, skip if already answered in research):

1. **사용자 관점**: "이 기능을 사용하는 사람은 누구이고, 어떤 상황에서 사용합니까?"
2. **핵심 행동**: "사용자가 반드시 할 수 있어야 하는 것은 무엇입니까? (최대 5개)"
3. **성공 기준**: "이 기능이 성공했다면, 무엇이 달라져 있습니까?"
4. **경계 조건**: "이 기능이 하지 않아야 할 것은 무엇입니까? (Non-goals)"
5. **비기능 요구**: "성능, 보안, 접근성 중 특별히 중요한 것이 있습니까?"
6. **기존 제약**: "변경 불가능한 것이 있습니까? (기존 API, DB 스키마, 외부 서비스 등)"

### Follow-up Questions (context-dependent):
- Ambiguous answer → "구체적으로 [X]는 어떤 경우를 말합니까?"
- Missing edge case → "[Y] 상황에서는 어떻게 동작해야 합니까?"
- Conflicting requirements → "[A]와 [B]가 충돌합니다. 우선순위는?"

## Phase 2: Requirements Synthesis

After the interview, synthesize into a structured PRD.

### EARS Format (Mandatory)
All requirements MUST use EARS patterns. Reference: `${CLAUDE_PLUGIN_ROOT}/settings/rules/ears-format.md`

Write to `tasks/requirements.md`:

```markdown
# Requirements: [Project Name]

## 1. Overview
[2-3 paragraphs summarizing the project from interview]

## 2. User Stories
### US-001: [Title]
As a [role], I want to [action], so that [benefit].

## 3. Functional Requirements
### FR-001: [Title]
[EARS pattern]
- Priority: Must / Should / Could
- Acceptance criteria: Given/When/Then (brief)

### FR-002: [Title]
...

## 4. Non-Functional Requirements
### NFR-001: [Title]
[EARS pattern]
- Metric: [measurable criterion]

## 5. Non-Goals (Explicit Exclusions)
- NG-001: [what this project will NOT do]

## 6. Constraints
- C-001: [immovable constraint from interview]

## 7. Open Questions
- OQ-001: [unresolved question, to be addressed in Analysis/Architect]

## 8. Glossary
| Term | Definition |
|------|-----------|
| | |
```

## Phase 3: User Review

Present the requirements document summary to the user:
- List all FR/NFR with IDs and priorities
- Highlight any assumptions you made
- Ask: "요구사항을 검토해주세요. 추가, 수정, 삭제할 항목이 있습니까?"

Iterate until the user approves.

## Phase 4: Update Progress

Update `tasks/si-progress.json`:
- Set `phases.prd.status = "completed"`
- Add `tasks/requirements.md` to artifacts
- Set `completedAt` to current timestamp

Inform: "요구사항 정의가 완료되었습니다. `/si:start`로 돌아가서 게이트 승인을 진행하세요."
