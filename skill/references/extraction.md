# Open Notebook — URL-extraction engines & Crawl4AI

Open Notebook (via the `content-core` library) picks a **URL-extraction engine**. Set it in
**Settings → Content Processing**, or via a variable (`CCORE_URL_ENGINE`).

| Engine | Rendering | Key/API | When |
|---|---|---|---|
| **Auto** (recommended) | tries Firecrawl → Jina → Crawl4AI → Simple | — | Default |
| **Firecrawl** | paid service (free tier), powerful | `FIRECRAWL_API_KEY` | Hard pages, high volume |
| **Jina** | Reader, good for articles → clean text | `JINA_API_KEY` | Articles |
| **Crawl4AI** | **browser rendering (Chromium)**, JS pages | `CRAWL4AI_API_URL` (+ `CRAWL4AI_API_TOKEN`) or local install | SPA / JS-loaded content |
| **Simple** | HTTP fetch + BeautifulSoup | — | Static HTML only |

On a non-JS page, **Simple and Crawl4AI return near-identical text** — the difference only shows on
JS-heavy pages. Reliable tell: a `[FETCH] ↓ <url>` line appears in the Crawl4AI server logs **only**
in Crawl4AI mode.

## Pointing Open Notebook at a remote Crawl4AI server

Set the **root** of your Crawl4AI server, and — if it is token-protected — its token:

- `CRAWL4AI_API_URL` = `{{CRAWL4AI_URL}}` (the server root)
- `CRAWL4AI_API_TOKEN` = `{{CRAWL4AI_API_TOKEN}}`

then pick the `Crawl4AI` engine (or `Auto`). Since **Open Notebook v1.15 / content-core 2.2**
(content-core PR #81, ON #1269), Open Notebook sends the token natively as a `Bearer` header, so a
Crawl4AI 0.9+ server — **secure-by-default** (token required) — works directly. **No reverse proxy.**

`OPEN_NOTEBOOK_ENABLE_CRAWL4AI` only controls the *local* Crawl4AI install — leave it `false` when
using a remote `CRAWL4AI_API_URL`.

> **Fallback for ON < v1.15 / content-core < 2.2 only** — older versions post to
> `{CRAWL4AI_API_URL}/crawl` **without** an `Authorization` header and expose no token variable, so a
> secure-by-default Crawl4AI returns **401** (a silent failure that falls back to Simple). There, run
> Crawl4AI without auth on a trusted internal network, or put an authenticating reverse proxy in front
> of it that injects the `Bearer` token, and point `CRAWL4AI_API_URL` at that proxy.

## content-core locally (standalone MCP)

The `content-core` MCP running on your machine reads the same variables as the library. Since
**content-core 2.2** it also sends `CRAWL4AI_API_TOKEN` as a `Bearer` header, so it can reach a
token-protected Crawl4AI directly; on **< 2.2** it can't (no token variable) — use
`simple`/`firecrawl`/`jina`, or install Crawl4AI locally
(`pip install content-core[crawl4ai]` + `python -m playwright install --with-deps`).

> For calling a Crawl4AI server directly (MCP / n8n / REST), see the companion kit
> [`crawl4ai-skill-mcp`](https://github.com/sergeesteves/crawl4ai-skill-mcp).
