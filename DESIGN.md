# DESIGN.md — why wiki-base is shaped the way it is

This is the **optional deep read** that [`README.md`](./README.md) defers to. The README
stays deliberately short so it costs little context to load; the thesis, the verbatim source
quotes, and the design tradeoffs that justify every decision live here instead. Nothing in this
file is required to *use* wiki-base — it is required only to understand *why* it looks the way
it does. Every claim traces to a source in [`SOURCES.md`](./SOURCES.md); the reference links at
the bottom of this file are the same URLs.

wiki-base is a lean, clone-and-go baseline that implements Andrej Karpathy's **LLM Wiki** idea
as four Claude Code skills — `/wiki-base:init`, `/wiki-base:ingest`, `/wiki-base:query`,
`/wiki-base:lint` — over a plain-markdown store. No vector database, no embeddings, no RAG
pipeline, no server. The wiki defaults to a `.wiki/` directory inside each project and can be
redirected to a central location with one field in `SCHEMA.md`.

---

## 1. The thesis: knowledge as a compiled artifact, not a per-query re-derivation

The whole design hangs on one distinction.

**RAG re-derives knowledge from your raw documents on *every* query and accumulates nothing.**
A retrieval-augmented pipeline embeds your corpus, and at question time it pulls back the
chunks that look nearest to the query, hands them to the model cold, and throws the result away.
The next question starts from the same undigested chunks. Effort does not compound; the system
is exactly as smart on its thousandth query as on its first.

**An LLM Wiki is the inverse.** Knowledge is *compiled once* into structured markdown and then
*kept current* as a living artifact, so every new question starts from everything already
learned. Karpathy frames the wiki as "a persistent, compounding artifact... The knowledge is
compiled once and then kept current, not re-derived on every query." [[gist]] The substrate is
deliberately boring: "The wiki is just a git repo of markdown files. You get version history,
branching, and collaboration for free." [[gist]]

The reason human wikis rot is well known — the bookkeeping. Cross-references go stale, updates
lag, duplicate pages drift apart, and nobody volunteers to reconcile them. Karpathy ties the
idea back to Vannevar Bush's 1945 Memex and notes that the part Bush could never solve was *who
does the maintenance* — and that the LLM is what finally answers that question. [[gist]] That is
the load-bearing insight for wiki-base: the tedious consistency work that kills human wikis is
precisely the work an LLM does for free, touching ten to fifteen pages in a single ingest pass.
When maintenance cost approaches zero, the wiki actually stays maintained, and the compounding
loop closes.

This also aligns with how Anthropic frames context engineering. The right move is not to
pre-load everything; it is to keep "lightweight identifiers (file paths, stored queries, web
links, etc.)" and "dynamically load data into context at runtime using tools." [[ctx]] A
compiled wiki *is* that set of lightweight identifiers: `index.md` plus slug-named pages are the
references, and `grep`/`glob`/`read` are the runtime loaders. The goal of "finding the smallest
possible set of high-signal tokens that maximize the likelihood of some desired outcome" [[ctx]]
is served far better by a distilled, cited page than by a wall of raw retrieved chunks — and it
sidesteps **context rot**, the documented decay in recall "as the number of tokens in the
context window increases." [[ctx]]

So the thesis in one line: **compile knowledge once, keep it current, and retrieve it
just-in-time** — instead of paying to rebuild understanding from scratch on every query.

---

## 2. The three-layer model

wiki-base stores knowledge in three layers, each with a different owner and a different mutability
rule. The separation is the spine of the whole system; collapsing any two layers reintroduces a
failure mode.

**Layer 1 — `raw/` (immutable sources, the source of truth).**
Everything ingested lands here untouched: code snapshots, fetched URLs rendered to markdown,
PDFs, transcripts, exported Confluence pages. The LLM **reads `raw/` but never modifies it.**
This is what makes citations verifiable: a `lint` audit can re-open the cited `raw/` file and
check that the claimed quote and locator actually exist. If the model could rewrite its own
sources, every citation would be circular. Immutability is the anchor that keeps the wiki honest.

