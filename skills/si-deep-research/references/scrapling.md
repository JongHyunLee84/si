# Scrapling MCP Server -- Tool Reference

Scrapling is an adaptive Python web scraping framework with built-in anti-bot bypass. Its MCP server exposes 6 tools that map to three underlying fetcher classes, each targeting a different protection level.

---

## Quick Decision Matrix

| Site characteristic | Tool to use | Why |
|---|---|---|
| Static HTML, no JS needed, low/no protection | `get` | Fastest. HTTP-only, no browser overhead. |
| Multiple static pages concurrently | `bulk_get` | Same as `get` but parallelized across URLs. |
| JS-rendered content, low-mid protection | `fetch` | Launches Playwright Chromium; renders JS. |
| Multiple JS pages concurrently | `bulk_fetch` | Same as `fetch` but parallelized across tabs. |
| Cloudflare Turnstile, aggressive WAFs, fingerprint checks | `stealthy_fetch` | Max stealth: solves Cloudflare, hides canvas, blocks WebRTC leaks. |
| Multiple heavily-protected pages concurrently | `bulk_stealthy_fetch` | Same as `stealthy_fetch` but parallelized. |

**Rule of thumb:** Start with `get`. Escalate to `fetch` if JS rendering is needed. Escalate to `stealthy_fetch` only when blocked by anti-bot protections.

---

## Tool Inventory

| MCP Tool | Underlying Class | Input | Protection Level | Browser? |
|---|---|---|---|---|
| `mcp__scrapling__get` | `Fetcher` | Single URL | Low-Mid | No (HTTP + TLS impersonation) |
| `mcp__scrapling__bulk_get` | `Fetcher` | List of URLs | Low-Mid | No |
| `mcp__scrapling__fetch` | `DynamicFetcher` | Single URL | Low-Mid | Yes (Playwright Chromium) |
| `mcp__scrapling__bulk_fetch` | `DynamicFetcher` | List of URLs | Low-Mid | Yes (shared browser, multi-tab) |
| `mcp__scrapling__stealthy_fetch` | `StealthyFetcher` | Single URL | High | Yes (stealth-patched browser) |
| `mcp__scrapling__bulk_stealthy_fetch` | `StealthyFetcher` | List of URLs | High | Yes (stealth browser, multi-tab) |

---

## Common Parameters (All Tools)

These parameters appear across all 6 tools:

| Parameter | Type | Default | Description |
|---|---|---|---|
| `extraction_type` | `"markdown"` / `"html"` / `"text"` | `"markdown"` | Output format. Markdown is best for AI consumption (fewest tokens). |
| `css_selector` | `string` or `null` | `null` | CSS selector to extract specific elements. Massively reduces token usage. Applied after `main_content_only` filter. |
| `main_content_only` | `bool` | `true` | Extract only `<body>` content, stripping nav/footer/etc. |

---

## Tool 1: `mcp__scrapling__get`

**What it does:** Makes a fast HTTP GET request with browser TLS fingerprint impersonation (via curl_cffi). No browser is launched. Best for static pages or APIs.

**When to use:** The page does not require JavaScript rendering and has no aggressive anti-bot protection. This is the fastest option.

### Parameters

| Parameter | Type | Default | Description |
|---|---|---|---|
| `url` | `string` | *required* | Target URL. |
| `impersonate` | `string` or `string[]` | `"chrome"` | Browser fingerprint to impersonate. Options include: `chrome`, `edge`, `safari`, `safari_ios`, `firefox`, `chrome_android`, and version-specific variants like `chrome131`, `chrome142`, `firefox135`, `safari184`, `tor145`, etc. Pass a list for random rotation. |
| `stealthy_headers` | `bool` | `true` | Auto-generates real browser headers and sets referer as if from Google search. |
| `follow_redirects` | `bool` | `true` | Follow HTTP redirects. |
| `max_redirects` | `int` | `30` | Max redirect hops. Use `-1` for unlimited. |
| `timeout` | `int/float` | `30` | Timeout in **seconds** (not ms). |
| `retries` | `int` | `3` | Retry attempts on failure. |
| `retry_delay` | `int` | `1` | Seconds between retries. |
| `params` | `object` | `null` | Query string parameters as key-value pairs. |
| `headers` | `object` | `null` | Custom request headers. |
| `cookies` | `object` | `null` | Cookies as key-value pairs (simple dict format). |
| `proxy` | `string` | `null` | Proxy URL. Format: `"http://username:password@host:port"`. |
| `proxy_auth` | `object` | `null` | Proxy credentials: `{"username": "...", "password": "..."}`. |
| `auth` | `object` | `null` | HTTP basic auth: `{"username": "...", "password": "..."}`. |
| `verify` | `bool` | `true` | Verify HTTPS certificates. |
| `http3` | `bool` | `false` | Use HTTP/3 protocol. May conflict with `impersonate`. |
| `extraction_type` | `string` | `"markdown"` | `"markdown"`, `"html"`, or `"text"`. |
| `css_selector` | `string` | `null` | CSS selector to narrow extraction. |
| `main_content_only` | `bool` | `true` | Body content only. |

