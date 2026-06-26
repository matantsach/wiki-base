# Ingest: code (the wiki-from-codebase pattern)

Read a repository and compile it into a navigable, cited wiki — optionally driven by
git so refreshes are incremental. Source of truth is the **git-tracked code at a
pinned commit**, not the wiki. Authoring conventions for codebase pages (slug
prefixes `mod-` / `api-` / `dec-` / `flow-`, reader-question index, "five deep pages
beat fifty shallow ones") live in `skills/init/references/codebase.md`; this file
covers the **ingest mechanics**.

## Readers

- **Code on disk** → native **Read / Grep / Glob**. No connector, no infra.
- **GitHub-hosted code** (private, or to avoid cloning) → the **official GitHub MCP
  server** (opt-in). Add it yourself; do not ship an active `.mcp.json` that would
  trigger auth on load:
  ```
  claude mcp add --transport http github https://api.githubcopilot.com/mcp/
  ```
  Repo: https://github.com/github/github-mcp-server

## Raw layer for code = git, not a copy

Do **not** duplicate source files into `.wiki/raw/`. The repo's own git history is
the immutable source layer: cite a live line as `path/to/file.ts:42@<short-SHA>`.
Copy into `.wiki/raw/` only **out-of-tree** artifacts you can't otherwise pin —
exported ADRs, external specs, architecture screenshots (binaries → `.wiki/assets/`).

## Ingest order (work outward from the edges)

1. Top-level docs (README, `docs/`) and decision records (`adr/`, `docs/decisions/`).
2. Build / deploy / CI configs.
3. Dependency manifests and entry points (`main`, server bootstrap, CLI).
4. Module internals only as deep as a question demands.

Two decision flavors: **Sourced** (formal ADRs) and **Reconstructed** (inferred from
the code — often the more valuable ones). Keep README boundaries clean: README =
operational (setup/test); wiki = rationale/relationships/tradeoffs. Link, don't duplicate.

## Codebase-only frontmatter (enables stale-detection; credit: yysun git-wiki)

```yaml
source_paths: [src/api/routes.ts, src/api/auth.ts]   # which code files this page documents
stale: false                                          # set true when a source file is deleted
```

`index.md` frontmatter carries the checkpoint:
```yaml
last_commit: <full-SHA>   # advanced only after a complete changeset
```

## Keeping a code wiki current (sync)

Refreshing an existing code wiki from the git diff since the last checkpoint — plus the
OPTIONAL post-commit / CI / `Stop`-hook automation — is the **sync** operation, folded into
`ingest`. It lives in one place to avoid drift:
[`references/codebase-sync.md`](./codebase-sync.md). Read that when you are updating a code
wiki rather than ingesting a brand-new source; it owns the `last_commit` checkpoint flow
(`git diff --name-status -M`), the `source_paths[]` → page mapping, deletion → `stale`
handling, and the documented-not-active automation recipes.

Then return to `SKILL.md` Mode A steps 3-6 (write page → index → entities/concepts →
log + backlink audit).
