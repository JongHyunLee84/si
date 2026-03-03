# Firecrawl MCP Server -- Complete Tool Reference

> Last updated: 2026-03-01. Source: tool schemas from mcp__firecrawl__* + docs.firecrawl.dev + firecrawl.dev/pricing.

Firecrawl is a web crawling, scraping, and search API that converts websites into clean markdown or structured data for LLM consumption. The MCP server exposes 11 tools spanning scraping, crawling, searching, extraction, autonomous agents, and browser automation.

---

## Tool Inventory

| MCP Tool Name | Purpose | Credits | When to Use |
|---|---|---|---|
| `firecrawl_scrape` | Scrape a single URL into markdown/HTML/JSON/screenshot | 1/page (+modifiers) | You know the exact URL and need its content |
| `firecrawl_crawl` | Crawl an entire site following links | 1/page crawled | Need content from many pages on one domain |
| `firecrawl_check_crawl_status` | Poll a crawl job for progress/results | 0 | After starting a crawl, check if done |
| `firecrawl_map` | Discover all URLs on a website (fast) | 1/call (flat) | Find URLs before deciding what to scrape |
| `firecrawl_search` | Web search + optional content scraping | 2/10 results (+scrape) | Open-ended questions; don't know which site has the answer |
| `firecrawl_extract` | LLM-powered structured extraction from URL(s) | Dynamic (token-based) | Need structured data from one or many URLs |
| `firecrawl_agent` | Autonomous AI research agent (async) | 5 free daily runs; then dynamic | Complex multi-site research; unknown URLs |
| `firecrawl_agent_status` | Poll agent job for results | 0 | Check if agent research is complete |
| `firecrawl_browser_create` | Launch a persistent browser sandbox session | 2/minute | Multi-step interactions, form filling, auth flows |
| `firecrawl_browser_execute` | Execute code in a browser session | (included in session) | Run bash/Python/JS in a live browser |
| `firecrawl_browser_delete` | Destroy a browser session | 0 | Clean up when done with browser |
| `firecrawl_browser_list` | List active/destroyed browser sessions | 0 | Check what sessions exist |

---

## Decision Flowchart

```
Need web data?
  |
  +-- Know the exact URL?
  |     +-- Single page? --> firecrawl_scrape
  |     +-- Many pages on same domain? --> firecrawl_crawl
  |     +-- Need structured JSON from page(s)? --> firecrawl_scrape (JSON format) or firecrawl_extract
  |
  +-- Don't know the URL?
  |     +-- General web question? --> firecrawl_search
  |     +-- Know the domain, need to find the right page? --> firecrawl_map (with search param)
  |     +-- Complex multi-source research? --> firecrawl_agent
  |
  +-- Need browser interaction (login, clicks, forms)?
        +-- firecrawl_browser_create --> firecrawl_browser_execute --> firecrawl_browser_delete
```

---

## Detailed Tool Reference

### 1. firecrawl_scrape

**Purpose:** Scrape content from a single URL. The primary and most-used tool.

**Required Parameters:**
- `url` (string, URI): The URL to scrape.

**Key Optional Parameters:**
- `formats` (array): Output format(s). Options:
  - `"markdown"` -- clean markdown (default, best for LLMs)
  - `"html"` -- cleaned HTML
  - `"rawHtml"` -- unmodified HTML
  - `"screenshot"` -- page screenshot (supports `fullPage`, `quality`, `viewport` sub-options)
  - `"links"` -- all links on the page
  - `"summary"` -- AI-generated summary
  - `"branding"` -- extract brand identity (colors, fonts, typography, spacing, UI components)
  - `{"type": "json", "schema": {...}, "prompt": "..."}` -- LLM-powered structured extraction (+4 credits)
- `onlyMainContent` (boolean): Strip nav/footer/sidebar. Recommended for article extraction.
- `actions` (array): Pre-scrape page interactions. Each action has a `type`:
  - `"wait"` (milliseconds), `"click"` (selector), `"write"` (text), `"press"` (key), `"scroll"` (direction), `"screenshot"` (fullPage), `"scrape"`, `"executeJavascript"` (script), `"generatePDF"`
- `waitFor` (number): Wait N milliseconds for JS to render before scraping.
- `maxAge` (number): Cache freshness window in ms. Default 172800000 (2 days). Set 0 for always-fresh.
- `storeInCache` (boolean): Whether to cache results. Default true.
- `location` (object): `{country: "US", languages: ["en"]}` for geo-targeted scraping.
- `proxy` (enum): `"basic"`, `"stealth"`, `"enhanced"` (+4 credits), `"auto"`.
- `includeTags` / `excludeTags` (string arrays): CSS selectors to include/exclude.
- `mobile` (boolean): Emulate mobile device.
- `removeBase64Images` (boolean): Strip base64 images from output.
- `parsers` (array): `["pdf"]` or `[{"type": "pdf", "maxPages": 100}]` for PDF parsing (+1 credit/page).

