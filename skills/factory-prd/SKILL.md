---
name: factory-prd
user-invocable: true
description: "인터뷰 기반 PRD — interview-then-plan 인터뷰 프로토콜 + product-manager PRD 구조 결합"
---

# Factory PRD

interview-then-plan의 인터뷰 규율과 product-manager의 PRD 구조를 결합하여, 구조화된 인터뷰 → Value Hypothesis → Scoped PRD → EARS 변환을 수행한다.

## Recommended Inputs

- `factory/research/research.md` — 있으면 참고 (선택)

## Why this skill exists

요구사항 정의에서 흔한 실패:
- **가정 기반 작성**: 사용자 의도를 대강 예상하고 PRD 작성 시작 → 방향 틀어짐
- **문제 정의 없는 솔루션**: "무엇을 만들지"만 있고 "왜 만드는지"가 없음
- **모호한 요구사항**: "사용자 친화적"처럼 측정 불가능한 표현

이 스킬의 원칙:
- **가정 금지** — 불명확한 것은 무조건 질문
- **확인 전 착수 금지** — 사용자 "맞아요" 없이 PRD 작성 안 함
- **문제 우선** — WHY/WHAT을 먼저 정의, HOW는 다루지 않음

---

## Scope Boundary

**This skill**: WHAT을 만들지, WHY 만드는지를 정의 — 요구사항, 사용자 스토리, 성공 지표.

**MUST NOT:**
- 기술 스택, 프레임워크, 라이브러리 지정 → `factory-architect`
- 컴포넌트 구조, API, 데이터 모델 설계 → `factory-architect`
- UI 레이아웃, 화면 흐름, 비주얼 디자인 정의 → `factory-ui-design`
- 구현 방법(HOW) 서술 — 요구사항은 동작만 정의
- Given/When/Then 인수 기준 작성 → `factory-architect` (PRD는 EARS만 사용)

**Output test**: 모든 요구사항은 "시스템이 무엇을 해야 하는가?"에 답해야 하며 "어떻게 만들어야 하는가?"에 답하지 않아야 한다. 요구사항에 클래스명, API 엔드포인트, UI 컴포넌트가 언급되면 경계 위반이다.

**경계 위반 시**: 기술적 내용을 Open Questions (섹션 9)으로 추출하고 "→ factory-architect" 또는 "→ factory-ui-design"으로 태그. 순수 EARS 동작 언어로 재작성.

---

## Phase 0: Initial Read

사전 자료 읽기:
- `factory/research/research.md` (Research 결과, 있으면)
- `factory/research/` (topic sub-reports, 있으면)

요청을 읽고 두 가지로 분류:

**명확한 것** (이미 알 수 있는 것)
**모호한 것** (아래 카테고리로 분류):
- **목적**: 왜 이걸 하려는가? 최종 목표는?
- **범위**: 어디까지? 어느 파일/기능/레이어?
- **제약**: 건드리면 안 되는 것, 유지해야 하는 것?
- **우선순위**: 여러 목표가 있을 때 순서는?
- **성공 기준**: 완료됐다는 걸 어떻게 알 수 있나?
- **맥락**: 이 작업의 배경, 이전에 시도했던 것?

---

## Phase 1: Mandatory Interview

**규칙:**
1. 모든 질문은 반드시 `AskUserQuestion` 도구를 사용 — 텍스트로만 출력하는 것은 금지
2. 한 번에 최대 4개 질문을 묶어서 보낼 수 있다 (배치 질문)
3. 각 질문마다 2~4개의 현실적인 선택지를 제공 — 사용자는 "Other"로 자유 입력 가능
4. 답변을 받으면 새로운 모호함이 생겼는지 확인, 있으면 다음 배치로 재질문
5. **"이 정도면 충분히 알겠다"고 AI가 판단해도 질문을 생략하지 않는다**
6. 사용자가 "그냥 알아서 해줘"라고 해도 핵심 항목(목적, 범위, 성공 기준)은 반드시 확인

### Core Questions (순서대로, 리서치에서 이미 답변된 것은 skip):

1. **사용자 관점**: "이 기능을 사용하는 사람은 누구이고, 어떤 상황에서 사용합니까?"
2. **핵심 행동**: "사용자가 반드시 할 수 있어야 하는 것은 무엇입니까? (최대 5개)"
3. **성공 기준**: "이 기능이 성공했다면, 무엇이 달라져 있습니까?"
4. **경계 조건**: "이 기능이 하지 않아야 할 것은 무엇입니까? (Non-goals)"
5. **비기능 요구**: "성능, 보안, 접근성 중 특별히 중요한 것이 있습니까?"
6. **기존 제약**: "변경 불가능한 것이 있습니까? (기존 API, DB 스키마, 외부 서비스 등)"

