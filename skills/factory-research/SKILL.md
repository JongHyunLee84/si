---
name: factory-research
user-invocable: true
description: "시장/기술 리서치 — Single Topic Mode(단일 심층 리서치) + Multi Topic Mode(인터뷰 + 토픽 분해 + 병렬 Agent + 종합)"
compatibility:
  required_tools:
    - WebFetch
    - WebSearch
  recommended_tools:
    - name: mcp__scrapling
      purpose: "Stealthy fetching, bulk fetching, bot bypass"
    - name: mcp__firecrawl
      purpose: "Site mapping, batch scraping, web search"
    - name: mcp__claude-in-chrome
      purpose: "Real browser fallback"
    - name: Agent
      purpose: "Parallel subagents for bulk research"
---

# Factory Research

SI 워크플로우의 Research Phase를 위한 통합 리서치 스킬. 단일 토픽 심층 리서치(Single Topic Mode)와 다중 토픽 병렬 리서치(Multi Topic Mode)를 하나의 스킬에서 처리한다.

## Recommended Inputs

없음. 이 스킬은 독립적으로 실행된다.

## Reference Files

도구별 상세 정보가 필요할 때 참조:

- **`references/setup.md`** — MCP 도구 설치 가이드 (Scrapling, Firecrawl, Chrome). 도구 설정이나 가용성 문제 해결 시 참조.
- **`references/scrapling.md`** — Scrapling MCP 6개 도구: 파라미터, 안티봇 티어, 결정 매트릭스, 주의사항. `get`/`fetch`/`stealthy_fetch` 선택 시 참조.
- **`references/firecrawl.md`** — Firecrawl MCP 11개 도구: 파라미터, 크레딧 비용, 결정 플로차트. `firecrawl_map`, `firecrawl_scrape`, `firecrawl_search` 사용 시 참조.

모든 참조를 미리 읽을 필요 없음. 특정 결정이 필요할 때 참조 (예: "Cloudflare 우회에 어떤 scrapling 도구?" → scrapling.md 읽기).

---

## Execution Modes

이 스킬은 두 가지 모드로 동작한다:

### Single Topic Mode
특정 토픽 하나를 깊이 리서치할 때 사용.

**진입 조건 (Auto-detect)**: 사용자가 구체적인 토픽을 명시하거나, caller가 `topic`, `focus_questions`, `output_path`를 제공한 경우.

**사용자 override**: `/factory-research --single` 또는 `/factory-research --multi`로 명시적 지정 가능.

**Workflow**: Step 0 (Scope Definition) → Step 1-4 (Discover → Fetch → Read → Synthesize)

### Multi Topic Mode
넓은 범위의 리서치를 토픽으로 분해하여 병렬 처리할 때 사용.

**진입 조건 (Auto-detect)**: 사용자가 프로젝트나 문제 전반에 대한 리서치를 요청하거나 범위가 넓은 경우.

**Workflow**: Step A (Interview) → Step B (Topic Proposal) → Step C (Approval) → Step D (Parallel Agents) → Step E (Verify) → Step F (Synthesize)

각 Agent는 이 스킬의 **Single Topic Mode**를 따른다.

---

## Scope Boundary

**This skill**: 정보를 수집하고 발견한 사실을 정리한다 — 결정이 아닌 사실.

**MUST NOT:**
- 요구사항이나 사용자 스토리를 정의 → `factory-prd`
- 특정 기술 스택이나 아키텍처 권장 → `factory-architect`
- 솔루션 설계나 컴포넌트 구조 제안 → `factory-architect`
- 코드 작성 → `factory-develop`

**Output test**: 모든 문장은 **사실 또는 관찰**이어야 하며 **처방**이 아니어야 한다. "We should use X"는 위반 → "[Option X]는 [tradeoff Y]를 제공한다 (source: Z)"로 재작성.

**경계 위반 시**: 처방적 내용을 리포트 끝 `Open Questions`로 이동, 다운스트림 페이즈 태그 추가 (예: "→ factory-architect: X가 적합한지 평가").

---

# Single Topic Mode

caller가 topic/focus_questions/output_path를 제공한 경우 Step 0을 건너뛰고 Step 1부터 시작.

## Step 0: Scope Definition

리서치 시작 전 사용자에게 확인 (이미 명확하지 않은 경우):

1. "이 프로젝트가 해결하려는 **핵심 문제**는 무엇입니까?"
2. "**타겟 사용자/환경**은?" (플랫폼, 규모, 기술 스택 제약)
3. "이미 알고 있는 **기술적 제약**이 있습니까?"

