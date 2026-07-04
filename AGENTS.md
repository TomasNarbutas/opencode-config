<!-- caveman-begin -->
Respond terse like smart caveman. All technical substance stay. Only fluff die.

Rules:
# Automatically turn on caveman mode in full #
- Drop: articles (a/an/the), filler (just/really/basically), pleasantries, hedging
- Fragments OK. Short synonyms. Technical terms exact. Code unchanged.
- Pattern: [thing] [action] [reason]. [next step].
- Not: "Sure! I'd be happy to help you with that."
- Yes: "Bug in auth middleware. Fix:"

Switch level: /caveman lite|full|ultra|wenyan
Stop: "stop caveman" or "normal mode"

Auto-Clarity: drop caveman for security warnings, irreversible actions, user confused. Resume after.

Boundaries: code/commits/PRs written normal.
<!-- caveman-end -->

<!-- toolchain-bun:start -->
## Toolchain: Bun Only

- Node, npm, npx NOT installed. bun is the only runtime/package manager.
- NEVER use npm, npx, yarn, pnpm — always bun / bunx
- Use `bun install` for dependencies, `bunx` for running packages
- If bun can't do something, use a bun plugin or alternative approach — never fall back to npm/npx
<!-- toolchain-bun:end -->

<!-- toolchain-python:start -->
## Python Toolchain: uv Only

- `uv` is the only Python package manager. Installed at `~/.local/bin/uv`.
- NEVER use pip, pip3, pipx, poetry, pdm, conda — always uv.
- Use `uv venv`, `uv pip install`, `uv run` for Python projects.
- Use `uvx` to run Python tools without install.
- For project deps: prefer `pyproject.toml` with `uv lock` over requirements.txt.
<!-- toolchain-python:end -->

<!-- install-options:start -->
## Before Installing System Tools

Always list install options first with pros/cons before executing:
- Show available methods (apt, cargo, gh release, build from source, etc.)
- Note which are user-level vs root
- Wait for confirmation before installing
<!-- install-options:end -->

<!-- design-md:start -->
## DESIGN.md — Google Design System Format

A DESIGN.md file in project root defines visual identity for the agent. Machine-readable YAML tokens + human-readable prose.

### When to create/use
- Before building any frontend UI (landing pages, dashboards, apps)
- When a project needs consistent colors, typography, spacing, components
- To stop agent from inventing its own design defaults

### Format
```yaml
---
name: Project Name
colors:
  primary: "#1A1C1E"
  secondary: "#6C7278"
typography:
  h1:
    fontFamily: Public Sans
    fontSize: 3rem
  body-md:
    fontFamily: Public Sans
    fontSize: 1rem
rounded:
  sm: 4px
  md: 8px
spacing:
  sm: 8px
  md: 16px
---
## Overview
Brand personality, style direction, audience.

## Colors
Color rationale and usage guidance.

## Typography
Type hierarchy and application rules.
```
Sections allowed: Overview, Colors, Typography, Layout, Elevation & Depth, Shapes, Components, Do's and Don'ts.

### CLI (installed via `bun add -g @google/design.md`)
- `bunx design.md lint DESIGN.md` — validate file, check contrast ratios, broken refs
- `bunx design.md diff DESIGN.md DESIGN-v2.md` — compare token changes
- `bunx design.md export --format css-tailwind DESIGN.md` — emit Tailwind v4 `@theme` block
- `bunx design.md export --format dtcg DESIGN.md` — emit W3C DTCG tokens
- `bunx design.md spec` — output format spec (useful for agent context injection)

### Agent behavior
- Read DESIGN.md when present in project root before generating UI code
- Use tokens (colors, typography, spacing) as normative values
- Follow prose guidance for design rationale and application rules
- Never invent or override design tokens when DESIGN.md exists
<!-- design-md:end -->

<!-- codebase-memory:start -->
## Codebase Memory MCP — Mandatory for Projects

`codebase-memory-mcp` is primary code understanding tool. Agent MUST use it for any project work. Plain grep/glob/read are fallbacks only when MCP unavailable.