### Key Notes
- Timeout is in **seconds** (unlike browser-based tools which use milliseconds).
- `impersonate` goes beyond User-Agent strings -- it mimics the full TLS fingerprint (cipher suites, extensions, ALPN).
- `stealthy_headers` automatically sets a Google-search referer for the target domain.
- No JavaScript execution; purely HTTP.

---

## Tool 2: `mcp__scrapling__bulk_get`

**What it does:** Same as `get` but accepts a list of URLs and fetches them concurrently.

**When to use:** Scraping multiple static pages at once.

### Parameters

Identical to `get` except:

| Parameter | Type | Default | Description |
|---|---|---|---|
| `urls` | `string[]` | *required* | List of target URLs (replaces `url`). |

All other parameters (`impersonate`, `stealthy_headers`, `css_selector`, etc.) apply uniformly to every URL.

---

## Tool 3: `mcp__scrapling__fetch`

**What it does:** Launches a Playwright Chromium browser to fetch and render the page. Handles JavaScript-heavy sites. Suitable for low-mid protection levels.

**When to use:** The page requires JavaScript rendering (SPAs, dynamically loaded content) but does not have aggressive anti-bot protection (no Cloudflare Turnstile, etc.).

### Parameters

| Parameter | Type | Default | Description |
|---|---|---|---|
| `url` | `string` | *required* | Target URL. |
| `headless` | `bool` | `true` | Run browser hidden (`true`) or visible (`false`). |
| `timeout` | `int/float` | `30000` | Timeout in **milliseconds**. |
| `wait` | `int/float` | `0` | Additional wait (ms) after page load before returning. |
| `network_idle` | `bool` | `false` | Wait until no network connections for 500ms. |
| `disable_resources` | `bool` | `false` | Block fonts, images, media, stylesheets, etc. ~25% speed boost. |
| `useragent` | `string` | `null` | Custom UA string. Auto-generated if omitted. |
| `cookies` | `SetCookieParam[]` | `null` | Playwright-format cookies (objects with `name`, `value`, and optional `domain`, `path`, `expires`, `httpOnly`, `secure`, `sameSite`, `url`, `partitionKey`). |
| `google_search` | `bool` | `true` | Set referer as if from Google search. Takes priority over `extra_headers` referer. |
| `extra_headers` | `object` | `null` | Additional HTTP headers. |
| `proxy` | `string` or `object` | `null` | Proxy: string URL or `{"server": "...", "username": "...", "password": "..."}`. |
| `real_chrome` | `bool` | `false` | Use locally installed Chrome instead of Chromium. |
| `cdp_url` | `string` | `null` | Connect to existing browser via Chrome DevTools Protocol URL. |
| `wait_selector` | `string` | `null` | Wait for this CSS selector to reach the specified state. |
| `wait_selector_state` | `"attached"` / `"detached"` / `"hidden"` / `"visible"` | `"attached"` | State condition for `wait_selector`. |
| `timezone_id` | `string` | `null` | Override browser timezone (e.g., `"America/New_York"`). |
| `locale` | `string` | `null` | Browser locale (e.g., `"en-GB"`, `"de-DE"`). Affects number/date formatting. |
| `extraction_type` | `string` | `"markdown"` | Output format. |
| `css_selector` | `string` | `null` | Narrow extraction to specific elements. |
| `main_content_only` | `bool` | `true` | Body content only. |

### Key Notes
- Timeout is in **milliseconds** (30000 = 30 seconds).
- `disable_resources` blocks: font, image, media, beacon, object, imageset, texttrack, websocket, csp_report, stylesheet.
- `real_chrome` requires Chrome installed on the host machine.
- Cookie format is Playwright's `SetCookieParam` (richer than simple key-value).

---

## Tool 4: `mcp__scrapling__bulk_fetch`

**What it does:** Same as `fetch` but opens multiple browser tabs to fetch a list of URLs concurrently.

**When to use:** Scraping multiple JS-rendered pages simultaneously.

### Parameters

Identical to `fetch` except:

| Parameter | Type | Default | Description |
|---|---|---|---|
| `urls` | `string[]` | *required* | List of target URLs (replaces `url`). |

All other parameters apply uniformly to every URL.

---

## Tool 5: `mcp__scrapling__stealthy_fetch`

