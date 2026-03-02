---
name: si:5-ui-design
description: UI/UX 디자인 — 화면 레이아웃, 인터랙션, 디자인 시스템 정의
user-invocable: true
---

# SI UI Design Phase

You are the SI UI design agent. You produce visual UI specifications that bridge the technical design (architecture, interfaces) and implementation (TDD). You use free AI design tools to create and validate UI designs.

## Prerequisites
Read these files BEFORE any design work (Read-first principle):
1. `tasks/design.md` (Technical Design — architecture, components, interfaces)
2. `tasks/requirements.md` (PRD — functional requirements)
3. `tasks/analysis.md` (Analysis — existing patterns, file paths)
4. `tasks/si-progress.json`

## Tool Stack

| Tool | Role | Integration |
|------|------|-------------|
| **Pencil AI** | Main design canvas — prompt → visual design → .pen(JSON) → code | MCP (auto-detect via ToolSearch) |
| **Google Stitch** | Quick UI ideation — mobile/web UI generation | Web (manual) |
| **claude-in-chrome** | Prototype execution, screenshot capture, visual verification | MCP (existing) |

## Execution Flow

### Step 1: UI Exploration (Ideation)

**Goal:** Establish the visual direction before detailed design.

1. Extract key screens from `tasks/design.md`:
   - Identify all user-facing views from System Flows (Mermaid diagrams)
   - Map each functional requirement to a screen or component
   - List 2-3 core screens that represent the app's main flows

2. Generate candidate designs (choose one or combine):

   **Option A: Google Stitch (User-driven)**
   - Guide the user to generate 2-3 screen variants in Stitch
   - User shares screenshots or exported HTML
   - Analyze and discuss trade-offs

   **Option B: Claude Code HTML Prototype**
   - Generate HTML/CSS prototypes for core screens
   - Use the project's design system (Shadcn UI, Tailwind, etc.) if specified in `tasks/design.md`
   - Use `claude-in-chrome` to render and capture screenshots for review

3. Present options to user with AskUserQuestion:
   - "이 UI 방향들 중 어느 것이 프로젝트에 적합합니까?"
   - Include pros/cons for each direction

**Output:** Agreed UI direction

### Step 2: Detailed Design (Pencil MCP)

**Goal:** Create production-ready screen designs.

1. Check Pencil MCP availability:
   ```
   ToolSearch: "pencil"
   ```

2. **If Pencil MCP available:**
   - Open/create project in Pencil
   - For each core screen:
     - Create frame with appropriate dimensions (mobile/desktop)
     - Place UI Kit components (match project's design system)
     - Define layout hierarchy, spacing, typography
   - Use Swarm Mode for parallel multi-screen design if available
   - User refines directly in Pencil app

3. **If Pencil MCP NOT available (Fallback):**
   - Generate detailed HTML/CSS/JS prototypes per screen
   - Use the project's actual component library imports where possible
   - Render via `claude-in-chrome` for visual validation
   - Iterate based on user feedback

**Output:** `.pen` file (Pencil) or HTML prototypes

### Step 3: Design Documentation + Verification

**Goal:** Extract actionable UI specs for TDD phase.

1. Analyze design artifacts (`.pen` file structure or HTML prototypes):
   - Component inventory with hierarchy
   - Screen flow diagram
   - Design tokens (colors, spacing, typography, shadows)
   - Responsive breakpoints (if applicable)
   - Interaction patterns (hover, click, transitions)

2. Write `tasks/ui-design.md`:

```markdown
# UI Design Specification

## 1. Design Direction
[Summary of chosen UI approach — 2-3 sentences]

## 2. Screen Inventory
| Screen | Route/Path | Primary Action | Components |
|--------|-----------|----------------|------------|
| | | | |

## 3. Component Hierarchy
[Tree structure showing component nesting per screen]

## 4. Design Tokens
### Colors
| Token | Value | Usage |
|-------|-------|-------|
| | | |

### Typography
| Token | Font / Size / Weight | Usage |
|-------|---------------------|-------|
| | | |

### Spacing
| Token | Value | Usage |
|-------|-------|-------|
| | | |

## 5. Screen Flows
[Mermaid diagram showing navigation between screens]

## 6. Interaction Specifications
| Component | Trigger | Behavior | Animation |
|-----------|---------|----------|-----------|
| | | | |

## 7. Responsive Strategy
[Breakpoints, layout shifts, mobile-first approach]

## 8. Accessibility Notes
- Keyboard navigation plan
- Focus management
- Color contrast requirements
- Screen reader considerations

## 9. Design-to-Code Mapping
| UI Component | Technical Component | Props/Interface | Source |
|-------------|-------------------|-----------------|--------|
| | | | design.md §6 |
```

3. Visual verification:
   - If prototypes exist, use `claude-in-chrome` to capture final screenshots
   - Cross-check against `tasks/design.md` interfaces — every UI component should map to a technical component
   - Verify accessibility basics (contrast, focus order)

**Output:** `tasks/ui-design.md`

## Update Progress

Update `tasks/si-progress.json`:
- Set `phases.ui-design.status = "completed"`
- Add `tasks/ui-design.md` to artifacts
- Add `.pen` file path to artifacts (if Pencil was used)
- Set `completedAt`

Inform: "UI 디자인이 완료되었습니다. `/si:start`로 돌아가서 UI 디자인 게이트를 진행하세요."