**Layer 2 — `wiki/` (the LLM-owned layer).**
This is the compiled artifact, and the LLM owns all of it: `index.md` (the categorized catalog
read first on every query — the RAG replacement), `log.md` (the append-only journal,
`## [YYYY-MM-DD] <op> | <title>`), `overview.md` (a single evolving synthesis), and `pages/` (a
flat, slug-named set of `source` / `entity` / `concept` / `analysis` pages). Pages are *living
documents, not journals*: the model edits in place and bumps `updated` rather than appending
dated change blocks, because a wiki that grows by accretion rots the same way an un-maintained
human wiki does. This is the layer where "compiled once, kept current" physically happens, and
it maps cleanly onto Anthropic's notion of **structured note-taking / agentic memory** — "the
agent regularly writes notes persisted to memory outside of the context window." [[ctx]] The
wiki is that sanctioned external memory.

**Layer 3 — the config + spec layer (`SCHEMA.md` and `CLAUDE.md`).**
`SCHEMA.md` is both the discovery anchor and the canonical specification: it carries the `Path:`
redirect field, the index categories, the page conventions, and the full **citation grammar**.
`CLAUDE.md` tells the maintainer agent its role and points it at `SCHEMA.md` first. This layer
exists so the rules live in exactly one place. An agent that opens `SCHEMA.md` can maintain the
wiki correctly *even without the plugin installed* — the spec is self-contained, which is what
makes the store portable rather than tool-locked.

Why three and not two: merge `raw/` into `wiki/` and you lose verifiable provenance; merge the
config into `CLAUDE.md` and you get the conflicting-instruction problem the memory docs warn
about. [[memory]] Keeping them distinct is what lets each layer have one owner and one rule.

---

## 3. Why this is built as Claude Code Skills + a plugin

The LLM Wiki idea does not require Claude Code — it is "just a git repo of markdown files"
[[gist]] — but packaging it as **skills inside a plugin** is what makes it lean at startup and
cheap to run.

**Progressive disclosure.** Agent Skills load in three levels: a skill's name and description are
always in context, the `SKILL.md` body loads only when the skill triggers, and bundled
`references/` files cost zero tokens until the model actually reads one. [[skills-overview]]
[[skills-eng]] That is exactly the budget profile a knowledge tool wants: at rest, the agent
carries four one-line skill descriptions and a small `CLAUDE.md`; depth (the citation grammar,
the codebase slug taxonomy, the MCP recipes, the git-sync automation) sits in `references/` and
in `SCHEMA.md`, pulled in only when a task needs it. Anthropic calls progressive disclosure the
scaling principle for skills precisely because metadata stays cheap while capability grows.
[[skills-eng]]

**Just-in-time context, by construction.** Claude Code itself is Anthropic's worked example of
the hybrid model this wiki imitates: "CLAUDE.md files are naively dropped into context up front,
while primitives like glob and grep allow it to navigate its environment and retrieve files
just-in-time." [[ctx]] wiki-base inherits that for free — `index.md` is the dropped-in map, and
the same `grep`/`glob`/`read` primitives walk to the few pages a question needs. Building on the
native skills + tools surface means we do not reimplement retrieval, file I/O, web fetching, or
PDF parsing; we describe a discipline and let the harness execute it. "Build the simplest thing
that works" is Anthropic's own guidance for agents, [[agents]] and a plugin of four skills over
markdown is about as simple as a self-maintaining knowledge base gets.

---

## 4. The human-vs-LLM division of labor

The wiki is a partnership with a sharp boundary, and Karpathy draws it explicitly: "The human's
job is to curate sources, direct the analysis, ask good questions, and think about what it all
means. The LLM's job is everything else." [[gist]] His IDE analogy makes the relationship
concrete — "Obsidian is the IDE; the LLM is the programmer; the wiki is the codebase." [[gist]]

