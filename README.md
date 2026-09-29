# pod-jupyter

The `jupyter` candy of the OpenCharly candy library, as a standalone repo
(kind-prefixed naming). It ships JupyterLab with a CRDT MCP server for
agent-driven notebook editing.

## What it provides

Installs JupyterLab into the pixi default environment (the `jupyter-lab` binary),
adds the spaCy `en_core_web_sm` NLP model and the `jupyterlab-quarto` extension,
and serves notebooks on port `8888` from the `/workspace` volume under
supervisord. A bundled MCP server (composed from `jupyter-mcp`) exposes
`notebook_*` / `cell_*` tools at `/mcp`, so an agent can create and edit notebooks
over the Model Context Protocol.

| Property | Value |
|---|---|
| Service | `jupyter` (`jupyter lab --ip=0.0.0.0 --port=8888`, `restart: always`) |
| Port | `8888` (JupyterLab HTTP + MCP at `/mcp`) |
| Requires | `layer-supervisord`, `plugin-mcp` (the out-of-process `mcp:` check verb) |
| Candy | `layer-jupyter-mcp` (the CRDT MCP extension) |
| Volume | `workspace` at `/workspace` |
| Env | `MCP_SERVER_NAME=jupyter` |
| mcp_provide | `jupyter` at `http://{{.ContainerName}}:8888/mcp` (http transport) |

## How to use it

```bash
charly box build jupyter
charly config jupyter
charly start jupyter
# open http://localhost:8888
```

The MCP server is reachable at `http://localhost:8888/mcp` (Streamable HTTP).
Register it with an MCP client, e.g. Claude Code:

```bash
claude mcp add --transport http --scope project jupyter http://localhost:8888/mcp
```

## The MCP tools

| Category | Tools |
|---|---|
| Notebook management | `notebook_list`, `notebook_create`, `notebook_get`, `notebook_watch`, `notebook_list_users` |
| Cell operations (CRDT) | `cell_get`, `cell_update`, `cell_insert`, `cell_delete`, `cell_execute` |
| Read-only diagnostic | `room_list` |

Clients do not manage CRDT rooms: the server auto-attaches each call to the room
for that path, or creates one. Idle rooms are swept after
`MCP_ROOM_IDLE_TIMEOUT_SEC` (default `600`s).

## Layout

- `charly.yml` — the `jupyter:` candy entity plus its `skill:` entity.
- `pixi.toml` / `pixi.lock` — the Python environment (JupyterLab, the data-science
  stack, spaCy).
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `CHANGELOG/` — per-CalVer history.
- `README.md` — this user overview.

## Related

- Owning skill: `/charly-jupyter:jupyter` — the box properties, the package
  matrix, and verification.
- `/charly-jupyter:jupyter-mcp` — the CRDT MCP server extension and its tool
  catalog.
- `/charly-jupyter:jupyter-ml` / `/charly-jupyter:jupyter-ml-notebook` — GPU
  variants that inherit the same MCP suite.
- [`opencharly/charly`](https://github.com/opencharly/charly) — the charly CLI and image builder
- [`opencharly/opencharly`](https://github.com/opencharly/opencharly) — the umbrella
