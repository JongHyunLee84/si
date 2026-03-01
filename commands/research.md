---
name: research
description: 시장/기술 리서치 — Deep research 패턴으로 체계적 조사
user_invocable: true
---

# SI Research Phase

You are conducting structured research for the SI workflow. Your goal is to produce a comprehensive but focused research report that informs the PRD phase.

## Input
- User's project idea or problem statement
- Any URLs, references, or constraints provided

## Research Protocol

### Step 1: Scope Definition
Ask the user (if not already clear):
1. "이 프로젝트가 해결하려는 핵심 문제는 무엇입니까?"
2. "타겟 사용자/환경은?" (플랫폼, 규모, 기술 스택 제약)
3. "이미 알고 있는 기술적 제약이 있습니까?"

### Step 2: Multi-angle Research
Use WebSearch and WebFetch to investigate:

**시장 조사** (필요한 경우):
- 유사 솔루션 / 경쟁 제품 확인
- 기존 접근 방식의 장단점

**기술 조사** (항상 수행):
- 관련 기술 / 프레임워크 / 라이브러리 최신 상태
- Best practices, known pitfalls
- API 시그니처, 레이트 리밋, 버전 호환성 (외부 의존성이 있는 경우)

**아키텍처 참고** (규모가 큰 경우):
- 유사 규모 프로젝트의 아키텍처 패턴
- 확장성, 유지보수성 관련 고려사항

### Step 3: PoC (Proof of Concept) — Optional
If the user requests or if technical feasibility is uncertain:
- Create a minimal spike to validate the core technical assumption
- Document what was proved/disproved

### Step 4: Research Report
Write to `tasks/research-report.md`:

```markdown
# Research Report: [Project Name]

## 1. Problem Statement
[1-2 paragraphs]

## 2. Market/Competition Analysis
| Solution | Pros | Cons | Relevance |
|----------|------|------|-----------|
| | | | |

## 3. Technical Landscape
### Recommended Stack
| Layer | Technology | Reason | Risk |
|-------|-----------|--------|------|
| | | | |

### Key Technical Findings
- [finding 1]
- [finding 2]

### Known Pitfalls
- [pitfall 1: mitigation]

## 4. PoC Results (if conducted)
- Hypothesis:
- Result:
- Conclusion:

## 5. Feasibility Assessment
- Technical feasibility: High / Medium / Low
- Effort estimate: S / M / L / XL
- Key risks:
  1. [risk: mitigation]

## 6. Recommendation
[2-3 sentences: proceed or pivot, and why]

## Sources
- [url1]
- [url2]
```

### Step 5: Update Progress
Update `tasks/si-progress.json`:
- Set `phases.research.status = "completed"`
- Add `tasks/research-report.md` to artifacts
- Set `completedAt` to current timestamp

Then inform the user: "리서치가 완료되었습니다. `/si:start`로 돌아가서 다음 단계를 확인하세요."
