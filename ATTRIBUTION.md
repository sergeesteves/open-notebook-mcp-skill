# Attribution

This kit documents how to drive an **Open Notebook** instance and the **content-core** extraction
engine through their MCP servers. It **reimplements nothing** and redistributes no upstream code — it
links to the projects below.

- **Open Notebook** — open-source, privacy-focused NotebookLM alternative, **MIT**:
  <https://github.com/lfnovo/open-notebook> · website <https://www.open-notebook.ai>
  Maintained by Luis Novo ([@lfnovo](https://github.com/lfnovo)). Open Notebook also ships its own
  MCP integration docs: <https://github.com/lfnovo/open-notebook/blob/main/docs/5-CONFIGURATION/mcp-integration.md>
- **open-notebook-mcp** — MCP server that drives an Open Notebook instance over its REST API
  (author: Epochal-dev): <https://github.com/Epochal-dev/open-notebook-mcp> ·
  <https://pypi.org/project/open-notebook-mcp>
- **content-core** — the standalone extraction/summarization engine Open Notebook uses internally,
  exposed here as its own MCP (author: [@lfnovo](https://github.com/lfnovo)):
  <https://github.com/lfnovo/content-core>

The tool lists and gotchas described here come from these projects' public documentation and from
running a standard, unmodified instance. Always verify the exact tools/fields against **your**
version — the MCP tool schema is served by your own MCP client.
