<!--
  CANONICAL SCHEMA.md TEMPLATE.
  /wiki-base:init instantiates this into <wiki-root>/SCHEMA.md, filling the {{...}} fields.
  This file is the SINGLE canonical home for the conventions block AND the full citation
  grammar — CLAUDE.md defers to it, so the load-bearing spec never drifts.
  Codebase-only frontmatter (source_paths[], stale) is defined separately in
  skills/init/references/codebase.md (credited to yysun), NOT here.
  A concrete, filled-in example is the shipped demo at .wiki/SCHEMA.md.
-->

# SCHEMA.md — wiki configuration & spec

This file is the discovery anchor for an LLM Wiki. Read it FIRST, then resolve the wiki
root from `Path:` below. It configures the wiki *and* fully specifies its conventions and
citation grammar, so any agent can maintain the wiki correctly even without the plugin.

## Configuration

- **Path:** `{{WIKI_ROOT}}`
  <!-- The redirect knob. A PORTABLE path to the wiki ROOT (SCHEMA.md is committed — don't
       hardcode a machine-specific absolute path). Default: `.wiki/`, the literal relative
       in-project root, which resolves on every clone. For a central/external wiki use a
       `~/`-relative path (e.g. `~/wikis/your-project`); raw/, assets/, and wiki/ then live
       there, and the in-project `.wiki/SCHEMA.md` becomes a thin redirect stub (just this
       Path:), NOT the wiki itself. See init step 4. -->
- **Domain:** `{{DOMAIN}}`  <!-- research | codebase -->
- **Categories:** `{{CATEGORIES}}`
  <!-- index.md groups pages under these headings.
       research default: Sources | Entities | Concepts | Analyses
       codebase default: Modules | APIs | Decisions | Flows -->
- **Source types:** `{{SOURCE_TYPES}}`  <!-- e.g. code, Confluence, URLs, PDFs, transcripts -->

## Layers

- `raw/` — **immutable** sources. The source of truth. Read it; never modify it.
- `assets/` — downloaded images / PDFs / attachments.
- `wiki/` — the LLM-owned layer you write and maintain:
  - `index.md` — categorized catalog. Read FIRST on every query; update on every page-changing op.
  - `log.md` — append-only journal (see below).
  - `overview.md` — single evolving synthesis (Current Understanding / Open Questions / Key Entities-Concepts).
  - `pages/` — FLAT, slug-named pages. No subdirectories.

## Conventions

- **`raw/` is immutable.** Never edit a source; the wiki is *derived* from it.
- **`log.md` is append-only.** Never rewrite past entries.
- **`index.md` is read first** on every query and updated on every page-changing op.
- **`overview.md`** is kept in sync whenever an ingest or update shifts the big picture.
- **Pages are living documents, not journals** — edit in place and bump `updated`; never append
  dated `## update` blocks. Quarantine superseded claims in an explicit `Superseded` block
  rather than deleting them silently.
- **Slugs** are lowercase-hyphen (e.g. `event-sourcing`). `pages/` is FLAT — no subdirectories.
- **Page types:** `source` (citable) · `entity` · `concept` · `analysis` (syntheses, never cited).
- **Cross-references** use Obsidian wikilinks: `[[slug]]`, alias `[[slug|display text]]`,
  anchor `[[slug#Heading]]`. Links should be bidirectional — every ingest runs a backlink audit.

### log.md format

Append one entry per operation, newest at the bottom:

```
## [YYYY-MM-DD] <op> | <title>
```

`<op>` is one of `init | ingest | query | update | lint | sync`. Sub-lines may record
`Pages written:` / `Pages updated:`. To see recent activity:

```
grep "^## \[" log.md | tail -5
```

## Required page frontmatter

```yaml
---
title: Human-readable title
tags: [source]          # or [entity] / [concept] / [analysis]
sources: [raw/example-note.md, https://example.com/page]
updated: YYYY-MM-DD
type: source            # optional, generic
---
```

<!-- Codebase wikis add `source_paths: [...]` and `stale: true` — defined in
     skills/init/references/codebase.md (credited to yysun), not in this generic template. -->

## Citation grammar (load-bearing — the canonical spec)

Cite every non-common-knowledge claim. Citations are markdown footnotes placed at
**paragraph / claim granularity** — never one footnote per sentence.

Two kinds of footnote:

- **Quote** — a verbatim excerpt from the source:

  ```
  [^1]: [[attention-is-all-you-need]] §3.2.2 — "We employ h = 8 parallel attention layers"
  ```

- **Synthesis** — your paraphrase across a span, tagged `[synthesis]`:

  ```
  [^2]: [[transformer-paper]] §3.2-3.4 [synthesis] — multi-head attention lets the model
        attend to information from different representation subspaces in parallel
  ```

**Three (and only three) valid citation targets:**

1. a SOURCE page — `[[source-slug]]`
2. a raw/asset file — `raw/FILE` or `assets/FILE`
3. a live URL

**A locator is MANDATORY** on every footnote — pick the one that fits the source:
`§<section>` · `p.<page>` · `[HH:MM:SS]` (transcripts) · a URL anchor · `(YYYY-MM-DD)` (when a
page has no finer locator). The locator is what makes a citation verifiable in a `lint` audit.

**Never citable:** `entity`, `concept`, and `analysis` pages. They synthesize across sources;
cite the underlying SOURCE pages, raw files, or URLs instead. Never invent a source — if a
claim isn't grounded in one of the three targets, say "not in the wiki" rather than guess.