**What it does:** Launches a stealth-patched browser with maximum anti-detection measures. Can solve Cloudflare Turnstile/Interstitial challenges, hide canvas fingerprints, block WebRTC leaks, and spoof browser fingerprints.

**When to use:** The target site has aggressive anti-bot protection (Cloudflare, DataDome, PerimeterX, etc.). This is the heaviest tool -- use only when `get` and `fetch` are blocked.

### Parameters

Inherits all parameters from `fetch`, plus these stealth-specific additions:

| Parameter | Type | Default | Description |
|---|---|---|---|
| `url` | `string` | *required* | Target URL. |
| `solve_cloudflare` | `bool` | `false` | Automatically solve Cloudflare Turnstile/Interstitial challenges (JS, interactive, invisible). **Set timeout >= 60000 when enabled.** |
| `hide_canvas` | `bool` | `false` | Inject random noise into canvas operations to prevent fingerprinting. |
| `block_webrtc` | `bool` | `false` | Force WebRTC to respect proxy settings, preventing local IP leaks. |
| `allow_webgl` | `bool` | `true` | Keep WebGL enabled. **Disabling is NOT recommended** -- many WAFs check for WebGL. |
| `additional_args` | `object` | `null` | Extra Playwright context settings. Takes highest priority over all other settings. |
| `headless` | `bool` | `true` | Run browser hidden or visible. |
| `timeout` | `int/float` | `30000` | Timeout in **milliseconds**. Use >= 60000 with `solve_cloudflare`. |
| `wait` | `int/float` | `0` | Additional wait (ms) after load. |
| `network_idle` | `bool` | `false` | Wait for network quiet. |
| `disable_resources` | `bool` | `false` | Block non-essential resources. |
| `useragent` | `string` | `null` | Custom UA. Auto-generated if omitted. |
| `cookies` | `SetCookieParam[]` | `null` | Playwright-format cookies. |
| `google_search` | `bool` | `true` | Google-search referer spoofing. |
| `extra_headers` | `object` | `null` | Additional headers. |
| `proxy` | `string` or `object` | `null` | Proxy configuration. |
| `real_chrome` | `bool` | `false` | Use installed Chrome. |
| `cdp_url` | `string` | `null` | CDP connection URL. |
| `wait_selector` | `string` | `null` | Wait for CSS selector. |
| `wait_selector_state` | `string` | `"attached"` | Selector state condition. |
| `timezone_id` | `string` | `null` | Browser timezone. |
| `locale` | `string` | `null` | Browser locale. |
| `extraction_type` | `string` | `"markdown"` | Output format. |
| `css_selector` | `string` | `null` | Narrow extraction. |
| `main_content_only` | `bool` | `true` | Body content only. |

### Anti-Bot Measures (Active by Default)
- Playwright detection fingerprint removal
- CDP/WebRTC leak isolation via sandboxed JS execution
- Headless-mode detection patches
- Timezone consistency enforcement
- Google-search referer spoofing
- Real browser User-Agent generation

### Anti-Bot Measures (Opt-In)
- `solve_cloudflare=true` -- Solves all Cloudflare challenge types
- `hide_canvas=true` -- Canvas fingerprint noise injection
- `block_webrtc=true` -- Prevents local IP leak through WebRTC

---

## Tool 6: `mcp__scrapling__bulk_stealthy_fetch`

**What it does:** Same as `stealthy_fetch` but opens multiple stealth browser tabs to fetch a list of URLs concurrently.

**When to use:** Scraping multiple heavily-protected pages simultaneously.

### Parameters

Identical to `stealthy_fetch` except:

| Parameter | Type | Default | Description |
|---|---|---|---|
| `urls` | `string[]` | *required* | List of target URLs (replaces `url`). |

All other parameters apply uniformly to every URL.

---

## Anti-Bot Capability Summary