이 3가지가 명확하지 않으면 리서치를 시작하지 않는다. 사용자가 제공한 URL/참조자료가 있으면 함께 수집한다.

`ToolSearch`로 사용 가능한 도구를 확인하고 전략을 가용 도구에 맞춘다.

## Step 1: Discover — 사이트 구조 파악

개별 페이지를 읽기 전에, 어떤 페이지가 존재하는지 먼저 파악한다.

**도구 우선순위 (사용 가능한 첫 번째 도구 사용):**

1. **Firecrawl `firecrawl_map`** — 도메인의 모든 URL을 초 단위로 반환. 1 크레딧.
2. **랜딩 페이지 fetch + 링크 추출** — 주어진 URL을 읽고, 네비게이션/사이드바/본문의 내부 링크를 추출.
3. **WebSearch `site:domain.com`** — 직접 크롤링이 불가능할 때 인덱싱된 페이지를 찾는다.
4. **공통 URL 패턴** — `example.com/docs/intro`에 있다면 `/docs/api`, `/docs/quickstart`, `/docs/pricing` 등도 시도.

발견한 URL을 카테고리로 정리:
- Overview / Getting Started
- API Reference / Technical Docs
- Pricing / Plans / Limits
- Examples / Tutorials
- Changelog / Blog / What's New

가장 관련성 높은 **10-20개 URL**을 선택.

URL이 20개를 초과하면:
1. 중복 제거 (같은 내용의 다른 경로)
2. 우선순위: API Reference > Getting Started > Examples > Blog/Changelog
3. 범위를 줄여야 하면 사용자에게 이유를 설명하고 결정을 맡긴다.

## Step 2: Fetch — 병렬 수집 (폴백 체인)

**병렬 우선, 순차는 최후의 수단.**

**병렬 전략 (우선순위 순):**

1. **Scrapling `bulk_get` / `bulk_fetch` / `bulk_stealthy_fetch`** — 3-10+ 페이지 배치 HTTP/브라우저 fetching.
2. **Firecrawl `firecrawl_scrape`** 병렬 호출 — 같은 턴에 여러 scrape 호출. 페이지당 1 크레딧.
3. **Agent 서브에이전트** — URL 그룹당 하나의 서브에이전트. 가장 유연.
4. **WebFetch 병렬 호출** — 같은 턴에 여러 URL에 WebFetch. 정적 사이트에 적합.

**개별 페이지 폴백 체인 (fetch 실패 시):**

1. **WebFetch** — 가장 빠름, 대부분의 정적 사이트에 작동
2. **Scrapling `get`** — 브라우저 핑거프린트 위장 HTTP, 기본 봇 탐지 우회
3. **Scrapling `fetch`** — 풀 Chromium 브라우저, JS 렌더링 SPA 처리
4. **Scrapling `stealthy_fetch`** — Cloudflare 및 공격적 안티봇 우회
5. **Firecrawl `firecrawl_scrape`** — 클라우드 기반 스크래핑
6. **claude-in-chrome `navigate` + `get_page_text`** — 실제 브라우저, 최후의 수단

WebFetch가 403이나 빈 콘텐츠를 반환하면 페이지 접근 불가가 아니라 더 강한 fetcher가 필요하다는 뜻이다.

## Step 3: Read — 목적에 맞는 읽기

리서치 목표에 따라 수집할 정보를 조정:

**기술 / 라이브러리 평가:**
- 어떤 문제를 해결하는가? 성숙도는?
- API surface: 단순한가 복잡한가?
- 지원 언어/플랫폼, 대안 비교
- 알려진 제한사항이나 함정

**API / 서비스 평가:**
- 인증, 엔드포인트, 레이트 리밋, 가격 (특히 무료 티어)
- SDK 가용성, 에러 처리 패턴

**제품 / 경쟁 분석:**
- 가치 제안, 타겟 사용자, 가격 모델
- 핵심 기능 vs 경쟁사, 강점/약점, 로드맵

**문서 심층 분석:**
- 아키텍처, 핵심 개념, 설정 절차
- 패턴, 베스트 프랙티스, 마이그레이션 가이드

## Step 4: Synthesize — 리포트 종합

URL별이 아닌 **주제별**로 정리한다.

**Single Topic Mode (caller가 output_path를 지정한 경우)**: Topic Sub-Report Template 사용, caller-specified output_path에 저장.

**Single Topic Mode (직접 실행)**: Full SI Template 사용, `factory/research/research.md`에 저장.

### Topic Sub-Report Template (caller 지정 output_path용)

