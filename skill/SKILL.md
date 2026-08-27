---
name: open-notebook
description: >
  Drive Open Notebook (notebooks, sources, notes, chat, vector search) and handle content
  extraction/summarization through its two MCP servers — open-notebook-mcp (pilot a self-hosted or
  local instance) and content-core (standalone extraction/summarization). Use whenever you want to
  create/list notebooks, add sources (URL/file/text), generate notes, search an Open Notebook base,
  or pick the right URL-extraction engine (Auto/Firecrawl/Jina/Crawl4AI/Simple).
license: MIT
---

# Open Notebook — pilot + extraction (MCP)

[Open Notebook](https://github.com/lfnovo/open-notebook) is an open-source, privacy-focused
alternative to NotebookLM: notebooks, sources, notes, chat over your research base, podcasts. You
drive it from Claude (or any MCP client) through **two distinct MCP servers** — don't mix them up:

| MCP | Role | When |
|---|---|---|
| **`open-notebook-mcp`** | **Pilot** your instance: notebooks, sources, notes, chat, search, models, settings | Manage/query your Open Notebook base |
| **`content-core`** | **Extract / summarize** any URL / file / text (standalone — does not talk to Open Notebook) | One-off extraction without a notebook |

Connection, config and the full tool list → [`references/mcp-setup.md`](references/mcp-setup.md).

## Connecting to your instance

- **URL**: `{{OPEN_NOTEBOOK_URL}}` — the **root**, without `/api` (the MCP appends `/api` itself).
  Locally that's typically `http://localhost:5055`.
- **Password**: if the instance has password protection enabled, `open-notebook-mcp` needs
  `OPEN_NOTEBOOK_PASSWORD` (env var / credential — never in a file or commit).
- **AI providers & hosting are out of scope here** — Open Notebook works with 18+ providers and can
  run locally or self-hosted; this skill only needs the URL (and password, if any).

## What you can ask for (pilot)

- "List my notebooks" · "Create a notebook *X*"
- "Add this URL / this PDF / this text as a source in *X*"
- "Summarize the key points into a note in *X*"
- "Search my notebooks for anything about *Y*" (vector + full-text search)
- "Start a chat on this source / this topic"

## Rules

1. **Never hard-code the instance URL or the password** in code, logs or a commit — env vars /
   credential only.
2. **URL-extraction engine choice**: `Auto` (recommended default); `Crawl4AI` for JS-heavy pages
   (browser rendering); `Simple` only for static HTML. See [`references/extraction.md`](references/extraction.md).
3. **Don't confuse the two servers**: to **manage notebooks** → `open-notebook-mcp`; to **extract a
   URL on the fly** → `content-core`.