**Credit Cost:** 1 base + modifiers (JSON +4, enhanced proxy +4, PDF +1/page). Modifiers stack.

**When to use:** You know exactly which URL to scrape. This is the default tool for getting web content.

**When NOT to use:** You don't know which URL has the info (use search). You need many pages (use crawl). You need to interact with the page (use browser for complex flows, or actions for simple ones).

**Example -- markdown:**
```json
{
  "url": "https://example.com/article",
  "formats": ["markdown"],
  "onlyMainContent": true
}
```

**Example -- structured JSON extraction:**
```json
{
  "url": "https://example.com/product",
  "formats": [{
    "type": "json",
    "prompt": "Extract product name, price, and description",
    "schema": {
      "type": "object",
      "properties": {
        "name": {"type": "string"},
        "price": {"type": "number"},
        "description": {"type": "string"}
      }
    }
  }]
}
```

**Example -- with actions (login flow):**
```json
{
  "url": "https://example.com/login",
  "formats": ["markdown"],
  "actions": [
    {"type": "write", "text": "user@example.com"},
    {"type": "press", "key": "Tab"},
    {"type": "write", "text": "password123"},
    {"type": "click", "selector": "button[type='submit']"},
    {"type": "wait", "milliseconds": 2000},
    {"type": "screenshot", "fullPage": true}
  ]
}
```

---

### 2. firecrawl_crawl

**Purpose:** Crawl an entire website by recursively following links from a starting URL. Returns a job ID (async).

**Required Parameters:**
- `url` (string): Starting URL to crawl.

**Key Optional Parameters:**
- `limit` (number): Max pages to crawl. Default 10,000. Set low to control costs.
- `maxDiscoveryDepth` (number): How many link-hops deep to follow.
- `includePaths` / `excludePaths` (string arrays): URL path patterns to include/exclude.
- `allowExternalLinks` (boolean): Follow links to other domains.
- `allowSubdomains` (boolean): Follow links to subdomains (e.g., blog.example.com).
- `crawlEntireDomain` (boolean): Include non-child URLs of the starting path.
- `deduplicateSimilarURLs` (boolean): Skip near-duplicate URLs.
- `sitemap` (enum): `"include"` (default), `"skip"`, `"only"`.
- `scrapeOptions` (object): All scrape parameters (formats, proxy, actions, etc.) applied to every page.
- `webhook` (string or object): Receive real-time notifications as pages are crawled.
- `delay` (number): Delay between requests in ms.
- `maxConcurrency` (number): Max parallel scrapes.
- `ignoreQueryParameters` (boolean): Treat URLs with different query params as the same page.

**Credit Cost:** 1 credit per page crawled (+ scrape option modifiers per page).

**Returns:** A job ID. Use `firecrawl_check_crawl_status` to poll for results.

**When to use:** Need comprehensive content from many pages on a single domain.

**When NOT to use:** Single page (use scrape). Just need URLs (use map). Results can exceed token limits -- set `limit` conservatively.

**Example:**
```json
{
  "url": "https://docs.example.com",
  "limit": 50,
  "maxDiscoveryDepth": 3,
  "scrapeOptions": {
    "formats": ["markdown"],
    "onlyMainContent": true
  }
}
```

**Gotcha:** Crawl responses can be very large. Keep `limit` and `maxDiscoveryDepth` low. Use `map` + targeted `scrape` for better control.

---

### 3. firecrawl_check_crawl_status

**Purpose:** Check the status and retrieve results of an ongoing or completed crawl job.

**Required Parameters:**
- `id` (string): The crawl job ID returned by `firecrawl_crawl`.

**Credit Cost:** 0 (free to poll).

**Returns:** Status (`scraping`, `completed`, `failed`) + progress counters + data array of scraped pages.

**Important:** Results expire after 24 hours.

---

### 4. firecrawl_map

**Purpose:** Quickly discover all URLs on a website without scraping content. Primarily uses sitemaps + SERP + cached crawl data.

**Required Parameters:**
- `url` (string, URI): The website to map.

**Key Optional Parameters:**
- `search` (string): Filter URLs by relevance to a search term. Results ordered by relevance.
- `limit` (number): Max URLs to return.
- `includeSubdomains` (boolean): Include subdomain URLs.
- `sitemap` (enum): `"include"` (default), `"skip"`, `"only"`.
- `ignoreQueryParameters` (boolean): Deduplicate URLs differing only in query params.

