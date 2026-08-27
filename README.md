# open-notebook-mcp-skill

An installable **Agent Skill** (+ setup guide) for driving [**Open Notebook**](https://github.com/lfnovo/open-notebook)
from **Claude** and other MCP clients — through two MCP servers: **`open-notebook-mcp`** (pilot your
instance) and **`content-core`** (standalone content extraction/summarization).

Open Notebook is an open-source, privacy-focused alternative to Google's NotebookLM (notebooks,
sources, notes, chat, vector search, podcasts). This repo **reimplements nothing**: it teaches an
assistant **how to call your Open Notebook instance over MCP**, including the real gotchas.

It is **agnostic** of where your instance runs (local, self-hosted, anywhere) and of which AI
providers it uses — it only needs the instance URL and, if enabled, a password.

## The two MCP servers (don't mix them up)

| MCP | Role | When |
|---|---|---|
| **`open-notebook-mcp`** | **Pilot** your instance: notebooks, sources, notes, chat, search, models, settings | Manage/query your Open Notebook base |
| **`content-core`** | **Extract / summarize** any URL / file / text (standalone — does not talk to Open Notebook) | One-off extraction without a notebook |

## Installation

**Agent Skill** — copy the `skill/` folder into your tool's skills directory, e.g.:

```bash
cp -r skill ~/.claude/skills/open-notebook
```

**Wire the MCP servers.** Both run over stdio via `uvx` (from [`uv`](https://github.com/astral-sh/uv)).

Claude Code (CLI) — run in an interactive terminal:

```bash
claude mcp add open-notebook \
  -e OPEN_NOTEBOOK_URL={{OPEN_NOTEBOOK_URL}} \
  -e OPEN_NOTEBOOK_PASSWORD='<from a secrets manager>' \
  -- uvx --with 'mcp<2' open-notebook-mcp

claude mcp add content-core -- uvx --from content-core content-core-mcp
```

Claude Desktop / VS Code (JSON) — see [`skill/references/mcp-setup.md`](skill/references/mcp-setup.md)
for the `mcpServers` block and the full tool list.

> Open Notebook also ships its own [MCP integration docs](https://github.com/lfnovo/open-notebook/blob/main/docs/5-CONFIGURATION/mcp-integration.md)
> if you prefer its official path.

## Configuration (you provide it)

- `{{OPEN_NOTEBOOK_URL}}` = the **root** URL of your instance — **without** `/api` (the MCP appends
  `/api` itself). Locally that's typically `http://localhost:5055`.
- **Password**: if your instance has password protection enabled, `open-notebook-mcp` needs
  `OPEN_NOTEBOOK_PASSWORD`.

> ⚠️ **Never** hard-code your instance URL, password or any API key in a version-controlled file —
> use environment variables or your client's credential store.

## Two gotchas that will bite you

- **`mcp<2` is mandatory** for `open-notebook-mcp` — the 2.x `mcp` SDK renamed `FastMCP` and the
  server crashes on start with no tools exposed. Launch it with `uvx --with 'mcp<2'` (as above).
- **URL is the root, without `/api`** — the MCP adds `/api/...` to every call. `http://host` → OK;
  `http://host/api` → `/api/api/...` → 404.

See [`skill/references/mcp-setup.md`](skill/references/mcp-setup.md) and
[`skill/references/extraction.md`](skill/references/extraction.md) for the full details, the tool
lists, and how Open Notebook's URL-extraction engines (Auto/Firecrawl/Jina/Crawl4AI/Simple) work.

## Credits & license

See [ATTRIBUTION.md](ATTRIBUTION.md). Open Notebook, content-core and open-notebook-mcp are the work
of their respective authors (Open Notebook & content-core: [@lfnovo](https://github.com/lfnovo);
open-notebook-mcp: [Epochal-dev](https://github.com/Epochal-dev)). This kit is licensed under **MIT**.
