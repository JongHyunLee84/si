# deep-research Prerequisites & Setup Guide

This skill works out of the box with WebFetch + WebSearch (always available). But many sites block simple HTTP requests with 403/429 errors. Installing the tools below dramatically expands what you can access.

## Capability comparison by configuration

| Configuration | Site discovery | Static pages | JS rendering | Bot bypass | Bulk fetching |
|---|---|---|---|---|---|
| **Minimum (no setup)** | URL pattern guessing | WebFetch | X | X | WebFetch parallel |
| **+ Firecrawl** | firecrawl_map (seconds) | firecrawl_scrape | firecrawl_scrape | Cloud proxy | firecrawl parallel scrape |
| **+ Scrapling** | Manual link extraction | scrapling get | scrapling fetch | stealthy_fetch | bulk_get/fetch |
| **Full setup** | firecrawl_map | Best tool per case | scrapling fetch | stealthy_fetch | Best tool per case |

## Installation

### 1. Scrapling (recommended)

HTTP/browser fetching with browser fingerprint impersonation. Runs locally for free.

```bash
# Using pipx (recommended — isolates from system Python)
pipx install "scrapling[ai]"
scrapling install

# Register MCP server (global)
claude mcp add scrapling -s user -- $(which scrapling) mcp
```

With pip only:
```bash
pip install "scrapling[ai]" && scrapling install
claude mcp add scrapling -s user -- $(which scrapling) mcp
```

### 2. Firecrawl (recommended)

Cloud scraping service. `firecrawl_map` discovers all URLs on a domain in seconds.

```bash
# 1. Get API key: https://firecrawl.dev/app/api-keys (free tier: 500 credits)
# 2. Register MCP server
claude mcp add firecrawl -s user \
  --env FIRECRAWL_API_KEY="your-key-here" \
  -- npx -y firecrawl-mcp
```

### 3. Claude in Chrome (optional)

Real Chrome browser automation as a last resort. For sites that block everything else.

- Install from Chrome Web Store: "Claude in Chrome"
- No separate MCP registration needed (extension auto-connects)

## Verifying installation

After installing, **restart Claude Code** and verify in a new session:

```
# Check registered MCP servers
claude mcp list

# Or verify tools exist via ToolSearch in a new session
ToolSearch: "scrapling"
ToolSearch: "firecrawl"
```

## Common pitfalls

- Omitting `-s user` (global registration) causes tools to register only to the project, making them unavailable to subagents
- Firecrawl free tier is 500 credits (one-time). `map` costs 1 credit flat, `scrape` costs 1 credit/page
- Scrapling's `stealthy_fetch` may need a browser download on first run
- If `scrapling install` fails, ensure you have sufficient disk space for Playwright browsers (~500MB)
