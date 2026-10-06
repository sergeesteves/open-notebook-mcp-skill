# Open Notebook — wiring the two MCP servers

Both servers run over **stdio via `uvx`** (provided by Astral's [`uv`](https://github.com/astral-sh/uv):
`winget install astral-sh.uv`, `brew install uv`, or `pip install uv`). ⚠️ **Never hard-code a
secret**: the password goes through an environment variable / credential, never a version-controlled
file.

## 1. `open-notebook-mcp` — pilot the instance

- Package: [`open-notebook-mcp`](https://pypi.org/project/open-notebook-mcp) (PyPI)
- Repo: [`Epochal-dev/open-notebook-mcp`](https://github.com/Epochal-dev/open-notebook-mcp)
- Talks to the **Open Notebook REST API** (default port `5055`). Point it at `{{OPEN_NOTEBOOK_URL}}`
  — the **root**, without `/api`.

**JSON config (Claude Desktop / VS Code)**:

```json
{
  "mcpServers": {
    "open-notebook": {
      "command": "uvx",
      "args": ["--with", "mcp<2", "open-notebook-mcp"],
      "env": {
        "OPEN_NOTEBOOK_URL": "{{OPEN_NOTEBOOK_URL}}",
        "OPEN_NOTEBOOK_PASSWORD": "<from a secrets manager>"
      }
    }
  }
}
```

**Claude Code (CLI)** — run in an interactive terminal:

```bash
claude mcp add open-notebook \
  -e OPEN_NOTEBOOK_URL={{OPEN_NOTEBOOK_URL}} \
  -e OPEN_NOTEBOOK_PASSWORD='<password>' \
  -- uvx --with 'mcp<2' open-notebook-mcp
```

**Exposed tools**: Notebooks (list/get/create/update/delete) · Sources (list/get/add link|file|text/update/delete) ·
Notes (list/get/create/update/delete) · Chat (create session/send/history/list) · Search (vector +
full-text, filterable by notebook) · Models (list/get/create/update) · Settings (get/update).

## 2. `content-core` — standalone extraction / summarization

- Repo: [`lfnovo/content-core`](https://github.com/lfnovo/content-core) (the extraction engine Open
  Notebook uses internally; exposed here **on its own**, it does not talk to Open Notebook).
- Tools: `extract_content`, `summarize_content`.

**Claude Code (CLI)**:

```bash
claude mcp add content-core -- uvx --from content-core content-core-mcp
```

> If the entry point differs by version: `uvx --from content-core content-core mcp` (subcommand), or
> install the MCP extra. The server reads the same variables as the library (`FIRECRAWL_API_KEY`,
> `JINA_API_KEY`, `CRAWL4AI_API_URL`, `CRAWL4AI_API_TOKEN`, `CCORE_URL_ENGINE`…).

## Gotchas

- **⚠️ `mcp<2` is mandatory**: `open-notebook-mcp` imports `mcp.server.fastmcp.FastMCP`, which the
  2.x `mcp` SDK renamed → `ModuleNotFoundError` on start, no tools exposed. The package does not cap
  its dependency, so pin it yourself: `uvx --with 'mcp<2' open-notebook-mcp` (already wired above).
- **⚠️ URL is the root, WITHOUT `/api`**: the MCP appends `/api/...` to every endpoint
  (`make_request("GET", "/api/notebooks")`). The base must be `http://host` — an extra `/api` yields
  `/api/api/notebooks` → **404**.
- **⚠️ v1.15 — source creation is multipart-only**: on Open Notebook **v1.15**, `POST /api/sources`
  accepts **`multipart/form-data` only**; a JSON body returns
  `422 {"type":"missing","loc":["body","type"]}`. JSON clients must post to **`POST /api/sources/json`**
  instead. Consequence: `open-notebook-mcp` (Epochal-dev) posts JSON to `/api/sources`, so **adding a
  source is broken on v1.15** (list / search / get still work). Until the MCP is patched: add sources
  from the Open Notebook UI, or call `/api/sources/json` directly.
- **`notebook_id` vs `notebooks`**: on create payloads, `notebooks: [id]` is the recommended form
  (multi-notebook). `notebook_id` still works (the API converts it to `notebooks: [id]`), but **never
  send both at once**.
- **Password**: a protected instance answers `401 Missing authorization header` when the password is
  absent → set `OPEN_NOTEBOOK_PASSWORD`. An instance without password protection needs no token.
- **Non-interactive session**: an MCP can't be added on the fly — provide the command and let the
  user run it in an interactive terminal.
- **Don't confuse the two**: to **manage notebooks** → `open-notebook-mcp`; to **extract a URL on
  the fly** → `content-core`.
