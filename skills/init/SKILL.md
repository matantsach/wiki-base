---
name: init
description: Bootstraps or reconfigures a Karpathy-style LLM Wiki for any domain — codebase documentation, research notes, competitive analysis, or long-term knowledge accumulation. Use when starting a wiki, choosing where the wiki lives (default .wiki/ in the project, or a central directory such as ~/wikis/your-project), or picking page categories. Detects a pre-existing demo wiki and offers to reset it. Creates SCHEMA.md plus index.md, log.md, overview.md, and a pages/ directory.
---

# init — bootstrap or reconfigure an LLM Wiki

You are setting up an LLM Wiki: a persistent, compounding markdown knowledge base
that compiles knowledge once and keeps it current — never re-derives per query.
This skill writes the on-disk layout and the `SCHEMA.md` that every other skill
reads first. Keep it pure markdown: no vector DB, no embeddings, no server.

## 1. Detect what already exists (do this first)

- Look for `.wiki/SCHEMA.md` in the project.
  - **Shipped demo present** — if `SCHEMA.md` carries the demo top-note (it ships as a
    throwaway worked example), offer to **reset**: clear `wiki/pages/*` and `raw/*`,
    keep the directory structure and a fresh `index.md`/`log.md`/`overview.md`. Confirm
    before deleting.
  - **Real wiki present** — do NOT overwrite. Offer to *reconfigure* instead (edit
    `Path:`, `Categories:`, or `Source types:` in the existing `SCHEMA.md`), leaving
    pages untouched.
  - **Nothing present** — proceed to a clean bootstrap.

## 2. Gather config — ask ONE question at a time

Wait for each answer before asking the next; offer the default in brackets.

1. **Wiki root path** — `[.wiki/]` in this project (versioned with the code; keep this the
   **relative** `.wiki/` so it stays portable across clones and machines), or a central
   directory such as `~/wikis/your-project` (prefer a `~/`-relative path over a
   machine-specific absolute one). Never write a `/Users/...`-style absolute path for an
   in-project wiki — `SCHEMA.md` is committed, so it must resolve on every clone.
2. **Domain** — `research` or `codebase` (sets the default categories and which
   reference guide applies).
3. **Source types** — what you will ingest (e.g. code, Confluence, URLs, PDFs,
   transcripts). Free-form; recorded for context.
4. **Index categories** — defaults by domain:
   - research → `Sources | Entities | Concepts | Analyses`
   - codebase → `Modules | APIs | Decisions | Flows`

## 3. Write the layout (at the ROOT)

