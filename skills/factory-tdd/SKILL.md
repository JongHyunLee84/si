---
name: factory-tdd
user-invocable: true
description: "테스트 주도 개발 — Red → Green → Refactor + 수용 기준 기반 테스트 플랜 자동 추출"
---

# Factory TDD

SI 워크플로우의 TDD Phase를 위한 엄격한 Red-Green-Refactor 스킬. 수용 기준(Given/When/Then)에서 테스트 플랜을 추출하고, AC별로 사이클을 트래킹한다.

## Recommended Inputs

- `factory/architect/architect.md` — 수용 기준(AC) 추출 (필수)
- `factory/ui-design/ui-design.md` — UI 수용 기준, 레이아웃 스펙 (선택)
- `factory/analysis/analysis.md` — 선택한 접근 방식, 파일 경로 (권장)
- `factory/prd/prd.md` — 트레이서빌리티용 요구사항 ID (권장)

Read these files BEFORE writing any code (Read-first principle).

## Scope Boundary

**This phase**: Write tests for acceptance criteria and minimal code to make them pass (R-G-R). The Iron Law governs HOW; this section governs WHAT.

**MUST NOT:**
- Change acceptance criteria or design decisions → `factory-architect` (flag for review if criteria seem wrong)
- Implement beyond what acceptance criteria require — scope is `factory/architect/architect.md` section 8 only
- Refactor code unrelated to current acceptance criteria → log as TODO for `factory-develop`
- Add dependencies not in the design → `factory-architect` must approve
- Write E2E tests → `factory-e2e` (mark as "deferred to factory-e2e")

**Scope test**: Every line of code must link to an AC-ID from the design. Code without an AC-ID traceability link is out of scope.

**When boundary is crossed**: STOP the R-G-R cycle. Design gap → "→ factory-architect". Scope expansion → log as TODO, continue with current AC.

## When to Use

**항상:**
- 새 기능
- 버그 수정
- 리팩토링
- 동작 변경

**예외 (사용자에게 확인):**
- 일회성 프로토타입
- 생성된 코드
- 설정 파일

"이번만 TDD 건너뛰자"? 멈춰라. 그건 합리화다.

---

## The Iron Law

```
NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST
```

테스트 전에 코드를 작성했다면? 삭제하고 처음부터 다시.

**예외 없음:**
- "참고용"으로 보관하지 않는다
- "수정하면서" 테스트를 쓰지 않는다
- 보지도 않는다
- 삭제는 삭제다

테스트로부터 새로 구현한다. 마침표.

**이 규칙의 문자를 어기는 것이 정신을 어기는 것이다.**

---

## Step 1: Extract Test Plan from Acceptance Criteria

`factory/architect/architect.md`의 수용 기준(Given/When/Then) section 8에서 테스트 플랜을 추출한다.

| AC ID | Requirement | Test Type | Test File | Status |
|-------|-------------|-----------|-----------|--------|
| AC-001 | FR-001 | Unit / Integration / E2E | [path] | Pending |
| AC-002 | FR-002 | Unit / Integration / E2E | [path] | Pending |

**Test Type Selection:**
- 순수 로직, 데이터 변환 → **Unit test**
- DB, API, 파일 I/O → **Integration test**
- 전체 사용자 흐름 → **E2E** (factory-e2e phase로 연기)

---

## Step 2: Red-Green-Refactor Cycles

각 수용 기준에 대해 우선순위 순으로:

### RED — Write Failing Test

1. 프로젝트 컨벤션에 맞는 테스트 파일 생성
2. Given/When/Then 시나리오에 직접 매핑되는 테스트 작성
3. 테스트 실행 — **반드시 FAIL 확인**
4. 테스트가 즉시 통과하면 → 기능이 이미 존재하거나 테스트가 잘못됨. 조사 필요.

**좋은 테스트:**
```typescript
test('retries failed operations 3 times', async () => {
  let attempts = 0;
  const operation = () => {
    attempts++;
    if (attempts < 3) throw new Error('fail');
    return 'success';
  };

  const result = await retryOperation(operation);

  expect(result).toBe('success');
  expect(attempts).toBe(3);
});
```
명확한 이름, 실제 동작 테스트, 하나의 행동.

**나쁜 테스트:**
```typescript
test('retry works', async () => {
  const mock = jest.fn()
    .mockRejectedValueOnce(new Error())
    .mockRejectedValueOnce(new Error())
    .mockResolvedValueOnce('success');
  await retryOperation(mock);
  expect(mock).toHaveBeenCalledTimes(3);
});
```
모호한 이름, 모의 동작 테스트.

**요구사항:** 하나의 행동, 명확한 이름, 실제 코드 (불가피한 경우만 mock).