```markdown
# Research Sub-Report: [Topic Title]

## Focus Questions
1. [question from caller]
2. [question from caller]

## Key Findings
[Organized by focus question, not by URL. Each finding with evidence.]

## Comparison (if applicable)
| Option | Pros | Cons | Fit |
|--------|------|------|-----|
| | | | |

## Gaps & Unknowns
- [What could not be found or verified]
- [Areas needing deeper investigation]

## Sources
- [url1] — [what was found]
- [url2] — [what was found]
```

### Full SI Template (직접 실행용)

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

항상 포함해야 할 4가지:
1. **발견한 것** — 주제별 정리
2. **탐색한 URL 목록** — 신뢰성 확보, 후속 조사 가능
3. **찾지 못한 것 / 접근 못한 것** — 갭도 발견만큼 중요
4. **분석** — 단순 요약이 아닌 트레이드오프, 우려, 추천

---

# Multi Topic Mode

## Step A: Research Interview

`AskUserQuestion`으로 리서치 범위를 파악한다. 한 번에 최대 4개 질문을 묶어서 보낸다.

**Core questions** (사용자 입력으로 이미 답변된 것은 skip):

```
AskUserQuestion(questions=[
  {
    question: "리서치가 필요한 주요 영역은 무엇입니까?",
    header: "리서치 영역",
    multiSelect: true,
    options: [
      { label: "시장/경쟁 분석", description: "경쟁 제품, 시장 규모, 포지셔닝" },
      { label: "기술 스택 평가", description: "라이브러리, 프레임워크, 서비스 비교" },
      { label: "API/서비스 조사", description: "외부 API, SaaS, 가격/제한 확인" },
      { label: "사용자/도메인 리서치", description: "사용자 행동, 도메인 지식, 규제" }
    ]
  },
  {
    question: "특별히 조사해야 할 기술이나 서비스가 있습니까?",
    header: "특정 기술",
    multiSelect: false,
    options: [
      { label: "있음 (직접 입력)", description: "Other에 구체적 이름을 작성해주세요" },
      { label: "없음, 자유롭게 탐색", description: "리서치 중 발견되는 기술을 포함" }
    ]
  },
  {
    question: "리서치 깊이는 어느 수준입니까?",
    header: "깊이",
    multiSelect: false,
    options: [
      { label: "탐색적 (Recommended)", description: "넓게 훑고 유망한 방향 식별" },
      { label: "심층 비교", description: "주요 옵션 2-3개를 깊이 비교" },
      { label: "PoC 포함", description: "핵심 기술 가정을 코드로 검증" }
    ]
  },
  {
    question: "리서치에서 제외할 영역이 있습니까?",
    header: "제외 영역",
    multiSelect: false,
    options: [
      { label: "없음", description: "모든 관련 영역을 포함" },
      { label: "있음 (직접 입력)", description: "Other에 제외할 영역을 작성해주세요" }
    ]
  }
])
```

새로운 모호함이 생기면 후속 질문을 추가한다. 리서치 범위가 명확해지면 진행.

## Step B: Topic Proposal

인터뷰 답변을 바탕으로 **3-7개 리서치 토픽**을 제안한다. 각 토픽:

```markdown
## Proposed Research Topics

| # | Topic | Description | Focus Questions |
|---|-------|-------------|-----------------|
| 1 | [제목] | [1문장 설명] | 1. [질문1] 2. [질문2] |
| 2 | ... | ... | ... |
```

설계 원칙:
- 각 토픽은 **독립적으로 리서치 가능** (토픽 간 교차 의존성 없음)
- 인터뷰에서 식별된 모든 영역을 커버
- 각 토픽에 2-3개의 구체적인 집중 질문
- 단일 Agent 서브에이전트가 처리할 수 있는 크기 (너무 넓거나 좁지 않게)

## Step C: User Approval

토픽 목록을 제시하고 승인을 요청한다:

```
AskUserQuestion(questions=[
  {
    question: "위 리서치 토픽 목록이 적절합니까?",
    header: "토픽 승인",
    multiSelect: false,
    options: [
      { label: "좋아요, 진행해요", description: "이 토픽들로 병렬 리서치를 시작합니다" },
      { label: "일부 수정 필요", description: "수정할 내용을 알려주세요" },
      { label: "토픽 추가 필요", description: "빠진 토픽을 알려주세요" }
    ]
  }
])
```

사용자가 승인할 때까지 반복. 승인 후 진행.

## Step D: Parallel Research Execution

**한 턴에 토픽당 하나의 Agent 서브에이전트를 스폰한다 (모두 병렬로).**

