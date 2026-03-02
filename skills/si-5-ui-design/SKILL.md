---
name: si-5-ui-design
user-invocable: true
description: UI/UX 디자인 — 화면 레이아웃, 인터랙션, 디자인 시스템 정의
---

# SI UI Design Phase

You are the SI UI design orchestrator. You delegate UI design work to the specialized skill and verify results.

## Prerequisites
Read these files BEFORE any design work (Read-first principle):
1. `tasks/design.md` (Technical Design — architecture, components, interfaces)
2. `tasks/requirements.md` (PRD — functional requirements)
3. `tasks/analysis.md` (Analysis — existing patterns, file paths)
4. `tasks/si-progress.json`

## Execution

### Step 1: Invoke UI Design Protocol

Follow the `si-ui-design` skill for the full UI design protocol.

Invoke: `/si-ui-design`

The skill handles:
- Context gathering from technical design and requirements
- Screen identification and prioritization
- Tool selection (Pencil MCP or HTML prototype fallback)
- Design execution with user feedback loops
- Design-to-spec conversion with 9-section template

### Step 2: Verify Output

After the skill completes, verify:
- `tasks/ui-design.md` exists and contains all 9 sections:
  1. Design Direction
  2. Screen Inventory
  3. Component Hierarchy
  4. Design Tokens
  5. Screen Flows
  6. Interaction Specifications
  7. Responsive Strategy
  8. Accessibility Notes
  9. Design-to-Code Mapping
- `.pen` file or HTML prototype exists as design artifact
- Design-to-Code Mapping references `tasks/design.md` §6 components

### Sub-reports (Optional)
중간 산출물이나 상세 분석이 있으면 `tasks/ui-design/`에 개별 파일로 저장.
최종 통합 파일은 `tasks/ui-design.md`에 작성.
서브리포트 경로는 `tasks/si-progress.json`의 `artifacts` 배열에 추가.

### Step 3: Update Progress

Update `tasks/si-progress.json`:
- Set `phases.ui-design.status = "completed"`
- Add `tasks/ui-design.md` to artifacts
- Add `.pen` file path to artifacts (if Pencil was used)
- Set `completedAt` to current timestamp

Inform:
"UI 디자인이 완료되었습니다. `/si-start`로 돌아가서 UI 디자인 게이트를 진행하세요."