### Verify RED — Watch It Fail

**필수. 절대 생략 불가.**

확인:
- 테스트가 실패함 (에러가 아님)
- 실패 메시지가 예상대로임
- 기능이 없어서 실패함 (오타 아님)

**테스트가 통과하면?** 기존 동작을 테스트하고 있다. 테스트를 수정.
**테스트가 에러?** 에러를 수정하고, 올바르게 실패할 때까지 재실행.

### GREEN — Minimal Code

테스트를 통과시키는 **최소한의** 코드를 작성.

- 최적화 없음, 정리 없음, 추가 기능 없음
- 테스트 실행 — **PASS 확인**
- 기존 모든 테스트 실행 — **회귀 없음 확인**

**테스트 실패?** 코드를 수정, 테스트를 수정하지 않는다.
**다른 테스트 실패?** 지금 수정.

### REFACTOR — Clean Up

GREEN 이후에만:
- 중복 제거
- 이름 개선
- 헬퍼 추출

모든 테스트가 녹색으로 유지되어야 한다. 새 동작을 추가하지 않는다.
리팩토링이 동작을 바꾸면 → **멈추고, 되돌리고, 다시 접근.**

---

## Step 3: Track Cycles

각 R-G-R 사이클 후 테스트 플랜 업데이트:

```
AC-001: ✅ Red → Green → Refactor (test: path/to/test.ts:42)
AC-002: 🔴 Red (writing test...)
AC-003: ⏳ Pending
```

---

## Step 4: Edge Cases & Error Paths

모든 수용 기준에 통과하는 테스트가 있은 후:
1. `factory/architect/architect.md`의 Error Handling 섹션 검토
2. 각 에러 시나리오에 대한 테스트 작성
3. R-G-R로 에러 처리 구현

---

## Step 5: Regression Guard

전체 테스트 스위트 실행:

```
[project-specific test command]
```

결과 보고:
- Total tests: N
- Passing: N
- Failing: N (각각 file:line 포함)
- New tests added this phase: N

---

## Good Tests

| 품질 기준 | Good | Bad |
|-----------|------|-----|
| **Minimal** | 하나만 테스트. 이름에 "and"가 있으면? 분리. | `test('validates email and domain and whitespace')` |
| **Clear** | 이름이 동작을 설명 | `test('test1')` |
| **Shows intent** | 원하는 API를 보여줌 | 코드가 무엇을 해야 하는지 모호하게 함 |

---

## Rules

1. **한 번에 하나의 사이클** — 하나의 AC에 대한 R-G-R을 완료한 후 다음으로
2. **테스트 이름은 동작을 설명** — 구현이 아님 (`"should display error when login fails"` not `"test handleLoginError"`)
3. **소유한 것을 mock하지 않는다** — 외부 의존성만 mock
4. **가장 작은 assertion** — 테스트당 하나의 논리적 assertion
5. **테스트가 문서다** — 테스트를 읽으면 기능을 이해할 수 있어야 한다

---

## Testing Anti-Patterns (인라인 참조)

### Anti-Pattern 1: Mock 동작 테스트
```typescript
// ❌ BAD: Mock이 존재하는지 테스트
test('renders sidebar', () => {
  render(<Page />);
  expect(screen.getByTestId('sidebar-mock')).toBeInTheDocument();
});
```
Mock의 존재를 확인하는 것은 실제 동작을 테스트하지 않는다.
→ 실제 컴포넌트를 테스트하거나 mock을 제거.

### Anti-Pattern 2: 프로덕션 코드에 테스트 전용 메서드
```typescript
// ❌ BAD: destroy()가 테스트에서만 사용됨
class Session {
  async destroy() { /* ... */ }
}
```
→ 테스트 유틸리티로 이동. 프로덕션 클래스를 오염시키지 않는다.

### Anti-Pattern 3: 의존성 이해 없는 Mock
```typescript
// ❌ BAD: Mock이 테스트가 의존하는 사이드 이펙트를 제거
vi.mock('ToolCatalog', () => ({
  discoverAndCacheTools: vi.fn().mockResolvedValue(undefined)
}));
```
→ 실제 구현으로 먼저 테스트를 실행하고, 무엇이 필요한지 관찰한 후 최소한으로 mock.

### Anti-Pattern 4: 불완전한 Mock
```typescript
// ❌ BAD: 실제 API 응답의 일부 필드만 mock
const mockResponse = { status: 'success', data: { userId: '123' } };
// metadata.requestId를 사용하는 다운스트림 코드에서 실패
```
→ 실제 API 응답의 전체 구조를 미러링.