**Credit Cost:** 1 credit per call (flat, regardless of URLs returned).

**Returns:** Array of `{url, title, description}` objects.

**When to use:**
- Need to find specific pages on a large site before scraping
- Scrape returned empty/wrong content -- use map to find the correct URL
- User needs to choose which pages to scrape
- Need a sitemap-like list cheaply

**When NOT to use:** Need actual page content (use scrape after mapping).

**Example -- find webhook docs:**
```json
{
  "url": "https://docs.example.com",
  "search": "webhook events"
}
```

**Tip:** Map is the cheapest discovery tool. Use `map` + `search` to find the right URL, then `scrape` that URL. This is faster and cheaper than `firecrawl_agent`.

---

### 5. firecrawl_search

**Purpose:** Web search with optional content scraping of results. The most powerful tool for open-ended questions.

**Required Parameters:**
- `query` (string): The search query. Supports search operators: `""`, `-`, `site:`, `inurl:`, `intitle:`, `related:`, `imagesize:`, `larger:`.

**Key Optional Parameters:**
- `limit` (number): Max results per source type.
- `sources` (array of objects): `[{"type": "web"}, {"type": "images"}, {"type": "news"}]`. Default: web only. `limit` applies per source type.
- `scrapeOptions` (object): If provided, each search result is also scraped. All scrape parameters supported (formats, actions, etc.).
- `tbs` (string): Time-based search filter. Values: `"qdr:h"` (hour), `"qdr:d"` (day), `"qdr:w"` (week), `"qdr:m"` (month), `"qdr:y"` (year), `"sbd:1"` (sort by date). Custom ranges: `"cdr:1,cd_min:12/1/2024,cd_max:12/31/2024"`.
- `location` (string): Location name for geo-targeted results (e.g., `"Germany"`).
- `filter` (string): Additional filter string.
- `enterprise` (array): Enterprise features like `"anon"`, `"zdr"`.

**Credit Cost:** 2 credits per 10 search results. If `scrapeOptions` provided, add 1 credit per result scraped (+ modifiers).

**Returns:** `{web: [...], images: [...], news: [...]}` with optional scraped content per result.

**When to use:** Don't know which website has the answer. Need current/recent information. General web research.

**When NOT to use:** Know the exact URL (use scrape). Need comprehensive site coverage (use crawl/map).

**Optimal workflow:** Search first WITHOUT scrapeOptions to find relevant URLs, then scrape specific relevant URLs individually.

**Example -- basic search:**
```json
{
  "query": "best practices for RAG pipelines 2025",
  "limit": 5,
  "sources": [{"type": "web"}]
}
```

**Example -- search with scraping:**
```json
{
  "query": "firecrawl pricing",
  "limit": 3,
  "scrapeOptions": {
    "formats": ["markdown"],
    "onlyMainContent": true
  }
}
```

---

### 6. firecrawl_extract

**Purpose:** Extract structured data from one or more URLs using LLM. Handles crawling, parsing, and collation internally.

**Required Parameters:**
- `urls` (string array): One or more URLs. Supports wildcards (e.g., `"https://example.com/*"`).

**Key Optional Parameters:**
- `prompt` (string): Natural language description of what data to extract. Required if no schema.
- `schema` (object): JSON Schema for structured output. Required if no prompt.
- `allowExternalLinks` (boolean): Follow links outside the specified domains.
- `enableWebSearch` (boolean): Enrich extraction with data from related pages found via search.
- `includeSubdomains` (boolean): Include subdomain pages.

**Credit Cost:** Dynamic, token-based. Each credit = 15 tokens.

**Returns:** Structured data matching your prompt/schema. Async -- returns a job that you poll (though SDK methods can wait).

**When to use:** Need structured data from pages. Multi-page extraction with a single schema. Don't want to write parsing logic.

**When NOT to use:** Need full page content (use scrape with markdown). Simple single-value extraction (use scrape with JSON format -- faster and cheaper).

**Example:**
```json
{
  "urls": ["https://docs.firecrawl.dev/*"],
  "prompt": "Extract all API endpoints with their methods, paths, and descriptions",
  "schema": {
    "type": "object",
    "properties": {
      "endpoints": {
        "type": "array",
        "items": {
          "type": "object",
          "properties": {
            "method": {"type": "string"},
            "path": {"type": "string"},
            "description": {"type": "string"}
          }
        }
      }
    }
  }
}
```

**Note:** Extract is being succeeded by the Agent tool (`/agent`). Agent is faster, more reliable, and does not require URLs.

---

### 7. firecrawl_agent