### "5 Whys" Purpose Deepening

Core Question #1 (사용자 관점) 답변 후, 목적이 추상적이면 최대 3단계 "왜?" 프로빙을 추가한다.

**메커니즘:**
- Round 1: Core Question #1 답변에서 목적 추출
- Round 2: "왜 [목적]이 필요한가요?" → 상위 목적 도출
- Round 3: "[상위 목적]이 어떤 비즈니스/사용자 결과를 만드나요?" → 측정 가능한 목적

**종료 조건:** 답변이 측정 가능한 비즈니스/사용자 결과에 도달하면 중단. 최대 3단계.

예시:
```
"로그인 기능 추가"
→ "왜 로그인이 필요한가요?" → "개인화된 경험을 제공하려고"
→ "개인화가 어떤 결과를 만드나요?" → "재방문율 30% 향상 목표"
→ 측정 가능한 목적 도출 완료 — 중단
```

### 질문 예시 (실제 코드):
```
AskUserQuestion(questions=[
  {
    question: "이 작업의 주요 목적은 무엇인가요?",
    header: "목적",
    multiSelect: false,
    options: [
      { label: "새 기능 추가", description: "기존에 없던 기능을 새로 만든다" },
      { label: "기존 기능 수정", description: "이미 있는 기능의 동작을 바꾼다" },
      { label: "리팩토링", description: "동작은 유지하고 코드 구조를 개선한다" },
      { label: "버그 수정", description: "잘못된 동작을 고친다" }
    ]
  },
  {
    question: "작업 범위는 어디까지인가요?",
    header: "범위",
    multiSelect: false,
    options: [
      { label: "단일 파일", description: "특정 파일 1개" },
      { label: "특정 기능/모듈", description: "연관된 파일 묶음" },
      { label: "전체 프로젝트", description: "프로젝트 전반에 걸쳐" }
    ]
  }
])
```

**선택지 생성 원칙:**
- 요청 맥락에서 가장 가능성 높은 답변 2~4개를 선택지로 제시
- 선택지가 불분명하거나 자유 서술이 필요한 경우 → 선택지를 줄이고 description에 힌트 제공
- 사용자는 항상 "Other"로 직접 입력 가능하므로 선택지를 억지로 맞출 필요 없음

### Scenario Walkthrough

Core Questions 답변 후, 구체적 사용 시나리오를 통해 숨겨진 요구사항을 발견한다.

```
AskUserQuestion(questions=[
  {
    question: "이 기능을 사용하는 가장 일반적인 시나리오를 단계별로 설명해주세요.",
    header: "시나리오",
    multiSelect: false,
    options: [
      { label: "직접 설명할게요", description: "단계별로 사용 흐름을 서술합니다 (Other로 입력)" },
      { label: "기존 유사 기능과 비슷해요", description: "어떤 기능인지 후속 질문으로 확인" },
      { label: "아직 구체적 시나리오는 없어요", description: "함께 시나리오를 만들어봅니다" }
    ]
  }
])
```

**시나리오에서 암묵적 요구사항 추출:**
- 사용자가 서술한 각 단계에서 전제 조건, 상태 전이, 필요한 UI 요소를 식별
- 예: "로그인하고 → 대시보드에서 → 프로필 클릭" → 네비게이션 구조, 인증 상태 관리, 라우팅이 숨겨진 요구사항
- 추출된 암묵적 요구사항은 Follow-up Questions에서 사용자에게 확인

### Follow-up Questions (맥락 의존):
- 모호한 답변 → "구체적으로 [X]는 어떤 경우를 말합니까?"
- 누락된 엣지 케이스 → "[Y] 상황에서는 어떻게 동작해야 합니까?"
- 충돌하는 요구사항 → "[A]와 [B]가 충돌합니다. 우선순위는?"

### Negative Scenario Probing

