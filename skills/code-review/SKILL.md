---
name: si:code-review
user-invocable: true
description: "SI 정량 채점 특화 코드 리뷰 — 수용 기준 대조 매트릭스, 90/70/fail 판정, severity 분류"
---

# SI Code Review

SI 워크플로우의 Acceptance Phase를 위한 코드 리뷰 스킬. 수용 기준에 대한 정량 채점과 severity 분류를 수행하고, 최종 준수율 판정을 내린다.

## Core Principle

**주관적 "looks good"이 아닌 정량적 준수율로 판정한다.**

---

## Step 1: Review Scope Identification

**Git SHA 기반 리뷰 범위 확보:**
```bash
BASE_SHA=$(git rev-parse HEAD~1)  # 또는 origin/main
HEAD_SHA=$(git rev-parse HEAD)
git diff --stat ${BASE_SHA}..${HEAD_SHA}
git diff ${BASE_SHA}..${HEAD_SHA}
```

리뷰 대상 파악:

1. `tasks/requirements.md` — 요구사항 ID
2. `tasks/design.md` — 수용 기준, 컴포넌트, 인터페이스
3. `tasks/analysis.md` — 선택한 접근 방식
4. 변경된 파일 (`git diff ${BASE_SHA}..${HEAD_SHA}`로 확인)

---

## Step 2: Acceptance Criteria Scoring Matrix

`tasks/design.md`의 **모든** 수용 기준에 대해 채점:

| AC ID | Requirement | Test Status | Implementation Status | Verdict |
|-------|-------------|-------------|----------------------|---------|
| AC-001 | FR-001 | Pass/Fail/No Test | Complete/Partial/Missing | ✅/⚠️/❌ |

**Verdict Rules:**
- ✅ **Pass**: 테스트 통과 AND 구현이 설계와 일치
- ⚠️ **Partial**: 테스트 통과하지만 구현이 설계에서 이탈 OR 사소한 갭
- ❌ **Fail**: 테스트 실패 OR 구현 누락

---

## Step 3: Compliance Score

```
Compliance = (✅ count / total AC count) × 100

Score: [N]% ([✅ count]/[total])
```

| Score | Verdict | Action |
|-------|---------|--------|
| **90%+** | **PASS** | 배포/머지 준비 완료 |
| **70-89%** | **NEEDS IMPROVEMENT** | 식별된 갭 수정 후 재검증 |
| **<70%** | **REDESIGN** | Architect Phase로 복귀, 근본적 문제 |

---

## Step 4: Issue Review (Severity Classification)

변경된 코드를 리뷰하고 이슈를 severity로 분류:

### Critical (Must Fix)
버그, 보안 이슈, 데이터 손실 리스크, 깨진 기능

### Important (Should Fix)
아키텍처 문제, 누락된 기능, 부족한 에러 처리, 테스트 갭

### Minor (Nice to Have)
코드 스타일, 최적화 기회, 문서 개선

**각 이슈에 반드시 포함:**
- **File:line** 참조
- **무엇이** 잘못되었는지
- **왜** 중요한지
- **어떻게** 수정하는지 (명확하지 않은 경우)

---

## Step 5: Quality Checklist

각 항목을 확인하고 증거 기록:

### Behavior
- [ ] 모든 기능 요구사항 구현됨
- [ ] 모든 비기능 요구사항 충족 (또는 예외 문서화)
- [ ] 에러 처리가 설계 시나리오를 커버
- [ ] 설계의 엣지 케이스가 처리됨

### Testing
- [ ] 유닛 테스트 통과
- [ ] 통합 테스트 통과
- [ ] E2E 테스트 통과
- [ ] 테스트 불안정성(flaky) 없음

### Build & Tooling
- [ ] 프로젝트 빌드 에러 없음
- [ ] 린터 통과 (또는 문서화된 억제)
- [ ] 타입 체크 통과 (해당 시)
- [ ] 새 경고 없음

