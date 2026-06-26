# wiki-base

**A Karpathy-style [LLM Wiki][gist] for Claude Code: ingest code, Confluence, URLs, PDFs, and transcripts into a cited, self-maintaining markdown wiki — then query it with citations. Pure markdown + agentic search. No RAG, no vectors, no server.**

## Why compile-once beats RAG

RAG re-derives knowledge from your raw documents on *every* query and accumulates nothing. An LLM Wiki is the inverse: knowledge is **compiled once** into structured markdown and **kept current** as a compounding artifact, so each question starts from everything you've already learned. The usual reason human wikis rot — the bookkeeping (cross-references, updates, dedup) — is exactly the tedious work an LLM does for free, touching 10–15 files in a single pass. The maintenance cost is near zero, so the wiki actually stays maintained.

This is faithful to Andrej Karpathy's [LLM Wiki gist][gist] ("just a git repo of markdown files"). The deep thesis and verbatim source quotes live in [DESIGN.md](./DESIGN.md) at zero context cost.

## Install

**As a Claude Code plugin** (if you fork it, swap `matantsach` for your own GitHub owner):

```
/plugin marketplace add matantsach/wiki-base
/plugin install wiki-base@wiki-base
```

The install string is `PLUGIN@MARKETPLACE`; both happen to be `wiki-base` here for a memorable `wiki-base@wiki-base`.

**As a clone-and-go starter** (the same repo *is* a starter wiki):

```
git clone https://github.com/matantsach/wiki-base
# open the folder in Claude Code, then:
/wiki-base:init        # bootstrap a wiki for your project
```

Or just explore the shipped demo `.wiki/` (a worked example) before initializing your own.

## Skills

| Skill | Role | What it does |
|-------|------|--------------|
| `/wiki-base:init` | maintain | Bootstrap or reconfigure a wiki — pick where it lives and its categories, or reset the shipped demo. |
| `/wiki-base:ingest` | ingest | Add a source (code, Confluence, URL, PDF, transcript) **or** revise a page in place (update folded in). One ingest touches ~10–15 pages and runs a backlink audit. Codebase git-sync lives in [`skills/ingest/references/codebase-sync.md`](./skills/ingest/references/codebase-sync.md). |
| `/wiki-base:query` | read | Answer a question **with citations** to source pages, raw files, or URLs. Reads `index.md` first; read-only; says "not in the wiki" rather than inventing. |
| `/wiki-base:lint` | maintain | Whole-wiki health check (broken links, orphans, contradictions, stale claims) **or** fact-check a single page's citations (audit folded in). |

Four skills, one trigger each. `update` folds into `ingest`, `audit` into `lint`, codebase `sync` into `ingest` — fewer overlapping triggers, every capability preserved.

## Tiny worked example

```
You:    /wiki-base:ingest ./docs/architecture.pdf
Claude: reads the PDF, drafts a cited source page in .wiki/wiki/pages/,
        updates index.md and related concept pages, appends a log entry.

You:    /wiki-base:query "why did we pick event sourcing?"
Claude: reads index.md, greps the matching pages, answers with footnote
        citations to the source page / raw file / URL — or says it isn't in the wiki.
```

## Storage model

The wiki defaults to **`.wiki/` inside your project**, versioned alongside the code (you get history, branching, and collaboration for free). The layout:

```
.wiki/
├── SCHEMA.md        # discovery anchor + the canonical spec (conventions + citation grammar)
├── raw/             # immutable sources — the LLM reads, never modifies
├── assets/          # downloaded images / PDFs / attachments
└── wiki/            # LLM-owned layer
    ├── index.md     # categorized catalog — read first on every query (the RAG replacement)
    ├── log.md       # append-only journal: "## [YYYY-MM-DD] <op> | <title>"
    ├── overview.md  # single evolving synthesis
    └── pages/       # FLAT, slug-named pages (source / entity / concept / analysis)
```

