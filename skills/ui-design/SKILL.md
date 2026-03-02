---
name: si:ui-design
description: "SI UI/UX 디자인 특화 — Pencil MCP 활용, HTML 프로토타입 폴백, 디자인-코드 매핑"
compatibility:
  required_tools:
    - WebFetch
  recommended_tools:
    - name: mcp__pencil
      purpose: "AI design canvas — prompt to visual design, .pen file generation"
    - name: mcp__claude-in-chrome
      purpose: "Browser-based prototype rendering and visual verification"
    - name: Agent
      purpose: "Parallel subagents for multi-screen design"
---

# SI UI Design

SI 워크플로우의 UI Design Phase를 위한 스킬. 기술 설계(tasks/design.md)를 기반으로 실현 가능한 UI를 설계하고, TDD 진입 전 "무엇을 만들지"를 시각적으로 합의한다.

## Reference Files

도구별 상세 정보가 필요할 때 참조:

- **`references/pencil-mcp.md`** — Pencil MCP 설치, 인증, 도구 상세(batch_design 6개 operation 등), Design-to-Code 기술 스택, 변수/컴포넌트/UI Kit, Import/Export, 단축키, CLI, 제한사항/트러블슈팅. Pencil 사용 시 참조.
- **`references/pencil-pen-format.md`** — .pen 파일 포맷 스펙: 오브젝트 타입, 레이아웃, 필/스트로크/이펙트, 컴포넌트/인스턴스, 변수/테마. .pen 파일 직접 읽기/수정 시 참조.

모든 참조를 미리 읽을 필요 없음. 특정 결정이 필요할 때 참조.

---

## Why this skill exists

기술 설계(ADR, 컴포넌트 구조, API 경계)만으로 코딩에 들어가면:
- 화면 레이아웃이 즉흥적으로 결정됨 — 일관성 부재
- 디자인 시스템 없이 컴포넌트가 파편화됨
- TDD에서 "무엇을 테스트할지"가 모호해짐 (시각적 기대가 없으므로)
- 사용자 피드백이 구현 후에야 발생 — 재작업 비용 증가

이 스킬은 Architect ↔ TDD 사이의 갭을 메운다.

---

## Design Principles

모든 UI 결정에 적용:

1. **레이아웃 계층**: 시각적 계층 → 정보 계층. 중요한 것이 눈에 먼저 들어와야 한다.
2. **일관성**: 같은 패턴에 같은 컴포넌트. 디자인 토큰으로 통일.
3. **접근성**: 키보드 네비게이션, 포커스 관리, 충분한 대비, 의미 있는 빈/에러 상태.
4. **최소 놀라움**: 사용자가 예상하는 위치에 예상하는 동작. 화려한 효과보다 명확한 인터랙션.
5. **기술 실현 가능성**: `tasks/design.md`의 컴포넌트 구조와 API 경계 안에서 설계.

---

## Step 0: Context Gathering

### 기술 설계에서 추출할 정보:
- `tasks/design.md` §2 Architecture — 컴포넌트 구조, 레이어
- `tasks/design.md` §4 System Flows — 사용자 흐름 (Mermaid)
- `tasks/design.md` §6 Components & Interfaces — 공개 인터페이스
- `tasks/design.md` §7 Data Models — 화면에 표시될 데이터

### 요구사항에서 추출할 정보:
- `tasks/requirements.md` — 기능 요구사항별 사용자 스토리
- 비기능 요구사항 중 UI 관련 (반응형, 성능, 접근성)

### 기존 코드에서 추출할 정보 (Extension/Simple Addition일 때):
- 기존 디자인 시스템/컴포넌트 라이브러리 (Shadcn UI, Material UI 등)
- 기존 레이아웃 패턴 (사이드바, 탑바, 그리드 등)
- 기존 디자인 토큰 (CSS 변수, theme config)

---

## Step 1: Screen Identification

System Flows와 기능 요구사항에서 화면 목록 추출:

| Screen | Route | User Goal | Data Required |
|--------|-------|-----------|---------------|
| | | | |

**우선순위 결정:**
- 핵심 사용자 흐름의 화면 2-3개를 먼저 디자인
- 나머지는 핵심 화면의 패턴을 재사용

---

## Step 2: Tool Selection

리서치 시작 시 `ToolSearch`로 가용 도구를 확인:

```
ToolSearch: "pencil"
```

### If Pencil MCP available:
→ `references/pencil-mcp.md` 참조하여 Pencil에서 디자인
→ .pen 파일을 프로젝트에 저장

### If Pencil MCP NOT available:
→ HTML/CSS/JS 프로토타입으로 폴백 (아래 "HTML Prototype Fallback" 참조)

---