핵심 기능(Core Question #2 답변)에 대해 부정 시나리오를 체계적으로 탐색한다.

```
AskUserQuestion(questions=[
  {
    question: "[핵심 기능]이 실패하면 어떻게 되어야 하나요?",
    header: "실패 처리",
    multiSelect: false,
    options: [
      { label: "에러 메시지 표시", description: "사용자에게 알리고 재시도 유도" },
      { label: "조용히 무시", description: "백그라운드에서 실패, 사용자는 모름" },
      { label: "폴백 동작", description: "대안 경로로 자동 전환" },
      { label: "치명적 오류", description: "작업 중단, 즉시 보고" }
    ]
  },
  {
    question: "잘못된 입력이 들어오면?",
    header: "입력 검증",
    multiSelect: false,
    options: [
      { label: "즉시 거부", description: "입력 시점에 바로 검증" },
      { label: "제출 시 검증", description: "모든 입력 후 한 번에 검증" },
      { label: "자동 보정", description: "가능한 범위에서 자동 수정" }
    ]
  }
])
```

이 답변을 PRD의 Functional Requirements에 에러/실패 처리 FR로 반영한다.

### Progressive Assumption Surfacing

매 인터뷰 배치 응답 처리 후, 현재까지의 가정을 3-column 형식으로 사용자에게 표시한다.

```
## 지금까지 제가 이해한 것
✅ 확인된 것:
- [인터뷰에서 명시적으로 확인된 항목]

⚠️ 제가 가정한 것 (아직 미확인):
- [답변에서 유추했지만 명시적 확인이 없는 항목]

❓ 아직 모르는 것:
- [아직 질문하지 않았거나 답변이 불충분한 항목]
```

**규칙:**
- 별도 도구 호출 불필요 — 텍스트 출력 후 다음 배치 질문을 이어서 진행
- ⚠️ 항목에 대해 사용자가 즉시 수정할 기회를 제공
- ❓ 항목은 다음 배치 질문에 우선 반영

### Contradiction Detection:
인터뷰 답변 간 모순을 적극 탐지한다:
- Non-Goal과 Success Metric이 충돌하는 경우 (예: "단순하게" vs "모든 케이스 커버")
- 제약사항과 기능 요구사항이 상충하는 경우
- 모순 발견 시 즉시 사용자에게 제시하고 우선순위 결정을 요청

### Trade-off Matrix

요구사항이 5개 이상일 때, Phase 2 진입 전에 명시적 트레이드오프를 확인한다.

```
AskUserQuestion(questions=[
  {
    question: "다음 중 가장 양보할 수 없는 것은?",
    header: "우선순위",
    multiSelect: false,
    options: [
      { label: "기능 완전성", description: "모든 기능이 포함되어야 함" },
      { label: "성능/속도", description: "빠른 응답이 가장 중요" },
      { label: "구현 속도", description: "빨리 완성하는 것이 최우선" },
      { label: "코드 품질", description: "유지보수성과 확장성 우선" }
    ]
  }
])
```

이 답변을 PRD의 Priority 섹션과 Non-Goals 섹션에 직접 반영한다.

### Completeness Gate (Phase 2 진입 조건)

**종료 조건:** 모든 카테고리에서 모호한 항목이 0개가 되면 아래 체크리스트를 자체 검증한다.

```
## PRD 인터뷰 완전성 체크
- [ ] 목적이 측정 가능한 결과로 표현되었는가?
- [ ] 주요 사용자 시나리오가 1개 이상 서술되었는가?
- [ ] 비목표(Non-goals)가 명시되었는가?
- [ ] 성공 기준에 수치가 포함되었는가?
- [ ] 실패/에러 케이스가 논의되었는가?
- [ ] 기존 제약사항이 확인되었는가?
- [ ] 가정 목록에 미확인 항목(⚠️)이 남아있지 않은가?
```

**미통과 항목이 있으면** 해당 카테고리에 대해 추가 질문 후 재검증.
**모두 통과하면** Phase 2로 이동.

---

## Phase 2: Brief & Confirmation

인터뷰 결과를 구조화된 브리프로 정리하여 사용자에게 제시:

```
PROJECT BRIEF
══════════════════════════════════════

목적
[왜 이 작업을 하는가]

범위
- In scope: [포함되는 것]
- Out of scope: [포함되지 않는 것]

제약사항
[건드리면 안 되는 것, 지켜야 하는 것]

성공 기준
[완료됐다는 걸 어떻게 판단하는가]

우선순위
1. [가장 중요한 것]
2. [그 다음]
```

`AskUserQuestion`으로 확인:

```
AskUserQuestion(questions=[
  {
    question: "위 브리프가 의도와 맞나요?",
    header: "브리프 확인",
    multiSelect: false,
    options: [
      { label: "맞아요, 진행해요", description: "브리프 확정 후 PRD 작성으로 이동" },
      { label: "일부 수정 필요해요", description: "수정할 항목을 말씀해주세요" },
      { label: "처음부터 다시 해요", description: "인터뷰를 다시 시작한다" }
    ]
  }
])
```

- "맞아요, 진행해요" → Phase 3
- "일부 수정 필요" → 해당 항목만 재질문 후 브리프 업데이트, 다시 확인
- "처음부터 다시" → Phase 1로 복귀

---

## Phase 3: PRD Synthesis (product-manager 구조)

확정된 브리프를 바탕으로 PRD를 작성한다. `factory/prd/prd.md`에 출력.

### 3.1 Value Hypothesis

```
IF we [intervention],
THEN [user outcome],
BECAUSE [mechanism].
```

반드시 반증 가능(falsifiable)해야 한다. "사용자가 좋아할 것이다"는 가설이 아님.

### 3.2 Scoped PRD

```markdown
# Requirements: [Project Name]

## 1. Overview
[2-3 paragraphs — Problem & Context]

## 2. User Persona & JTBD
### Persona: [Name/Role]
- Key characteristics:
- Jobs-to-be-done: [When I..., I want to..., So I can...]

### Value Hypothesis
IF we [intervention], THEN [outcome], BECAUSE [mechanism].

## 3. User Stories
### US-001: [Title]
As a [role], I want to [action], so that [benefit].

## 4. Functional Requirements
### FR-001: [Title]
[EARS pattern]
- Priority: Must / Should / Could
- Acceptance criteria: Given/When/Then (brief)

## 5. Non-Functional Requirements
### NFR-001: [Title]
[EARS pattern]
- Metric: [measurable criterion]

## 6. Non-Goals (Explicit Exclusions)
- NG-001: [what this project will NOT do]

## 7. Constraints
- C-001: [immovable constraint from interview]

## 8. Success Metrics
| Metric | Current | Target | Measurement |
|--------|---------|--------|-------------|

## 9. Open Questions
- OQ-001: [unresolved question, to be addressed in Analysis/Architect]

## 10. Glossary
| Term | Definition |
|------|-----------|
```

### 3.3 EARS Format (Mandatory)

모든 요구사항(FR, NFR)은 반드시 EARS 패턴을 사용. Reference: `settings/rules/ears-format.md`

5가지 패턴:
1. **Ubiquitous**: "The [system] shall [action]."
2. **Event-Driven**: "When [event], the [system] shall [action]."
3. **State-Driven**: "While [state], the [system] shall [action]."
4. **Optional**: "Where [feature is enabled], the [system] shall [action]."
5. **Unwanted**: "If [condition], then the [system] shall [action] (to prevent [consequence])."

규칙: 요구사항당 패턴 하나, 정량화 (시간/횟수/크기), 모호한 단어 금지 ("appropriate", "reasonable", "user-friendly"), 요구사항당 행동 하나.

---

## Phase 4: User Review

요구사항 문서 요약을 사용자에게 제시:
- 모든 FR/NFR을 ID와 우선순위와 함께 나열
- 만든 가정을 강조
- "요구사항을 검토해주세요. 추가, 수정, 삭제할 항목이 있습니까?"

사용자가 승인할 때까지 반복.

---

## Core Principles

| 원칙 | 내용 |
|------|------|
| 가정 금지 | 불명확한 것은 항상 질문 |
| 확인 전 착수 금지 | "맞아요" 없이 PRD 작성 안 함 |
| 배치 질문 | 관련 질문은 묶어서 한 번에 |
| 문제 우선 | WHY/WHAT을 먼저, HOW는 다루지 않음 |
| 반증 가능한 가설 | Value Hypothesis는 증거로 부정 가능해야 함 |
| Not-doing 필수 | 범위에서 제외하는 것이 포함하는 것만큼 중요 |
| EARS 강제 | 자유 형식 요구사항 금지 |

---

## Failure Modes to Watch

- **문제 정의 없이 솔루션 나열**: "무엇을 만들지"만 있고 "왜"가 없음
- **측정 불가능한 성공 기준**: "사용자 경험 향상" 같은 표현
- **Not-doing 리스트 누락**: 범위 확장 방지 실패
- **가설 없는 기능**: 누구의 어떤 문제를 해결하는지 불명확
- **EARS 위반**: 자유 형식으로 요구사항 작성
- **부정 시나리오 누락**: 성공 경로만 정의하고 실패/에러 처리 미정의

---

## Output

- `factory/prd/prd.md` — 최종 PRD

PRD가 완료되었습니다. 결과는 `factory/prd/prd.md`에 저장되었습니다.