**Purpose:** Autonomous AI research agent that independently browses the web, follows links, and extracts data. Runs asynchronously.

**Required Parameters:**
- `prompt` (string, max 10,000 chars): Natural language description of what you need.

**Key Optional Parameters:**
- `urls` (string array, URI): Optional URLs to focus the agent on.
- `schema` (object): JSON Schema for structured output.

**Credit Cost:** 5 free daily runs. Dynamic pricing beyond that.

**Returns immediately:** A job ID. Must poll with `firecrawl_agent_status`.

**Polling protocol:**
1. Call `firecrawl_agent` -- get job ID
2. Poll `firecrawl_agent_status` every 15-30 seconds
3. Keep polling for at least 2-3 minutes (complex research can take 5+ minutes)
4. Stop when status is `"completed"` or `"failed"`

**When to use:**
- Complex research across multiple unknown sites
- Multi-source data gathering
- Content behind JavaScript-heavy SPAs that fail with regular scrape
- You don't know which URLs contain the data

**When NOT to use:** Simple single-page scraping (use scrape -- faster and cheaper). You know the URL (use scrape with JSON format).

**Example:**
```json
{
  "prompt": "Find the top 5 AI startups founded in 2024 with their funding amounts and key products",
  "schema": {
    "type": "object",
    "properties": {
      "startups": {
        "type": "array",
        "items": {
          "type": "object",
          "properties": {
            "name": {"type": "string"},
            "funding": {"type": "string"},
            "product": {"type": "string"}
          }
        }
      }
    }
  }
}
```

---

### 8. firecrawl_agent_status

**Purpose:** Poll the status of an agent job and retrieve results.

**Required Parameters:**
- `id` (string): The agent job ID.

**Returns:** `{status, progress, results}`. Statuses: `"processing"`, `"completed"`, `"failed"`.

**Important:** Be patient. Keep polling for 2-3+ minutes before giving up.

---

### 9. firecrawl_browser_create

**Purpose:** Launch a persistent, isolated browser sandbox session via CDP (Chrome DevTools Protocol).

**Optional Parameters:**
- `ttl` (number, 30-3600): Total session lifetime in seconds. Default 300 (5 min).
- `activityTtl` (number, 10-3600): Idle timeout in seconds. Default 120 (2 min).
- `streamWebView` (boolean): Enable live view streaming.

**Credit Cost:** 2 credits per browser minute.

**Returns:** `{id, cdpUrl, liveViewUrl, interactiveLiveViewUrl}`.

**When to use:** Multi-step browser automation (login, navigate, click, fill forms). Content behind authentication. Parallel browsing across sites.

**When NOT to use:** Simple page scraping (use scrape). Search queries (use search).

**Rate limit:** Up to 20 concurrent browser sessions on all plans.

---

### 10. firecrawl_browser_execute

**Purpose:** Execute code inside an active browser session.

**Required Parameters:**
- `sessionId` (string): The browser session ID.
- `code` (string): Code to execute.

**Optional Parameters:**
- `language` (enum): `"bash"` (default, uses agent-browser CLI), `"python"`, `"node"`.

**Returns:** `{success, result}` with stdout/stderr/exit code.

**Recommended: Use bash with agent-browser commands:**
```json
{
  "sessionId": "...",
  "code": "agent-browser open https://example.com",
  "language": "bash"
}
```

**Common agent-browser commands:**
| Command | Description |
|---|---|
| `agent-browser open <url>` | Navigate to URL |
| `agent-browser snapshot` | Get accessibility tree with clickable refs |
| `agent-browser snapshot -i -c` | Interactive elements only, compact |
| `agent-browser click @e5` | Click element by ref |
| `agent-browser type @e3 "text"` | Type into element |
| `agent-browser fill @e3 "text"` | Clear and fill element |
| `agent-browser get text @e1` | Get text content |
| `agent-browser get title` | Get page title |
| `agent-browser get url` | Get current URL |
| `agent-browser screenshot [path]` | Take screenshot |
| `agent-browser scroll down` | Scroll page |
| `agent-browser wait 2000` | Wait 2 seconds |

**For Playwright scripting, use Python:**
```json
{
  "sessionId": "...",
  "code": "await page.goto('https://example.com')\ntitle = await page.title()\nprint(title)",
  "language": "python"
}
```

---

### 11. firecrawl_browser_delete

**Purpose:** Destroy a browser session and free resources.

**Required Parameters:**
- `sessionId` (string): The session to destroy.

**Credit Cost:** 0 (session already billed per minute).

**Important:** Always delete sessions when done to stop billing.

---

### 12. firecrawl_browser_list