## Step 3: Design Execution

### Pencil MCP Path

1. Pencil 프로젝트 생성/열기
2. 화면별 디자인 생성:
   - `batch_design`으로 프롬프트 기반 화면 생성 (디바이스 크기: 모바일 390x844, 데스크톱 1440x900 등)
   - 프로젝트의 UI Kit 컴포넌트 사용
3. 디자인 토큰 설정:
   - `set_variables`로 색상, 타이포그래피, 간격 등 디자인 토큰 적용
   - `get_variables`로 현재 토큰 확인
4. 결과 확인:
   - `get_screenshot`으로 시각적 결과 확인
   - `snapshot_layout`으로 레이아웃 구조 검증
5. 복수 화면이면 Swarm Mode 활용
6. 사용자가 Pencil에서 직접 미세 조정

### HTML Prototype Fallback

Pencil이 없을 때 Claude Code가 직접 프로토타입 생성:

```
1. 프로젝트의 기존 디자인 시스템을 사용하여 HTML/CSS 생성
2. 가능하면 실제 컴포넌트 라이브러리 CDN 링크 사용
3. claude-in-chrome으로 렌더링:
   - tabs_create_mcp → 새 탭
   - navigate → file:///path/to/prototype.html
   - read_page → 스크린샷 캡처
4. 사용자에게 보여주고 피드백 수렴
5. 피드백 반영하여 반복
```

**HTML 프로토타입 작성 규칙:**
- 프로젝트 기술 스택에 맞는 도구 사용 (Tailwind, CSS Modules 등)
- 실제 데이터와 유사한 더미 데이터 사용 (Lorem ipsum 지양)
- 반응형 레이아웃 포함
- 인터랙션 상태 표현 (hover, active, disabled, error, empty)

---

## Step 4: Design-to-Spec Conversion

디자인 결과물을 `tasks/ui-design.md`로 변환:

### Pencil MCP 도구로 추출 (Pencil 사용 시):
- `batch_get` → 디자인 요소 데이터 → Screen Inventory
- `snapshot_layout` → 레이아웃 구조 → Component Hierarchy
- `get_variables` → 디자인 변수 → Design Tokens
- `get_screenshot` → 화면 캡처 → Screen Flows 참고 자료

### HTML 프로토타입에서 추출 (폴백 시):
- DOM 구조 → Component Hierarchy
- CSS 변수/클래스 → Design Tokens
- 라우팅/네비게이션 → Screen Flows
- 이벤트 핸들러 → Interaction Specifications

### 필수 검증:
1. **Coverage**: 모든 기능 요구사항이 최소 하나의 화면에 매핑됨
2. **Consistency**: 같은 패턴에 같은 컴포넌트 사용
3. **Feasibility**: 모든 UI 컴포넌트가 `tasks/design.md`의 기술 컴포넌트에 매핑됨
4. **Accessibility**: 키보드, 대비, 포커스 계획 존재

---

## Step 5: Visual Verification

`claude-in-chrome`으로 최종 검증:

1. 프로토타입/코드 변환 결과를 브라우저에서 렌더링
2. 스크린샷 캡처
3. `tasks/design.md`의 인터페이스와 교차 검증
4. 접근성 기본 확인 (대비, 포커스 순서)

---

## Common Failure Patterns — 자기 점검

이 행동을 하고 있으면 멈추고 수정:

- **기술 설계 무시**: `tasks/design.md`의 컴포넌트 구조와 맞지 않는 UI 설계
- **과잉 디자인**: 핵심 화면 대신 모든 가능한 화면을 상세 설계
- **토큰 미정의**: 색상/간격/타이포를 하드코딩하고 토큰화하지 않음
- **접근성 후순위**: "나중에 접근성" → 처음부터 기본은 포함
- **디자인-코드 갭**: UI 명세와 기술 컴포넌트의 매핑이 없음
- **사용자 피드백 생략**: 후보 방향을 보여주지 않고 단일 안으로 진행

---

## When Stuck

| Problem | Solution |
|---------|----------|
| 디자인 시스템이 없음 | 프로젝트 기술 스택에서 추론 (Next.js → Shadcn UI, React → Material UI 등) |
| Pencil MCP 연결 안됨 | HTML 프로토타입으로 폴백. `references/pencil-mcp.md`의 설치 가이드 참조 |
| 사용자가 시각적 피드백 못 줌 | ASCII 목업 + 설명으로 대안 제시 |
| 기존 디자인이 없음 (신규 프로젝트) | 경쟁 앱 참고 제안 + 기본 레이아웃 패턴 적용 |
| 화면이 너무 많음 | 핵심 2-3개만 상세 설계, 나머지는 패턴 재사용 명시 |