wiki-base encodes that split directly in the skill surface:

- **Human (sourcing, direction, questions).** The human decides *what* enters the wiki and
  *what matters*. They point `/wiki-base:ingest` at a source, set the domain and categories in
  `/wiki-base:init`, and pose the questions that `/wiki-base:query` answers. Judgment, taste, and
  intent stay human.
- **LLM (summarizing, cross-referencing, bookkeeping).** Everything mechanical and tedious is the
  model's: distilling a raw source into a cited page, weaving `[[wikilinks]]` and keeping them
  bidirectional, updating `index.md` and `overview.md`, appending to `log.md`, and catching
  contradictions, orphans, and stale claims in `/wiki-base:lint`. This is the consistency labor
  that no human sustains — and the reason the compounding loop survives contact with reality.

The plugin is opinionated about keeping the model on its side of that line. `/wiki-base:query` is
read-only and is told *not* to answer from general knowledge and to say "not in the wiki" rather
than invent — grounding answers in the curated sources the human chose, in line with Anthropic's
guidance to allow "I don't know" and restrict claims to provided sources. [[reduce]]

---

## 5. Key design decisions and tradeoffs

Each decision below is a deliberate *exclusion* as much as an inclusion. Leanness is the product,
so the interesting question for every feature is "what did we leave out, and why is that correct?"

### (a) Markdown + agentic `grep`/`Read` search — no vector DB, no RAG

**Decision.** Retrieval is `index.md` read first, then agentic `grep`/`glob`/`read` to the few
pages that matter, loaded just-in-time. There is no embedding model, no vector index, and no
retrieval service anywhere in the stack.

**Rationale.** Karpathy reports this works directly: "The LLM reads the index first to find
relevant pages, then drills into them. This works surprisingly well at moderate scale (~100
sources, ~hundreds of pages) and avoids the need for embedding-based RAG infrastructure." [[gist]]
Anthropic's context-engineering guidance backs the same shape — lightweight identifiers loaded at
runtime via tools, with `glob`/`grep` over files as the canonical example. [[ctx]] On-demand
loading is not just simpler, it is dramatically cheaper: Anthropic's code-execution-with-MCP work
shows a tiered title → summary → full-content load pattern cutting one workload from **150,000
tokens to 2,000 — a 98.7% reduction.** [[cem]] `index.md` → `pages/` is exactly that tiering.

**The contrast that proves the point — DeepWiki / deepwiki-open.** The open-source DeepWiki stack
is the maximalist version of this same goal, and its architecture is instructive precisely
because of how much machinery it carries: a FAISS vector store, an embedding service, a RAG
retrieval pipeline, and a FastAPI + Next.js web service to host it all. [[deepwiki-arch]]
[[deepwiki-open]] Sibling projects pile on more — OpenDeepWiki ships a .NET server, SQL and
vector databases, an admin console, OAuth UI, and chat webhooks. [[opendeepwiki]] wiki-base
**borrows the good ideas and rejects the stack**: keep the navigable structure, an
overview/architecture page, "links to sources" so every claim points at its raw file
[[devin-deepwiki]], and free fenced Mermaid diagrams for query answers; **drop** the embeddings,
FAISS, the RAG pipeline, and the web server. The borrowed parts are the parts that survive in
plain markdown; the rejected parts are infrastructure we get for free from the agent and the
filesystem.