### Gate Function (Mock 사용 전 확인)
```
BEFORE mocking:
  1. "실제 메서드의 사이드 이펙트는?"
  2. "이 테스트가 그 사이드 이펙트에 의존하는가?"
  3. "내가 이 테스트에 무엇이 필요한지 완전히 이해했는가?"

  의존한다면 → 실제 느린/외부 연산만 mock, 고수준 메서드 아님
  확실하지 않으면 → 실제 구현으로 먼저 실행, 관찰, 그 후 최소 mock
```

---

## Common Rationalizations — 전부 거짓

| 변명 | 현실 |
|------|------|
| "너무 단순해서 테스트 불필요" | 단순한 코드도 깨진다. 테스트에 30초. |
| "나중에 테스트 쓸게" | 나중에 쓴 테스트는 즉시 통과. 즉시 통과는 아무것도 증명 안 함. |
| "이미 수동으로 테스트했어" | 임시 ≠ 체계적. 기록 없음, 재실행 불가. |
| "X시간 작업 삭제는 낭비" | 매몰 비용 오류. 검증 안 된 코드 유지가 기술 부채. |
| "참고로 남기고 테스트부터" | 참고를 보면 적응하게 됨. 그건 tests-after. 삭제는 삭제. |
| "먼저 탐색이 필요해" | 탐색 좋다. 그 다음 버리고 TDD로 시작. |
| "테스트 어려움 = 설계 불명확" | 테스트가 말해주는 것을 들어라. 테스트하기 어려움 = 사용하기 어려움. |
| "TDD가 속도를 늦춰" | TDD가 디버깅보다 빠르다. 실용적 = test-first. |
| "TDD는 교조적, 나는 실용적" | TDD가 실용적임. 버그를 커밋 전에 발견. |
| "tests-after도 같은 목적" | tests-after = "이게 뭘 하지?" tests-first = "이게 뭘 해야 하지?" |

---

## Red Flags — 멈추고 처음부터 다시

- 테스트 전에 코드 작성
- 구현 후 테스트 추가
- 테스트가 즉시 통과
- 왜 테스트가 실패했는지 설명 못함
- "나중에" 테스트 추가
- "이번만" 합리화
- "이미 수동 테스트했어"
- "참고로 남기기" 또는 "기존 코드 적응"
- "X시간 작업했는데 삭제는 낭비"

**이 모든 것은: 코드를 삭제하고 TDD로 다시 시작하라는 의미.**

---

## Verification Checklist

작업 완료 전:

- [ ] 모든 새 함수/메서드에 테스트 있음
- [ ] 각 테스트가 실패하는 것을 확인함
- [ ] 각 테스트가 예상 이유로 실패함 (기능 부재, 오타 아님)
- [ ] 각 테스트를 통과시키는 최소 코드 작성
- [ ] 모든 테스트 통과
- [ ] 출력 깨끗 (에러, 경고 없음)
- [ ] 테스트가 실제 코드 사용 (불가피한 경우만 mock)
- [ ] 엣지 케이스와 에러 커버됨

체크박스를 다 못 채우면? TDD를 건너뛴 것. 다시 시작.

---

## When Stuck

| 문제 | 해결 |
|------|------|
| 어떻게 테스트할지 모르겠음 | 원하는 API를 먼저 작성해라. assertion부터 써라. 사용자에게 물어라. |
| 테스트가 너무 복잡 | 설계가 너무 복잡한 것. 인터페이스를 단순화. |
| 모든 것을 mock 해야 함 | 코드가 너무 결합됨. 의존성 주입을 사용. |
| 테스트 설정이 거대 | 헬퍼를 추출. 여전히 복잡하면? 설계를 단순화. |

---

## Debugging Integration

버그 발견? 버그를 재현하는 실패 테스트를 먼저 작성. TDD 사이클을 따른다. 테스트가 수정을 증명하고 회귀를 방지한다.

**테스트 없이 버그를 수정하지 않는다.**

---

## Example: Bug Fix (전체 워크드 예제)

**버그:** 빈 이메일이 허용됨

**RED**
```typescript
test('rejects empty email', async () => {
  const result = await submitForm({ email: '' });
  expect(result.error).toBe('Email required');
});
```

**Verify RED**
```bash
$ npm test
FAIL: expected 'Email required', got undefined
```

**GREEN**
```typescript
function submitForm(data: FormData) {
  if (!data.email?.trim()) {
    return { error: 'Email required' };
  }
  // ...
}
```

**Verify GREEN**
```bash
$ npm test
PASS
```

**REFACTOR**
여러 필드에 대한 검증이 필요하면 추출.

## Output

- Test files (per project conventions)
- `factory/tdd/` — TDD session notes, test plan tracking (optional)

## Completion

TDD가 완료되었습니다. 모든 수용 기준에 대한 테스트가 통과합니다.