### Documentation & Code Quality
- [ ] 코드가 프로젝트 컨벤션을 따름 (analysis에서 확인)
- [ ] 공개 인터페이스가 설계 시그니처와 일치
- [ ] 디버그 코드나 임시 해킹 남지 않음
- [ ] 변경 범위가 Change Impact Map과 일치

### Production Readiness
- [ ] 마이그레이션 전략 (스키마 변경 시)
- [ ] 하위 호환성 고려됨
- [ ] 문서 완비
- [ ] 명백한 버그 없음

---

## Step 6: Deviation Report

준수율이 100% 미만이면 각 편차를 나열:

```markdown
### DEV-001: [AC ID] [Title]
- **Gap**: [무엇이 누락되었거나 다른지]
- **Severity**: Critical / Major / Minor
- **Recommendation**: [해결을 위한 구체적 행동]
```

---

## Step 7: Final Report

`tasks/acceptance-report.md`에 작성:

```markdown
# Acceptance Report: [Project Name]

## Summary
- **Date**: [ISO 8601]
- **Compliance Score**: [N]%
- **Verdict**: PASS / NEEDS IMPROVEMENT / REDESIGN

## Scoring Detail
[Step 2의 테이블]

## Issues Found

### Critical (Must Fix)
[이슈 목록 with file:line]

### Important (Should Fix)
[이슈 목록 with file:line]

### Minor (Nice to Have)
[이슈 목록 with file:line]

## Quality Checklist
[Step 5의 결과]

## Deviations
[Step 6, 있는 경우]

## Strengths
[잘된 점을 구체적으로. file:line 참조 포함.]

## Verification Evidence
- Test command: [command]
- Test results: [N/N passing]
- Build command: [command]
- Build result: [success/failure]
- Lint command: [command]
- Lint result: [N warnings, N errors]

## Recommendations
[코드 품질, 아키텍처, 프로세스 개선 사항]

## Sign-off
- [ ] 모든 수용 기준 검토됨
- [ ] 준수율 산출됨
- [ ] 품질 체크리스트 완료됨
- [ ] 편차 문서화됨 (있는 경우)
```

---

## Critical Rules

**DO:**
- 실제 severity로 분류 (모든 것이 Critical은 아님)
- 구체적으로 (file:line, 모호하지 않게)
- **왜** 이슈가 중요한지 설명
- 강점을 인정
- 명확한 판정 제시

**DON'T:**
- 확인하지 않고 "looks good" 말하기
- 사소한 것을 Critical로 표시
- 리뷰하지 않은 코드에 피드백
- 모호하게 ("에러 처리 개선")
- 명확한 판정 회피
- 주관적 "이 정도면 괜찮다"로 점수 부풀리기

---

## Review Red Flags

**절대 하지 않는다:**
- "간단해서" 리뷰를 건너뛰기
- Critical 이슈를 무시하기
- 수정하지 않은 Important 이슈가 있는 채 진행하기
- 유효한 기술적 피드백에 근거 없이 반박하기

**리뷰어가 틀렸을 때:**
- 기술적 근거로 반박한다
- 코드/테스트로 정상 동작을 증명한다
- 명확하지 않으면 설명을 요청한다

---

## Example Output

```
### Strengths
- Clean database schema with proper migrations (db.ts:15-42)
- Comprehensive test coverage (18 tests, all edge cases)
- Good error handling with fallbacks (summarizer.ts:85-92)

### Issues

#### Important
1. **Missing help text in CLI wrapper**
   - File: index-conversations:1-31
   - Issue: No --help flag, users won't discover --concurrency
   - Fix: Add --help case with usage examples

2. **Date validation missing**
   - File: search.ts:25-27
   - Issue: Invalid dates silently return no results
   - Fix: Validate ISO format, throw error with example

#### Minor
1. **Progress indicators**
   - File: indexer.ts:130
   - Issue: No "X of Y" counter for long operations
   - Impact: Users don't know how long to wait

### Assessment
**Compliance Score: 92% — PASS**
**Reasoning:** Core implementation is solid with good architecture and tests. Important issues are easily fixed and don't affect core functionality.
```