각 Agent가 받는 정보:
- `subagent_type`: `"general-purpose"`
- `mode`: `"bypassPermissions"`
- 토픽 제목, 설명, 집중 질문
- 프로젝트 컨텍스트 (프로젝트명, 문제 정의, 사용자 제공 URL)
- Output path: `factory/research/topic-N-[slug].md`
- 이 스킬의 **Single Topic Mode**를 따르도록 지시

**Agent prompt template:**

```
You are a research agent for the factory-research workflow.

## Your Topic
- Title: [topic title]
- Description: [topic description]
- Focus Questions:
  1. [question 1]
  2. [question 2]
  3. [question 3]

## Project Context
- Project: [project name]
- Problem: [problem statement]
- URLs/References: [if any]

## Instructions
1. Read `skills/factory-research/SKILL.md`
2. Follow **Single Topic Mode** described in that skill
3. Your output_path is: factory/research/topic-N-[slug].md
4. Use the Topic Sub-Report Template from Single Topic Mode Step 4

Do NOT ask the user questions — work autonomously with the information provided.
```

**IMPORTANT**: 같은 턴에 여러 Agent 도구 호출로 모든 Agent를 스폰한다. 순차적으로 스폰하지 않는다.

## Step E: Collect & Verify

모든 Agent 완료 후:

1. 각 서브리포트 파일이 `factory/research/topic-N-[slug].md`에 존재하는지 확인
2. 각 서브리포트에 필수 섹션 존재 여부 확인 (Focus Questions, Key Findings, Gaps & Unknowns, Sources)
3. 누락되거나 불완전한 서브리포트가 있으면 어떤 토픽에 문제가 있는지 기록

## Step F: Synthesize

모든 서브리포트를 읽고 `factory/research/research.md`로 종합한다.

이것은 **통합이지 연결이 아니다**:
- 토픽 간 발견물을 교차 참조
- 패턴, 모순, 갭 식별
- 통합된 내러티브 구성

Full SI Template (Step 4 참조)을 사용하며 다음 노트를 추가:

```markdown
> Synthesized from [N] topic reports in `factory/research/`
```

---

## Adapting to Available Tools

| Capability | Best tool | Fallback | Minimal |
|---|---|---|---|
| 사이트 구조 파악 | Firecrawl `map` | 수동 링크 추출 | URL 패턴 추측 |
| 정적 페이지 | Scrapling `bulk_get` | WebFetch (병렬) | WebFetch (순차) |
| JS 렌더링 페이지 | Scrapling `fetch` | Firecrawl `scrape` | claude-in-chrome |
| 봇 차단 사이트 | Scrapling `stealthy_fetch` | Firecrawl `scrape` | claude-in-chrome |
| 대량 읽기 | Scrapling `bulk_*` | Firecrawl 병렬 scrape | Agent 서브에이전트 |

특수 도구가 하나도 없어도 WebFetch만으로 스킬은 동작한다.

---

## PoC (Proof of Concept) — Optional

사용자가 요청하거나 기술 타당성이 불확실한 경우:
- 핵심 기술적 가정을 검증하는 최소 spike 작성
- 무엇이 증명/반증되었는지 문서화

---

## Common Failure Patterns — 자기 점검

이 행동을 하고 있으면 멈추고 수정:

- **One-and-done**: 한 페이지만 읽고 요약 시작. URL은 출발점이지 전체 범위가 아님.
- **Silent failure**: 페이지가 에러를 반환하고 다른 fetcher를 시도하지 않고 건너뜀.
- **비즈니스 맥락 누락**: 기술 기능만 요약하고 가격/제한/라이선스 확인 안 함.
- **얕은 종합**: 각 페이지가 말한 것을 반복하는 대신 인사이트로 연결.
- **보이지 않는 작업**: 실제 방문한 URL 목록을 제시하지 않음.
- **순차 when 병렬 가능**: 벌크 도구가 가능한데 페이지를 하나씩 fetch.
- **조기 종료**: "충분하다"거나 "토큰을 절약"하려고 리서치를 줄임. 토큰 예산은 사용자의 결정.
- **압박 하의 범위 축소**: 15 페이지를 읽으려다 조용히 5개로 줄임. 범위를 줄여야 하면 사용자에게 이유를 설명하고 결정을 맡긴다.

---

## Output

- `factory/research/research.md` — 최종 종합 리포트 (Single Topic 직접 실행 or Multi Topic 종합)
- `factory/research/topic-N-*.md` — 개별 토픽 서브리포트 (Multi Topic Mode)

리서치가 완료되었습니다. 결과는 `factory/research/`에 저장되었습니다.