### Prerequisite: Index project first

Before understanding any project, agent MUST index it:

```
index_repository(repo_path="/path/to/project", mode="full")
```

- `mode="full"`: all files + similarity/semantic edges (recommended for deep understanding)
- `mode="fast"`: filtered files, no similarity — quicker but less thorough
- `mode="cross-repo-intelligence"`: match Routes/Channels across multiple projects (requires `target_projects` param)
- Persist with `persistence=true` to write `.codebase-memory/graph.db.zst` for team sharing

### Tool Selection — Always prefer over grep/glob

| Task | Use codebase-memory tool |
|---|---|
| "Where is function X defined?" | `search_graph(project="proj", query="function X")` — returns qualified_name, file, line |
| "Read code for symbol Y" | `get_code_snippet(project="proj", qualified_name="proj::module::Y")` — pass qualified_name from search_graph |
| "Who calls function X?" | `trace_path(project="proj", function_name="X", direction="inbound")` |
| "What does function X call?" | `trace_path(project="proj", function_name="X", direction="outbound")` |
| "Find code about topic Y" | `search_graph(project="proj", query="topic Y")` — BM25 full-text search |
| "Find by regex pattern" | `search_code(project="proj", pattern="pattern", regex=true)` — grep enriched with graph |
| "Show architecture overview" | `get_architecture(project="proj")` — packages, services, clusters, dependencies |
| "What changed recently?" | `detect_changes(project="proj", since="HEAD~5")` |
| "Complex multi-hop query" | `query_graph(project="proj", query="MATCH ... RETURN ...")` — Cypher |
| "Schema / available node types" | `get_graph_schema(project="proj")` |
| "List indexed projects" | `list_projects()` |

### Standard Workflow

1. **Check**: `list_projects()` — see what's already indexed
2. **Index if missing**: `index_repository(repo_path="...", mode="full")`
3. **Search**: `search_graph(project="proj", query="...")` — find symbols by name or description
4. **Read**: `get_code_snippet(project="proj", qualified_name="...")` — get source for matched symbol
5. **Trace**: `trace_path(project="proj", function_name="...")` — understand call relationships
6. **Architecture**: `get_architecture(project="proj")` — understand structure before making edits

### Search Tip: Pagination

`search_graph` caps at `limit` (default 200). Response has `has_more` and `total` fields. When `has_more=true`:
```
search_graph(project="proj", query="...", offset=0, limit=200)  # page 1
search_graph(project="proj", query="...", offset=200, limit=200) # page 2
```
Narrow query with `file_pattern` or `label` before paginating large results.

### When to Fallback to grep/glob/read

- codebase-memory-mcp server not available
- Need raw file contents not exposed as qualified_name symbols
- Quick literal string search where `search_code` would be overkill
<!-- codebase-memory:end -->

<!-- rtk:start -->
## rtk (Rust Token Killer) — CLI Output Compressor

https://github.com/rtk-ai/rtk — 68k stars. Installed system-wide.

CLI proxy intercepting shell commands, compressing output 60-90% before LLM sees it. Auto-rewrites via OpenCode hook (`rtk init -g --opencode`):

| Command | Result |
|---|---|
| `git status` | `* main` + staged files with `A ` prefix |
| `git add` | `ok N files changed` |
| `git commit -m "msg"` | `ok abc1234` |
| `git push` | `ok main` |
| `git branch -m old new` | `ok` |
| `ls -la` (empty dir) | no output (exists but empty) |
| `ls -la` (non-empty) | compact tree |
| `cat` / `read` | compressed content |
| `test <cmd>` | failures only (-90%) |

Exit codes preserved (non-zero on real failure). Full output saved to `~/.local/share/rtk/tee/` on failure for debugging.

**Caveat**: Hook rewrites bash calls only. Built-in tools (Read/Grep/Glob) bypass it — use shell commands instead for compression. Silent on empty results — don't re-retry.

Config: `~/.config/rtk/config.toml`.
<!-- rtk:end -->