**Purpose:** List browser sessions, optionally filtered by status.

**Optional Parameters:**
- `status` (enum): `"active"` or `"destroyed"`.

**Credit Cost:** 0.

---

## Pricing Summary

### Credit Costs Per Operation

| Operation | Base Cost | With JSON Format | With Enhanced Proxy | With Both |
|---|---|---|---|---|
| Scrape | 1 credit/page | 5 credits/page | 5 credits/page | 9 credits/page |
| Crawl | 1 credit/page | 5 credits/page | 5 credits/page | 9 credits/page |
| Map | 1 credit/call | -- | -- | -- |
| Search | 2 credits/10 results | +4/result scraped | +4/result scraped | +8/result scraped |
| Browser | 2 credits/minute | -- | -- | -- |
| Agent | 5 free/day; dynamic after | -- | -- | -- |
| Extract | Dynamic (1 credit = 15 tokens) | -- | -- | -- |
| PDF Parsing | +1 credit/PDF page | -- | -- | -- |

### Plans

| Plan | Monthly Price | Credits/Month | Concurrent Requests |
|---|---|---|---|
| Free | $0 | 500 (one-time) | 2 |
| Hobby | $9/mo | 3,000 | 5 |
| Standard | $47/mo | 100,000 | 50 |
| Growth | $177/mo | 500,000 | 100 |
| Scale | $599/mo | 1,000,000 | 150 |
| Enterprise | Custom | Custom | Custom |

Annual billing available at discount. Credits do not roll over. Auto-recharge available on paid plans.

---

## Common Gotchas and Tips

### Scrape Tips
- **Use `onlyMainContent: true`** for articles/docs to strip nav/footer noise.
- **Use JSON format for specific data points**, not markdown. JSON costs +4 credits but gives structured output.
- **Set `maxAge: 0`** only when you absolutely need fresh data. Cached responses are 5x faster and more reliable.
- **Add `waitFor: 5000`** for JavaScript-heavy pages (SPAs) that return empty content.
- **Use actions sparingly** -- they add complexity. For complex multi-step flows, use the browser tools instead.

### Search Tips
- **Search first without scrapeOptions**, then scrape relevant results individually. This avoids wasting credits scraping irrelevant results.
- **Use search operators** (`site:`, `intitle:`, `""` for exact match) to narrow results.
- **`limit` applies per source type** -- `limit: 5` with web+news = up to 10 results total.

### Map Tips
- **Use `search` parameter** to find specific pages. Results are ranked by relevance.
- **Map is the cheapest discovery tool** (1 credit flat). Use map + targeted scrape instead of crawl when possible.
- **If scrape returns empty**, use map with search to find the correct URL, then scrape that.

### Crawl Tips
- **Always set `limit`**. Default is 10,000 pages which burns credits fast.
- **Keep `maxDiscoveryDepth` low** (2-3) unless you need deep pages.
- **Use `deduplicateSimilarURLs: true`** to avoid scraping near-duplicate pages.
- **Crawl responses can exceed token limits**. For large sites, use map + batch scrape for better control.

### Extract Tips
- **For single-page extraction**, prefer `scrape` with JSON format (faster, cheaper).
- **Use `extract` for multi-page** or domain-wide extraction where you need a unified schema.
- **`enableWebSearch: true`** enriches results but increases cost.
- **Being succeeded by Agent** -- prefer Agent for new use cases.

### Agent Tips
- **Be patient with polling** -- complex research takes 1-5+ minutes.
- **Poll every 15-30 seconds**, not faster.
- **5 free daily runs** -- use wisely for complex tasks, not simple scrapes.
- **Provide URLs when possible** to focus the agent and reduce research time.

### Browser Tips
- **Always delete sessions** when done to stop per-minute billing.
- **Use bash (agent-browser)** as the default language -- simplest for AI agents.
- **Use Python** for complex Playwright scripting.
- **`snapshot`** is the key command for AI agents -- it returns an accessibility tree with clickable element refs.
- **TTL defaults**: session=5min, idle=2min. Increase for longer workflows.
- **Max 20 concurrent sessions** on all plans.

### General Best Practices
1. **Start with the cheapest tool that works**: map (1 credit) > scrape (1 credit) > search (2 credits) > extract (dynamic) > agent (dynamic)
2. **Cache aggressively**: Use default `maxAge` unless freshness is critical.
3. **Escalation path for failed scrapes**: scrape -> add `waitFor` -> use map to find correct URL -> try extract -> use browser -> use agent (last resort).
4. **Monitor credit usage** via dashboard or API.
5. **Use `proxy: "auto"`** to let Firecrawl choose the best proxy strategy.