**Redirect to a central/external dir** (e.g. `~/wikis/your-project`): set the `Path:` field in `.wiki/SCHEMA.md` to an absolute path. The in-project `.wiki/SCHEMA.md` always stays as a breadcrumb so every skill can locate the wiki; `raw/`, `assets/`, and `wiki/` then live at the external root. One config field, no code change. (The breadcrumb is our design choice; the canonical template is [`skills/init/references/schema-template.md`](./skills/init/references/schema-template.md).)

**The shipped `.wiki/` is a throwaway demo.** Clear it before real use: `rm -rf .wiki` then `/wiki-base:init`, or just run `/wiki-base:init` — it detects the demo `SCHEMA.md` and offers to reset.

## Make the wiki discoverable in your projects

When you `/wiki-base:init` a wiki in a repo, init offers to drop a small **managed pointer** into that repo's `CLAUDE.md` (between `<!-- wiki-base:begin -->` / `<!-- wiki-base:end -->` markers — created if absent, never clobbering your content) so Claude **proactively consults the wiki** instead of waiting for an explicit `/wiki-base:query`. It mirrors the block to `AGENTS.md` only if you already use one (other agents read `AGENTS.md`, not `CLAUDE.md`). It deliberately does *not* also write a `.claude/rules/` copy — a no-`paths` rule loads at the same priority as `CLAUDE.md`, so two copies would double-load the same text.

**Check it in.** In-project wikis are "just a git repo of markdown files": `git add .wiki && commit` and a clone carries the wiki — and the committed pointer — for free, with PR-reviewable knowledge diffs. If `raw/`/`assets/` would carry large binaries or sensitive material, init can add `.wiki/assets/*` to your `.gitignore`. (A central/external wiki isn't checked in — only the `.wiki/SCHEMA.md` redirect stub is.)

**Teammates without the plugin** can still read the wiki — it's plain markdown. For the full skill workflow without installing the plugin, init can vendor the four skills into `.claude/skills/` (opt-in; they lose the `wiki-base:` prefix and can drift from the plugin).

## Scope: lean by design

No vector database, no embeddings, no RAG pipeline, no server. Retrieval is `index.md` + agentic `grep`/`glob`/`read`, loaded just-in-time. This works comfortably to **~100 sources / hundreds of pages**. Anthropic's context-engineering guidance frames this as a hybrid — agentic, just-in-time context *augments* embedding-based retrieval, and on-demand tool/data loading can cut token use by up to **98.7%** (150,000 → 2,000 tokens in Anthropic's [code-execution-with-mcp][cem] example). See [DESIGN.md](./DESIGN.md) for the full treatment ([context-engineering][ctx]).

If a wiki ever outgrows `index.md`, a markdown search engine (e.g. `qmd`) is an opt-in CLI/MCP upgrade — never required, never bundled.

## Optional automation

Freshness for codebase wikis is documented, never auto-installed: a git `post-commit` hook, a headless CI staleness check, or an in-session `Stop`-hook nudge. Copy-paste recipes live in [`skills/ingest/references/codebase-sync.md`](./skills/ingest/references/codebase-sync.md). Defaults stay pure-markdown + agentic search.

## Cross-agent portability

`AGENTS.md` is a symlink to `CLAUDE.md` so there is exactly one source of truth. Note this **reverses** the direction in Claude Code's memory docs (which symlink `CLAUDE.md → AGENTS.md`); we keep `CLAUDE.md` as the real file because this is a Claude-Code-first plugin and the authoring guidance targets it. The content is tool-agnostic, so the direction is cosmetic.

## Local testing

```
claude --plugin-dir ./        # load the plugin from this checkout
claude plugin validate ./     # validate the manifests
```

## Credits & license

Built on public ideas, not a fork — see [NOTICE](./NOTICE):

- **Andrej Karpathy** — the [LLM Wiki idea][gist] (the north star).
- **[kfchou/wiki-skills](https://github.com/kfchou/wiki-skills)** (MIT) — the skill set, directory schema, and citation grammar.
- **[yysun/awesome-agent-world](https://github.com/yysun/awesome-agent-world)** git-wiki (MIT) — the git-checkpoint codebase-sync pattern.

Original files are [MIT](./LICENSE).

[gist]: https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f
[ctx]: https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
[cem]: https://www.anthropic.com/engineering/code-execution-with-mcp
