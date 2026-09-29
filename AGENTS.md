# AGENTS.md — pod-jupyter

Standalone candy repo for the `jupyter` candy — JupyterLab with a CRDT MCP server
for agent-driven notebook editing, on `8888`. The candy lives in `charly.yml` at
the repo root plus its pixi environment.

Canonical files:

- `charly.yml` — the `jupyter:` candy entity (description, `require`, `candy`,
  `env`, `distro`, `port`, `mcp_provide`, `volume`, `service`, `plan`) and its
  `skill:` entity.
- `pixi.toml` / `pixi.lock` — the Python environment (JupyterLab, the
  data-science stack, spaCy).
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `CHANGELOG/` — per-CalVer history.
- `README.md` — user overview only; never agent guidance.

## Load these skills first (R0)

- `/charly-jupyter:jupyter` — the owning skill: box properties, the package
  matrix, and verification. Load before editing, building, deploying, or
  troubleshooting this candy.
- `/charly-jupyter:jupyter-mcp` — the CRDT MCP server extension and its tool
  catalog.
- `/charly-pod:pod` — the `kind: pod` / deploy schema reference (this candy is
  composed into a box; tree-position nesting, volumes, ports).
- `/charly-check:check` — the check/R10 framework: the `check:` step verbs, the
  `mcp:` verb, and `charly check run <bed>`.
- `/charly-image:layer` — the candy authoring reference (`charly.yml` schema,
  `plan:` step verbs, `mcp_provide`, service declarations).
- `/charly-internals:git-workflow` — before any git/PR action.

## Build / validate / test

- `charly box validate` at the repo root — the structural check: the manifest
  must parse and validate at the installed charly.
- The live R10 witness is a composing box's `check` bed; the candy's own `check:`
  steps assert the pixi `jupyter-lab` binary, the spaCy model load, the mounted
  `/workspace` volume, the running `jupyter` service, the `/api` `200`, and the
  `mcp:` `ping` / `list-tools` / `call` surface (including the CRDT invariants).
- The merge gate is the **org-wide** `charly/pr-validator` (required check
  `validate / validate`, defined in `opencharly/.github`); this repo has **no**
  per-repo candy gate. Its only workflow file is
  `.github/workflows/tag-on-merge.yml`.

## Modify this repo

- Edit the `jupyter:` candy entity in `charly.yml`; the `skill:` entity in the
  same file is the owning skill's source — a candy change and its skill change
  land together.
- Python dependency changes belong in `pixi.toml` / `pixi.lock`, not in the plan.
- The MCP server name is the `MCP_SERVER_NAME` env (`jupyter` by default); the
  `mcp_provide` URL, the service, and the `port:` field must stay in step.
- The `workspace` volume at `/workspace` is the persistent notebook store; keep
  the service's `--notebook-dir` in step.
- The `skill:` entity is the source for `/charly-jupyter:jupyter`; never edit the
  generated `SKILL.md` — regenerate it.

## Landing

- PR-only. Every change lands through a pull request; the org-required
  `charly/pr-validator` validates the diff and body and arms native auto-merge on
  PASS. Direct pushes to `main` are blocked.
- History lives in `CHANGELOG/` (written by `tag-on-merge` at merge time); the PR
  body IS the changelog.
- The authoritative rulebook is the umbrella `AGENTS.md` in
  `opencharly/opencharly` and `charly/AGENTS.md` in the charly repo. Do not
  restate its rules here.
