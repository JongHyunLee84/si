# Pencil MCP Reference

Pencil AI는 프롬프트로 UI를 디자인하고 코드로 변환하는 AI 디자인 도구. MCP를 통해 Claude Code와 연동하여 자동 감지된다.

- **공식 사이트**: https://www.pencil.dev
- **공식 문서**: https://docs.pencil.dev

---

## Supported AI Assistants

> Source: [AI Integration](https://docs.pencil.dev/getting-started/ai-integration)

Pencil MCP를 지원하는 AI 도구:

| AI Tool | Type | Notes |
|---------|------|-------|
| **Claude Code** | CLI / IDE | 기본 권장. `claude` CLI 인증 필요 |
| **Cursor** | IDE | Extensions에서 Pencil 설치, Settings → Tools & MCP에서 연결 확인 |
| **Claude Desktop** | Desktop App | MCP 서버 자동 연결 |
| **Windsurf** | IDE (Codeium) | Pencil 익스텐션 설치 후 사용 |
| **Codex CLI** | CLI (OpenAI) | `/mcp` 명령으로 서버 확인 |
| **Antigravity** | IDE | Pencil 익스텐션 설치 후 사용 |
| **OpenCode** | CLI | Pencil 익스텐션 설치 후 사용 |

---

## Installation

> Source: [Installation](https://docs.pencil.dev/getting-started/installation)

### 1. Pencil 앱 설치

**방법 A: VS Code / Cursor 익스텐션** (권장)
- VS Code 또는 Cursor의 Extensions에서 "Pencil" 검색 후 설치
- 설치 확인: `.pen` 파일 생성 시 에디터 우상단에 Pencil 아이콘 표시

**방법 B: 데스크톱 앱**
- **macOS**: https://www.pencil.dev 에서 `.dmg` 다운로드 후 Applications에 드래그
- **Linux**: `.deb` (`sudo dpkg -i`) 또는 `.AppImage` (`chmod +x` 후 실행)
- **Windows**: 데스크톱 앱 없음. VS Code / Cursor 익스텐션 사용

> ⚠️ `brew install --cask pencil`은 **다른 도구**(The Pencil Project wireframing tool)를 설치함. 사용하지 않는다.

### 플랫폼별 주의사항

| Platform | Notes |
|----------|-------|
| macOS | 데스크톱 앱 + 익스텐션 모두 사용 가능 |
| Windows | 익스텐션만 지원 (데스크톱 앱 없음) |
| Linux | 데스크톱 앱 사용 가능. X11이 Wayland/Hyprland보다 안정적 |
| Cursor | "You need Cursor Pro" 메시지가 나올 수 있음 (Cursor 플랜 제한) |

### 2. Claude Code CLI 설치 (AI 기능 필수)

```bash
# npm으로 설치
npm install -g @anthropic-ai/claude-code-cli
```

### 3. MCP 자동 감지

Pencil 앱이 실행 중이면 Claude Code가 MCP 서버를 자동 감지한다.
별도의 `claude mcp add` 등록이 필요 없다.

> Source: [AI Integration](https://docs.pencil.dev/getting-started/ai-integration)

---

## Authentication

> Source: [Authentication](https://docs.pencil.dev/getting-started/authentication)

Pencil은 **두 단계 인증**이 필요하다: Pencil 활성화 + Claude Code 인증.

### Step 1: Pencil 활성화 (이메일)

1. Pencil 첫 실행 시 이메일 입력
2. 수신된 활성화 코드 입력
3. 활성화 완료

**알려진 이슈:**
- "Invite for your email address was not found" → 익스텐션 재설치 후 재시도
- 반복적 활성화 프롬프트 → IDE 재시작 또는 익스텐션 재설치
- 이메일 미수신 → 스팸 폴더 확인, 다른 이메일 시도

### Step 2: Claude Code 인증

**방법 A: CLI 인증** (권장)

```bash
claude
# 브라우저가 열리고 Anthropic 계정으로 로그인
# 인증 정보가 로컬에 저장됨
```

검증: `claude --version` 실행 또는 Pencil에서 `Cmd/Ctrl + K` 패널 작동 확인.

**방법 B: API 키** (비권장)

`ANTHROPIC_API_KEY` 환경 변수 설정. CLI 인증과 **충돌할 수 있으므로** 동시 사용 금지.

### 인증 충돌 해결

| 증상 | 원인 | 해결 |
|------|------|------|
| "Claude Code isn't connected" | 인증 미완료 | 터미널에서 `claude` 실행 후 Pencil 복귀 |
| "Invalid API key" / "Please run /login" | 다중 인증 방법 충돌 | `ANTHROPIC_API_KEY` 환경 변수 제거, CLI 재인증 |
| CLI 작동하지만 Pencil 미연결 | 커스텀 프로바이더 충돌 | IDE 재시작, 충돌 환경 변수 제거 |

### 보안 참고

MCP 서버는 **로컬에서만** 실행된다. 디자인 데이터는 AI 프롬프트 사용 시에만 Claude에 전송된다.

---

## MCP Connection Verification

> Source: [AI Integration](https://docs.pencil.dev/getting-started/ai-integration)

### IDE별 연결 확인

| IDE | 확인 방법 |
|-----|----------|
| Claude Code | `ToolSearch: "pencil"` → 도구 목록 확인 |
| Cursor | Settings → Tools & MCP → Pencil 서버 확인 |
| VS Code | MCP 설정 확인, `.pen` 파일 열기 |
| Codex CLI | `/mcp` 명령 → 서버 목록에 Pencil 표시 |

### 연결 실패 시

1. Pencil 앱이 실행 중인지 확인
2. `.pen` 파일이 프로젝트에 있는지 확인
3. Claude Code CLI 인증 완료 확인
4. IDE / Pencil 재시작
5. Pencil 앱을 최신 버전으로 업데이트

---

## MCP Tools — 상세

> Source: [AI Integration](https://docs.pencil.dev/getting-started/ai-integration)

### `batch_design` — 디자인 생성/수정 (핵심 도구)

6가지 operation을 지원하는 배치 디자인 도구:

| Operation | 설명 | 사용 시점 |
|-----------|------|----------|
| **insert** | 새 디자인 요소 추가 | 화면/컴포넌트 초기 생성 |
| **copy** | 기존 요소 복제 | 유사 화면 변형 생성 |
| **update** | 기존 요소 속성 수정 | 색상, 크기, 텍스트 변경 |
| **replace** | 기존 요소를 새 요소로 교체 | 컴포넌트 전면 교체 |
| **move** | 요소 위치/순서 변경 | 레이아웃 재배치 |
| **delete** | 요소 제거 | 불필요한 요소 정리 |

이미지 생성 및 배치도 지원.

**사용 예시:**
- "Design a dashboard with sidebar and main content area"
- "Change all primary buttons to blue"
- "Create a button component with variants"

### `batch_get` — 디자인 데이터 일괄 조회

디자인 요소 데이터를 읽는 도구. 패턴 기반 검색 지원.

| 기능 | 설명 |
|------|------|
| 요소 조회 | 특정 요소의 속성/구조 읽기 |
| 패턴 검색 | 이름/타입으로 요소 검색 |
| 계층 탐색 | 컴포넌트의 자식 요소 구조 파악 |

**사용 시점:** 현재 디자인 상태를 파악하고 코드 변환 전 정보 수집.

### `get_screenshot` — 캔버스 스크린샷

현재 캔버스의 시각적 렌더링을 캡처.

- 디자인 결과물 시각 확인
- Before/After 비교
- 사용자에게 결과 공유

### `snapshot_layout` — 레이아웃 구조 스냅샷

레이아웃의 구조적 정보를 추출.

- 포지셔닝 이슈 감지
- 겹치는 요소 식별
- 컴포넌트 배치 구조 파악

### `get_editor_state` — 에디터 상태 조회

현재 에디터 컨텍스트 정보 조회.

- 현재 선택된 요소
- 줌 레벨
- 활성 파일 정보

### `get_variables` / `set_variables` — 디자인 변수(토큰) 관리

| Tool | 용도 |
|------|------|
| `get_variables` | 색상, 타이포, 간격 등 디자인 토큰 읽기 |
| `set_variables` | 디자인 토큰 값 설정/변경, 테마별 값 정의 |

CSS 변수 및 Tailwind config와 양방향 동기화 가능.

### 도구 활용 워크플로우

```
1. get_editor_state   → 현재 상태 파악
2. batch_design       → 디자인 생성/수정
3. get_screenshot     → 시각적 결과 확인
4. batch_get          → 요소 데이터 추출
5. snapshot_layout    → 구조 검증
6. get_variables      → 토큰 확인
```

---

## Design-to-Code Technologies

> Source: [Design to Code](https://docs.pencil.dev/design-and-code/design-to-code)

Pencil이 코드 생성 시 지원하는 기술 스택:

### Frameworks

| Framework | Notes |
|-----------|-------|
| React | JavaScript / TypeScript |
| Next.js | App Router 지원 |
| Vue | — |
| Svelte | — |
| Plain HTML/CSS | 프레임워크 없이 |

### Styling

| Option | Notes |
|--------|-------|
| **Tailwind CSS** | 권장 |
| CSS Modules | — |
| Styled Components | — |
| Plain CSS | — |

### Component Libraries

| Library | Notes |
|---------|-------|
| Shadcn UI | React 기반, Tailwind 스타일링 |
| Radix UI | 접근성 중심 프리미티브 |
| Chakra UI | — |
| Material UI | — |
| Custom components | 프로젝트 자체 컴포넌트 |

### Icon Libraries

- 캔버스 내 빌트인: Material Icons
- 코드 생성 시: Lucide, Heroicons, FontAwesome, React Icons (프롬프트에서 지정)

### 코드 생성 팁

프롬프트에서 기술 스택을 **구체적으로 명시**:
- "Generate Next.js 14 code with Tailwind CSS"
- "Use Shadcn UI components for this layout"
- "Use Lucide icons instead of Material Icons"

---

## Variables System

> Source: [Variables](https://docs.pencil.dev/core-concepts/variables)

Variables는 디자인 토큰으로, CSS custom properties와 유사하게 동작한다. 한 번 정의하면 프로젝트 전체에 재사용되고, 변경 시 모든 인스턴스에 자동 전파된다.

### Variable Types

| Type | 용도 | 예시 |
|------|------|------|
| **color** | 색상값 | `#3b82f6`, `#RRGGBBAA` |
| **number** | 숫자값 (간격, 크기) | `16`, `1.5` |
| **boolean** | 켜기/끄기 | `true`, `false` |
| **string** | 텍스트값 (폰트명 등) | `"Inter"` |

### 생성 방법 3가지

**1. CSS에서 추출**
```
AI에게 "globals.css에서 변수를 생성해줘" 요청
→ 색상, 간격, 폰트 자동 추출
```

**2. Figma에서 가져오기**
```
Figma에서 토큰 테이블 복사/붙여넣기
또는 토큰 테이블 스크린샷 붙여넣기
→ AI가 변수 자동 생성
```

**3. 수동 정의**
```
Pencil에서 직접 변수 정의 및 테마별 값 설정
```

### CSS 양방향 동기화

- Pencil 변수 → CSS 변수/Tailwind config 내보내기
- CSS 변수/Tailwind config → Pencil 변수 가져오기

### 테마 지원

변수에 테마별 다른 값 정의 가능 (예: light/dark mode).
.pen 파일에서 `$variable-name` 구문으로 바인딩.

---

## Components

> Source: [Components](https://docs.pencil.dev/core-concepts/components)

### Blue vs Purple 구분

| 색상 | 의미 | 설명 |
|------|------|------|
| **Blue** 바운딩 박스 | 일반 요소 | 프레임, 도형, 텍스트 등 기본 요소 |
| **Purple/Magenta** 바운딩 박스 | 재사용 가능 컴포넌트 | Figma 컴포넌트 / React 컴포넌트와 유사 |

### 컴포넌트 생성

1. 요소 선택
2. **`Cmd/Ctrl + Option/Alt + K`** 누르기
3. 선택 해제 후 다시 선택 → Purple 하이라이트 확인

### 컴포넌트의 장점

- 하나를 수정하면 **모든 인스턴스 자동 업데이트**
- 디자인 시스템 구축에 활용
- 코드 컴포넌트와 동기화

### 인스턴스와 오버라이드

.pen 파일에서 `ref` 타입으로 인스턴스 생성, `descendants` 객체로 속성 오버라이드 가능.
상세 스펙은 `references/pencil-pen-format.md` 참조.

---

## UI Kits

> Source: [Styles and UI Kits](https://docs.pencil.dev/design-and-code/styles-and-ui-kits)

Pencil에서 사용 가능한 프리빌트 디자인 킷:

| UI Kit | 설명 |
|--------|------|
| **Shadcn UI** | Popular React component library |
| **Halo** | Modern design system |
| **Lunaris** | Design system |
| **Nitro** | Design system |

공통 UI 패턴(버튼, 카드, 폼, 네비게이션 등)에 즉시 사용 가능한 컴포넌트 제공.

프로젝트의 `factory/architect/architect.md` Technology Stack에서 프레임워크를 확인하고 적절한 UI Kit 선택.

---

## Import / Export

> Source: [Import and Export](https://docs.pencil.dev/core-concepts/import-and-export)

### Import

**이미지 가져오기:**
- **드래그 앤 드롭** (가장 안정적, 모든 플랫폼)
- 지원 형식: PNG, JPEG, SVG
- macOS에서 File 메뉴 import는 불안정 → 드래그 앤 드롭 사용

**Figma에서 가져오기:**
- Figma에서 프레임 복사 → Pencil에 붙여넣기
- ⚠️ **이미지는 전송되지 않음** — SVG 사용 또는 이미지 별도 재import 필요

**아이콘:**
- 빌트인 Material Icons 라이브러리
- 커스텀 SVG path 지원
- 코드 생성 시 프롬프트에서 아이콘 라이브러리 지정 (Lucide, Heroicons 등)

### Export

**디자인 → 코드:**
- `Cmd/Ctrl + K`로 AI 프롬프트 열기
- 기술 스택 지정하여 코드 생성 (위 "Design-to-Code Technologies" 참조)

**이미지 내보내기:**
- 프레임 우클릭 → Export (PNG, SVG)

---

## Swarm Mode

> Source: [Pencil CLI](https://docs.pencil.dev/for-developers/pencil-cli) — CLI의 병렬 실행 기반

복수 화면을 병렬로 디자인하는 모드:

1. 화면 목록과 공통 디자인 토큰을 정의
2. Swarm Mode 활성화 — 각 화면을 병렬로 생성
3. 결과를 리뷰하고 일관성 확인

**사용 시점:**
- 3개 이상의 화면을 디자인할 때
- 공통 레이아웃/컴포넌트를 공유하는 화면들

**주의:**
- 각 화면의 디자인 토큰이 일관적인지 수동 확인 필요
- 복잡한 인터랙션은 Swarm 후 개별 조정

---

## Keyboard Shortcuts

> Source: [Keyboard Shortcuts](https://docs.pencil.dev/core-concepts/keyboard-shortcuts)

### 프롬프트 / 명령

| Shortcut | Action |
|----------|--------|
| `Cmd/Ctrl + K` | AI 프롬프트 패널 열기 |
| `Cmd/Ctrl + Shift + P` | Command Palette |

### 선택

| Shortcut | Action |
|----------|--------|
| Click | 요소 선택 |
| `Cmd/Ctrl + Click` | Direct select (가장 깊은 요소) |
| `Shift + Click` | 선택에 추가 |
| `Cmd/Ctrl + A` | 전체 선택 |
| `Shift + Enter` | 상위 요소 선택 |
| `Cmd/Ctrl + Enter` | 상위 요소 선택 (대안) |

### 편집

| Shortcut | Action |
|----------|--------|
| `Cmd/Ctrl + C` | 복사 |
| `Cmd/Ctrl + V` | 붙여넣기 |
| `Cmd/Ctrl + X` | 잘라내기 |
| `Cmd/Ctrl + D` | 복제 |
| `Delete / Backspace` | 삭제 |
| `Cmd/Ctrl + G` | 그룹화 |
| `Cmd/Ctrl + Option/Alt + G` | 프레임 생성 |
| `Cmd/Ctrl + Option/Alt + K` | 재사용 컴포넌트로 변환 |

### 캔버스 네비게이션

| Shortcut | Action |
|----------|--------|
| `Spacebar + Drag` | 캔버스 팬 |
| `Shift + Scroll` | 가로 팬 |
| `Cmd/Ctrl + Scroll` | 줌 인/아웃 |
| `Cmd/Ctrl + 0` | 전체 맞춤 줌 |
| `Cmd/Ctrl + 1` | 100% 줌 |
| `Cmd/Ctrl + 2` | 200% 줌 |

### 파일

| Shortcut | Action |
|----------|--------|
| `Cmd/Ctrl + S` | 저장 |
| `Cmd/Ctrl + N` | 새 파일 |
| `Cmd/Ctrl + O` | 파일 열기 |

---

## CLI (Experimental)

> Source: [Pencil CLI](https://docs.pencil.dev/for-developers/pencil-cli)

### 설치

macOS/Linux: File 메뉴 → "Install `pencil` command into PATH" → 터미널 재시작.

### 실행

```bash
pencil --agent-config config.json
```

### Agent Config JSON 구조

```json
[
  {
    "file": "path/to/design.pen",
    "prompt": "Design a login screen with email and password fields",
    "model": "claude-4.5-sonnet",
    "attachments": ["specs/login.md", "assets/logo.png"]
  }
]
```

| Field | Required | Description |
|-------|----------|-------------|
| `file` | Yes | .pen 파일 경로 (절대 또는 실행 위치 기준 상대) |
| `prompt` | Yes | 실행할 디자인 프롬프트 |
| `model` | Yes | AI 모델 |
| `attachments` | No | 첨부 파일 배열 (.md, .jpg, .png 등) |

### 사용 가능 모델

| Model | Notes |
|-------|-------|
| `claude-4.5-haiku` | 빠름, 간단한 작업 |
| `claude-4.5-sonnet` | 균형 |
| `claude-4.6-opus` | 최고 품질, 복잡한 디자인 |

### 핵심 주의사항

- ⚠️ **.pen 파일을 미리 생성해야 함** — CLI가 파일을 자동 생성하지 않음
- 복수 config entry → **병렬로 Pencil 윈도우 열어 동시 실행**
- 데스크톱 앱 설치 필수 (CLI 단독 실행 불가)

### Headless 로드맵

현재 개발 중인 headless 버전 계획:
- 서버 사이드/에이전트 사용에 최적화된 최소 footprint
- CLI로 .pen 파일 직접 조작
- npm 패키지로 배포 예정

---

## File Organization

> Source: [.pen Files](https://docs.pencil.dev/core-concepts/pen-files)

### 배치 규칙

- `.pen` 파일은 **프로젝트 워크스페이스 내에 코드와 함께** 배치
- 서술적 파일명 사용: `dashboard.pen`, `components.pen`, `login.pen`
- IDE에서 일반 코드 파일처럼 열림

### Git 통합

- `.pen`은 JSON 기반 텍스트 포맷 → diff, merge, branch 가능
- Git commit을 버전 히스토리로 활용
- 정기적으로 커밋하여 진행 기록 유지

### Auto-save 없음

현재 auto-save 미지원. **`Cmd/Ctrl + S`로 수시 저장** 필수.

---

## Known Limitations

> Source: [Troubleshooting](https://docs.pencil.dev/troubleshooting)

| 제한사항 | 상세 | 대안 |
|----------|------|------|
| **Auto-save 없음** | 자동 저장 미구현 (향후 예정) | `Cmd/Ctrl + S`로 수시 저장, Git 커밋 활용 |
| **Undo/Redo 제한** | 일반 디자인 에디터보다 제한적 | 주요 변경 전 저장, 점진적 수정 |
| **실시간 협업 없음** | Git 기반 워크플로우만 지원 | 브랜치로 병렬 작업 후 PR 머지 |
| **Figma 이미지 미전송** | Figma 복사/붙여넣기 시 이미지 누락 | SVG 사용 또는 이미지 별도 import |
| **단축키 커스텀 불가** | 기본 단축키 변경 불가 | 기본 단축키 학습 |
| **Windows 데스크톱 앱 없음** | 익스텐션만 사용 가능 | VS Code / Cursor 익스텐션 |
| **CLI 파일 생성 불가** | CLI가 .pen 파일 자동 생성하지 않음 | 사전에 빈 .pen 파일 생성 |

---

## Troubleshooting

> Source: [Troubleshooting](https://docs.pencil.dev/troubleshooting)

### MCP 서버 연결 안됨

1. Pencil 앱 실행 중인지 확인
2. `.pen` 파일이 프로젝트에 있는지 확인
3. IDE별 연결 확인:
   - **Cursor**: Settings → Tools & MCP
   - **VS Code**: MCP 설정 확인
   - **Codex CLI**: `/mcp` 명령
4. Claude Code CLI 인증 완료 확인 (`claude` 실행)
5. IDE 재시작

### 인증 오류

| 증상 | 해결 |
|------|------|
| "Claude Code isn't connected" | 터미널에서 `claude` 실행 후 복귀 |
| "Invalid API key" | `ANTHROPIC_API_KEY` 환경 변수 제거, `claude` 재인증 |
| 활성화 코드 미수신 | 스팸 폴더 확인, 다른 이메일 시도 |
| 반복 활성화 요청 | 익스텐션 재설치 |

### Codex CLI 주의사항

> ⚠️ Pencil이 Codex의 `config.toml`을 수정하거나 복제할 수 있음. 첫 사용 전 `config.toml` 백업 권장.

### macOS 이미지 Import 실패

File 메뉴 대신 **드래그 앤 드롭** 사용.

### 캔버스-Export 불일치

캔버스와 내보낸 결과가 다를 경우 알려진 버그. 스크린샷 캡처 후 재시도하거나 디자인 조정.

### 폴더 접근 제한

Permission 프롬프트 수락, 시스템 폴더 권한 업데이트, IDE를 적절한 권한으로 실행.

### 요소 네비게이션 팁

- 중첩 요소 직접 선택: `Cmd/Ctrl + Click`
- 상위 요소로 이동: `Shift + Enter` 또는 `Cmd/Ctrl + Enter`
- Layers 패널에서 계층 구조 확인

### 버그 리포트 시 포함 정보

OS 버전, IDE 종류/버전, Pencil 버전, Claude Code CLI 버전, 에러 메시지, 재현 단계, 스크린샷, 콘솔 로그, 최소 `.pen` 파일.
