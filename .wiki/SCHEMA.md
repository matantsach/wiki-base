> **This `.wiki/` is a throwaway demo.** It ships as a worked example so you can see the
> layout, `[[wikilinks]]`, footnote citations, and the index / log / overview control files
> before pointing wiki-base at real sources. **Clear it before real use:** `rm -rf .wiki`
> then `/wiki-base:init`, or just run `/wiki-base:init` — it detects this demo `SCHEMA.md`
> and offers to reset.

# SCHEMA.md — wiki configuration & spec

This file is the discovery anchor for an LLM Wiki. Read it FIRST, then resolve the wiki
root from `Path:` below. It configures the wiki *and* fully specifies its conventions and
citation grammar, so any agent can maintain the wiki correctly even without the plugin.

This is a concrete, filled-in instance of the canonical template at
[`skills/init/references/schema-template.md`](../skills/init/references/schema-template.md).

## Configuration

- **Path:** `.wiki/`
  <!-- The redirect knob. Absolute path to the wiki root. Default: this in-project `.wiki/`.
       To use a central/external wiki (e.g. `~/wikis/wiki-base`), set this to that absolute
       path; raw/, assets/, and wiki/ then live there, and this in-project SCHEMA.md stays
       as the breadcrumb every skill uses to locate the wiki. -->
- **Domain:** research
- **Categories:** Sources | Entities | Concepts | Analyses
- **Source types:** URLs, PDFs, transcripts, local notes, code

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
     skills/init/references/codebase.md (credited to yysun), not in this generic spec. -->

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

## Retrieval

`index.md` + agentic `grep` / `glob` / `read`, just-in-time. **No vector DB, no embeddings, no
RAG.** This works to ~100 sources / hundreds of pages. An optional, pluggable markdown search
engine (e.g. qmd) is an opt-in upgrade for larger wikis, never required.
