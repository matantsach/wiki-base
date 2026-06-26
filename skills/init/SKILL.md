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

1. **Wiki root path** — `[.wiki/]` in this project (versioned with the code), or a
   central directory such as `~/wikis/your-project`. Resolve to an absolute path.
2. **Domain** — `research` or `codebase` (sets the default categories and which
   reference guide applies).
3. **Source types** — what you will ingest (e.g. code, Confluence, URLs, PDFs,
   transcripts). Free-form; recorded for context.
4. **Index categories** — defaults by domain:
   - research → `Sources | Entities | Concepts | Analyses`
   - codebase → `Modules | APIs | Decisions | Flows`

## 3. Write the layout (at the ROOT)

Generate from `references/schema-template.md` (the canonical conventions + citation
grammar — do not restate it inline). Create at the resolved **ROOT** (the absolute path
from step 2):

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

Fill `SCHEMA.md` placeholders: `Path:` = the **absolute** ROOT; `Domain:`, `Categories:`,
and `Source types:` from the answers. Seed `index.md` with the chosen category headings
(empty), and a one-line `overview.md` skeleton (Current Understanding / Open Questions /
Key Entities-Concepts).

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

  - **Path:** /absolute/path/to/external/ROOT
  ```

  If a demo or older `.wiki/SCHEMA.md` already sits here, **overwrite it** with this stub —
  a stale anchor pointing at `.wiki/` is the bug where queries ignore your external wiki.
  Only this stub lives in-project; `raw/`, `assets/`, and `wiki/` live at the ROOT.

## 5. Log it

Append one line to `ROOT/wiki/log.md`:

```
## [YYYY-MM-DD] init | <wiki name>
```

Then tell the user the wiki is ready and point them at `/wiki-base:ingest` to add the
first source.

## References

- `references/schema-template.md` — canonical `SCHEMA.md`: conventions + full citation grammar.
- `references/codebase.md` — dev-focused authoring guide for codebase wikis.
