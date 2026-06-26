# Sources

Annotated bibliography for **wiki-base**. Every design decision in this repo traces to one of
the sources below. Entries are grouped by tier — **Karpathy primary**, **Anthropic official**,
and **reputable community** (named OSS projects with real traction and official vendor docs).
No generic blog / SEO posts are used. Each entry gives the title, URL, and what it backs here.

## Karpathy primary

### Andrej Karpathy — "LLM Wiki" gist
- **URL:** <https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f>
  (raw mirror: <https://gist.githubusercontent.com/karpathy/442a6bf555914893e9891c11519de94f/raw>)
- **Backs:** The product's north star — the basis for `CLAUDE.md`, `DESIGN.md`, `.wiki/SCHEMA.md`,
  and all four skills. Specifically: the **compile-once / keep-current** pattern (ingest mutates
  existing pages rather than only appending) that distinguishes the wiki from RAG / NotebookLM;
  the **three-layer model** (immutable `raw/` → LLM-owned `wiki/` → `SCHEMA.md`/`CLAUDE.md`
  config); the human-vs-LLM division of labor ("Obsidian is the IDE; the LLM is the programmer;
  the wiki is the codebase"; the human curates sources, the LLM owns the wiki); the **Ingest**
  flow (read → discuss → write summary page → update index → revise entity/concept pages →
  append log; ~10–15 pages touched per source), **Query** (read `index.md` first, answer with
  citations, file good answers back as pages), and **Lint** (contradictions, stale claims,
  orphans, missing-concept pages, missing cross-refs, fillable data gaps) workflows; the
  `index.md` content catalog and the `log.md` append-only journal format
  `## [YYYY-MM-DD] <op> | <title>` with its grep recipe; **"No vector databases"** / index-first
  search to ~100 sources / hundreds of pages; the "persistent, compounding artifact" and
  "just a git repo of markdown files" framings; the Memex (Vannevar Bush, 1945) lineage; and
  qmd named as an optional, pluggable search upgrade.

### Andrej Karpathy on X — "LLM Knowledge Bases"
- **URL:** <https://x.com/karpathy/status/2039805659525644595>
- **Backs:** First-party confirmation that the pattern targets "manipulating knowledge" at
  ~100-article / ~400,000-word scale — the `index.md` sweet spot cited in `README.md` / `DESIGN.md`.
  (X.com returns HTTP 402 to WebFetch; the verbatim opening was captured via WebSearch metadata
  of the canonical status URL.)

## Anthropic official

### Create plugins (Claude Code docs)
- **URL:** <https://code.claude.com/docs/en/plugins>
- **Backs:** The `.claude-plugin/plugin.json` manifest, the rule that component directories
  (`skills/`, `agents/`, `hooks/`) live at the repo root — NOT inside `.claude-plugin/` — and
  the `skills/<name>/SKILL.md` namespaced invocation (`/wiki-base:<skill>`). Backs the repo tree
  and plugin-manifest layout.

### Create and distribute a plugin marketplace (Claude Code docs)
- **URL:** <https://code.claude.com/docs/en/plugin-marketplaces>
- **Backs:** `.claude-plugin/marketplace.json` (`name` + `owner` + `plugins[]`), `source: "./"`
  so one repo is both marketplace and plugin (resolves only when added via Git), and the
  `/plugin marketplace add OWNER/wiki-base` → `/plugin install wiki-base@wiki-base` flow.

### Plugins reference (Claude Code docs)
- **URL:** <https://code.claude.com/docs/en/plugins-reference>
- **Backs:** `plugin.json` is optional with `name` the only required field; the rule to set
  `version` explicitly (omitting it on a git-hosted plugin makes every commit a new version);
  the `displayName` / `license` / `keywords` hygiene fields.

### Agent Skills overview (Claude docs)
- **URL:** <https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview>
- **Backs:** SKILL.md requires only `name` + `description` (64 / 1024-char limits, lowercase-hyphen
  name, no "claude"/"anthropic"); the three-level progressive-disclosure model. Backs every
  `skills/*/SKILL.md` frontmatter and the `init` / `ingest` / `query` / `lint` naming.

### Skill authoring best practices (Claude docs)
- **URL:** <https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices>
- **Backs:** The <500-line SKILL.md cap and "references one hop deep" rule (the `references/`
  design); third-person, trigger-rich descriptions; eval-driven minimal authoring; the
  verify-citations loop folded into `ingest`/`lint`; and domain-partitioned markdown + grep as
  the RAG-free retrieval pattern.

### Extend Claude with skills (Claude Code docs)
- **URL:** <https://code.claude.com/docs/en/skills>
- **Backs:** Claude-Code-specific SKILL.md fields — `allowed-tools` (read-only `query`; `lint`
  with `Write`/`Edit`) and `disable-model-invocation`; dynamic context injection. Backs the
  `allowed-tools` choices on `query` and `lint`.

### anthropics/skills (GitHub)
- **URL:** <https://github.com/anthropics/skills>
- **Backs:** The `skills/<name>/SKILL.md` + `references/` layout. (The MIT license choice in
  `LICENSE` / `NOTICE` is wiki-base's own, chosen for compatibility with the MIT-licensed
  upstreams it credits.)

### Best practices for Claude Code
- **URL:** <https://code.claude.com/docs/en/best-practices>
- **Backs:** `CLAUDE.md` authoring discipline — the "would removing this cause a mistake?" test,
  anti-bloat, reserving `IMPORTANT` / `YOU MUST` emphasis for load-bearing rules, treat-it-like-code
  iteration, the over-specified-CLAUDE.md failure pattern, and the include/exclude table
  (link to docs, don't inline).

### How Claude remembers your project (memory)
- **URL:** <https://code.claude.com/docs/en/memory>
- **Backs:** The <200-line CLAUDE.md target; HTML comments stripped from context (maintainer
  notes); the `@path` import caveat (imports don't save tokens); the project-vs-user memory
  hierarchy; AGENTS.md interop (the symlink and its documented direction-reversal note); the
  conflicting-instructions failure mode → de-duping conventions into `.wiki/SCHEMA.md`; and the
  `.gitignore` of `CLAUDE.local.md`.

### Effective context engineering for AI agents (Anthropic Engineering)
- **URL:** <https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents>
- **Backs:** The core justification for markdown + agentic search over RAG — just-in-time
  retrieval via lightweight identifiers (page paths) + grep/glob/read; the explicit **hybrid**
  stance (augment embedding retrieval, not replace it); CLAUDE.md + glob/grep as Anthropic's own
  example; **context rot** and "the smallest set of high-signal tokens"; right-altitude skill
  authoring and the warning against bloated/ambiguous tool sets; and structured note-taking
  (the wiki as sanctioned agentic memory). Backs `DESIGN.md`, the no-RAG guardrail in `CLAUDE.md`,
  and the lean four-skill surface.

### Building effective agents (Anthropic Engineering)
- **URL:** <https://www.anthropic.com/engineering/building-effective-agents>
- **Backs:** "Build the simplest thing that works" (no vector/RAG infrastructure);
  agents-vs-workflows (open-ended agentic query loop vs more deterministic ingest); and markdown
  as a natural agent-computer interface "close to what the model has seen." Backs `DESIGN.md` and
  the query/ingest skill shapes.

### Code execution with MCP (Anthropic Engineering)
- **URL:** <https://www.anthropic.com/engineering/code-execution-with-mcp>
- **Backs:** On-demand / tiered loading (title → summary → full page, mirrored by
  `index.md` → `pages/`) and the **98.7% (150k → 2k token)** saving figure quoted in `README.md` /
  `DESIGN.md`; plus keeping raw dumps out of context (distill in ingest before the model sees them).

### Equipping agents for the real world with Agent Skills (Anthropic Engineering)
- **URL:** <https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills>
- **Backs:** Progressive disclosure as the scaling principle — metadata always loaded, bundled
  reference files cost zero tokens until read. Backs the lean-at-startup design (small `CLAUDE.md`,
  `index.md`, and skill descriptions; depth pushed into `references/`).

### Create custom subagents (Claude Code docs)
- **URL:** <https://code.claude.com/docs/en/sub-agents>
- **Backs:** Routing verbose sources (large PDFs / Confluence dumps / transcripts) through a
  subagent so the raw dump burns the subagent's context and only the distilled, cited note
  returns; and the read-only-lint pattern (no Edit/Write). Backs the subagent guidance inlined in
  `ingest` and referenced by `codebase-sync.md`.

### Automate actions with hooks (Claude Code docs)
- **URL:** <https://code.claude.com/docs/en/hooks-guide>
- **Backs:** The OPTIONAL, documented-not-active automation in `codebase-sync.md` — `PostToolUse`
  `Edit|Write` vs the more robust `Stop` + `git status --porcelain` once-per-turn; project-scoped
  (not global) hook config; the fact that there is **no `git commit` hook event**; and
  `disableAllHooks` as the kill switch.

### Run Claude Code programmatically (headless)
- **URL:** <https://code.claude.com/docs/en/headless>
- **Backs:** The CI lane in `codebase-sync.md` — `claude --bare -p ... --output-format json`,
  permission modes, the pipe-the-diff staleness check (warn, don't fail) vs an `acceptEdits`
  refresh, asserting plugin load via `system/init` `plugin_errors`, and capping spend via
  `total_cost_usd`.

### Introducing the Model Context Protocol (Anthropic news)
- **URL:** <https://www.anthropic.com/news/model-context-protocol>
- **Backs:** MCP as the vendor-neutral standard for auth-bearing sources and the per-source-server
  pattern — bespoke connectors "don't scale." Backs the opt-in MCP appendix in the `ingest`
  reference files (`skills/ingest/references/ingest-confluence.md`, `ingest-code.md`, `ingest-web-and-files.md`).

### PDF support (Claude docs)
- **URL:** <https://platform.claude.com/docs/en/build-with-claude/pdf-support>
- **Backs:** Native PDF ingestion with no OCR/RAG stack (32 MB / 600-page / no-encryption limits;
  split large files); transcripts and other local files via standard file handling. Backs the
  PDF and transcript branches in `skills/ingest/references/ingest-web-and-files.md`.

### Connect Claude Code to tools via MCP (Claude Code docs)
- **URL:** <https://code.claude.com/docs/en/mcp>
- **Backs:** `claude mcp add` transports / scopes and plugin-bundled vs project `.mcp.json`; the
  decision to document `claude mcp add` as opt-in rather than ship an active `.mcp.json` that
  would trigger OAuth on load.

### Reduce hallucinations (Claude docs)
- **URL:** <https://platform.claude.com/docs/en/test-and-evaluate/strengthen-guardrails/reduce-hallucinations>
- **Backs:** The anti-hallucinated-citation playbook in `ingest` / `query` / `lint` — allow
  "I don't know" ("not in the wiki"), ground claims in verbatim quotes, restrict to provided
  sources, and retract any claim whose quote can't be found.

## Reputable community

### kfchou/wiki-skills — Claude Code plugin
- **URL:** <https://github.com/kfchou/wiki-skills>
- **Backs:** The skill-set precedent (init / ingest / query / lint / update / audit) that
  wiki-base consolidates to four, and the well-tuned trigger phrasing ("Do not answer from
  general knowledge", "Run after every 5–10 ingests"). Credited in `NOTICE` (MIT).

### kfchou/wiki-skills — wiki-init/SKILL.md (incl. the SCHEMA.md template)
- **URL:** <https://raw.githubusercontent.com/kfchou/wiki-skills/main/skills/wiki-init/SKILL.md>
- **Backs:** The directory schema instantiated in `.wiki/SCHEMA.md` and
  `skills/init/references/schema-template.md` — SCHEMA.md as discovery anchor with a `Path:`
  field; flat, slug-named `pages/`; the `index.md` / `log.md` / `overview.md` control files;
  the research vs codebase category defaults; the full footnote **citation grammar** (Quote vs
  `[synthesis]`, three targets, mandatory locators, entity/concept/analysis never cited); and the
  required page frontmatter (`title`, `tags`, `sources`, `updated`).

### kfchou/wiki-skills — codebase.md
- **URL:** <https://raw.githubusercontent.com/kfchou/wiki-skills/main/skills/wiki-init/codebase.md>
- **Backs:** `skills/init/references/codebase.md` — the `mod-` / `api-` / `dec-` / `flow-` slug
  taxonomy, "five deep pages beat fifty shallow ones," the Sourced-vs-Reconstructed decision
  split, and the README-vs-wiki boundary (link, don't duplicate).

### kfchou/wiki-skills — marketplace.json + plugin.json
- **URL:** <https://raw.githubusercontent.com/kfchou/wiki-skills/main/.claude-plugin/marketplace.json>
- **Backs:** The packaging template mirrored in `.claude-plugin/` (root `marketplace.json` +
  `plugin.json`, `skills/<name>/SKILL.md`). wiki-base keeps the marketplace description in sync
  with the actual skill set (kfchou's undercounts the skills).

### kfchou/wiki-skills — wiki-ingest/SKILL.md
- **URL:** <https://raw.githubusercontent.com/kfchou/wiki-skills/main/skills/wiki-ingest/SKILL.md>
- **Backs:** The "living documents, not journals" rule (edit in place, bump `updated`, never add
  dated update blocks) enforced in `ingest` and lint Mode A; and the required frontmatter.

### kfchou/wiki-skills — wiki-lint/SKILL.md
- **URL:** <https://raw.githubusercontent.com/kfchou/wiki-skills/main/skills/wiki-lint/SKILL.md>
- **Backs:** Lint Mode A severity tiers and concrete thresholds — orphan = 0 inbound links;
  stale = >90 days untouched AND contains "current/latest/recent/state-of-the-art" or a year
  literal ≥2 years old; missing-concept = referenced 3+ times with no page; report →
  `lint-<date>.md`, always logged.

### kfchou/wiki-skills — wiki-audit/SKILL.md
- **URL:** <https://raw.githubusercontent.com/kfchou/wiki-skills/main/skills/wiki-audit/SKILL.md>
- **Backs:** Lint Mode B single-page citation audit — Phase A (list uncited claims), Phase B
  (group footnotes by source, read each source once, verdicts supported / unsupported / partial /
  source-missing), report → `audit-<page>-<date>.md`.

### yysun/awesome-agent-world
- **URL:** <https://github.com/yysun/awesome-agent-world>
- **Backs:** The home of the `git-wiki` skill; credited in `NOTICE` (MIT) for the git-checkpoint
  codebase pattern.

### yysun — Bringing the LLM Wiki idea to a codebase (DEV Community)
- **URL:** <https://dev.to/yysun/bringing-the-llm-wiki-idea-to-a-codebase-22go>
- **Backs:** The git-commit-triggered incremental-maintenance pattern in
  `skills/ingest/references/codebase-sync.md` — a `last_commit` SHA checkpoint in `index.md`
  frontmatter, `git diff --name-status -M`, updating only touched pages, and marking
  deleted-source pages `stale`. (First-party post by the AppRun author publishing his own named
  OSS skill — included per the source bar as a primary publication, not a relay.)

### yysun/awesome-agent-world — git-wiki/SKILL.md
- **URL:** <https://raw.githubusercontent.com/yysun/awesome-agent-world/main/skills/git-wiki/SKILL.md>
- **Backs:** The reader-question-first index template and the `source_paths[]` / `stale`
  codebase-only frontmatter (defined in `codebase.md`, credited to yysun); the query rule to
  verify specific values against live source; and lint's "missing coverage for changed modules"
  check.

### DeepWiki-Open — Architecture overview (official docs)
- **URL:** <https://asyncfunc.mintlify.app/reference/architecture>
- **Backs:** The borrow/reject table in `DESIGN.md` — BORROW the navigable structure, an
  overview/architecture page, and Mermaid diagrams (free fenced code); REJECT FAISS, the
  embedding service, the RAG pipeline, and the FastAPI/Next.js web service.

### AsyncFuncAI/deepwiki-open (GitHub)
- **URL:** <https://github.com/AsyncFuncAI/deepwiki-open>
- **Backs:** The "Ask" / "Deep Research" query-mode concept (reframed as agentic search over
  markdown) and the Mermaid-diagram borrow cited in `skills/query/SKILL.md`; REJECT the
  embeddings/FAISS/WebSocket implementation.

### Devin DeepWiki docs (Cognition)
- **URL:** <https://docs.devin.ai/work-with-devin/deepwiki>
- **Backs:** Two lean borrows in `DESIGN.md` — an architecture/overview page and "links to
  sources" (every claim cites the raw file), mapping onto the immutable `raw/` layer + cited query.

### AIDotNet/OpenDeepWiki (GitHub)
- **URL:** <https://github.com/AIDotNet/OpenDeepWiki>
- **Backs:** The "catalog" (table-of-contents = `index.md`) borrow, and the clearest example of
  scope deliberately excluded — a .NET server, SQL/vector DBs, MCP-host endpoints, an admin
  console, OAuth UI, and chat webhooks.

### MCP Architecture overview (modelcontextprotocol.io)
- **URL:** <https://modelcontextprotocol.io/docs/learn/architecture>
- **Backs:** Tools vs Resources, and stdio (local) vs Streamable HTTP + OAuth (remote) — the rule
  in the `ingest` reference files that local sources use native reads while remote SaaS sources
  use HTTP+OAuth MCP servers.

### Getting started with the Atlassian Rovo MCP Server (Atlassian Support)
- **URL:** <https://support.atlassian.com/atlassian-rovo-mcp-server/docs/getting-started-with-the-atlassian-remote-mcp-server/>
- **Backs:** The Confluence/Jira branch in `skills/ingest/references/ingest-confluence.md` — the official remote MCP at
  `https://mcp.atlassian.com/v1/mcp/authv2`, OAuth 2.1, Cloud-only, via
  `claude mcp add --transport http atlassian ...`.

### Supported tools — Atlassian Rovo MCP Server (Atlassian Support)
- **URL:** <https://support.atlassian.com/atlassian-rovo-mcp-server/docs/supported-tools/>
- **Backs:** The exact Confluence ingest tool sequence in `skills/ingest/references/ingest-confluence.md` —
  `searchConfluenceUsingCql` / `getPagesInConfluenceSpace` → `getConfluencePage`.

### GitHub MCP Server (github/github-mcp-server)
- **URL:** <https://github.com/github/github-mcp-server>
- **Backs:** The GitHub-hosted-code branch in `skills/ingest/references/ingest-code.md` — the official GitHub MCP at
  `https://api.githubcopilot.com/mcp/` (OAuth/PAT) via `claude mcp add --transport http github ...`.

### MCP filesystem reference server (modelcontextprotocol/servers)
- **URL:** <https://github.com/modelcontextprotocol/servers/tree/main/src/filesystem>
- **Backs:** The official-option note that code/transcripts on disk need no extra server inside
  Claude Code (native Read/Grep cover it); the filesystem/git MCP servers are the official option
  for other MCP hosts.

### MCP fetch reference server (modelcontextprotocol/servers)
- **URL:** <https://github.com/modelcontextprotocol/servers/tree/main/src/fetch>
- **Backs:** The URL → markdown step in `skills/ingest/references/ingest-web-and-files.md` (native WebFetch inside Claude Code; the
  fetch MCP for other hosts; `start_index` for long pages).

### Obsidian Help — Internal links
- **URL:** <https://obsidian.md/help/links>
- **Backs:** The `[[slug]]` cross-reference syntax used throughout the wiki — basic links,
  aliases (`[[slug|text]]`), and heading anchors (`[[slug#Heading]]`) — and Obsidian graph-view
  compatibility (hubs and orphans).

### Google Search Central — URL structure best practices
- **URL:** <https://developers.google.com/search/docs/crawling-indexing/url-structure>
- **Backs:** The slug convention for `pages/` filenames — hyphens over underscores, consistent
  lowercase casing.
