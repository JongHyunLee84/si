---
name: si-deep-research
user-invocable: true
description: "SI 리서치 특화 deep-research — 스코프 정의 질문 통합, SI 템플릿 출력, 도구 폴백 체인 유지"
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

# SI Deep Research

SI 워크플로우의 Research Phase를 위한 체계적 리서치 스킬. 단일 페이지 읽기가 아닌 다각도 탐색을 강제한다.

## Reference Files

도구별 상세 정보가 필요할 때 참조:

- **`references/setup.md`** — MCP 도구 설치 가이드 (Scrapling, Firecrawl, Chrome). 도구 설정이나 가용성 문제 해결 시 참조.
- **`references/scrapling.md`** — Scrapling MCP 6개 도구: 파라미터, 안티봇 티어, 결정 매트릭스, 주의사항. `get`/`fetch`/`stealthy_fetch` 선택 시 참조.
- **`references/firecrawl.md`** — Firecrawl MCP 11개 도구: 파라미터, 크레딧 비용, 결정 플로차트. `firecrawl_map`, `firecrawl_scrape`, `firecrawl_search` 사용 시 참조.

모든 참조를 미리 읽을 필요 없음. 특정 결정이 필요할 때 참조 (예: "Cloudflare 우회에 어떤 scrapling 도구?" → scrapling.md 읽기).

---

## Why this skill exists

AI가 URL 하나를 읽고 요약하는 것은 책의 목차만 읽고 내용을 안다고 하는 것과 같다. 이 스킬은:
- **광범위한 탐색**을 먼저 수행하고 종합하는 리서치 워크플로우를 강제
- **403/429 에러에 멈추지 않고** 폴백 체인으로 대응
- **토큰 절약을 위한 조기 종료를 방지** — 리서치의 깊이는 사용자의 결정

---

## Step 0: Scope Definition (SI 통합)

리서치 시작 전 사용자에게 확인 (이미 명확하지 않은 경우):

1. "이 프로젝트가 해결하려는 **핵심 문제**는 무엇입니까?"
2. "**타겟 사용자/환경**은?" (플랫폼, 규모, 기술 스택 제약)
3. "이미 알고 있는 **기술적 제약**이 있습니까?"

이 3가지가 명확하지 않으면 리서치를 시작하지 않는다. 사용자가 제공한 URL/참조자료가 있으면 함께 수집한다.

---

## Step 0.5: Tool Availability Check

리서치 시작 시 `ToolSearch`로 사용 가능한 도구를 확인한다. 전략을 가용 도구에 맞춘다 (아래 "Adapting to Available Tools" 참조).

---

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

---

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

---

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

---

## Step 4: Synthesize — SI 템플릿으로 종합

URL별이 아닌 **주제별**로 정리한다. `tasks/research-report.md`에 작성:

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

## PoC (Proof of Concept) — Optional

사용자가 요청하거나 기술 타당성이 불확실한 경우:
- 핵심 기술적 가정을 검증하는 최소 spike 작성
- 무엇이 증명/반증되었는지 문서화

---

## Adapting to Available Tools

이 스킬은 어떤 MCP 서버가 설치되어 있든 동작한다. 리서치 시작 시 `ToolSearch`로 가용 도구를 확인하고 전략을 조정한다:

| Capability | Best tool | Fallback | Minimal |
|---|---|---|---|
| 사이트 구조 파악 | Firecrawl `map` | 수동 링크 추출 | URL 패턴 추측 |
| 정적 페이지 | Scrapling `bulk_get` | WebFetch (병렬) | WebFetch (순차) |
| JS 렌더링 페이지 | Scrapling `fetch` | Firecrawl `scrape` | claude-in-chrome |
| 봇 차단 사이트 | Scrapling `stealthy_fetch` | Firecrawl `scrape` | claude-in-chrome |
| 대량 읽기 | Scrapling `bulk_*` | Firecrawl 병렬 scrape | Agent 서브에이전트 |

특수 도구가 하나도 없어도 WebFetch만으로 스킬은 동작한다. 중요한 것은 리서치 **행동** (한 페이지에서 멈추지 않고 폭넓게 탐색)이지 특정 도구가 아니다.

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