Generate from `references/schema-template.md` (the canonical conventions + citation
grammar — do not restate it inline). Create the directories at the **ROOT** chosen in step 2
(for the default that is this repo's `.wiki/`; for external it is the `~/wikis/<project>` dir):

```
<ROOT>/
├── SCHEMA.md            # instantiated from references/schema-template.md
├── raw/                 # immutable sources (LLM never modifies)
├── assets/              # downloaded images / PDFs / attachments (+ .gitkeep)
└── wiki/
    ├── index.md         # categorized catalog, read first on every query
    ├── log.md           # append-only journal
    ├── overview.md      # evolving synthesis
    └── pages/           # FLAT, slug-named, no subdirectories
```

Fill `SCHEMA.md` placeholders: set `Path:` to a **PORTABLE** root (SCHEMA.md is committed —
never hardcode a machine-specific path):
- in-project → `Path: .wiki/` (the literal relative default; resolves to this repo's `.wiki/`
  on every clone — do NOT write `/Users/<you>/.../.wiki`).
- external → `Path: ~/wikis/<project>` (`~/`-relative) when under home; a bare absolute path
  only as a last resort, and warn the user it won't be portable to other machines.
Then `Domain:`, `Categories:`, and `Source types:` from the answers. Seed `index.md` with the
chosen category headings (empty), and a one-line `overview.md` skeleton (Current Understanding
/ Open Questions / Key Entities-Concepts).

For **codebase** wikis, follow `references/codebase.md` for slug prefixes
(`mod-`/`api-`/`dec-`/`flow-`), the reader-question index template, and the
codebase-only frontmatter (`source_paths[]`, `stale`).

## 4. Write the discovery anchor — `.wiki/SCHEMA.md` (ALWAYS)

Every skill finds the wiki by reading `.wiki/SCHEMA.md` **in the project where init was
invoked** and following its `Path:`. That anchor MUST exist in the project after init, in
both modes — getting this wrong is exactly what makes `query` silently fall back to the
wrong `.wiki/`:

- **In-project ROOT (default `.wiki/`)** — the ROOT *is* the anchor folder, so the
  `.wiki/SCHEMA.md` you wrote in step 3 already serves as the anchor. Nothing more to do.
- **External ROOT (e.g. `~/wikis/<project>`)** — you wrote the full wiki at the ROOT in
  step 3. Now ALSO write a thin **redirect stub** at the in-project `.wiki/SCHEMA.md`:

  ```
  # SCHEMA.md — redirect
  This project's wiki lives at another ROOT. Resolve `Path:` and read its real
  `SCHEMA.md` there; do not treat this `.wiki/` folder as the wiki.

  - **Path:** ~/wikis/<project>     # ~/-relative if under home; absolute only as a last resort
  ```

  If a demo or older `.wiki/SCHEMA.md` already sits here, **overwrite it** with this stub —
  a stale anchor pointing at `.wiki/` is the bug where queries ignore your external wiki.
  Only this stub lives in-project; `raw/`, `assets/`, and `wiki/` live at the ROOT.

## 5. Make the wiki discoverable & check-in-ready (ask the user about each)

A wiki only helps if agents and teammates know it exists. After writing it, **explicitly
present all three options below** — name each one and its default; do NOT silently skip the
skills-vendoring offer just because its default is off. Apply the user's choices.

### a. Ambient pointer — so Claude consults the wiki unprompted  `[yes]`
Without it, Claude only uses the wiki when you run `/wiki-base:query`. Write a **managed
block** into the repo's root `CLAUDE.md` (the project memory Claude Code loads every
session). Use the markers so it's idempotent and never clobbers existing content:

  ```
  <!-- wiki-base:begin — managed by /wiki-base:init; edit outside these markers -->
  ## Project knowledge base (wiki-base)
  This repo has a compile-once knowledge base (Karpathy-style LLM Wiki). Find it via
  `.wiki/SCHEMA.md` → its `Path:` → the wiki ROOT, and **read `<ROOT>/wiki/index.md`
  first.** Plain markdown, readable by any agent. **Consult it before answering questions
  about this project, and keep it current as the code/docs change.** With the wiki-base
  plugin: `/wiki-base:query` · `/wiki-base:ingest` · `/wiki-base:lint`.
  <!-- wiki-base:end -->
  ```

  If `CLAUDE.md` exists, replace any existing `wiki-base:begin..end` block in place (else
  append); if absent, create it with just this block. Mirror the block to `AGENTS.md`
  **only if that file already exists** (other agents read AGENTS.md, not CLAUDE.md — don't
  create one unprompted). Do NOT also drop a `.claude/rules/` copy: a no-`paths` rule loads
  at the same priority as CLAUDE.md, so a second copy just double-loads the same text.

### b. Check-in readiness  `[commit it]`
In-project wikis are meant to be committed — a clone then carries the wiki for free. Tell
the user they can `git add .wiki && commit`. If `raw/`/`assets/` would hold large binaries
or sensitive material, offer to add to the repo `.gitignore`: `.wiki/assets/*` then
`!.wiki/assets/.gitkeep`. (External-ROOT wikis aren't checked in — only the
`.wiki/SCHEMA.md` redirect stub is.)

### c. Vendor skills for plugin-less teammates  `[no — rely on the plugin]`
**Ask directly:** "Will everyone who clones this repo have the wiki-base plugin installed?"
If not, offer to copy the four skills (`init`, `ingest`, `query`, `lint`) from the plugin
into the repo's `.claude/skills/<name>/SKILL.md` (with their `references/`) so a clone is
fully self-contained, then `git add .claude/skills`. Resolve the plugin's skills via
`${CLAUDE_PLUGIN_ROOT}/skills/`. Trade-off: vendored skills are invoked **without** the
`wiki-base:` prefix (`/init`, `/ingest`, …) and can drift from the plugin as it updates.
Default off — the plugin already provides the skills in every project, and the wiki is
readable markdown even with no skills at all.

## 6. Log it

Append one line to `ROOT/wiki/log.md`:

```
## [YYYY-MM-DD] init | <wiki name>
```

Then tell the user the wiki is ready and point them at `/wiki-base:ingest` to add the
first source.

## References

- `references/schema-template.md` — canonical `SCHEMA.md`: conventions + full citation grammar.
- `references/codebase.md` — dev-focused authoring guide for codebase wikis.
