# codebase-sync — keep a codebase wiki current with git

This is the **sync** operation for codebase wikis, folded into `ingest`: instead of "read a new
source," you read the *git diff since the last sync* and update the pages it touched. The pattern
is yysun's git-checkpoint refresh — git-tracked code is the source of truth, the wiki is derived.

Part 1 is the by-hand operation. Part 2 is OPTIONAL automation that only decides *when* Part 1
runs — it is documented, never auto-installed, and adds zero infrastructure.

## Part 1 — git-checkpoint incremental refresh

The wiki tracks a checkpoint commit; sync replays everything that changed since then.

1. **Find the checkpoint.** Read `last_commit` (a full SHA) from `index.md` frontmatter.
2. **Find HEAD.** `git rev-parse HEAD` — the target you are syncing up to.
3. **Diff.**
   - `last_commit` set → `git diff --name-status -M <last_commit> HEAD` (the `-M` makes it
     rename-aware) for added / modified / renamed / deleted files.
   - `last_commit` absent (first sync) → broad pass over `git ls-files`; this is effectively an
     initial ingest of the whole tree.
4. **Map files → pages.** Each codebase page's `source_paths[]` frontmatter lists the code files it
   documents. Update or create only the affected pages — run the normal ingest steps for each, and
   leave unrelated pages alone.
5. **Handle deletions.** A page whose source file was DELETED gets `stale: true` plus a one-line
   note. **Never auto-delete a page** — a human decides whether the knowledge still matters.
6. **Advance the checkpoint.** Set `index.md` `last_commit:` to HEAD **only after the whole
   changeset is processed**, so an interrupted run re-runs cleanly from the old checkpoint.
7. **Log it.** Append `## [YYYY-MM-DD] sync | <short_sha>..<short_sha>` to `log.md`, listing pages
   written / updated / marked stale.

**Rules.**
- Operate only on **committed** content. Ignore staged and unstaged working-tree changes — they
  aren't durable and shouldn't drive the wiki.
- If git metadata is unavailable (no repo, or a shallow clone with no base commit), **stop and tell
  the user** — do NOT fall back to scanning the filesystem. A filesystem scan can't tell you what
  *changed*, which is the whole point of sync.
- Verify volatile values (counts, names, config defaults) against the live file at HEAD, never from
  memory.

## Part 2 — OPTIONAL automation (documented, never auto-installed)

Part 1 runs by hand inside a Claude Code session. If you want freshness without remembering to run
it, copy in one of the recipes below. **None ship active.**

### Claude Code hooks are NOT git hooks
Claude Code's lifecycle hooks (`SessionStart`, `PreToolUse`, `PostToolUse`, `Stop`, …) fire on
events *inside a Claude session*. There is **no `post-commit` event** — committing happens in git,
outside the session. So "refresh on every commit" needs a real git hook or CI, not a Claude Code
hook.

### Outside-session: a git `post-commit` hook (the git-commit-triggered pattern)
A real git hook at `.git/hooks/post-commit` can invoke headless Claude Code. Default to a
**non-mutating** staleness check that warns and never blocks the commit:

```sh
#!/bin/sh
# .git/hooks/post-commit — OPTIONAL. Copy in by hand, then: chmod +x .git/hooks/post-commit
# Warn (never fail) when the new commit may have staled .wiki/ pages.
git diff --name-only HEAD~1 HEAD \
  | claude --bare -p "Given these changed files, list which .wiki/ pages look stale and why. Cite page paths. Do not edit anything." \
      --output-format json \
  | jq -r '.result'
```

`--bare` skips auto-discovery (hooks, skills, plugins, MCP, CLAUDE.md) so the hook behaves the same
on every machine; `--output-format json` exposes `total_cost_usd` (cap spend) and the `system/init`
`plugin_errors` array (assert the plugin loaded). A **mutating** variant — opt-in, more trust — lets
Claude edit `.wiki/` and open a PR instead of just warning:

```sh
git diff --name-only HEAD~1 HEAD \
  | claude --bare -p "Update the affected .wiki/ pages for these changes, following .wiki/SCHEMA.md, then summarize the diff." \
      --permission-mode acceptEdits --plugin-dir /path/to/wiki-base
```

Prefer the non-mutating check until you trust it — a post-commit hook that silently rewrites files
surprises teammates.

### CI: a headless staleness gate
Same idea in CI (GitHub Actions, GitLab CI): pipe the branch/PR diff into
`claude --bare -p "…" --output-format json`, parse `.result`, and post it as a warning comment.
Keep it **non-blocking** — fail the job only on `plugin_errors`, not on staleness.

### In-session: nudge yourself to sync
For freshness while you're already working you don't need a git hook. Lean on `ingest`'s subagent
guidance and add a lightweight, **project-scoped** `Stop` hook that notices uncommitted churn once
per turn:

```json
// .claude/settings.json  (project-scoped, NOT ~/.claude — opt-in)
{
  "hooks": {
    "Stop": [
      { "hooks": [ { "type": "command",
        "command": "git status --porcelain | grep -q . && echo 'Working tree changed — consider /wiki-base:ingest (sync) to refresh .wiki/.'" } ] }
    ]
  }
}
```

Prefer `Stop` over `PostToolUse` with an `Edit|Write` matcher: `PostToolUse` misses Bash-driven
changes (codegen, `git mv`, scripts), while `Stop` catches them all once the turn ends. Kill any
hook instantly with `"disableAllHooks": true`.

All of this only orchestrates *when* the Part 1 sync runs. The defaults stay pure markdown +
agentic search — no servers, no database, no background workers.

---

**Sources**
- yysun, *git-wiki* skill — the git-checkpoint codebase-wiki pattern (MIT):
  https://raw.githubusercontent.com/yysun/awesome-agent-world/main/skills/git-wiki/SKILL.md
- yysun, *Bringing the LLM Wiki idea to a codebase*:
  https://dev.to/yysun/bringing-the-llm-wiki-idea-to-a-codebase-22go
- Anthropic — Run Claude Code programmatically (`--bare`, `--output-format json`,
  `--permission-mode`): https://code.claude.com/docs/en/headless
- Anthropic — Automate actions with hooks (`Stop` / `PostToolUse` events, matchers,
  `disableAllHooks`): https://code.claude.com/docs/en/hooks-guide
- Anthropic — Subagents (context isolation for per-source reads):
  https://code.claude.com/docs/en/sub-agents
