---
name: si-prd
user-invocable: true
description: "인터뷰 기반 PRD — interview-then-plan 인터뷰 프로토콜 + product-manager PRD 구조 결합"
---

# SI PRD Skill

interview-then-plan의 인터뷰 규율과 product-manager의 PRD 구조를 결합하여, 구조화된 인터뷰 → Value Hypothesis → Scoped PRD → EARS 변환을 수행한다.

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

## Phase 0: Initial Read

사전 자료 읽기:
- `tasks/research-report.md` (Research phase 산출물, 있으면)
- `tasks/research/` (topic sub-reports, 있으면)
- `tasks/si-progress.json` (프로젝트 컨텍스트)

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

### Follow-up Questions (맥락 의존):
- 모호한 답변 → "구체적으로 [X]는 어떤 경우를 말합니까?"
- 누락된 엣지 케이스 → "[Y] 상황에서는 어떻게 동작해야 합니까?"
- 충돌하는 요구사항 → "[A]와 [B]가 충돌합니다. 우선순위는?"

### Contradiction Detection:
인터뷰 답변 간 모순을 적극 탐지한다:
- Non-Goal과 Success Metric이 충돌하는 경우 (예: "단순하게" vs "모든 케이스 커버")
- 제약사항과 기능 요구사항이 상충하는 경우
- 모순 발견 시 즉시 사용자에게 제시하고 우선순위 결정을 요청

**종료 조건:** 모든 카테고리에서 모호한 항목이 0개가 되면 Phase 2로 이동.

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

확정된 브리프를 바탕으로 PRD를 작성한다. `tasks/requirements.md`에 출력.

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

## Output

- `tasks/requirements.md` — final PRD

### Sub-reports (Optional)
중간 산출물이나 상세 분석이 있으면 `tasks/prd/`에 개별 파일로 저장.
최종 통합 파일은 `tasks/requirements.md`에 작성.
서브리포트 경로는 `tasks/si-progress.json`의 `artifacts` 배열에 추가.

---

## Failure Modes to Watch

- **문제 정의 없이 솔루션 나열**: "무엇을 만들지"만 있고 "왜"가 없음
- **측정 불가능한 성공 기준**: "사용자 경험 향상" 같은 표현
- **Not-doing 리스트 누락**: 범위 확장 방지 실패
- **가설 없는 기능**: 누구의 어떤 문제를 해결하는지 불명확
- **EARS 위반**: 자유 형식으로 요구사항 작성