| Capability | `get` | `fetch` | `stealthy_fetch` |
|---|---|---|---|
| TLS fingerprint impersonation | Yes (via curl_cffi) | No (uses Playwright's) | Patched/stealth |
| Stealthy headers + Google referer | Yes | Yes | Yes |
| JavaScript rendering | No | Yes | Yes |
| Cloudflare Turnstile solving | No | No | Yes (`solve_cloudflare`) |
| Canvas fingerprint hiding | No | No | Yes (`hide_canvas`) |
| WebRTC IP leak prevention | No | No | Yes (`block_webrtc`) |
| WebGL presence | N/A | Default | Default (keep enabled) |
| Headless detection bypass | N/A | Partial | Full |
| Playwright fingerprint removal | N/A | No | Yes |
| CDP leak isolation | N/A | No | Yes |
| Browser impersonation versions | 50+ browser variants | Auto UA | Stealth UA |
| Proxy support | Yes | Yes | Yes |
| HTTP/3 | Yes | No | No |
| Real Chrome support | No | Yes | Yes |
| CDP remote browser | No | Yes | Yes |

---

## Parameter Differences: Timeout Units

This is the most common source of confusion:

| Tool | Timeout unit | Default |
|---|---|---|
| `get` / `bulk_get` | **Seconds** | `30` |
| `fetch` / `bulk_fetch` | **Milliseconds** | `30000` |
| `stealthy_fetch` / `bulk_stealthy_fetch` | **Milliseconds** | `30000` |

---

## Cookie Format Differences

| Tool | Cookie format |
|---|---|
| `get` / `bulk_get` | Simple dict: `{"name": "value"}` |
| `fetch` / `bulk_fetch` | Playwright `SetCookieParam[]`: `[{"name": "...", "value": "...", "domain": "..."}]` |
| `stealthy_fetch` / `bulk_stealthy_fetch` | Playwright `SetCookieParam[]`: same as fetch |

---

## Common Gotchas and Tips

### 1. Always use `css_selector` to reduce tokens
The biggest advantage of Scrapling's MCP server over alternatives is CSS selector support. Instead of passing entire pages to the AI, narrow down to the content you need:
```
css_selector: "article.post-content"
css_selector: "#main-table tbody tr"
css_selector: "div.product-info"
```

### 2. `solve_cloudflare` requires longer timeout
When enabling Cloudflare solving, set `timeout` to at least `60000` (60 seconds). The default 30-second timeout will often cause failures during challenge solving.

### 3. Do not disable WebGL
Many WAFs check for WebGL presence. Keeping `allow_webgl=true` (the default) avoids detection. Only disable if you have a specific reason.

### 4. `google_search` referer overrides `extra_headers` referer
When `google_search=true` (default), any referer you set in `extra_headers` will be overridden. Disable `google_search` first if you need a custom referer.

### 5. `disable_resources` for speed
For browser-based tools (`fetch`, `stealthy_fetch`), enabling `disable_resources=true` blocks fonts, images, media, stylesheets, and other non-essential resources. This gives roughly 25% speed improvement and is safe for most data extraction tasks.

### 6. `network_idle` for SPAs
For single-page applications that load data asynchronously after initial page load, enable `network_idle=true` to wait until all AJAX/fetch calls complete (500ms network quiet).

### 7. `wait_selector` for specific elements
Instead of guessing wait times, use `wait_selector` with a CSS selector for the element you actually need:
```
wait_selector: "table.data-loaded"
wait_selector_state: "visible"
```

### 8. `real_chrome` for maximum authenticity
Using `real_chrome=true` launches your locally installed Chrome instead of Chromium. This produces the most authentic browser fingerprint but requires Chrome to be installed on the host.

### 9. Escalation strategy
Start with `get` (fastest, cheapest). If you get blocked or need JS:
- Blocked by JS requirement -> `fetch`
- Blocked by anti-bot -> `stealthy_fetch`
- Blocked by Cloudflare -> `stealthy_fetch` + `solve_cloudflare=true`
- Still blocked -> `stealthy_fetch` + `real_chrome=true` + `solve_cloudflare=true` + `hide_canvas=true` + `block_webrtc=true`

### 10. Bulk tools share configuration
All parameters on bulk tools apply uniformly to every URL. You cannot set different parameters per URL in a single bulk call. If you need different settings per URL, make separate single-URL calls.

### 11. `extraction_type="markdown"` is the best default for AI
Markdown output is the most token-efficient format for AI consumption. Use `"html"` only when you need to preserve exact structure, and `"text"` when you need zero formatting.

### 12. Proxy format varies by tool
- `get`/`bulk_get`: String URL only: `"http://user:pass@host:port"`
- `fetch`/`stealthy_fetch` (and bulk variants): String URL or dict: `{"server": "...", "username": "...", "password": "..."}`

---

## Installation and Setup

```bash
# Install with MCP server support
pip install "scrapling[ai]"

# Install browser dependencies
scrapling install

# Claude Desktop config (claude_desktop_config.json)
{
  "mcpServers": {
    "ScraplingServer": {
      "command": "/path/to/scrapling",
      "args": ["mcp"]
    }
  }
}

# Claude Code
claude mcp add ScraplingServer "/path/to/scrapling" mcp

# Optional: HTTP transport mode
scrapling mcp --http --host '127.0.0.1' --port 8000
```

**Requirements:** Python >= 3.10

---

## Sources

- GitHub: https://github.com/D4Vinci/Scrapling
- Docs: https://scrapling.readthedocs.io/en/latest/
- MCP Server Docs: https://scrapling.readthedocs.io/en/latest/ai/mcp-server.html
- PyPI: https://pypi.org/project/scrapling/ (v0.4.1, Feb 2026)