**Tradeoff.** Pure index-first search has a ceiling — roughly ~100 sources / hundreds of pages,
exactly the regime Karpathy describes. [[gist]] [[xpost]] That is a real limit, accepted on
purpose. If a wiki ever outgrows `index.md`, a markdown search engine (e.g. `qmd`) is an opt-in
CLI/MCP upgrade — never bundled, never required. We pay a scale ceiling to buy zero infrastructure,
and for the target use (one project's knowledge) that is the right trade.

### (b) Folding `update` → `ingest` and `audit` → `lint` to keep exactly four skills

**Decision.** Where the structural ancestor exposes six operations (init / ingest / query / lint /
update / audit), wiki-base exposes **four**. Revising a page in place is folded into
`/wiki-base:ingest` (adding a source and updating an existing page are the same "compile this into
the wiki" motion). Single-page citation fact-checking is folded into `/wiki-base:lint` as Mode B.
Codebase git-`sync` is also folded into `/wiki-base:ingest`.

**Rationale.** Anthropic's skill-authoring guidance warns against bloated, overlapping tool sets
and favors a minimal, eval-driven surface. [[skills-bp]] Every additional skill is another
description permanently in context and another chance for two triggers to collide. Folding
preserves every capability while cutting the trigger surface — fewer near-duplicate descriptions
for the model to disambiguate, which is itself a context-engineering win. [[ctx]] The named
operations did not disappear; they became modes of a smaller, sharper set.

### (c) `.wiki/` in-project default, redirectable in one field

**Decision.** The wiki lives in `.wiki/` inside the project by default, versioned alongside the
code. Pointing it at a central or external directory (e.g. `~/wikis/your-project`) is a single
edit to the `Path:` field in `.wiki/SCHEMA.md`; an in-project `SCHEMA.md` breadcrumb always stays
behind so every skill can still locate the wiki.

**Rationale.** "The wiki is just a git repo of markdown files. You get version history, branching,
and collaboration for free" [[gist]] — and you only get that for free if the wiki sits *inside* a
repo by default. In-project also means the wiki travels with a `git clone`, reviews in the same
pull request as the code it documents, and needs no global configuration to find. The redirect
exists for the real cases that break the default (a wiki spanning many repos, or one kept private
from a public codebase), and it is one config field rather than a code change — a knob, not a fork.

### (d) A lean `CLAUDE.md` that defers conventions to `SCHEMA.md`

**Decision.** `CLAUDE.md` is kept short (a sub-200-line target) and states only the role, the
three layers, the four operations, and the load-bearing invariants. The *full* conventions,
categories, and citation grammar live once in `SCHEMA.md`; `CLAUDE.md` points there and does not
restate them.

**Rationale.** Claude Code's memory and best-practices docs are explicit about this: keep
`CLAUDE.md` small, reserve `IMPORTANT`/`YOU MUST` emphasis for genuinely load-bearing rules, apply
the "would removing this cause a mistake?" test per line, and avoid the over-specified-`CLAUDE.md`
failure mode. [[memory]] [[ccbp]] Stating a rule in two files invites the two copies to drift —
the conflicting-instructions failure the memory docs call out — so conventions are de-duplicated
into `SCHEMA.md` and cited, not inlined. [[memory]] The payoff is also a context-budget payoff: a
small always-on `CLAUDE.md` keeps the smallest set of high-signal tokens resident, with depth
pulled just-in-time. [[ctx]]

### (e) Ingestion via native `Read`/`WebFetch` first; official MCP as opt-in

**Decision.** Local sources — code, PDFs, transcripts, notes — are ingested with the harness's
native `Read` / `WebFetch` and standard file handling. URLs go through native `WebFetch`. PDFs are
read directly with no OCR or RAG stack. [[pdf]] Auth-bearing remote SaaS sources use the **official
vendor MCP servers** (Atlassian Rovo for Confluence/Jira [[atlassian]], the GitHub MCP server for
hosted code [[github-mcp]]), and those are documented as **opt-in** — added explicitly with
`claude mcp add`, never shipped as an active `.mcp.json` that would trigger OAuth on load. [[ccmcp]]

**Rationale.** Reach for the heavier tool only when the lighter one cannot do the job. Local files
and public URLs need no server at all — native reads cover them, which keeps the common path
zero-config. [[fsmcp]] [[fetchmcp]] For sources behind authentication, MCP is the vendor-neutral
standard built for exactly that, and using each vendor's official server beats hand-rolled
connectors, which "don't scale." [[mcpnews]] Making MCP opt-in rather than bundled honors the same
leanness rule as everything else: no auth prompt, no network dependency, and no extra surface
until the user actually has a Confluence space or a private repo to ingest.

---

## 6. Provenance & credits

wiki-base is built on public ideas, not forked from any one codebase. Three sources are
load-bearing, and all original files in this repo are **MIT-licensed** (see [`NOTICE`](./NOTICE)).

- **The idea — Andrej Karpathy's LLM Wiki.** The north star: knowledge as a compiled, compounding
  git repo of markdown files that the LLM maintains; the three-layer model; the human-vs-LLM
  division of labor; index-first search without vector databases; and the Memex lineage. Every
  conceptual decision above traces here. [[gist]] [[xpost]]
- **Structural inspiration — `kfchou/wiki-skills` (MIT).** The skill-set precedent that wiki-base
  consolidates from six operations to four, the directory schema instantiated in `SCHEMA.md`, the
  flat slug-named `pages/` layout, the control files (`index.md` / `log.md` / `overview.md`), and
  the footnote citation grammar. Credited in `NOTICE`. [[kfchou]]
- **The codebase-sync pattern — yysun's git-commit-triggered wiki (MIT).** The incremental
  maintenance approach for codebase wikis — a `last_commit` SHA checkpoint, `git diff` to find
  what changed, and updating only the touched pages while marking deleted-source pages stale —
  comes from yysun's `git-wiki` skill and the accompanying write-up. It lives in
  `skills/ingest/references/codebase-sync.md` and is credited in `NOTICE`. [[yysunpost]]
  [[yysunrepo]]

---

## Sources

This file cites a subset of the project bibliography. The complete annotated list — every URL,
grouped by tier (Karpathy primary, Anthropic official, reputable community), with a note on
exactly what each source backs — is in [`SOURCES.md`](./SOURCES.md). Do not add citations here
that are not in `SOURCES.md`.

[gist]: https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f
[xpost]: https://x.com/karpathy/status/2039805659525644595
[ctx]: https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
[agents]: https://www.anthropic.com/engineering/building-effective-agents
[cem]: https://www.anthropic.com/engineering/code-execution-with-mcp
[skills-eng]: https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills
[skills-overview]: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview
[skills-bp]: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices
[memory]: https://code.claude.com/docs/en/memory
[ccbp]: https://code.claude.com/docs/en/best-practices
[reduce]: https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/reduce-hallucinations
[pdf]: https://platform.claude.com/docs/en/build-with-claude/pdf-support
[ccmcp]: https://code.claude.com/docs/en/mcp
[mcpnews]: https://www.anthropic.com/news/model-context-protocol
[atlassian]: https://support.atlassian.com/atlassian-rovo-mcp-server/docs/getting-started-with-the-atlassian-remote-mcp-server/
[github-mcp]: https://github.com/github/github-mcp-server
[fsmcp]: https://github.com/modelcontextprotocol/servers/tree/main/src/filesystem
[fetchmcp]: https://github.com/modelcontextprotocol/servers/tree/main/src/fetch
[deepwiki-arch]: https://asyncfunc.mintlify.app/reference/architecture
[deepwiki-open]: https://github.com/AsyncFuncAI/deepwiki-open
[devin-deepwiki]: https://docs.devin.ai/work-with-devin/deepwiki
[opendeepwiki]: https://github.com/AIDotNet/OpenDeepWiki
[kfchou]: https://github.com/kfchou/wiki-skills
[yysunrepo]: https://github.com/yysun/awesome-agent-world
[yysunpost]: https://dev.to/yysun/bringing-the-llm-wiki-idea-to-a-codebase-22go
